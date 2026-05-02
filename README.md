# PlatePulse

> **Know what you actually ate.** Turn your Swiggy order history into a clear, honest picture of your nutrition — no food logging, no guesswork.

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

| View | What you see |
|------|-------------|
| **Protein tracker** | Weekly protein intake over the selected date range, plotted day by day |
| **Macro breakdown** | Weekly totals for protein, carbs, and fats — your real nutritional fingerprint |
| **Spend analytics** | Weekly food spend trends, avg order value, and cost-per-macro insights |
| **Cuisine patterns** | Your most-ordered cuisines and how they map to your nutrition goals |
| **Top nutrition orders** | Highest and lowest macro orders — know your best and worst meals at a glance |

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

Currently in development as a Swiggy Builders Club project. Prototype available — see [`plate-pulse-prototype.html`](plate-pulse-prototype.html).

Built by [Sid Jagtap](https://linkedin.com/in/siddhant-jagtap) — product at Shopflo, building PlatePulse as a side project because ordering Swiggy 5x/week with zero visibility into your macros is a problem worth solving.
