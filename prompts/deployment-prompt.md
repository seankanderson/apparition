# Deployment Prompt — Get Help Deploying

Use these prompts when you're stuck getting your app live.

---

## Prompt: Generate a Railway-ready project

```
Review my ASP.NET Core project and make sure it is ready to deploy to Railway.

Check for:
1. A valid .csproj file in a consistent location
2. ASPNETCORE_URLS is not hardcoded (Railway sets it automatically)
3. The connection string is read from environment variable ConnectionStrings__Default
4. No hardcoded localhost URLs
5. appsettings.json does not contain any secrets
6. Static files are served with app.UseStaticFiles()

Output:
- A list of any issues found with exact file names and line numbers
- Fixed versions of any files that need changes
- The Railway environment variables I need to set
```

---

## Prompt: Check readiness for AWS App Runner

```
Review my ASP.NET Core project and make sure it is ready to deploy to AWS App Runner.

Check for:
1. A Dockerfile exists at the repo root
2. The Dockerfile uses mcr.microsoft.com/dotnet/aspnet:8.0 as the runtime image
3. ASPNETCORE_URLS is set to http://+:8080 in the Dockerfile (not hardcoded in code)
4. EXPOSE 8080 is in the Dockerfile
5. The ENTRYPOINT uses the correct .dll name matching the .csproj project name
6. The connection string is read from environment variable ConnectionStrings__Default
7. No secrets in committed files

Output any issues found with exact file and line, plus the fixed version.
```

---

## Prompt: Check readiness for Azure App Service

```
Review my ASP.NET Core project and make sure it is ready to deploy to Azure App Service.

Check for:
1. A valid .csproj file at the repo root (or note where it is)
2. The connection string is read from environment variable ConnectionStrings__Default
3. ASPNETCORE_URLS is NOT hardcoded — Azure App Service sets the port automatically
4. No hardcoded localhost URLs
5. appsettings.json does not contain any secrets
6. Static files are served with app.UseStaticFiles()

Output any issues found with exact file and line, plus the fixed version.
Also output the exact App Settings I need to add in the Azure Portal.
```

---

## Prompt: Generate a Dockerfile for Render or AWS App Runner

```
Generate a production Dockerfile for my ASP.NET Core (.NET 8) app for deployment to Render or AWS App Runner.

Requirements:
- Multi-stage build: SDK image for build, runtime image for final
- Final image: mcr.microsoft.com/dotnet/aspnet:8.0
- App listens on port 8080 (set ASPNETCORE_URLS=http://+:8080)
- EXPOSE 8080
- Entry point: dotnet App.dll (replace App with my project name: [YOUR PROJECT NAME])
- Keep the Dockerfile minimal — no unnecessary layers
```

---

## Prompt: Debug a failed deployment

```
My ASP.NET Core app failed to deploy. Here is the error from the deployment log:

[PASTE YOUR ERROR LOG HERE]

My project structure is:
[LIST YOUR FILES: e.g. App.csproj, Program.cs, /Pages/Index.cshtml, etc.]

My environment variables are set to:
- ConnectionStrings__Default = (set, not shown)
- ASPNETCORE_ENVIRONMENT = Production

Diagnose the error and provide the exact fix. Do not change the stack (Dapper, Razor Pages, no EF).
```

---

## Prompt: Set up a custom domain

```
I have deployed my ASP.NET Core app to Railway.
My custom domain is: [YOUR DOMAIN]
My Railway-assigned URL is: [YOUR APP].railway.app

What DNS records do I need to add at my domain registrar to point my domain to Railway?
Give me the exact record type, name, and value to enter.
```

---

## Prompt: Add HTTPS redirect

```
Add HTTPS redirect to my ASP.NET Core app for production.

In Program.cs, add:
- app.UseHttpsRedirection() only when ASPNETCORE_ENVIRONMENT == "Production"
- Do not add it in development (it breaks localhost)

Show me exactly where in Program.cs to add the conditional.
```
