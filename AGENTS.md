# Apparition — Instructions for AI Agents

Read this file before doing anything else. It tells you what this project is, what every file does, and the rules you must follow.

---

## What This Project Is

Apparition is an **AI Template Starter Kit** for non-technical founders building their first web app. It is a toolkit — a collection of prompts, agent definitions, and reference docs — not an app itself.

**Your role** is to help a user build *their* app using this toolkit. Their app lives in a separate folder next to this one. You write to their app; you read from this toolkit.

---

## What to Do Based on What the User Asks

| User says...                                            | What to do                                                                                                  |
| ------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| "Build me an app" / "Let's start" / "I have an idea"    | Load `agents/apparition-build.agent.md`, then run the guided interview in `prompts/onboarding-questions.md` |
| "Add a feature" / "I need a new page" / "Can you add X" | Load `agents/apparition-feature.agent.md`                                                                   |
| "Deploy" / "How do I go live" / "Set up Railway"        | Load `agents/apparition-deploy.agent.md`                                                                    |
| "How do I set up my folder" / "Where do things go"      | Read `docs/workspace-setup.md` and explain it                                                               |
| "How does the database work"                            | Read `docs/database.md`                                                                                     |
| "Why did you choose this stack"                         | Read `docs/architecture.md`                                                                                 |
| Any question about git, GitHub, GitLab, Bitbucket       | Read `docs/git-setup.md`                                                                                    |
| Deployment questions                                    | Read `docs/deployment.md`                                                                                   |
| Something broke                                         | Read `docs/troubleshooting.md`                                                                              |

**Always confirm** which folder you are working in before writing any files. Apparition is a toolkit — the user's app is a sibling folder. Never write generated app code into the Apparition toolkit folder.

---

## File Map

### Root
| File              | Purpose                                             |
| ----------------- | --------------------------------------------------- |
| `AGENTS.md`       | This file. Universal entry point for any AI agent.  |
| `README.md`       | Human-readable introduction to the toolkit.         |
| `CHANGELOG.md`    | Version history.                                    |
| `CONTRIBUTING.md` | How to contribute to the Apparition toolkit itself. |

### `docs/`
Reference documentation. Read before explaining anything to the user.

| File                      | Purpose                                                                               |
| ------------------------- | ------------------------------------------------------------------------------------- |
| `docs/architecture.md`    | Every stack decision and why it was made. Read this before suggesting any technology. |
| `docs/database.md`        | PostgreSQL + JSONB patterns with Dapper code examples.                                |
| `docs/deployment.md`      | Railway and Render step-by-step deployment guides.                                    |
| `docs/git-setup.md`       | GitHub, GitLab, and Bitbucket account and repo setup.                                 |
| `docs/workspace-setup.md` | How to organise the parent folder so both the toolkit and the user's app are visible. |
| `docs/customization.md`   | How to adapt the template for a specific use case.                                    |
| `docs/troubleshooting.md` | Common errors and how to fix them.                                                    |

### `prompts/`
AI prompts that work as raw copy-paste (ChatGPT, Claude, etc.) or as agent instructions.

| File                              | Purpose                                                                                           |
| --------------------------------- | ------------------------------------------------------------------------------------------------- |
| `prompts/onboarding-questions.md` | The guided interview. Run this when a user wants to build a new app. Ask questions one at a time. |
| `prompts/master-prompt.md`        | Fill-in-the-blank build prompt. Used after the interview is complete.                             |
| `prompts/auth-prompt.md`          | Detailed instructions for implementing cookie auth if needed beyond the default.                  |
| `prompts/feature-prompt.md`       | Template for adding a new feature to an existing app.                                             |
| `prompts/deployment-prompt.md`    | Deployment assistance prompt.                                                                     |

### `agents/`
Agent definition files. Load the relevant one based on what the user needs.

| File                                 | Purpose                                                                                   |
| ------------------------------------ | ----------------------------------------------------------------------------------------- |
| `agents/apparition-build.agent.md`   | Runs the onboarding interview and scaffolds a complete new app. Load this for new builds. |
| `agents/apparition-feature.agent.md` | Safely adds features to an existing app without breaking what already works.              |
| `agents/apparition-deploy.agent.md`  | Guides the user through deploying to Railway or Render.                                   |

### `.github/`
| File                              | Purpose                                                                                   |
| --------------------------------- | ----------------------------------------------------------------------------------------- |
| `.github/copilot-instructions.md` | VS Code Copilot adapter — defers to this file for rules, adds VS Code-specific behaviour. |

---

## Stack — Follow These Rules Exactly

| Layer               | Technology                            | Rule                                                                                                                                                                         |
| ------------------- | ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Frontend — SSR      | Razor Pages (.cshtml)                 | No React, no Vue, no Blazor, no HTMX                                                                                                                                         |
| Frontend — Reactive | Alpine.js via CDN                     | One `<script>` tag only — no npm, no build step                                                                                                                              |
| Frontend — Styling  | Bootstrap 5 via CDN                   | Always included. `site.css` = brand color overrides only (under 25 lines). Page-specific styles go in `@section Styles` blocks inside `.cshtml` files — never in `site.css`. |
| Backend             | ASP.NET Core Minimal API (.NET 8)     | No MVC controllers                                                                                                                                                           |
| Database            | PostgreSQL + JSONB                    | Use the `documents` table; no new tables without strong justification                                                                                                        |
| ORM                 | Dapper                                | Never Entity Framework                                                                                                                                                       |
| Auth                | ASP.NET cookie auth                   | Always included. No JWT, no Auth0, no third-party identity                                                                                                                   |
| Hosting             | Railway (primary) · Render (fallback) |                                                                                                                                                                              |

**Never suggest an alternative to any of these.** If a deviation is genuinely necessary, flag it explicitly before implementing and explain why.

---

## Rendering Mode — SSR vs. Reactive

Razor Pages handles everything by default. Add Alpine.js only when a specific interaction must happen without a page reload.

| Need                                    | Approach                                           |
| --------------------------------------- | -------------------------------------------------- |
| Form submit, navigation, CRUD           | Razor Pages — full page render, no JS              |
| Toggle, show/hide, modal, counter       | Alpine.js `x-data` / `x-show` — no API call needed |
| Live search, inline edit, status change | Alpine.js + `fetch()` to Minimal API               |
| Auto-refreshing dashboard               | Alpine.js + `setInterval` + `fetch()`              |
| Real-time (WebSockets, live chat)       | Out of scope for Apparition                        |

To enable Alpine.js, add to `Pages/Shared/_Layout.cshtml`:
```html
<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script>
```

---

## Security Rules — Non-Negotiable

### API Keys and Third-Party Services
- **Never put API keys in front-end code.** Not in `.cshtml`, not in Alpine.js, not in any JavaScript.
- All third-party calls (email, SMS, payments, AI, maps, etc.) go through a Minimal API proxy endpoint in `/Api/`.
- The endpoint reads the key from an environment variable. The client never sees it.

### API Endpoint Authorization
- **Every `/api/*` endpoint defaults to `.RequireAuthorization()`.**
- Use `.AllowAnonymous()` only when the endpoint meets ALL of:
  - Returns no user data or PII
  - No write operations
  - No third-party cost (no LLM calls, email sends, SMS, etc.)
  - Cannot be used for spam, scraping, or data exfiltration
- When in doubt, require auth. It is always easier to open an endpoint than to recover from a breach.

### Auth is Always Included
Every app gets authentication. The minimum is a single admin account. There is no "no auth" option.

---

## Code Conventions

- Use `async/await` throughout — avoid unnecessary async chains
- Inject `IDbConnection` via DI — never `new NpgsqlConnection()` inline
- Serialize/deserialize JSONB with `System.Text.Json` (not Newtonsoft)
- Connection string always from `ConnectionStrings__Default` environment variable
- No abstract interfaces — no `IRepository<T>`, no `IService<T>`
- Prefer `record` types for DTOs
- No unit test project in the template

### Generated App Structure
```
/Pages        Razor Pages — one .cshtml + .cshtml.cs per page
/Api          Minimal API endpoints — grouped by domain
/Data         Dapper data access — one class per domain concept
Program.cs    ALL wiring lives here — keep under 60 lines
schema.sql    ALL CREATE TABLE and CREATE INDEX statements
```

---

## What You Must Never Do

- Write app code into the Apparition toolkit folder
- Suggest Entity Framework or any migration framework
- Add JavaScript build tooling (webpack, vite, etc.)
- Add React, Vue, Svelte, or any JS framework requiring a build step
- Generate abstract service or repository layers
- Hardcode connection strings, API keys, or secrets in any committed file
- Create a new database table without first explaining why the documents table won't work
- Split the app into multiple projects
- Add NuGet packages without naming them and explaining why

---

## Communication Style

The user is a non-technical founder. They are not a developer.

- Use plain English — no jargon, no acronyms without explanation
- Be direct and brief — one short paragraph before showing code
- If something might break an existing feature, say so clearly before proceeding
- Never show partial code snippets — always generate complete, ready-to-save files
- After every task, state exactly which files were created or changed
