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

### App crashes on startup — "Format of the initialization string does not conform to specification starting at index 0"

**Symptom:** The app deploys without a build error but immediately crashes. Railway logs show a connection string parse error mentioning "index 0".

**Cause:** Railway injects the database connection as a `postgresql://` URI (`DATABASE_URL`). ASP.NET Core's `NpgsqlConnection` expects ADO.NET format (`Host=...;Port=...`). If `Program.cs` passes the URI directly to `NpgsqlConnection`, it fails to parse it.

**Fix — for apps generated with the current scaffold:**
If your `Program.cs` already contains `ResolveConnectionString()`, this should not happen. Check that:
1. `DATABASE_URL` is being read (not `ConnectionStrings__Default`) — the helper checks `DATABASE_URL` first
2. The Railway PostgreSQL service is linked to your app service (so `DATABASE_URL` is actually injected)

**Fix — for apps without `ResolveConnectionString()`:**
Add this helper to `Program.cs` and update the `IDbConnection` registration to use it:

```csharp
string ResolveConnectionString()
{
    var raw = Environment.GetEnvironmentVariable("DATABASE_URL")
              ?? builder.Configuration.GetConnectionString("Default")
              ?? throw new InvalidOperationException("No database connection string configured.");

    if (!raw.StartsWith("postgres://", StringComparison.OrdinalIgnoreCase) &&
        !raw.StartsWith("postgresql://", StringComparison.OrdinalIgnoreCase))
        return raw;

    var uri      = new Uri(raw);
    var userInfo = uri.UserInfo.Split(':', 2);
    var host     = uri.Host;
    var port     = uri.IsDefaultPort ? 5432 : uri.Port;
    var database = uri.AbsolutePath.TrimStart('/');
    var username = Uri.UnescapeDataString(userInfo[0]);
    var password = userInfo.Length > 1 ? Uri.UnescapeDataString(userInfo[1]) : "";
    return $"Host={host};Port={port};Database={database};Username={username};Password={password};SSL Mode=Require;Trust Server Certificate=true";
}

// Replace your existing IDbConnection registration with:
builder.Services.AddScoped<IDbConnection>(_ => new NpgsqlConnection(ResolveConnectionString()));
```

Then commit, push, and redeploy.

---



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

### "relation 'documents' does not exist" (or any table name)

**First, check whether auto-migration is wired up.** Apps scaffolded with the current toolkit apply `schema.sql` at startup automatically. If the error still appears, one of these is the cause:

1. **`schema.sql` is not in the published output** — check that the `.csproj` contains:
   ```xml
   <Content Include="schema.sql">
     <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
   </Content>
   ```
   If this line is missing, `schema.sql` exists in the repo but is not present in the deployed container. `dotnet publish` only copies files explicitly marked as content.

2. **Wrong path used to locate `schema.sql`** — the startup code must use `AppContext.BaseDirectory`, not `Directory.GetCurrentDirectory()`. In a deployed container, `GetCurrentDirectory()` returns `/` or the container working directory, not the folder where the published binary lives. Only `AppContext.BaseDirectory` reliably points to the published output folder where `schema.sql` was copied.

3. **`schema.sql` uses `CREATE TABLE` without `IF NOT EXISTS`** — the startup migration runs but fails silently on the second deploy because the table already exists. Change every `CREATE TABLE` and `CREATE INDEX` to use `IF NOT EXISTS`.

4. **Startup migration code is missing from `Program.cs`** — search for `schema.sql` in `Program.cs`. If it's not there, the app never applies it. Add:
   ```csharp
   var schemaPath = Path.Combine(AppContext.BaseDirectory, "schema.sql");
   if (File.Exists(schemaPath))
       await db.ExecuteAsync(await File.ReadAllTextAsync(schemaPath));
   ```

**Quick manual fix for any of the above:** Go to your Railway PostgreSQL service → **Data → Query**, paste the contents of `schema.sql`, and run it. Then fix the root cause so it doesn't require manual intervention next time.

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

## Known Build Gotchas

These are issues that have caused real build failures. Check here before spending time debugging.

---

### `SignOutAsync` build error — missing using directive

**Symptom:** `CS1061: 'HttpContext' does not contain a definition for 'SignOutAsync'`

**Cause:** `HttpContext.SignOutAsync()` is an extension method from `Microsoft.AspNetCore.Authentication`. The `using` for cookie auth alone isn't enough.

**Fix:** Add to the top of the file (PageModel or endpoint):
```csharp
using Microsoft.AspNetCore.Authentication;
```

---

### `@@click`, `@@change` — Alpine.js in Razor files

**Symptom:** Alpine.js event bindings silently do nothing. No error in the browser, no JS console output.

**Cause:** Razor treats `@` as a C# expression start. A single `@click` gets parsed as a Razor directive and stripped from the output HTML.

**Fix:** Always double the `@` in `.cshtml` files:
```html
<!-- Wrong (silently stripped by Razor) -->
<button @click="doSomething()">Click me</button>

<!-- Correct -->
<button @@click="doSomething()">Click me</button>
<select @@change="filter = $event.target.value">...</select>
```

This applies to all Alpine.js event directives: `@@click`, `@@change`, `@@input`, `@@submit`, `@@keydown`, etc.

---

### Stripe.net pulls in a vulnerable `Newtonsoft.Json`

**Symptom:** After adding Stripe.net, `dotnet build` or a security scanner warns about a vulnerable version of `Newtonsoft.Json`.

**Cause:** Stripe.net's transitive dependency pins an older, vulnerable `Newtonsoft.Json` version.

**Fix:** Always follow `dotnet add package Stripe.net` with an explicit override:
```
dotnet add package Newtonsoft.Json --version 13.0.3
```

This forces the resolver to use the safe version. Add both lines to your setup notes.

---

### `create_file` won't overwrite scaffold files

**Symptom:** After `dotnet new web`, trying to replace `Program.cs` or other scaffold files with `create_file` fails silently or errors — the original content remains.

**Cause:** `dotnet new web` creates `Program.cs`, `appsettings.json`, and other files at scaffold time. `create_file` refuses to overwrite existing files.

**Fix:** Use `replace_string_in_file` (not `create_file`) for any file that may already exist from the scaffold. When in doubt, check with `read_file` first to confirm whether a file exists before deciding which tool to use.

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
