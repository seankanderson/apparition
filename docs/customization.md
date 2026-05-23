# Customization Guide

Apparition is a starting point, not a cage. This guide shows you how to shape it into exactly the app you want to build.

---

## The Golden Rule

**Change one thing at a time, test it, then move on.**

This applies whether you're writing code yourself or prompting AI. Small steps = fewer broken things = less frustration.

---

## Changing the App Name

1. Rename the project folder from `App` to `YourAppName`
2. Open `App.csproj` (or whatever it's named) and change the `<AssemblyName>` and `<RootNamespace>` values
3. Update `README.md` with your app's name and description
4. In Railway/Render, update the service name in the dashboard

---

## Adding Pages

Every page in a Razor Pages app is a pair of files:

```
/Pages
  /MyNewPage
    Index.cshtml       ← HTML template
    Index.cshtml.cs    ← C# code-behind (handles form submissions, loads data)
```

**Prompt to use:**
> Use the `apparition-feature` Copilot agent or paste from [prompts/feature-prompt.md](../prompts/feature-prompt.md)

---

## Adding API Endpoints

Add a new file in `/Api` for each domain area:

```csharp
// /Api/Products.cs
public static class ProductsApi
{
    public static void Map(WebApplication app)
    {
        app.MapGet("/api/products", async (IDbConnection db) =>
        {
            // ...
        });
    }
}
```

Then register it in `Program.cs`:
```csharp
ProductsApi.Map(app);
```

---

## Adding Authentication

Authentication is not included by default but can be added in one step.

Use the auth prompt: [prompts/auth-prompt.md](../prompts/auth-prompt.md)

Or invoke the `apparition-build` agent in Copilot Chat and ask:
> "Add cookie-based login and logout to my Apparition app"

---

## Changing the Database Schema

To add a new type of data:

1. Add a new `INSERT` pattern in your data class under `/Data`
2. Add a new section to `schema.sql` if you need a new index
3. Re-run the new statements against your database

You rarely need a new table. Try storing the new data as a new `type` value in the `documents` table first.

---

## Styling Your App

Apparition ships with minimal styling. To make it look great:

**Option 1 — Bootstrap (easiest)**
Add to your `_Layout.cshtml`:
```html
<link rel="stylesheet"
      href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
```

**Option 2 — Tailwind CSS via CDN**
```html
<script src="https://cdn.tailwindcss.com"></script>
```

**Option 3 — Custom CSS**
Edit `wwwroot/css/site.css` — it's loaded on every page.

---

## Adding File Uploads

Ask the `apparition-feature` agent:
> "Add a file upload field to [page name] that saves files to [Railway Volume / Azure Blob / S3]"

---

## Adding Email

The simplest approach for transactional email is [Resend](https://resend.com) — one API call, generous free tier.

Ask the `apparition-feature` agent:
> "Add email sending using Resend when a user submits [form name]"

---

## Environment-Specific Configuration

- `appsettings.json` — base config, committed to source control (no secrets here)
- `appsettings.Development.json` — local dev overrides, committed (no secrets)
- Environment variables — production secrets, never committed

---

## When to Reach Beyond the Template

These are signs you've outgrown the one-table pattern and should talk to a developer:

- You need real-time features (WebSockets, live updates)
- You're processing payments at scale (> $10k/month GMV)
- You need complex reporting across millions of rows
- You have compliance requirements (HIPAA, SOC2, PCI)

The template will still get you to those problems. That's a good problem to have.
