---
name: portfolio-snapshot
description: Capture your full Indian net worth (stocks, mutual funds, EPF, FDs, ESOPs, US stocks, manual assets) from INDmoney or a CAS statement, explain what changed since the last snapshot, lay out invest/rebalance/sell options against your target allocation, and project when you'll reach your FIRE target. Use for "update my portfolio", "what's my net worth", "when will I hit FIRE".
---

# Portfolio snapshot

Goal: one dated snapshot of total portfolio value by asset class, appended to a running tracker, plus:
(a) what changed since the last snapshot (default ~15 days),
(b) decision support on where new money could go and whether anything is worth trimming, against the user's own target allocation,
(c) a FIRE projection: when the user will probably reach their target.

This is decision support, not investment advice. Claude is not a licensed advisor.

## Setup (first run)
Keep three small files together in one folder on the user's computer (connect it in the Claude desktop app so scheduled runs can read and append):

- `net_worth_snapshots.csv`: one row per run. Columns:
  `date,source,headline_total_inr,indstocks,mutual_funds,epf,esops_rsus,digital_assets,fixed_income,us_stocks,nps,bonds,ppf,sum_of_asset_tiles,one_day_change`
- `targets.csv`: the user's target allocation. Columns: `asset_class,target_pct,band_pp`. The user sets it; Claude never suggests targets.
- `fire.csv`: FIRE inputs. Columns: `key,value`, with keys:
  - `target_inr`: e.g. 50000000 for ₹5 Cr
  - `target_basis`: `nominal` (actual future amount) or `today` (inflation-adjusted)
  - `monthly_investment`: SIPs + stock buys + EPF (employee + employer) + expected ESOP/RSU vests ÷ 12
  - `annual_stepup_pct`: default 10
  - `inflation_pct`: default 6
  - `target_date`: optional

If a file is missing, ask the user once and create it. In an unattended run, use the defaults and list what's missing under "Data gaps".

## Asset classes
Indian stocks & ETFs | Mutual funds | EPF | PPF / NPS | FDs & bonds | US stocks | ESOPs/RSUs | Gold | Digital assets | Cash | Other (manual)

## Step 1 - Primary source: INDmoney (built-in browser)
1. Load the built-in-browser skill and open https://www.indmoney.com/dashboard.
2. If a login/OTP screen appears, stop and ask the user to sign in themselves in the browser pane. Never type credentials or OTPs. In an unattended run, message the user that the session expired and stop.
3. Read the page text (get_page_text / read_page, not screenshots). Extract the headline total ("MY ASSETS"), 1-day change, top gainer/loser, and value + share of each asset row.
4. Locate values by label, not position. If a value can't be found confidently, mark it "not found".
5. On the web, clicking an asset row opens INDmoney's explore pages, not holdings. Don't drill in. Holding-level and invested values come from the CAS (Step 2) or from the user.
6. Sum the asset rows in code and report any gap vs the headline total (the headline may include bank balances or other items).
7. Note any section shown without a value (not linked) and any stale sync dates (EPF can lag by weeks).

## Step 2 - Fallback / cross-check: CAS from Gmail
Use when INDmoney is unavailable, or for holding-level detail.
1. Search Gmail for the latest Consolidated Account Statement from NSDL or CDSL (or CAMS/KFintech for an MF-only CAS).
2. Download the PDF into its own empty directory.
3. Ask the user for the PDF password at that moment. Never save it anywhere.
4. Decrypt and extract with pypdf / pdfplumber. Parse ISIN, name, units, NAV/price, value, and transactions in the period.
5. A CAS covers demat equity, ETFs, bonds and mutual funds only. EPF, FDs, US stocks and ESOPs are not covered.

## Step 3 - Manual assets
Read any manual entries (PPF, NPS, gold, real estate, cash) the user keeps alongside the snapshots, and add them in.

## Step 4 - Log the snapshot
Append today's row to `net_worth_snapshots.csv`; never overwrite. If no folder is connected, create the file with today's row and send it to the user.

## Step 5 - What changed (last ~15 days)
Compare against the most recent snapshot that is 10-20 days old (or the latest one if none is).
1. Total and per-asset-class change in INR and %.
2. Split each change into new money vs market movement where possible:
   - Net new money = SIPs, lump sums, sells, vests, EPF contributions (from the CAS or the user; otherwise assume `monthly_investment` and say so).
   - Market movement = change - net new money.
   - EPF: changes are mostly contributions; interest is credited once a year.
   - ESOPs/RSUs: track the employer's share price and USD/INR; vests add units.
3. Context: Nifty 50, Sensex and USD/INR change over the same dates (web search).
4. The 3 biggest contributors to the change.
5. Allocation drift in percentage points since the last snapshot.

## Step 6 - Invest / rebalance / sell: decision support
1. Drift table: current % vs `targets.csv`. Flag classes outside their band.
2. Investing more: split the planned amount across underweight classes toward target, without selling. Show INR per class.
3. Trimming: for overweight classes, the INR amount that brings them back to target, with facts:
   - Tax: look up current Indian rules each run, because they change. As of the 2024 Budget: equity STCG 20% under 12 months; LTCG 12.5% above ₹1.25L per year; debt MFs bought after Apr 2023 taxed at slab rate.
   - Exit loads; lock-ins (ELSS 3 years, EPF and PPF restricted, ESOP trading windows).
   - Rebalancing with new money instead of selling avoids the tax.
4. Risk flags, factual only: single stock or employer equity above 10%; total equity share; liquid buffer vs the emergency fund the user wants.
5. "Things to decide": 2-4 plain questions for the user, not instructions.

## Step 6b - FIRE projection
1. Starting value = investable assets: the sum of the asset rows, excluding vehicles and other non-investment items.
2. Return assumptions per asset class in three scenarios (overridable in `fire.csv`). These are assumptions, not forecasts:
   - Equity (Indian stocks, equity MFs, US stocks, ESOPs/RSUs): 9% / 11% / 13%
   - EPF: the current declared EPF rate (look it up each run) in all three
   - Fixed income: 6.5% / 7% / 7.5%; digital assets: 0%
   - Blend them by the current allocation, or by targets.csv if present.
3. Simulate month by month in code: grow the balance, add the monthly investment, apply the step-up each year. If `target_basis = today`, inflate the target at `inflation_pct`. Cap at 40 years.
4. Report:
   - Progress (% of target).
   - Probable date (month and year) per scenario, leading with the base case. If the step-up is unknown, show 0% and 10%.
   - Sensitivity: months gained with ₹10k / ₹25k more per month; months lost with returns 2 points lower.
   - If `target_date` is set: the monthly investment needed to hit it in the base case.
   - Whether the date moved since the last report, and why.
   - What the target buys: annual spend at 3% and 4% withdrawal; for nominal targets, the target's value in today's rupees at the projected date.
5. After 6 or more snapshots spanning at least 6 months: the realized annualized return (XIRR), shown as a fourth "your actual pace" scenario.

## Step 7 - Report back
1. Headline: total, change since last snapshot, the market's change over the same period.
2. What changed: table by asset class (value, share, change, new money vs market) plus 2-3 sentences on why.
3. Allocation vs target: drift table.
4. Options: invest-more split, trim amounts with tax notes, risk flags.
5. FIRE: progress, the probable date and its range, sensitivity, and movement since last time.
6. Things to decide.
7. Data gaps.
End with one line: decision support only, not investment advice; check tax specifics with a CA.

## Rules
- Read-only on every financial site. Never click buy/sell/redeem/transfer, change settings, or link accounts. The user places any trades.
- Never store credentials, PAN, account numbers, OTPs or PDF passwords.
- Do all arithmetic in code, not in prose.
- Don't name specific funds or stocks to buy, or predict prices. Present projections as ranges based on labeled assumptions.
- If both INDmoney and CAS are used, prefer INDmoney for totals and flag differences above 2%.

## Optional: schedule it
Ask Claude to run this as a scheduled task, e.g. on the 1st and 15th of each month. The Claude desktop app must be open, and the INDmoney sign-in in the built-in browser must still be valid.
