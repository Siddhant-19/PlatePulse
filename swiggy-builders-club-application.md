# Swiggy Builders Club — Application Pack

A complete pack for the Swiggy MCP access form: product name shortlist, then field-by-field answers ready to paste.

---

## 1. Product name — 7 options to brainstorm from

| # | Name | Style | Why it works |
|---|------|-------|--------------|
| 1 | **MacroSwig** | Witty / portmanteau | Macros (protein/carbs/fats) + Swiggy. Says exactly what it does in 8 letters. Memorable, slightly cheeky. |
| 2 | **PlateLens** | Brandable / soft | Reads your plate like a lens. Category-agnostic, lets the brand expand beyond Swiggy later. |
| 3 | **NomNomKnow** | Playful / consumer | Onomatopoeic, instantly readable as a food app. Skews younger / D2C. |
| 4 | **OrderIQ** | Descriptive / B2B-leaning | Intelligence layer on your order history. Clean, easy to spell, dot-com feel. |
| 5 | **Thali** | Brandable / Indian | Indian word for a balanced meal plate. Culturally resonant, instantly evokes nutrition + portion thinking. Ownable as a single word. |
| 6 | **CalorieCart** | Descriptive / direct | Says the unlock: nutrition awareness right inside the order flow. Good SEO. |
| 7 | **EatTrace** | Brandable / utilitarian | Trace what you eat. Short, ownable, neutral enough for both casual users and health-serious users. |

**Top pick: `Thali`** — it's the most ownable name on the list, instantly meaningful in the Indian context (Swiggy's market), and naturally extends the metaphor of "what's actually on your plate this week." Strong fallback: **MacroSwig** if you want the Swiggy connection front and center for the application reviewer.

---

## 2. Form answers — field by field

These are written against the exact questions in the Builders Club form. Paste each answer into the matching field.

### Team / Project Name
**Thali** — solo project (Sid Jagtap, indie builder).

### GitHub or Portfolio URL
`https://github.com/<your-handle>/thali` *(replace with your handle — create the repo as a placeholder before submitting; an empty repo with a README is fine)*

### LinkedIn
`https://linkedin.com/in/<your-handle>` *(replace with your URL)*

### What are you building? (2–3 sentences)
Thali is a personal nutrition dashboard that turns a user's Swiggy Food order history into a weekly view of macros (protein, carbs, fats), spend, and eating patterns. We pull the user's past orders through the Swiggy Food MCP, enrich each item with nutrition data from open food databases, and show them a single-screen "this is your eating week" dashboard — no food logging required. The goal is to nudge healthier ordering by closing the gap between what people *think* they ate and what they actually ate.

### Which MCP servers do you need?
- ✅ **Swiggy Food** *(required)*
- ⬜ Swiggy Instamart
- ⬜ Swiggy Dineout

### What type of integration is this?
**Web App** *(single-page dashboard. Not an AI agent — Thali is read-only and doesn't place orders.)*

### Tech stack & architecture overview
Frontend is a Next.js single-page app on Vercel. Backend is a thin Node service (Hono on Render) with Postgres on Neon for the per-user nutrition cache and a curated Indian-dish lookup table. MCP integration uses the official `@modelcontextprotocol/sdk` to call the Swiggy Food server at `https://mcp.swiggy.com/food` over OAuth 2.1 + PKCE — user clicks "Connect Swiggy", consents on Swiggy's side (phone+OTP handled by Swiggy), tokens stored encrypted-at-rest with refresh-token rotation. We call `get_orders` for the selected date window (90d default) and `get_order_details` per order to extract items, then enrich each item via a two-layer pipeline (USDA FoodData Central + a hand-curated Indian-dish table, with an LLM fallback for unmatched items, all clearly labelled as estimates the user can correct). All Swiggy data is fetched per-user on-demand, scoped to that user's account, and never shared with third parties.

### Redirect URI(s) for auth flows
- `https://thali.app/api/auth/swiggy/callback` *(production — replace with your real domain)*
- `http://localhost:3000/api/auth/swiggy/callback` *(local development)*

### Expected request volume
**< 1K/day** for the closed beta. Each user generates roughly 1 `get_orders` call + ~30–60 `get_order_details` calls per dashboard load, cached for 24h. Volume scales linearly with active users; we'll target ~100 beta users to start.

### Demo link, GitHub repo, or anything else
- **Low-fi prototype:** [link to your hosted prototype, or attach the `thali-prototype.html` from this pack]
- **Repo:** `https://github.com/<your-handle>/thali`
- **About me:** Product at Shopflo (checkout infra, India). Building Thali as a side project to scratch a personal itch — I order on Swiggy 5x/week and have no idea what my macros look like.
- **Why we won't abuse this:** Thali is **strictly read-only** — no cart, no checkout, no write tools. Lower risk surface for review. Happy to start gated to my own account, expand to 50 invited beta users, then open up.

---

## Notes for Sid before submitting

1. **Replace the four placeholders** before pasting: GitHub URL, LinkedIn URL, prototype link, and the production redirect URI (`thali.app` is a stand-in — use whatever domain you actually own or `vercel.app` subdomain).
2. **Create the GitHub repo first**, even if empty with just a README — reviewers click these links.
3. **Host the prototype somewhere fetchable** (Vercel, Netlify drop, or even a GitHub Pages branch). The `.html` file in this pack is self-contained.
4. **If you want to pitch as Shopflo-internal** instead of indie, swap the Team Name line to "Shopflo (internal experiment)" and adjust the LinkedIn/GitHub fields accordingly.
5. **Auth nuance:** the form asks for the redirect URI used by *your* app, not by Claude/ChatGPT. The Swiggy manifest already whitelists `claude.ai`, `chatgpt.com`, etc. for those clients — you need to register your own callback for Thali separately.
