# Troubleshooting

Common problems and how to fix them.

---

## Local Development

### "Cannot connect to database"

**Symptom:** App starts but crashes with a connection error on first request.

**Fixes:**
1. Make sure PostgreSQL is running locally (`pg_ctl status` or check your system services)
2. Check `appsettings.Development.json` — confirm the host, database name, username, and password match your local setup
3. If you just installed PostgreSQL, make sure you've run `schema.sql` against your local database:
   ```bash
   psql -U postgres -d myapp -f schema.sql
   ```

---

### "Port 5000 is already in use"

**Symptom:** `dotnet run` fails with an address-in-use error.

**Fix:** Kill whatever is using port 5000:
```bash
# Windows
netstat -ano | findstr :5000
taskkill /PID <pid> /F

# Mac/Linux
lsof -ti:5000 | xargs kill
```

Or change the port in `launchSettings.json`.

---

### Page shows an error but no message

1. In `Program.cs`, make sure you have `app.UseDeveloperExceptionPage()` in development
2. Check the terminal where `dotnet run` is running for the full stack trace

---

## Deployment (Railway)

### Build fails with "not a .NET project"

Railway sometimes misdetects the project. Fix:
1. Make sure your `.csproj` file is in the repository root (or the `/App` subfolder)
2. Set the **Root Directory** in Railway service settings to the folder containing your `.csproj`

---

### App crashes on startup in Railway

1. Check the **Logs** tab in Railway
2. Most common cause: missing `ConnectionStrings__Default` environment variable
3. Make sure you added the variable in **Service → Variables**, not in the PostgreSQL service

---

### "relation 'documents' does not exist"

The schema hasn't been run against the production database.

1. Go to your Railway PostgreSQL service → **Data → Query**
2. Paste the contents of `schema.sql` and run it

---

### App deploys but shows a blank page or 404

1. Make sure `ASPNETCORE_ENVIRONMENT` is set to `Production`
2. Check that your Razor Pages route matches what you're visiting (e.g. `/` maps to `Pages/Index.cshtml`)
3. Check Railway logs for any startup errors

---

## Deployment (Render)

### Build times out

Render's free tier has a 30-minute build limit. If your build exceeds it:
1. Use the Dockerfile approach (see [deployment.md](deployment.md)) — it's more predictable
2. Upgrade to a paid plan if you need faster builds

---

### Health check fails

Render pings your app's root URL (`/`) to confirm it's alive. If it gets anything other than a 200 response, it marks the deploy as failed.

Fix: Make sure your `Pages/Index.cshtml` returns a 200 OK (the default for a valid Razor Page).

---

## AI / Copilot Issues

### AI generated code that doesn't compile

1. Copy the exact compiler error message
2. Paste it back into Copilot Chat: *"This code gives the following error: [paste error]. Fix it."*
3. If the same error keeps coming back, try the [master prompt](../prompts/master-prompt.md) from scratch with a smaller scope

---

### AI keeps adding Entity Framework or complex patterns

Your prompt may not be explicit enough. Add this line to any prompt:
> "Use Dapper for all database access. Do not use Entity Framework or any ORM other than Dapper."

---

### AI output is correct but doesn't match the file structure

Add this line to your prompt:
> "The project structure is: /Pages for Razor Pages, /Api for Minimal API endpoints, /Data for Dapper data access, and Program.cs for all wiring."

---

## Getting More Help

If you're stuck and none of the above helps:

1. Open `apparition-build.agent.md` in VS Code and invoke the agent — describe exactly what's broken
2. Check the [GitHub Discussions](https://github.com/your-org/apparition/discussions) for this repo
3. File an issue with the full error message and what you were trying to do
