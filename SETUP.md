# Setup Guide

Start here if you've never run a Python project before. Every step assumes you have nothing installed.

**What you end up with:** an email every weekday morning with the 5 jobs worth your time, each with tailored resume bullets, honest gaps, and a draft cover note. You read it, edit it, and apply yourself. The tool never submits anything.

---

## Step 1 — Install Python

You need **Python 3.10 or newer**.

### Windows

Download Python from [python.org/downloads](https://www.python.org/downloads/).

On the first screen of the installer, tick **"Add python.exe to PATH"** before clicking Install.

### macOS

If you have Homebrew:

```bash
brew install python
```

Otherwise, download Python from [python.org/downloads](https://www.python.org/downloads/).

### Linux

```bash
sudo apt install python3 python3-venv python3-pip
```

### Check Python

Open a fresh terminal.

**Windows:**

```bash
python --version
```

**macOS / Linux:**

```bash
python3 --version
```

You should see `Python 3.10.x` or higher.

If you see `command not found`, reinstall Python and make sure the PATH option is enabled.

> From here on, wherever you see `python`, use `python3` on macOS/Linux.

---

## Step 2 — Get the Code

### Option 1: Download ZIP

Click the green **Code** button on the repository → **Download ZIP** → unzip it.

### Option 2: Clone with Git

```bash
git clone https://github.com/<owner>/jobhunt.git
cd jobhunt
```

If you want the application to email you automatically every morning, click **Fork** at the top-right of the repository first, then clone your own fork.

---

## Step 3 — Create a Virtual Environment and Install Dependencies

A virtual environment keeps this project's packages separate from the rest of your system.

### Windows PowerShell

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

If PowerShell refuses with **"running scripts is disabled on this system"**, run this once in the same window:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Then activate the environment again:

```powershell
.\.venv\Scripts\Activate.ps1
```

If you get `No module named pip`, run:

```powershell
python -m ensurepip --upgrade
```

and retry the installation.

### macOS / Linux

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

You'll know it worked when your terminal prompt starts with `(.venv)`.

You need to reactivate the environment every time you open a new terminal.

---

## Step 4 — Prove It Works

Before configuring API keys or resumes, check the installation.

Run:

```bash
python -m jobhunt run --mock --scorer keyword
```

You should see output similar to:

```text
[2/5] filtering
  prefilter: 12 -> 5 (dropped title=5 location=1 stale=1)
[3/5] screening 5 jobs (keyword stub — DEV ONLY)
  3 scored >= 7.0
[5/5] digest
  wrote out/digest.html

funnel: 12 scanned -> 5 passed filters -> 5 new -> 3 in digest
```

Open `out/digest.html` in your browser.

If this works, your installation is fine and the remaining setup is configuration.

> The keyword scorer is for testing only. It has no idea what the words mean and should not be used to judge real jobs.

---

## Step 5 — Get a Free AI API Key

Pick one provider.

### Gemini — Recommended

Gemini is the recommended free option because it can read PDF resumes directly.

1. Go to [Google AI Studio](https://aistudio.google.com/apikey)
2. Sign in with a Google account.
3. Click **Create API key**.
4. Pick or create a project.
5. Copy the API key.

> A Gemini subscription is not an API key. Google One AI Premium / Gemini Advanced is a separate consumer product. You need Google AI Studio for API access.

### Other Options

| Provider | Where | Card Required? | Reads PDF Resumes |
|---|---|---|---|
| **Groq** | console.groq.com/keys | No | No — export resume to `.txt` |
| **Ollama** | ollama.com | No | No — export resume to `.txt` |
| **Anthropic** | console.anthropic.com | Yes, prepaid | Yes |

---

## Step 6 — Create Your `.env`

Create the environment file.

### Windows

```bash
copy .env.example .env
```

### macOS / Linux

```bash
cp .env.example .env
```

Open `.env` and add:

```env
LLM_PROVIDER=gemini
GEMINI_API_KEY=paste-your-key-here
```

### Check Available Gemini Models

Model names can change. Run:

```bash
python -c "import os,sys,requests; sys.path.insert(0,'.'); from jobhunt.cli import _load_env; _load_env(); print('\n'.join(m['name'].replace('models/','') for m in requests.get('https://generativelanguage.googleapis.com/v1beta/models', params={'key':os.environ['GEMINI_API_KEY']}).json()['models'] if 'generateContent' in m.get('supportedGenerationMethods',[])))"
```

Choose a fast/cheap model for screening and a stronger model for drafting.

Example:

```env
SCREEN_MODEL=gemini-3.5-flash-lite
DRAFT_MODEL=gemini-3.6-flash
```

If either model returns a `404`, use a model from the list printed by the command above.

> `.env` is already included in `.gitignore`. Never commit it or share your API key in a chat, screenshot, or video. If you accidentally expose your key, regenerate it immediately.

---

## Step 7 — Turn Your Resume into a Profile

Place your resume in the project folder.

Run:

```bash
python -m jobhunt profile --resume resume.pdf
```

This creates `profile.json`.

Open `profile.json` and fix anything that is incorrect.

Check especially:

- `years_experience`
- `target_titles`
- `core_skills`

These fields are used when jobs are scored.

For Groq or Ollama, export your resume to `.txt` first and pass that file instead.

---

## Step 8 — Tune Your Filters

Open `config.yaml`.

Configure the job titles, locations, remote preference, age limit, and score threshold.

Example:

```yaml
filters:
  include_titles:
    - 'software engineer'
    - '\bsde\b'

  exclude_titles:
    - '\b(staff|principal|senior)\b'
    - '\b(sales|marketing|recruit)\b'

  locations:
    - bangalore
    - bengaluru

  allow_remote: true
  max_age_days: 30

score_threshold: 7.0
max_per_digest: 5
```

### Important Filter Notes

`SDE` does not match `Software Development Engineer`.

Use:

```yaml
- '\bsde\b'
- 'software development engineer'
```

if you want to match both.

Also exclude seniority levels you do not want. Otherwise, the AI may spend its quota rejecting unsuitable senior roles.

---

## Step 9 — Pick Your Companies

Open `companies.yaml`.

The `slug` is the last part of the company's public careers board URL.

Examples:

| Careers Board URL | ATS | Slug |
|---|---|---|
| `boards.greenhouse.io/stripe` | `greenhouse` | `stripe` |
| `jobs.lever.co/netlify` | `lever` | `netlify` |
| `jobs.ashbyhq.com/ramp` | `ashby` | `ramp` |

To find a company's board, search:

```text
<company> careers greenhouse
```

or:

```text
<company> careers lever
```

or:

```text
<company> careers ashby
```

Start with around **10–15 companies** you would actually consider joining.

A dead slug prints `HTTP 404` and the run continues, so a wrong entry will not break the application.

> This tool does not use LinkedIn or Naukri. Neither provides a public API for this use case.

---

## Step 10 — Run the Application

For your first real run, use a small limit:

```bash
python -m jobhunt run --limit 10
```

Check:

```text
out/tracker.csv
```

Review the `score` and `reason` columns.

### Common Results

| What You See | Meaning | Fix |
|---|---|---|
| `prefilter: 3000 -> 0` | Filters are too tight | Loosen titles, locations, or age |
| Everything scores 3–4 | Filters are too loose | Tighten `exclude_titles` |
| A few jobs score 7–9 | Filters are working | Remove `--limit` |

Once everything looks correct, run:

```bash
python -m jobhunt run
```

Then open:

```text
out/digest.html
```

---

## Step 11 — Send the Digest by Email

For Gmail, you need an **App Password**.

Your normal Gmail password will not work for SMTP when using this setup.

### Create a Gmail App Password

1. Enable **2-Step Verification** on your Google account.
2. Go to [Google App Passwords](https://myaccount.google.com/apppasswords).
3. Create an App Password.
4. Copy the generated 16-character password.

Add the following to `.env`:

```env
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=you@gmail.com
SMTP_PASS=your-16-char-app-password
MAIL_TO=you@gmail.com
```

Then test:

```bash
python -m jobhunt run --send
```

---

## Step 12 — Automate Daily Runs with GitHub Actions

GitHub Actions can run the application automatically every weekday without your laptop being turned on.

### 1. Push Your Configuration

Push:

```text
config.yaml
companies.yaml
```

to your fork.

The following files are intentionally gitignored:

```text
.env
profile.json
resume
seen.json
```

Your API key and profile should never be committed to the repository.

### 2. Add GitHub Secrets

In your fork, go to:

**Settings → Secrets and variables → Actions**

Add these under **Secrets**:

| Secret | Value |
|---|---|
| `PROFILE_JSON` | Entire contents of your local `profile.json` |
| `GEMINI_API_KEY` | Your Gemini API key |
| `SMTP_USER` | Your Gmail address |
| `SMTP_PASS` | Your 16-character App Password |
| `MAIL_TO` | Email address where the digest should be sent |

### 3. Add GitHub Variables

Under **Variables**, add:

| Variable | Value |
|---|---|
| `LLM_PROVIDER` | `gemini` |
| `SCREEN_MODEL` | `gemini-3.5-flash-lite` |
| `DRAFT_MODEL` | `gemini-3.6-flash` |

### 4. Enable the Workflow

Go to:

**Actions → daily job digest**

Enable workflows if they are disabled.

### 5. Test the Workflow

Go to:

**Actions → daily job digest → Run workflow**

Enable `dry_run` and run the workflow.

A dry run builds the digest and uploads it as an artifact without sending an email.

### 6. Automatic Schedule

Once the workflow is working, it runs automatically at:

**06:00 IST every weekday**

To change the time, edit:

```text
.github/workflows/daily.yml
```

The cron schedule uses UTC, so subtract **5 hours 30 minutes** from your intended IST time.

---

## Step 13 — Track Applications

When you apply to a job, mark it:

```bash
python -m jobhunt applied "greenhouse:cloudflare:7462799"
```

View your statistics:

```bash
python -m jobhunt stats
```

Application data is stored in:

```text
out/tracker.csv
```

You can open this file in Excel or Google Sheets.

The file:

```text
seen.json
```

prevents the same job from appearing repeatedly.

Do not delete `seen.json` unless you want to start over.

---

# Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `python: command not found` | Python is not in PATH | Reinstall Python and tick **Add python.exe to PATH** |
| `No module named pip` | Incomplete Python installation | Run `python -m ensurepip --upgrade` |
| PowerShell: `running scripts is disabled` | Default execution policy | Run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass` |
| `No module named jobhunt` | Wrong folder or virtual environment is not active | `cd` into the project root and activate `.venv` |
| `GEMINI_API_KEY is not set` | `.env` is missing or the key is blank | Check `.env` is next to `config.yaml` |
| `gemini HTTP 404` | Model name is no longer available | Run the model-list command from Step 6 |
| `gemini stopped early (finishReason=MAX_TOKENS)` | The model reached its output limit | Raise the relevant token limits in `jobhunt/llm.py` |
| `prefilter: 3000 -> 0` | Filters are too tight | Loosen `include_titles` or `locations` |
| `HTTP 404` next to a company | Invalid/dead ATS slug | Fix or remove the company from `companies.yaml` |
| SMTP `Username and Password not accepted` | Normal Gmail password is being used | Use a 16-character App Password |
| Digest arrives empty | No jobs passed the score threshold | Lower `score_threshold` or loosen filters |

---

# Ground Rules

- **Read before you send.** Cover notes are drafts generated from your `profile.json`. Verify every claim before sending.
- **Keep your profile accurate.** Incorrect profile information can affect job matching.
- **The tool never applies for you.** It only finds, matches, scores, and drafts.
- **Never commit API keys or personal information to Git.**
