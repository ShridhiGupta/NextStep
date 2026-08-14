# NextStep

A personal job-search agent. Every morning it scans the public job boards
(Greenhouse / Lever / Ashby) of the companies you care about, filters out the
~99% of postings that don't fit, scores the rest against your resume with an
LLM, drafts application material for the best few, and emails you a digest.

**It never submits an application** — you read the digest, edit the drafts,
and apply yourself.

## Quick start

```bash
python -m venv .venv && source .venv/bin/activate   # Windows: .\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python -m nextstep run --mock --scorer keyword       # smoke test, no API key needed
```

## Setup

1. Copy `.env.example` to `.env` and fill in your `GEMINI_API_KEY`, plus
   `SMTP_USER` / `SMTP_PASS` (Gmail App Password) / `MAIL_TO` for email.
2. Build your profile: `python -m nextstep profile --resume resume.pdf`
   — then check `profile.json` for accuracy.
3. Edit `companies.yaml` (companies to watch) and `config.yaml`
   (job titles, locations, score threshold).

## Daily use

```bash
python -m nextstep run --send
```

The digest lands in your inbox with only jobs you haven't seen before
(`seen.json` tracks history — reset it to `{}` only for a fresh start).

Useful flags: `--limit 10` (cost guard while tuning), `--no-draft`
(screen only), `--mock` (offline fixtures).

Track applications: `python -m nextstep applied "<job_id>"` ·
stats/CSV export: `python -m nextstep stats`

## Tests

```bash
python -m pytest tests -q    # 63 tests, no network, no API key
```