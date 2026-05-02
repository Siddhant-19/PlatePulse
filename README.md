# PlatePulse

> **Know what you actually ate.** Turn your Swiggy order history into a clear, honest picture of your nutrition — no food logging, no guesswork.

**[Live prototype →](https://plate-pulse.vercel.app/plate-pulse-prototype.html)** · **[Architecture →](architecture.md)**

---

## What is PlatePulse?

Most people who order on Swiggy regularly have no real idea what their weekly macros look like. They assume they're eating unhealthy, feel vaguely guilty, and do nothing about it — because the data to act on simply isn't surfaced anywhere.

PlatePulse fixes that.

It connects to your Swiggy account, pulls your order history, enriches each item with nutrition data, and presents a clean weekly dashboard of your eating habits — protein, carbs, fats, spend, cuisine patterns, and more. No manual logging. No switching apps. Just your real data, finally made useful.

---

## How it works

1. **Authenticate** — Sign in with your Swiggy phone number + OTP. PlatePulse gets a scoped auth token through the Swiggy Food MCP. Read-only, no orders placed on your behalf.
2. **Fetch orders** — Pull your Swiggy Food order history for a date range you choose (default: last 90 days).
3. **Enrich nutrition** — Each item in your orders is matched to nutrition data (calories, protein, carbs, fats) from a curated Indian-dish database + USDA FoodData Central, with clearly labelled estimates where exact data isn't available.
4. **View your dashboard** — A single-screen analytics view with everything you need to understand your eating week.

---

## Dashboard analytics

**Primary — Nutrition**

| View | What you see |
|------|-------------|
| **Macro metrics** | At-a-glance cards for protein, calories, carbs, and fats — each with a progress bar vs. your daily goal |
| **Protein trend** | Weekly protein intake plotted against your target — see the gap clearly, week by week |
| **Calorie trend** | Daily calorie average per week with a healthy-range band (1,600–2,200 kcal) |
| **Macro split** | Stacked bar showing % of calories from protein, carbs, and fats each week |
| **Order highlights** | Your best nutrition order and heaviest meal flagged — know which choices move the needle |
| **Cuisine patterns** | Most-ordered cuisines mapped to your eating habits |

**Secondary — Habits & Spend**

| View | What you see |
|------|-------------|
| **Weekly spend** | ₹ per week on Swiggy Food over the last 6 weeks |
| **Order frequency** | How often you order, and whether it's changing |

---

## Why build this?

The health-conscious market in India is growing fast — but the tools haven't caught up. People already order on Swiggy multiple times a week. The data to understand their eating habits exists. It just isn't surfaced in a way that's actionable.

PlatePulse closes that gap. The goal isn't to make people feel bad about ordering out — it's to give them the insight to make slightly better choices, one order at a time. Understanding your habits is the first step to improving them.

---

## Tech stack

- **Frontend:** Next.js (single-page dashboard, hosted on Vercel)
- **Backend:** Node / Hono on Render, Postgres on Neon for nutrition cache
- **MCP integration:** Swiggy Food MCP via `@modelcontextprotocol/sdk` — OAuth 2.1 + PKCE, tokens encrypted at rest with refresh-token rotation
- **Nutrition data:** USDA FoodData Central + hand-curated Indian-dish lookup table, LLM fallback for unmatched items

---

## Status

Currently in development as a Swiggy Builders Club project.

| Resource | Link |
|---|---|
| Live prototype | [plate-pulse.vercel.app/plate-pulse-prototype.html](https://plate-pulse.vercel.app/plate-pulse-prototype.html) |
| Architecture | [architecture.md](architecture.md) |

Built by [Sid Jagtap](https://linkedin.com/in/siddhant-jagtap) — product at Shopflo, building PlatePulse as a side project because ordering Swiggy 5x/week with zero visibility into your macros is a problem worth solving.
