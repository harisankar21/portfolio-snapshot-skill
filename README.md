# Portfolio Snapshot: a Claude skill for Indian investors

A [Claude](https://claude.ai) skill that reads your full net worth from **INDmoney** (stocks, mutual funds, EPF, ESOPs/RSUs, FDs, US stocks, digital assets), or from your **NSDL/CDSL CAS** statement. It then tells you:

- **What changed**, usually over the last ~15 days, split into new money vs market movement, with Nifty, Sensex and USD/INR shown for context
- **How you compare with your target allocation**: where new money could go and what's overweight, with Indian tax facts (STCG/LTCG, exit loads, lock-ins)
- **When you'll probably hit FIRE**: a month-by-month projection under conservative, base and optimistic scenarios, with sensitivity to how much you invest and what returns you get

> ⚠️ **Decision support, not investment advice.** The skill is read-only on every financial site: it never clicks buy, sell or redeem. You place any trade yourself. Check tax specifics with a CA.

## Install

1. Download `portfolio-snapshot.zip` from the [latest release](../../releases/latest).
2. In Claude, open **Settings → Capabilities → Skills**, upload the zip (or the `portfolio-snapshot/` folder), and turn it on.
3. Use the **Claude desktop app**: the skill reads INDmoney through Claude's built-in browser.

## First run

1. Copy the files from `templates/` into a folder on your computer and connect that folder in the Claude desktop app.
2. Edit `fire.csv` (FIRE target, monthly investment, step-up, inflation) and `targets.csv` (your target allocation). If they're missing, the skill asks for them.
3. Ask Claude: **"update my portfolio"**. Sign in to INDmoney yourself when the browser pane asks (Claude never types OTPs or passwords).
4. Optional: ask Claude to run it as a **scheduled task**, e.g. on the 1st and 15th of each month.

## What it needs

| Need | Why |
|---|---|
| Claude desktop app with the built-in browser | Reads your INDmoney dashboard |
| INDmoney account with your accounts linked (UAN for EPF, etc.) | Primary data source |
| Gmail connector (optional) | CAS statement fallback, for holding-level detail |

## Privacy

- Your data stays in your own Claude session and your own folder. This repo collects nothing, and the skill sends no data anywhere.
- The skill never stores credentials, PAN, account numbers, OTPs or PDF passwords.
- **Don't commit your real `net_worth_snapshots.csv` to a public fork.**

## Limitations

- On the web, INDmoney shows totals for each asset class but not each holding. For holding-level detail and invested amounts, use the CAS statement.
- INDmoney's sign-in in the built-in browser can expire; scheduled runs then ask you to sign in again.
- Projections rest on labeled assumptions. They're ranges, not promises.

## Feedback

Using it? ⭐ the repo or say hi in [Discussions](../../discussions); that's the only way I know anyone uses it. Bugs and ideas go in [Issues](../../issues).

## License

MIT
