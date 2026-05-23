# Feature Prompt — Add a New Feature

Use this prompt when you want to add a new feature to an existing Apparition app.

Fill in the `[brackets]` and paste into **GitHub Copilot Chat** (Agent mode).

---

```
Add a new feature to my existing ASP.NET Core Razor Pages + Dapper + PostgreSQL app.

## What the feature does
[Describe the feature in plain English. Example: "A comments section on each invoice page.
Users can add a comment with their name and the date is recorded automatically."]

## New pages or changes to existing pages
[List what UI changes are needed. Example:
- Add a comment form at the bottom of /Pages/Invoices/Detail.cshtml
- Show a list of existing comments above the form
- No new pages needed]

## New data
[Describe any new data this feature stores. Example:
- Comment: invoiceId, authorName, body, createdAt
- Store as type = 'comment' in the documents table]

## New API endpoints (if needed)
[List any new API endpoints. Example:
- POST /api/invoices/{id}/comments — saves a new comment]

## Stack rules (enforce these)
- Razor Pages for UI, Minimal API for any new endpoints
- Dapper for all database access — no Entity Framework
- Store new data as JSONB in the existing documents table if possible
- Single project — add files to /Pages, /Api, or /Data as appropriate
- Do not change Program.cs unless a new service registration is strictly required
- Do not add NuGet packages unless absolutely necessary — ask first if you need one

## Output required
- All new or modified .cshtml and .cshtml.cs files
- Any new /Data class
- Any new /Api endpoint file
- Any new schema.sql additions (new indexes only — no new tables unless unavoidable)
```

---

## Common Feature Requests

Copy-paste these directly into Copilot Chat (no additional prompt needed):

### Add search
> "Add a search bar to [page name] that filters [type] documents by [field name]. Use a SQL ILIKE query on the JSONB field. Keep everything in the existing Razor Page."

### Add pagination
> "Add simple next/previous pagination to [page name]. Show 20 items per page. Pass page number as a query string parameter."

### Add file upload
> "Add a file upload field to [page name]. Save the file to the wwwroot/uploads folder. Store the filename in the document's JSONB data."

### Add email notification
> "When [event happens], send an email to [recipient] using the Resend API (https://resend.com). Store the Resend API key in an environment variable called RESEND_API_KEY."

### Add a dashboard with counts
> "Add a /dashboard page that shows: total count of [type A], total count of [type B], and [metric]. Use SQL COUNT queries with JSONB filters."

### Export to CSV
> "Add a 'Download CSV' button to [list page] that exports all [type] records as a CSV file. Use a Minimal API endpoint at /api/[type]/export.csv."
