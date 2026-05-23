# Apparition

> **An AI-powered starter kit for non-technical founders who want to build real apps and get to market — without things going sideways.**

Apparition gives you a production-ready full-stack template, a set of copy-paste AI prompts, and a set of Copilot agents that guide you step-by-step from idea to deployed app. No computer science degree required.

---

## What You Get

| Component               | What it does                                                           |
| ----------------------- | ---------------------------------------------------------------------- |
| **Stack Template**      | ASP.NET Core + Razor Pages + PostgreSQL — pre-wired and ready to clone |
| **Prompt Library**      | Battle-tested AI prompts you paste into Copilot, ChatGPT, or Claude    |
| **AI Agents**           | Agent definitions that work with VS Code, Claude, Cursor, and others   |
| **Styled & Responsive** | Bootstrap 5 built in — every app looks great on mobile out of the box  |
| **Deploy Guides**       | Step-by-step instructions for Railway and Render                       |

The hard decisions — how the app is built, how it stays secure, how it handles logins and sensitive data — are already made. You don't need to worry about the engineering, just the creativity. You  answer simple questions about what you want, and the AI handles the rest as defined by experienced software developers.

Nothing you build from Apparition goes to waste, your app can always be picked up by experienced software engineers at any time and be quickly expanded or upgraded easily. 

---

## The Stack (and why it was chosen for you)

```
Frontend:   Razor Pages  (plain HTML — no React, no build pipeline)
            Alpine.js via CDN  (lightweight reactivity when needed — no build step)
Styling:    Bootstrap 5 via CDN  (responsive grid, full component library, no install)
            site.css  (brand color overrides only — under 25 lines)
            Page styles  (kept inline per page — never added to the central file)
Backend:    ASP.NET Core Minimal API  (.NET 8)
Database:   PostgreSQL with JSONB columns
ORM:        Dapper  (not Entity Framework — intentionally)
Auth:       Cookie-based  (always included — no third-party service needed)
Hosting:    Railway (primary) · Render (fallback)
```

Every decision was made to keep AI output small, predictable, and deployable on first try. Razor Pages handles full-page interactions; Alpine.js (one CDN script tag) handles anything that needs to feel instant — live search, status toggles, modals — without a build pipeline.

**Styling is built in.** Bootstrap 5 is included via CDN — no installation, no build step. When you answer questions about your app's look and feel during the interview, the agent generates a small brand-color override file and that's it. No sprawling stylesheet. Every page keeps its own custom styles inline, right next to the markup that needs them. This keeps the central CSS file tiny (under 25 lines) and means the AI only reads and writes the styles relevant to what it's working on — not the entire project's stylesheet every time.

See [docs/architecture.md](docs/architecture.md) for the full reasoning.

---

## Before You Start — Folder Setup

> **Apparition is a toolkit. Your app lives next to it, not inside it.**

The right setup looks like this:

```
dev\                    ← open THIS folder in VS Code
  ├── apparition\       ← this toolkit (read-only reference)
  └── my-invoice-app\   ← your actual app (built here)
```

Open VS Code at the **parent folder** so both are visible in the sidebar at once. This lets the Copilot agents read the toolkit docs and write to your app at the same time.

New to this? Read [docs/workspace-setup.md](docs/workspace-setup.md) for a full step-by-step guide (Windows and Mac).

---

## Quick Start

### Option 1 — Let the AI interview you (recommended for first-timers)

Open **GitHub Copilot Chat** in VS Code, switch to the `apparition-build` agent, and type:

> "Let's build my app"

The agent will ask you a series of simple questions — one at a time — about your idea, features, data, and where to host it. When you're done answering, it builds everything for you.

Don't have VS Code set up yet? Paste the contents of `prompts/onboarding-questions.md` directly into ChatGPT or Claude and follow the same interview flow.

### Option 2 — Fill in the master prompt yourself

Open `prompts/master-prompt.md`, fill in the `[brackets]`, and paste it into any AI chat.

### Option 3 — Clone and start coding

```bash
git clone https://github.com/your-org/apparition.git my-app
cd my-app
```

Then use the prompts in `prompts/` as needed.

### Deploy

Follow [docs/deployment.md](docs/deployment.md). Your app will be live on Railway in about 10 minutes.
Need to set up GitHub/GitLab/Bitbucket first? See [docs/git-setup.md](docs/git-setup.md).

---

## Folder Structure

```
apparition/
├── README.md                          ← you are here
├── docs/                              ← guides and reference
│   ├── architecture.md                ← why this stack
│   ├── database.md                    ← PostgreSQL JSONB patterns
│   ├── deployment.md                  ← Railway + Render step-by-step
│   ├── git-setup.md                   ← GitHub / GitLab / Bitbucket setup
│   ├── workspace-setup.md             ← folder organization (start here if confused)
│   ├── customization.md               ← how to make it yours
│   └── troubleshooting.md             ← when things go wrong
├── prompts/                           ← AI prompts (works raw or via agent)
│   ├── onboarding-questions.md        ← START HERE — guided interview
│   ├── master-prompt.md               ← fill-in-the-blank build prompt
│   ├── auth-prompt.md                 ← add login/logout
│   ├── feature-prompt.md              ← add any new feature
│   └── deployment-prompt.md           ← deploy assistance
├── agents/                            ← AI agent definitions (VS Code, Claude, Cursor, etc.)
│   ├── apparition-build.agent.md      ← scaffold your app (runs interview)
│   ├── apparition-deploy.agent.md     ← deploy your app
│   └── apparition-feature.agent.md    ← add features safely
├── AGENTS.md                          ← AI entry point — any agent reads this first
└── .github/
    └── copilot-instructions.md        ← VS Code Copilot adapter (defers to AGENTS.md)
```

---

## Philosophy

- **One project. One file to wire everything.** No microservices. No monorepos.
- **No magic.** Every line of code is readable by a curious non-developer.
- **Secure by default.** Login is always included. Your API keys never end up where they shouldn't. The AI follows the security rules whether you think to ask or not.
- **Fast by design.** Server-rendered HTML lands in the browser in one round trip. No JavaScript framework to boot, no API waterfall, no client-side hydration. Pages are lightweight and snappy, which means cheaper hosting — you're serving small HTML responses, not running a front-end build server.
- **Built to conserve AI tokens.** Styles live with the component they belong to. The central CSS file stays under 25 lines. Data models are flat JSONB — no sprawling schema to paste into every prompt. Files are short by design. Every one of these choices means the AI reads less before it can act, which saves you money on AI usage and keeps responses faster and more accurate.
- **AI-first, not AI-dependent.** The app works without Copilot. Copilot just makes it faster.
- **Deploy on day one.** The template is production-ready out of the box.

---

## Docs

- [Onboarding interview](prompts/onboarding-questions.md) ← start here
- [Architecture decisions](docs/architecture.md)
- [Database patterns](docs/database.md)
- [Workspace & folder setup](docs/workspace-setup.md)
- [Git setup (GitHub / GitLab / Bitbucket)](docs/git-setup.md)
- [Deployment guide](docs/deployment.md)
- [Customization guide](docs/customization.md)
- [Troubleshooting](docs/troubleshooting.md)

---

## License

MIT — use it, fork it, ship it.
