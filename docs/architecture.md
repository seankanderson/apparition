# Architecture Decisions

This document explains every major stack choice made for Apparition and *why* it was made specifically for AI-assisted, non-technical development.

---

## The Core Problem This Solves

Most web app templates are built for experienced developers. They use complex frameworks, multi-repo setups, and heavy abstractions that confuse AI models and overwhelm beginners. When AI writes code for these stacks it produces:

- Too many files
- Too much boilerplate
- Too many things that can break in unexpected ways

Apparition is designed so that **AI always writes the same 4–5 patterns** and they always work.

---

## Stack Decisions

### Frontend: Razor Pages (not React, not Blazor)

**Chosen because:**
- Plain HTML + C# = lowest possible token usage when prompting AI
- No component trees, no state management, no build pipeline
- AI produces correct, working Razor code on first try ~90% of the time
- A non-developer can read and understand the output

**Rejected alternatives:**
| Framework | Why rejected                                                      |
| --------- | ----------------------------------------------------------------- |
| React     | Massive token usage; AI often produces broken component wiring    |
| Vue       | Still component complexity; adds build step                       |
| Blazor    | Less mature; more abstraction layers; larger AI confusion surface |
| HTMX      | Good option for v2 — too niche for first-time users today         |

---

### Reactive Layer: Alpine.js (CDN only)

Razor Pages handles the majority of interactions well. When a specific page needs instant feedback — a toggle, a live search, an inline edit, a status change without a page reload — Alpine.js is added from CDN.

**Why Alpine.js and not HTMX, React, or Vue:**
- No build step — one `<script>` tag in `_Layout.cshtml` is all that's needed
- Declarative HTML attributes (`x-data`, `x-show`, `x-model`, `@click`) are readable by non-developers
- AI generates correct Alpine.js code reliably with very short prompts
- Calls the existing Minimal API endpoints via `fetch()` natively
- ~15kb — no impact on page load

**Decision framework:**

| Need                                 | Approach                                               |
| ------------------------------------ | ------------------------------------------------------ |
| Form submit, page navigation, CRUD   | Razor Pages — full page render, no JS                  |
| Toggle, show/hide, modal, counter    | Alpine.js `x-data` / `x-show` — no API call needed     |
| Live search, filter, inline edit     | Alpine.js + `fetch()` to Minimal API                   |
| Status change without page reload    | Alpine.js + `fetch()` to Minimal API                   |
| Dashboard that auto-refreshes        | Alpine.js + `setInterval` + `fetch()`                  |
| Real-time (WebSockets, live updates) | Out of scope — Apparition is not a real-time framework |

**Most apps use both.** The server renders the full page on load; Alpine handles the specific interactions that would feel broken with a page reload.

**To enable Alpine.js in a project**, add to `Pages/Shared/_Layout.cshtml`:
```html
<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script>
```

---

### Backend: ASP.NET Core Minimal API (.NET 8)

**Chosen because:**
- Minimal API = a few lines per endpoint, no controller classes
- .NET 8 LTS = stable, well-supported, Railway/Render compatible
- AI handles Minimal API extremely well
- Single `Program.cs` wires the entire app

**Key rule:** Everything lives in one project. No separate API project. No class libraries.

---

### Database: PostgreSQL + JSONB

**Chosen because:**
- JSONB lets you store any business object without designing a schema upfront
- Non-technical users don't have to understand table design
- AI can generate a working data layer with one table pattern every time
- Cheaper and simpler than Cosmos DB, MongoDB Atlas, or Firestore

**The universal table pattern:**
```sql
CREATE TABLE documents (
    id   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    type TEXT NOT NULL,
    data JSONB NOT NULL
);

CREATE INDEX idx_documents_type ON documents(type);
```

Store invoices, users, orders, products — anything — in `data`.

---

### ORM: Dapper (not Entity Framework Core)

**Chosen because:**
- EF Core migrations are token-heavy and frequently break for non-technical users
- EF tracking bugs are nearly impossible to explain to someone learning
- Dapper = write SQL, get results — no magic, no surprises
- AI writes clean, correct Dapper code with very short prompts

---

### Auth: Cookie-based (ASP.NET built-in)

**Chosen because:**
- Zero third-party services to set up
- No API keys to manage
- Works offline and in local dev without configuration
- Simple mental model: login sets a cookie, logout clears it

**Not included by default** but add it with the [auth prompt](../prompts/auth-prompt.md).

---

### Hosting: Railway (primary) · Render (fallback)

**Railway chosen because:**
- Auto-detects .NET projects and runs `dotnet publish` automatically
- Built-in PostgreSQL with one click
- GitHub integration — push to deploy
- Free tier is generous enough to validate an idea

**Render as fallback** because it accepts the same repo with a Dockerfile if Railway detection fails.

---

## Project Structure Rules

These rules exist so AI always generates code in the right place:

```
/App
  /Pages        Razor Pages UI — one .cshtml + .cshtml.cs pair per page
  /Api          Minimal API endpoint files — grouped by domain
  /Data         Dapper data access — one class per entity/concept
  Program.cs    All wiring lives here — keep it under 60 lines
  appsettings.json
```

**Never:**
- Create a `/Services` folder of abstract interfaces
- Add a separate test project in the template
- Use `IRepository<T>` patterns
- Split into multiple projects

---

## Token Budget Philosophy

Every architectural decision was evaluated against a single question:

> *If a non-technical user copies this into an AI prompt, will the AI produce working code in one shot?*

Simpler stack = fewer tokens = fewer mistakes = less frustration = ships faster.
