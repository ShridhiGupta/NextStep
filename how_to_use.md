# How to use NextStep

This guide assumes setup is already done (see [SETUP.md](SETUP.md)):
dependencies installed, `.env` filled in, and `profile.json` generated.

## Every morning

1. Open the project folder in VS Code and open the terminal.
2. Activate the virtual environment:
   ```powershell
   .\.venv\Scripts\Activate.ps1        # Windows
   # source .venv/bin/activate         # macOS/Linux
   ```
3. Run:
   ```bash
   python -m nextstep run --send
   ```
4. Check your inbox — the digest contains only jobs you haven't seen before.

That's it. Don't reset `seen.json` daily; it's what stops the tool from
showing you the same jobs again.

## Reading the digest

Each job card shows:
- **Score badge** (0–10) — how well the job matches your profile, with a
  one-line reason.
- **Why it fits** — a short fit summary.
- **Resume bullets for this role** — your real experience rewritten for
  this job. Copy them into your resume before applying.
- **Honest gaps** — what you're missing and how to address it.
- **Cover note** — a draft. **Always edit it before sending.**
- **Ask them** — questions for the interview/recruiter call.
- **Open & apply** — link to the actual posting. You apply yourself.

## After you apply

Mark the job as applied (the job id is printed at the bottom of each card):

```bash
python -m nextstep applied "greenhouse:stripe:5501001"
```

See your stats and export a spreadsheet:

```bash
python -m nextstep stats      # writes out/tracker.csv
```

## Common commands

| Command | What it does |
|---|---|
| `python -m nextstep run --send` | daily run + email the digest |
| `python -m nextstep run` | build `out/digest.html` without emailing |
| `python -m nextstep run --limit 10` | process at most 10 jobs (cheap testing) |
| `python -m nextstep run --no-draft` | score only, skip drafting (faster/cheaper) |
| `python -m nextstep run --mock --scorer keyword` | offline smoke test, no API key |
| `python -m nextstep profile --resume resume.pdf` | rebuild profile from resume |
| `python -m pytest tests -q` | run the test suite |

## Changing what jobs you see

- **Different companies** → edit `companies.yaml`. Verify the slug works:
  open `https://boards-api.greenhouse.io/v1/boards/<slug>/jobs` in a browser.
- **Different roles/locations** → edit `filters` in `config.yaml`
  (`include_titles`, `exclude_titles`, `locations`, `allow_remote`).
- **More/fewer jobs in the digest** → adjust `score_threshold` (lower = more)
  and `max_per_digest` in `config.yaml`.
- **Updated your resume?** → re-run
  `python -m nextstep profile --resume resume.pdf` and review `profile.json`.

## Troubleshooting

| Symptom | Fix |
|---|---|
| "No new matches today" | Normal — all current matches were already emailed. New postings appear as companies publish them. |
| A board shows 0 jobs every day | The slug is dead; check it in the browser and update `companies.yaml`. |
| `screen batch failed` / digest missing drafts | Gemini free-tier rate limit (429/503). Re-run later; it retries unscored jobs automatically. |
| Email not arriving | Check `SMTP_USER`/`SMTP_PASS` in `.env` — `SMTP_PASS` must be a Gmail **App Password**, not your login password. |
| `No module named nextstep` | You're in the wrong folder or the venv isn't activated. |
| Want a completely fresh start | Set `seen.json` contents to `{}` — every current posting counts as new again. |
