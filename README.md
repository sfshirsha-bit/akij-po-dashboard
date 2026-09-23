# Akij PO Dashboard — GitHub Pages + Scheduled Sync

Live PO dashboard hosted on GitHub Pages. Your office laptop fetches data from
iBOS (MSSQL) every 30 minutes and pushes it to this repo, so the dashboard is
always online even when the laptop is off (it shows the last-pushed snapshot).

## Files
- `index.html` — the dashboard (reads `po_data.json`).
- `po_data.json` — the data snapshot (auto-updated by the laptop).
- `fetch_push.js` — Node script: connects to MSSQL, writes `po_data.json`, commits & pushes.
- `run_fetch.bat` — wrapper that runs `fetch_push.js` (called by Task Scheduler).

## One-time setup

### 1. Create the GitHub repo
1. Create a GitHub account (https://github.com).
2. New repository → name it `akij-po-dashboard` → **Public** → Create.

### 2. Upload these files
- Easiest: on the repo page, click **Add file → Upload files**, drag in all 4 files, commit.
- (Or use git from your laptop — see "Git CLI" below.)

### 3. Enable GitHub Pages
- Repo **Settings → Pages → Source: `main` branch → / (root) → Save**.
- After ~1 minute your dashboard is live at:
  `https://YOUR-USERNAME.github.io/akij-po-dashboard/`

### 4. On your office laptop — clone the repo
```
git clone https://github.com/YOUR-USERNAME/akij-po-dashboard.git
cd akij-po-dashboard
```
Then drop `fetch_push.js` and `run_fetch.bat` into the cloned folder (if not already there).

### 5. Install Node dependency (once)
```
npm install mssql
```

### 6. Set up git credentials (once)
So `git push` doesn't ask for a password every 30 min:
```
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global credential.helper wincred
```
First push will prompt for your GitHub username + a **Personal Access Token**
(create one: GitHub → Settings → Developer settings → Personal access tokens →
Tokens (classic) → `repo` scope). Paste the token as the password; Windows caches it.

### 7. Point the batch file at the repo
Edit `run_fetch.bat`, set `REPO_DIR` to your cloned repo path:
```
set REPO_DIR=C:\path\to\akij-po-dashboard
```

### 8. Schedule every 30 min
1. Open **Task Scheduler** (Start → "Task Scheduler").
2. **Create Basic Task** → name "PO Dashboard Sync".
3. Trigger: **Daily**, recur every **30 minutes** (or set a repeat interval).
4. Action: **Start a program** → browse to `run_fetch.bat`.
5. Finish.

## How it works
- Laptop on office Wi-Fi/VPN → Task Scheduler runs `run_fetch.bat` → `fetch_push.js`
  connects to MSSQL, writes fresh `po_data.json`, commits & pushes.
- Dashboard on GitHub Pages always shows the latest `po_data.json` (with a
  "last updated" timestamp). When the laptop is off/home, it shows the last synced snapshot.

## Security note
The DB credentials live inside `fetch_push.js` on your laptop only — never commit
them to GitHub. If you move this repo elsewhere, keep that file out of the public repo
or use environment variables instead.
