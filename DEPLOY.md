# Deployment Guide — GitHub + Render

This app can be deployed in two ways:
1. **GitHub Codespaces** — Test live without leaving GitHub
2. **Render** — Permanent production URL with auto-deploy from GitHub

---

## Option 1: GitHub Codespaces (Quick Test)

No external platform needed. Runs entirely in GitHub's cloud.

### Steps:
1. Push this repo to GitHub
2. On your repo page, click **"Code"** → **"Codespaces"** tab
3. Click **"Create codespace on main"**
4. Wait ~30 seconds for the container to boot
5. In the terminal, run:
   ```bash
   python app.py
   ```
6. A popup appears: *"Your application running on port 5000 is available"*
7. Click **"Open in Browser"**
8. You get a URL like: `https://weave-cad-abc123-xyz.app.github.dev`

> The URL is public and shareable. It stays active while your Codespace is running.

---

## Option 2: Render (Permanent Production URL)

### Step 1: Push to GitHub
Upload all files to a GitHub repository.

### Step 2: Create Render Account
1. Go to [render.com](https://render.com)
2. Sign up with your GitHub account
3. Authorize Render to access your repos

### Step 3: Create Web Service
1. On Render dashboard, click **"New +"** → **"Web Service"**
2. Select your `weave-cad` repository
3. Render auto-detects settings from `render.yaml`:
   - **Name**: `weave-cad`
   - **Runtime**: Python 3
   - **Build Command**: `pip install -r requirements.txt`
   - **Start Command**: `gunicorn app:app --bind 0.0.0.0:$PORT --workers 2 --timeout 120`
4. Click **"Create Web Service"**
5. Wait 2-3 minutes for build
6. Your app is live at: `https://weave-cad.onrender.com`

### Step 4: Enable Auto-Deploy (GitHub Actions)

Every push to `main` branch will automatically redeploy on Render.

#### Setup:
1. On Render, go to your service → **Settings** → copy **Service ID**
2. Go to Render Account Settings → **API Keys** → create new key
3. On GitHub repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**
4. Add two secrets:
   - `RENDER_SERVICE_ID` = your service ID from step 1
   - `RENDER_API_KEY` = your API key from step 2

Now every `git push` automatically updates your live app!

---

## File Structure

```
weave-cad-app/
├── app.py                      # Flask backend
├── requirements.txt            # Python dependencies
├── render.yaml                 # Render platform config
├── README.md                   # Documentation
├── DEPLOY.md                   # This file
├── .gitignore                  # Git ignore rules
├── .devcontainer/
│   └── devcontainer.json       # GitHub Codespaces config
├── .github/
│   └── workflows/
│       ├── deploy.yml          # Auto-deploy to Render
│       └── ci.yml              # Test on every push
├── templates/
│   └── index.html              # Frontend
└── uploads/
    └── .gitkeep                # Empty uploads folder
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Build fails on Render | Check Render Events log; usually missing dependency in `requirements.txt` |
| Port already in use | Ensure `app.py` reads `PORT` from environment, not hardcoded |
| App sleeps after 15 min | Free tier limitation; upgrade to paid or use GitHub Codespaces |
| Auto-deploy not working | Check `RENDER_SERVICE_ID` and `RENDER_API_KEY` secrets are correct |
| Codespace URL not working | Make sure port 5000 is forwarded; check "Ports" tab in Codespace |

---

## Free Tier Limits

| Platform | Limit |
|----------|-------|
| Render Free | Sleeps after 15 min inactivity; 512 MB RAM; 0.1 CPU |
| GitHub Codespaces | 120 hours/month; 2 core; 8 GB RAM; 32 GB storage |
| GitHub Actions | 2,000 minutes/month for private repos; unlimited for public |
