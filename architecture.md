# PlatePulse — Architecture

> High-level design of how PlatePulse is built and how it talks to the Swiggy MCP.

---

## System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                            USER BROWSER                             │
│                                                                     │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                    Next.js Frontend                         │   │
│   │                                                             │   │
│   │   [ Auth Screen ] → [ Fetch Screen ] → [ Dashboard ]       │   │
│   └──────────────────────────┬──────────────────────────────────┘   │
└─────────────────────────────-│-------------------------------------─┘
                               │ HTTPS
                               ▼
┌──────────────────────────────────────────────────────────────────────┐
│                         PLATEPULSE BACKEND                           │
│                      Node.js / Hono  (Render)                        │
│                                                                      │
│   ┌───────────────┐   ┌─────────────────┐   ┌────────────────────┐  │
│   │  Auth Handler │   │  Orders Service │   │ Nutrition Enricher │  │
│   │               │   │                 │   │                    │  │
│   │ OAuth 2.1+PKCE│   │ get_orders()    │   │ item → macro lookup│  │
│   │ Token storage │   │ get_order_detail│   │ + LLM fallback     │  │
│   └──────┬────────┘   └────────┬────────┘   └─────────┬──────────┘  │
│          │                     │                       │             │
│          └─────────────────────┼───────────────────────┘             │
│                                │                                     │
│   ┌────────────────────────────▼──────────────────────────────────┐  │
│   │                    Postgres (Neon)                            │  │
│   │   user_tokens  |  nutrition_cache  |  indian_dish_table       │  │
│   └───────────────────────────────────────────────────────────────┘  │
└──────────────────────────────┬───────────────────────────────────────┘
                               │ MCP over HTTPS
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        SWIGGY FOOD MCP                              │
│                   https://mcp.swiggy.com/food                       │
│                                                                     │
│   ┌──────────────────┐         ┌────────────────────────────────┐   │
│   │   OAuth Server   │         │         MCP Tools              │   │
│   │  (phone + OTP)   │         │  get_orders(date_from, date_to) │   │
│   │  issues tokens   │         │  get_order_details(order_id)   │   │
│   └──────────────────┘         └────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                                                        │
                               ┌────────────────────────┘
                               ▼
┌──────────────────────────────────────────────────────┐
│               NUTRITION DATA SOURCES                 │
│                                                      │
│   ┌──────────────────┐   ┌──────────────────────┐   │
│   │ USDA FoodData    │   │ Indian Dish Table     │   │
│   │ Central (API)    │   │ (curated, Postgres)   │   │
│   └──────────────────┘   └──────────────────────┘   │
│            │                        │                │
│            └──────────┬─────────────┘                │
│                       ▼                              │
│            ┌──────────────────┐                      │
│            │  LLM Fallback    │  (unmatched items)   │
│            │  (Claude API)    │                      │
│            └──────────────────┘                      │
└──────────────────────────────────────────────────────┘
```

---

## Auth Flow — Swiggy MCP OAuth 2.1 + PKCE

```
User                  PlatePulse             Swiggy MCP OAuth
 │                        │                        │
 │  Enter phone number     │                        │
 │───────────────────────▶│                        │
 │                        │   Initiate OAuth PKCE  │
 │                        │   (code_challenge)      │
 │                        │───────────────────────▶│
 │                        │                        │
 │        Redirect to Swiggy consent + OTP screen  │
 │◀────────────────────────────────────────────────│
 │                        │                        │
 │  Enter OTP on Swiggy   │                        │
 │──────────────────────────────────────────────── ▶│
 │                        │                        │
 │                        │◀──── auth_code ─────── │
 │                        │                        │
 │                        │  Exchange code +       │
 │                        │  code_verifier         │
 │                        │───────────────────────▶│
 │                        │                        │
 │                        │◀── access_token ────── │
 │                        │    refresh_token        │
 │                        │    scope: food.orders:read
 │                        │                        │
 │  Token stored encrypted│                        │
 │  in Postgres. User     │                        │
 │  redirected to         │                        │
 │  dashboard.            │                        │
 │◀───────────────────────│                        │
```

**Key principle:** PlatePulse never sees the user's Swiggy password. The OTP flow is handled entirely by Swiggy. PlatePulse only receives a scoped read-only token.

---

## Data Flow — Orders to Dashboard

```
Swiggy MCP                Backend                    Frontend
    │                        │                           │
    │   get_orders(90d)       │                           │
    │◀───────────────────────│                           │
    │──── [ order list ] ───▶│                           │
    │                        │                           │
    │  get_order_details()   │                           │
    │  (per order, ~47 calls)│                           │
    │◀───────────────────────│                           │
    │──── [ items + qty ] ──▶│                           │
    │                        │                           │
    │                  ┌─────▼──────────────────────┐   │
    │                  │  Nutrition Enrichment        │   │
    │                  │                              │   │
    │                  │  for each item:              │   │
    │                  │  1. check nutrition_cache    │   │
    │                  │  2. lookup indian_dish_table │   │
    │                  │  3. query USDA FoodData API  │   │
    │                  │  4. LLM fallback (Claude)    │   │
    │                  │                              │   │
    │                  │  output: { protein, carbs,  │   │
    │                  │           fats, calories }   │   │
    │                  └─────┬──────────────────────-─┘   │
    │                        │                           │
    │                  ┌─────▼──────────────────────┐   │
    │                  │  Analytics Aggregation       │   │
    │                  │                              │   │
    │                  │  → weekly macro totals       │   │
    │                  │  → daily protein average     │   │
    │                  │  → calorie trend             │   │
    │                  │  → cuisine frequency         │   │
    │                  │  → spend per week            │   │
    │                  │  → highlight orders          │   │
    │                  └─────┬──────────────────────-─┘   │
    │                        │                           │
    │                        │──── JSON analytics ──────▶│
    │                        │                           │
    │                        │              ┌────────────▼──────────┐
    │                        │              │   Dashboard Render     │
    │                        │              │                        │
    │                        │              │  Protein trend chart   │
    │                        │              │  Macro split chart     │
    │                        │              │  Calorie trend chart   │
    │                        │              │  Cuisine bars          │
    │                        │              │  Spend chart           │
    │                        │              │  Insight banner        │
    │                        │              └────────────────────────┘
```

---

## Nutrition Enrichment Pipeline

```
  Order Item Name (e.g. "Grilled Chicken Bowl")
              │
              ▼
  ┌───────────────────────┐
  │   nutrition_cache     │  ← Postgres. Hit? Return cached macros.
  │   (per item name)     │
  └──────────┬────────────┘
             │ Miss
             ▼
  ┌───────────────────────┐
  │  indian_dish_table    │  ← Curated ~500 common Indian dishes.
  │  (exact match)        │    Hit? Return + write to cache.
  └──────────┬────────────┘
             │ Miss
             ▼
  ┌───────────────────────┐
  │  USDA FoodData API    │  ← Fuzzy name search. Hit? Return + cache.
  └──────────┬────────────┘
             │ Miss
             ▼
  ┌───────────────────────┐
  │  Claude API (LLM)     │  ← "Estimate macros for: [item name]"
  │  (estimate + label)   │    Labelled as "estimate" in UI.
  └───────────────────────┘
```

**Cache TTL:** Nutrition data is cached indefinitely per item name — dishes don't change their macros. User order data is cached for 24h, then re-fetched from Swiggy MCP.

---

## Swiggy MCP — Tools Used

| MCP Tool | When called | Purpose |
|---|---|---|
| `get_orders` | Once per dashboard load | Fetch all order metadata for the selected date range |
| `get_order_details` | Once per order (~47 calls per 90d load) | Fetch full item list, quantities, and item-level properties from each order |

**Scope requested:** `food.orders:read` — the narrowest scope available. PlatePulse never calls write tools (`add_to_cart`, `place_order`, etc.).

---

## Request Volume Estimate

```
Per dashboard load (90d window, ~47 orders):
  1  ×  get_orders()          =   1 MCP call
  47 ×  get_order_details()   =  47 MCP calls
                                ──────────────
  Total:                        ~48 MCP calls per load

Cached for 24h → repeat visits = 0 MCP calls

At 100 beta users, 1 fresh load/day:
  100 users × 48 calls = ~4,800 MCP calls/day  (<< 1K/day per user)
```

---

## Component Map

```
platepulse/
├── app/                          # Next.js app router
│   ├── page.tsx                  # Auth entry (phone + OTP screens)
│   ├── dashboard/
│   │   └── page.tsx              # Dashboard (charts, metrics, insight)
│   └── api/
│       ├── auth/
│       │   ├── swiggy/route.ts   # Initiates OAuth PKCE flow
│       │   └── callback/route.ts # Handles Swiggy redirect + token exchange
│       └── analytics/route.ts    # Returns aggregated nutrition JSON
│
├── server/                       # Hono backend (Render)
│   ├── swiggy/
│   │   ├── client.ts             # MCP SDK wrapper (get_orders, get_order_details)
│   │   └── auth.ts               # Token storage + refresh rotation
│   ├── nutrition/
│   │   ├── enricher.ts           # 4-layer enrichment pipeline
│   │   ├── usda.ts               # USDA FoodData Central API client
│   │   └── llm.ts                # Claude API fallback estimator
│   └── analytics/
│       └── aggregator.ts         # Weekly rollups, highlight orders
│
└── db/
    ├── schema.sql                # user_tokens, nutrition_cache, indian_dish_table
    └── seed/
        └── indian_dishes.json    # ~500 curated dish → macro mappings
```
