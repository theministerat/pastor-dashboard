# Aaron's Personal Dashboard

A personal operations dashboard for Ford Street United Methodist Church, built for MiniMax Code.

## Setup

1. **Create a GitHub repository**:
   - Go to https://github.com/new
   - Name it `pastor-dashboard` (or your preferred name)
   - Select "Public"
   - Don't initialize with README (we have one)

2. **Connect and push**:
   ```bash
   cd pastor-dashboard
   git remote add origin https://github.com/YOUR_USERNAME/pastor-dashboard.git
   git branch -M main
   git push -u origin main
   ```

3. **Enable GitHub Pages**:
   - Go to your repo settings on GitHub
   - Go to "Pages" (under "Code and automation")
   - Under "Build and deployment" → "Source", select "Deploy from a branch"
   - Under "Branch", select "gh-pages" and "/ (root)"
   - Click Save

4. **Wait for deployment**:
   - GitHub Actions will automatically deploy on push
   - You'll get a URL like `https://yourusername.github.io/pastor-dashboard/`

## Usage

- Open the dashboard in your browser
- Click "Sync" to refresh data (this copies a prompt to paste in chat with Mavis)
- For sermon prep tasks, click the buttons to copy prompts for the relevant skills

## Files

- `dashboard.html` - Main dashboard (also deployed as index.html)
- `pastor-dashboard-refresh.SKILL.md` - Skill for refreshing the dashboard
- `.github/workflows/deploy.yml` - GitHub Actions workflow for auto-deployment