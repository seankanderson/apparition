# Git Setup Guide

Your code needs to live somewhere online so Railway or Render can pull it and deploy it automatically. This is called a **Git repository** (or "repo"). Think of it as a Google Drive folder for your code that keeps a full history of every change.

You have three options. All three work with Railway and Render.

---

## Which service should I choose?

| Service       | Best for                                                  | Sign-up URL                      |
| ------------- | --------------------------------------------------------- | -------------------------------- |
| **GitHub**    | Most people — largest community, most tutorials online    | https://github.com/signup        |
| **GitLab**    | People who want more control or prefer open-source tools  | https://gitlab.com/users/sign_up |
| **Bitbucket** | People who already use Atlassian tools (Jira, Confluence) | https://id.atlassian.com/signup  |

**Not sure?** Choose GitHub. It's what most tutorials, Stack Overflow answers, and AI tools are optimized for.

---

## Option A: GitHub

### Step 1 — Create an account
Go to https://github.com/signup and follow the steps. The free plan is all you need.

### Step 2 — Create a repository
1. Go to https://github.com/new
2. Fill in:
   - **Repository name:** your app name in lowercase with dashes (e.g. `my-invoice-app`)
   - **Visibility:** Private (you can make it public later)
   - **Important:** Do NOT check "Add a README file" — leave all checkboxes empty
3. Click **Create repository**
4. Copy the URL — it looks like: `https://github.com/yourusername/your-repo`

### Step 3 — Connect to Railway
1. Go to [railway.app](https://railway.app) and sign in with GitHub
2. Click **New Project → Deploy from GitHub repo**
3. Select your repository from the list

### Step 4 — Connect to Render (if using Render instead)
1. Go to [render.com](https://render.com) and sign in with GitHub
2. When creating a Web Service, select **Connect a repository**
3. Choose your repository

---

## Option B: GitLab

GitLab gives you more built-in tools (CI/CD, wikis, issue boards) and lets you host your own instance if you ever need full control.

### Step 1 — Create an account
Go to https://gitlab.com/users/sign_up. The free plan includes unlimited private repositories.

### Step 2 — Create a project
1. Go to https://gitlab.com/projects/new
2. Click **Create blank project**
3. Fill in:
   - **Project name:** your app name
   - **Visibility Level:** Private
   - Uncheck "Initialize repository with a README"
4. Click **Create project**
5. Copy the URL — it looks like: `https://gitlab.com/yourusername/your-project`

### Step 3 — Connect to Railway
1. Go to [railway.app](https://railway.app)
2. Click **New Project → Deploy from GitLab repo**
3. Authorize Railway to access your GitLab account
4. Select your repository

### Step 4 — Connect to Render (if using Render instead)
1. Go to [render.com](https://render.com)
2. When creating a Web Service, choose **GitLab** as the source
3. Authorize Render and select your repository

---

## Option C: Bitbucket

Good choice if you're already in the Atlassian ecosystem (Jira, Trello, Confluence).

### Step 1 — Create an account
Go to https://id.atlassian.com/signup. Free plan includes unlimited private repos.

### Step 2 — Create a repository
1. Go to https://bitbucket.org/repo/create
2. Fill in:
   - **Repository name:** your app name
   - **Access level:** Private
   - Uncheck "Include a README?"
3. Click **Create repository**
4. Copy the URL — it looks like: `https://bitbucket.org/yourusername/your-repo`

### Step 3 — Connect to Railway
1. Go to [railway.app](https://railway.app)
2. Click **New Project → Deploy from Bitbucket repo**
3. Authorize Railway to access your Bitbucket account
4. Select your repository

### Step 4 — Connect to Render (if using Render instead)
1. Go to [render.com](https://render.com)
2. When creating a Web Service, choose **Bitbucket** as the source
3. Authorize Render and select your repository

---

## Pushing Your Code (First Time)

After the AI generates your project files, run these commands in the terminal inside your project folder. The AI will give you the exact commands, but here's what they look like:

**GitHub / GitLab / Bitbucket — same commands:**
```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOURUSERNAME/YOUR-REPO.git
git push -u origin main
```

Replace the last URL with your actual repository URL.

> **First time using git?** The AI will walk you through this step by step.
> You only need to do this once — after the first push, every future update is just:
> ```bash
> git add .
> git commit -m "describe what you changed"
> git push
> ```

---

## Keeping Your Code Safe

A few rules that will save you headaches:

- **Never commit secrets.** API keys, passwords, and connection strings go in environment variables — not in code files. The template enforces this.
- **Push often.** Every time you get something working, push. It's your save point.
- **Private repos are free** on all three services. Keep your app private until you're ready to share.
