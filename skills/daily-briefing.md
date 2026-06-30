---
name: daily-briefing
description: >
  Compiles Gautam's personal morning briefing as an interactive widget. Searches for live data
  across Indian and global markets, Adobe (ADBE) stock and company news, top tech and AI headlines,
  and generates one AI term of the day explained in plain language with a real-world analogy.
  Trigger whenever the user says "morning briefing", "daily briefing", "compile my briefing",
  "recompile briefing", or invokes /daily-briefing. Never pulls FluffyJaws, Jira, Slack, or
  any work connectors — this is a personal briefing only.
user-invocable: true
---

# Daily Briefing Skill

Compiles Gautam's personal morning briefing. This is a **personal-only** skill. Do NOT invoke
FluffyJaws, Jira, Confluence, Slack, or any work connectors regardless of what other skills
or connectors are available. Only use `web_search`.

**Important**: This skill includes a full investment research step (Step 1.5) that runs before
the widget is rendered. This takes 10–15 minutes. Do not render the widget until Step 1.5 is
fully complete and STOCK_GUIDE has all 7 fields populated. Announce to the user at the start:
*"Running your morning briefing — includes full investment research on today's random stock pick, plus checking your action plan tracker for triggered signals. This will take around 10–15 minutes."*

---

## Step 0 — Check action plan tracker for triggered signals

Before picking today's stock, read the tracker file at:
`~/Documents/AI Projects/AI Berkshire/action_plan_tracker.md`

If the file does not exist, skip to Step 0.5 (no prior plans to check).

If it exists:
1. Parse every entry that has **Status: Active**
2. For each active entry, search for the company's current stock price:
   `[CompanyName] [TICKER] current stock price today [date]`
3. Compare current price against each entry's stored conditions:
   - **Buy zone triggered**: current price ≤ upper bound of buy zone range
   - **Add signal triggered**: current price ≤ add signal price threshold (extract from Add signal action text)
   - **Sell signal triggered**: current price ≥ sell signal price threshold (extract from Sell signal action text)
4. Build **TRIGGERED_ALERTS** — a list of objects, one per triggered condition:
   - `company`: company name + ticker
   - `signal_type`: "Buy Zone Reached" | "Add Signal" | "Sell Signal"
   - `condition`: the original action text
   - `current_price`: current price found
   - `date_added`: when the plan was added to tracker

If no signals are triggered, TRIGGERED_ALERTS is an empty list.

---

## Step 0.5 — Pick today's random stock

Before any searches, select one stock from the list below. Do NOT pick a stock that already has an Active entry in the tracker — pick the next one in the random sequence to ensure variety. Use the current time (seconds or
milliseconds) as entropy — pick the stock at index `(current_second % 200)`. This ensures
the pick is different each time and spread across both markets.

**US stocks (100)**:
Apple, Microsoft, Alphabet, Meta, Adobe, Salesforce, ServiceNow, Workday, Veeva Systems,
Synopsys, Cadence Design Systems, PTC Inc, Fair Isaac (FICO), Tyler Technologies,
Jack Henry & Associates, Paycom, Datadog, Crowdstrike, Cloudflare, Palo Alto Networks,
Fortinet, Arista Networks, Verisign, HubSpot, Paylocity,
Nvidia, Texas Instruments, Analog Devices, Broadcom, KLA Corp,
Lam Research, Applied Materials, Qualcomm, Amphenol,
Amazon, Netflix, Booking Holdings, MercadoLibre, Airbnb, Spotify,
Visa, Mastercard, S&P Global, Moody's, MSCI Inc, FactSet Research,
Verisk Analytics, MarketAxess, CME Group, Intercontinental Exchange,
JPMorgan, Berkshire Hathaway, BlackRock, T. Rowe Price,
Progressive Corp, Erie Indemnity, Markel, Brown & Brown,
UnitedHealth, Intuitive Surgical, Idexx Laboratories, Zoetis, Danaher,
West Pharmaceutical, Bio-Techne, Mettler-Toledo, Waters Corp, Steris,
Automatic Data Processing, Cintas, Rollins, Illinois Tool Works, Roper Technologies,
Fortive, TransDigm, Heico Corp, Graco, Nordson,
Copart, Gartner, Axon Enterprise, W.W. Grainger, Accenture,
Costco, O'Reilly Automotive, AutoZone, Tractor Supply, Pool Corp,
NVR Inc, Winmark Corp, Ulta Beauty,
LVMH (ADR), Hermès (ADR), Ferrari, Hubbell, Parker Hannifin,
Intuit, Fiserv, RLI Corp, Snowflake

**India stocks (100)**:
TCS, Infosys, Wipro, HCL Technologies, Tech Mahindra,
Mphasis, KPIT Technologies, Persistent Systems, Coforge, LTIMindtree,
HDFC Bank, ICICI Bank, Kotak Mahindra Bank, Axis Bank, SBI,
Bajaj Finance, Bajaj Finserv, Cholamandalam Investment, Muthoot Finance, Shriram Finance,
SBI Life Insurance, HDFC Life Insurance, ICICI Lombard, CAMS, Nippon India AMC,
Angel One, BSE Ltd, CDSL, MCX, 360 One WAM,
Hindustan Unilever, Nestle India, Britannia Industries, Marico, Dabur India,
Colgate-Palmolive India, Godrej Consumer Products, ITC Ltd, Tata Consumer Products, Varun Beverages,
Asian Paints, Berger Paints, Pidilite Industries, Havells India, Polycab India,
Avenue Supermarts (DMart), Titan Company, Trent, Page Industries, Metro Brands,
Vedant Fashions (Manyavar), Jubilant FoodWorks, Westlife Foodworld,
Sun Pharmaceutical, Divi's Laboratories, Dr. Reddy's Laboratories, Cipla, Abbott India,
Syngene International, Astral Ltd, Poly Medicure,
Larsen & Toubro, Siemens India, ABB India, Cummins India, SKF India,
Schaeffler India, AIA Engineering, Carborundum Universal, Grindwell Norton, Timken India,
SRF Ltd, Navin Fluorine, Aarti Industries, Deepak Nitrite, Fine Organic Industries,
Maruti Suzuki, Bajaj Auto, Eicher Motors (Royal Enfield), TVS Motor, Hero MotoCorp,
Sona BLW Precision, Minda Corporation, Motherson Sumi,
Reliance Industries, Ultratech Cement, Shree Cement, Dalmia Bharat,
Indian Hotels (Taj), IndiGo (InterGlobe Aviation), Zomato,
Dixon Technologies, Kaynes Technology, Data Patterns, KEI Industries, Campus Activewear,
Torrent Pharmaceuticals, Balkrishna Industries (BKT), IRCTC, Godrej Properties

Store the selected company name as **STOCK_PICK** for use in searches and the widget.

---

## Step 1 — Run all searches in parallel

Fire these 4 searches in parallel before generating any output:

1. `Indian stock market Nifty Sensex today [current date]`
2. `S&P 500 NASDAQ today [current date]`
3. `top tech AI news today [current date]`
4. `Adobe ADBE stock price news [current date]`

Use today's actual date in every query. Do not use "latest" or "recent" — always include the
specific date so results are not stale.

**Note:** STOCK_PICK data is collected in Step 1.5 as part of the full research — no need
to pre-search it here.

---

## Step 1.5 — Full investment research on the stock pick

Run the **complete 7-module investment research** on STOCK_PICK — the same framework used
by the `/investment-research` skill. Do not skip or abbreviate any module. Search 5 from
Step 1 is your starting data; run as many additional searches as needed to complete the
full analysis.

The 7 modules to execute in order:

1. **Data collection** — revenue structure, 5-year financials (revenue, net income, gross margin,
   operating margin, FCF), competitive landscape, current valuation (price, market cap, P/E, P/FCF,
   FCF yield). Cross-validate key figures from 2 independent sources. Run
   `python3 ~/Documents/AI\ Projects/AI\ Berkshire/ai-berkshire/tools/financial_rigor.py verify-market-cap`
   to verify market cap and
   `python3 ~/Documents/AI\ Projects/AI\ Berkshire/ai-berkshire/tools/financial_rigor.py verify-valuation`
   to verify P/E and FCF yield.

2. **Business quality** (Duan Yongping) — one-line business definition, revenue model,
   subscription vs one-time, gross margin vs peers, operating leverage, customer stickiness.

3. **Moat assessment** (Buffett) — score each of the 5 moat types (brand, switching costs,
   network effects, scale, technology). Is the moat widening or narrowing?

4. **Contrarian risks** (Munger) — list all plausible failure paths with probability and impact.
   Collect the bear case. Identify the #1 way this investment could go wrong.

5. **Management quality** (Duan + Buffett) — CEO track record on key decisions, capital
   allocation discipline, insider ownership, succession risk.

6. **Industry & civilizational trend** (Li Lu) — is this sector in structural growth or decline?
   TAM size and penetration. What does this business look like in 20 years?

7. **Valuation & margin of safety** (Buffett + Duan) — reverse DCF (what does current price
   imply?), run three-scenario valuation:
   `python3 ~/Documents/AI\ Projects/AI\ Berkshire/ai-berkshire/tools/financial_rigor.py three-scenario`
   with bull/base/bear growth rates and P/E assumptions. Derive buy zone and 12-month base target.

**Data sources by market:**
- US stocks: macrotrends.net (primary) + stockanalysis.com (secondary)
- India stocks (NSE/BSE): screener.in (primary) + moneycontrol.com (secondary)

**After completing all 7 modules:**

1. Save the complete research report to:
   `~/Documents/AI Projects/AI Berkshire/[CompanyName]_Investment_Research.md`
   (full English report, same format as the Adobe research report)

2. Extract **only** the following fields for **STOCK_GUIDE** (used in the widget):
   - Current price + currency
   - P/E ratio (GAAP TTM preferred; non-GAAP forward if TTM unavailable)
   - One-line rating: ✅ Buy Zone | ⚖️ Cautiously Constructive | ⏳ Wait for Better Price | ❌ Avoid
   - Rating rationale: one sentence derived from the research
   - Action guide (4 rows): No position / Existing position / Sell signal / Add signal
   - Buy zone (price range)
   - 12-month base case target price

Do not display the full research in chat. The widget card is the only output visible to the user.

**After extracting STOCK_GUIDE, append a new entry to the tracker file:**
`~/Documents/AI Projects/AI Berkshire/action_plan_tracker.md`

Create the file if it does not exist. Append this block at the end (do not overwrite existing entries):

```
## [CompanyName] ([TICKER]) — Added [YYYY-MM-DD]
- **Price when added**: [current price + currency]
- **Buy zone**: [buy zone range]
- **12-month target**: [target price]
- **P/E when added**: [P/E value]
- **Rating**: [rating emoji + label]
- **Rationale**: [one-sentence rationale]
- **No position**: [action text from action guide]
- **Existing position**: [action text from action guide]
- **Sell signal**: [action text from action guide — must include a specific price threshold if one was derived]
- **Add signal**: [action text from action guide — must include a specific price threshold if one was derived]
- **Status**: Active

---
```

If STOCK_PICK already has an Active entry in the tracker (added on a different date), add a new entry anyway — price conditions may have changed since the last research run.

---

## CHECKPOINT — Do not proceed to Step 2 until STOCK_GUIDE is complete

Before moving on, confirm that STOCK_GUIDE contains all 7 fields:
current price, P/E, rating, rationale, action guide (4 rows), buy zone, 12-month target.
If any field is missing, complete the research before continuing.
Never render the widget with an incomplete or empty STOCK_GUIDE.

---

## Step 2 — Pick the AI term of the day

Choose ONE AI concept that meets all of these criteria:
- Genuinely useful to a practitioner-level AI user (Gautam has IIM Ahmedabad EPAIB, is fluent
  with Claude, Claude Code, MCP, agents, and LLMs)
- Not a basic entry-level term (skip: "machine learning", "neural network", "prompt engineering",
  "LLM", "fine-tuning" — he knows these)
- Ideally relevant to something in the news today, or something timely in the AI/agent space
- Has a crisp real-world analogy that makes it click immediately

Good examples: Inference-Time Compute, Activation Steering, Mixture of Experts, Constitutional AI,
Speculative Decoding, Chain-of-Thought vs Tree-of-Thought, KV Cache, RLHF vs DPO, World Models,
Grounding vs Hallucination, Context Poisoning, Agentic Scaffolding, Reward Hacking.

---

## Step 3 — Render the briefing widget

Use `show_widget` (visualize tool) to render the briefing as an interactive HTML widget.

### Widget structure (in order, top to bottom)

**Header row**
- Title: "Morning Briefing" with a Live badge
- Date: full date in `font-family: var(--font-mono)`
- Subtitle: "Ask Claude to recompile anytime ↗"
- Button: "Recompile" — calls `sendPrompt('Recompile my morning briefing')`

**Row 1 — two columns:**
- Left: AI Term of the Day card (amber `#eda100` left border)
  - Term name (20px, font-weight 500)
  - Type label (monospace, muted, uppercase)
  - 3–4 sentence plain-language explanation
  - Analogy line (italic, muted, prefixed with 💡)
- Right: Markets card (green eyebrow)
  - Rows: NIFTY 50, SENSEX, S&P 500, NASDAQ, Nifty IT, Crude Oil (Brent)
  - Each row: index name (mono) + change % with ↑/↓/→ and color (green up / red down / muted flat)
  - 2-sentence market sentiment summary below the rows

**Row 2 — full width:**
Adobe card (red `#e60000` left border)
- Price line: current price (22px mono) + day change + % change (colored) + meta (52-wk range, avg analyst target)
- Tag pills: key signals — e.g. YTD change, latest revenue beat, AI ARR, leadership changes, pending deals
  - Use tag colors: warn (yellow bg) for risks, ok (green bg) for positives, info (blue bg) for neutral/pending
- Two-column news grid inside the card: 2 items per column, 4 total Adobe stories
  - Each item: bold headline + 2-sentence summary

**Row 2.5 — full width (only render if TRIGGERED_ALERTS is non-empty):**
Action Alerts card (orange `#ea580c` left border, eyebrow: "⚡ ACTION ALERTS — EXECUTE NOW")
- Intro line (muted, 13px): "The following tracked stocks have hit a signal threshold since you last checked."
- For each alert in TRIGGERED_ALERTS, render one row:
  - Signal badge: red pill for "Sell Signal", green pill for "Buy Zone Reached" or "Add Signal"
  - Company name + ticker (bold, 14px)
  - Current price (monospace)
  - Signal condition text (muted, 12px, italic) — the original action text from the tracker
  - "Added [date_added]" label (monospace, muted, 11px)
- Footer line (muted, 11px): "Mark as executed or dismissed by editing ~/Documents/AI Projects/AI Berkshire/action_plan_tracker.md → change Status to ✅ Executed or 🗑️ Dismissed"

If TRIGGERED_ALERTS is empty, do NOT render this card at all — no placeholder, no "no alerts" message.

**Row 3 — full width:**
Stock Pick card (purple `#7c3aed` left border, eyebrow: "🎲 TODAY'S RANDOM PICK")
- Header line: company name (18px, font-weight 600) + ticker/exchange in monospace + current price + P/E
- Rating pill: colored badge matching the rating — green for ✅, amber for ⚖️, blue for ⏳, red for ❌
- Rating rationale: one sentence in muted text below the badge
- Action guide: 4-row table with columns "Position" and "Action"
  - Rows: No position | Existing position | Sell signal | Add signal
- Footer line (muted, 12px): "Buy zone: [range] · 12-month target: [price] · Quick analysis only — run /investment-research [company] for full deep dive"

**Row 4 — two columns:**
- Left: Tech News (blue eyebrow) — 3 items, headline + 2-sentence summary each
- Right: AI News (amber eyebrow) — 3 items, headline + 2-sentence summary each

**Footer**
- Timestamp + sources list (right-aligned, monospace, muted)

### Design rules
- Background: transparent (host provides bg). Cards: `var(--surface-2)` with `0.5px solid var(--border)`, `border-radius: 12px`
- Eyebrow labels: 10px, uppercase, letter-spacing 0.09em, colored by section
- Body text: 13px, `var(--text-secondary)`, line-height 1.6
- Snippets/meta: 12px, `var(--text-muted)`
- All colors via CSS variables — never hardcode light/dark values except accent colors
- No emoji except 💡 on the analogy line
- Recompile button: `var(--fill-accent)` background, calls `sendPrompt()`
- No position:fixed, no overflow:hidden on outer container, no iframes

---

## Step 4 — After the widget

Write 2–3 lines of plain prose below the widget summarising the single most important thing
from each section (markets, Adobe, AI). Keep it tight — one sentence per section max.
Do not repeat what is already visible in the widget. Think of it as the "so what" layer.

---

## Failure handling

- If a search returns no usable data for a section, render that card with a "No data retrieved —
  try recompiling" message in muted text. Do not leave the card empty or omit it.
- If Adobe stock price is unavailable, show the last known price with a "as of [date]" label.
- Never fabricate market numbers. If live data is unavailable, say so explicitly in the card.

---

## What this skill does NOT do

- Does not pull FluffyJaws, Jira, Confluence, Slack, M365, or any work connector
- Does not include simulation portfolio tracking (separate skill/request)
- Does not send to Telegram (separate Hermes automation)
- Does not store or log any data
