# Master Prompt — Build My App

Copy this entire prompt, fill in the `[brackets]`, and paste it into **GitHub Copilot Chat** (Agent mode) or **ChatGPT / Claude**.

---

```
Build a full-stack web application with the following specifications.

## What the app does
[Describe your app in plain English. Example: "A simple invoicing tool for freelancers.
Users can create invoices, mark them as paid, and view a list of all invoices."]

## App name and tagline
- Name: [Your app name, e.g. InvoiceKit]
- Tagline: [One short line that describes the app, e.g. "Invoicing for freelancers" — leave blank if none]

Use the name in `<title>@ViewData["Title"] — AppName</title>` and as the navbar brand in `_Layout.cshtml`.
If a tagline is provided, show it as a small subtitle under the brand name in the navbar.

## Pages / Screens needed
[List every screen the user will see. Example:
- Home page with a welcome message and a "Create Invoice" button
- Invoice list page showing all invoices with status
- Create invoice form
- Invoice detail page]

## Data the app stores
[Describe the information the app needs to remember. Example:
- Invoice: client name, invoice number, line items (description + amount), total, status (draft/sent/paid), due date]

## Stack requirements (do not change these)
- ASP.NET Core (.NET 8)
- Razor Pages for all UI — no React, no SPA, no frontend build pipeline
- Minimal API for any backend endpoints
- PostgreSQL database
- JSONB columns for data storage — use the document-per-row pattern
- Dapper for all database access — do NOT use Entity Framework
- Cookie-based authentication if login is needed
- Single project structure: /Pages, /Api, /Data, Program.cs

## Project structure rules (follow exactly)
- /Pages — one .cshtml + .cshtml.cs pair per page
- /Api — Minimal API endpoints grouped by domain
- /Data — Dapper data access, one class per domain concept
- Program.cs — all service registration and middleware, keep under 60 lines
- schema.sql at the project root — all CREATE TABLE and CREATE INDEX statements

## Output required
1. Complete project file structure (every file listed)
2. schema.sql with all tables and indexes
3. Program.cs
4. All Razor Page files (.cshtml and .cshtml.cs)
5. All data access classes in /Data
6. Any API endpoint files in /Api
7. appsettings.json and appsettings.Development.json
8. Properties/launchSettings.json (so `dotnet run` works out of the box with http://localhost:5000)
9. wwwroot/images/ — if the user has placed logo or brand asset files in planning/assets/, copy them here and reference them in _Layout.cshtml

## Constraints
- No Entity Framework, no migrations, no DbContext
- No dependency injection abstractions (no IRepository<T>, no service interfaces)
- No JavaScript frameworks
- No background jobs or hosted services
- Keep each file short — prefer more files over long files
- All database access uses Dapper and raw SQL
- Connection string comes from environment variable: ConnectionStrings__Default

## Deployment target
Railway (primary). The app must work with: dotnet publish -c Release
```

---

## Tips for Better Results

- **Be specific about your pages.** The more detail you give about what each screen shows, the less back-and-forth you'll need.
- **Describe your data like a form.** Think about what fields you'd fill in to create one record.
- **One feature at a time.** If your app has 10 features, build 2–3 first, deploy, then add more.
- **If something doesn't compile**, paste the error back into the chat: *"This error appeared: [paste error]. Fix it without changing the stack."*
