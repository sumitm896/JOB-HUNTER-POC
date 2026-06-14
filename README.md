# Job Hunter 🎯

An automated job-search pipeline that runs on a schedule, finds roles that
match your profile, scores each one against your actual strengths (not just
keywords), verifies the listings are real, and emails you a daily shortlist.

It's a **proof of concept** — a working tool I built to stop refreshing job
boards by hand. It runs, it's cheap, and it's driven entirely by one settings
file so anyone can point it at their own search without touching the code.

---

## What it does

- **Searches** for your target roles using Claude with live web search.
- **Scores** each listing 0–100 on *transferable fit* — it weights your core
  strengths over exact tool matches, so it won't discard a great role just
  because the stack is unfamiliar.
- **Verifies** every job URL actually resolves before it reaches your inbox,
  because language models will happily invent a link that 404s. Links that
  can't be confirmed get an "⚠ Unconfirmed" badge rather than being dropped.
- **De-duplicates** against the last 14 days using Google Sheets as memory,
  so you never get pinged about the same job twice.
- **Emails** you a clean HTML digest, ranked by match score.
- **Logs** cost and token usage per run, so you always know what it's spending
  (it runs on Claude Haiku with prompt caching — typically a few cents a day).

---

## How it works

```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐     ┌──────────┐
│ Search      │ ──▶ │ Score &      │ ──▶ │ Verify URLs │ ──▶ │ Email    │
│ (Claude +   │     │ de-duplicate │     │ (web_fetch) │     │ digest   │
│ web search) │     │ (Sheets)     │     │             │     │ (Gmail)  │
└─────────────┘     └──────────────┘     └─────────────┘     └──────────┘
                          │                                        │
                          └──────────────  Google Sheets  ─────────┘
                                       (seen jobs + run log)
```

| File | Role |
|------|------|
| `job_hunter.py` | Main pipeline: search, score, de-dup, email |
| `verify.py` | Confirms each job URL is real before sending |
| `sheet.py` | Google Sheets I/O — de-dup memory and run logs |
| `.env.example` | The settings file template (copy to `.env`) |

---

## Setup

**1. Clone and install**

```bash
git clone https://github.com/sumitm896/JOB-HUNTER-POC.git
cd JOB-HUNTER-POC
pip install -r requirements.txt
```

**2. Create your settings file**

Copy the template and fill it in. You never edit the Python.

```bash
cp .env.example .env
```

Open `.env` and set, at minimum:

- `ANTHROPIC_API_KEY` — from [console.anthropic.com](https://console.anthropic.com)
- `GMAIL_ADDRESS` and `GMAIL_APP_PASSWORD` — a Gmail
  [app password](https://myaccount.google.com/apppasswords), not your real one
- `TARGET_ROLES` — the job titles you want, comma-separated
- `SEARCH_FOCUS` — a plain-English description of what you're after

Google Sheets logging is optional. To enable it, set `GOOGLE_SHEET_ID` and
`GOOGLE_CREDENTIALS_FILE`; leave them blank to run without it.

**3. Run it**

```bash
python job_hunter.py
```

A digest lands in your inbox.

---

## Make it yours

Everything is controlled from `.env` — no code changes needed:

```dotenv
TARGET_ROLES=Data Analyst, BI Developer, Analytics Engineer
SEARCH_LOCATION=Berlin (incl. remote)
SEARCH_FOCUS=Junior data roles, Python-heavy teams, willing to relocate within the EU. Avoid pure dashboarding-only roles.
```

Point it at a different career entirely just by rewriting those lines.

---

## Run it on a schedule (optional)

The pipeline is built to run unattended via GitHub Actions on a daily cron.
In that setup you don't commit a `.env` file — you add the same keys as
**repository secrets** (Settings → Secrets and variables → Actions), and the
workflow passes them in as environment variables. The script reads from a
local `.env` when present and falls back to real environment variables
otherwise, so the same code works both locally and in CI.

---

## What's next

The current bottleneck is search quality: open-web search returns thin, stale
results for niche roles. The next module, `ats.py`, queries the **Greenhouse**
and **Lever** applicant-tracking-system APIs directly — pulling fresh,
structured listings straight from companies' own job boards instead of relying
on web-search snippets.

---

## A note on cost & honesty

This is a personal project, not a polished product. It uses Claude Haiku with
prompt caching to keep runs to a few cents, verifies URLs to avoid sending you
ghost listings, and is transparent about what it can't confirm. If you fork it,
keep your real keys in `.env` (which is git-ignored) and never commit them.

---

*Built by Sumit Mishra.*
