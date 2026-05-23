# Deployment Guide

You'll have your app live in about 10 minutes following this guide. Railway is the recommended path — pick the option that matches where you already have an account.

| Option          | Best for                    | Database                      |
| --------------- | --------------------------- | ----------------------------- |
| **A — Railway** | Everyone — simplest overall | Built-in PostgreSQL           |
| **B — Render**  | Railway fallback            | Built-in PostgreSQL           |
| **C — AWS**     | Already using AWS           | Amazon RDS                    |
| **D — Azure**   | Already using Azure         | Azure Database for PostgreSQL |

---

## Before You Deploy

Make sure you have:
- [ ] A GitHub account
- [ ] Your project code pushed to a GitHub repository
- [ ] An account on your chosen platform (free tier is fine to start)

---

## Option A: Railway (Recommended)

Railway auto-detects .NET projects and handles everything including the database.

### Step 1 — Create a Railway project

1. Go to [railway.app](https://railway.app) and sign in with GitHub
2. Click **New Project → Deploy from GitHub repo**
3. Select your repository
4. Railway will detect it as a .NET project automatically

### Step 2 — Add a PostgreSQL database

1. In your Railway project dashboard, click **+ New**
2. Select **Database → PostgreSQL**
3. Wait ~30 seconds for it to provision

### Step 3 — Connect the database to your app

When you add a PostgreSQL service to your Railway project, Railway automatically injects a `DATABASE_URL` environment variable into your app service. **You do not need to copy or paste the connection string manually** — the app reads `DATABASE_URL` directly.

All you need to add manually is:

```
ASPNETCORE_ENVIRONMENT = Production
```

Click on your **app service** → **Variables** tab → **New Variable**, and add just that one variable.

> **Why not `ConnectionStrings__Default`?** Railway's `DATABASE_URL` is a `postgresql://` URI, not ADO.NET format. The scaffolded `Program.cs` includes a `ResolveConnectionString()` helper that converts it automatically. Do not paste the URI as `ConnectionStrings__Default` — it will cause a parse error on startup.

### Step 4 — Deploy

Railway deploys automatically on every push to your `main` branch. Your app is now live.

On the very first startup, the app runs `schema.sql` automatically and creates all tables. You should see no errors in the Railway logs.

> **If you see "relation does not exist" in the logs**, the startup schema migration did not run. This happens in apps scaffolded before auto-migration was added. Fix: go to your Railway PostgreSQL service → **Data → Query**, paste the contents of `schema.sql`, and run it manually. Then redeploy.

**Your URL:** shown on the Railway service card under **Domains**.

---

## Option B: Render (Fallback)

Use Render if Railway isn't working for your project.

### Step 1 — Create a Render account

Go to [render.com](https://render.com) and sign in with GitHub.

### Step 2 — Create a PostgreSQL database

1. Dashboard → **New → PostgreSQL**
2. Choose the free plan
3. Copy the **Internal Database URL** once it's created

### Step 3 — Create a Web Service

1. Dashboard → **New → Web Service**
2. Connect your GitHub repository
3. Set:
   - **Runtime:** Docker (add a `Dockerfile` — see below)
   - **Environment Variables:**
     ```
     ConnectionStrings__Default = <your Internal Database URL>
     ASPNETCORE_ENVIRONMENT    = Production
     ```

### Dockerfile for Render

Create a `Dockerfile` in the root of your project:

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/publish .
ENV ASPNETCORE_URLS=http://+:8080
EXPOSE 8080
ENTRYPOINT ["dotnet", "App.dll"]
```

### Step 4 — Deploy

Render deploys automatically. On first startup, the app runs `schema.sql` and creates all tables automatically.

> **If tables are missing**, use the **Render Shell** tab on your PostgreSQL service to run `schema.sql` manually, then trigger a new deploy.

---

## Option C: AWS (App Runner + RDS)

Use this if you already have an AWS account. App Runner builds your container directly from GitHub — no Docker knowledge needed.

### Step 1 — Add a Dockerfile

Create a `Dockerfile` in the root of your project (same file as shown in Option B above):

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/publish .
ENV ASPNETCORE_URLS=http://+:8080
EXPOSE 8080
ENTRYPOINT ["dotnet", "App.dll"]
```

Replace `App.dll` with your project name (e.g. `MyApp.dll`).

### Step 2 — Create a PostgreSQL database on Amazon RDS

1. AWS Console → **RDS → Create database**
2. Engine: **PostgreSQL**, Template: **Free tier**
3. Set a DB instance identifier (e.g. `myapp-db`), master username (`postgres`), and a strong password
4. DB name: `myapp`
5. Under **Connectivity** → check **Publicly accessible: Yes**
6. Click **Create database** — takes about 5 minutes

### Step 3 — Create an App Runner service

1. AWS Console → **App Runner → Create service**
2. Source type: **Source code repository → GitHub** → Connect your account → select your repo → branch: `main`
3. Deployment trigger: **Automatic**
4. Build: App Runner detects the Dockerfile automatically
5. Port: **8080**
6. Under **Environment variables**, add:
   ```
   ConnectionStrings__Default = Host=<RDS endpoint>;Port=5432;Database=myapp;Username=postgres;Password=<your password>
   ASPNETCORE_ENVIRONMENT    = Production
   ```
7. Click **Create and deploy** — first build takes ~5 minutes

### Step 4 — Run the schema

1. Download [TablePlus](https://tableplus.com) (free tier)
2. Create a new connection → PostgreSQL → enter your RDS endpoint, username, and password
3. Open your `schema.sql` file, paste it into the query window, and run it

**Your URL:** shown on the App Runner service page under **Default domain**.

---

## Option D: Azure (App Service + Azure Database for PostgreSQL)

Use this if you already have an Azure account. Azure App Service has native .NET 8 support — no Docker required.

### Step 1 — Create a PostgreSQL database

1. Azure Portal → **Create a resource** → search **Azure Database for PostgreSQL**
2. Choose **Flexible Server → Create**
3. Set server name, admin username (`pgadmin`), and a strong password
4. Under **Networking** → check **Allow public access from any Azure service within Azure to this server**
5. Click **Review + create** — takes about 5 minutes

### Step 2 — Create an App Service

1. Azure Portal → **Create a resource → Web App**
2. Settings:
   - **Runtime stack:** .NET 8 (LTS)
   - **Operating system:** Linux
   - **Region:** same region as your database
   - **Pricing plan:** Free F1 (to start)
3. Click **Review + create**

### Step 3 — Connect GitHub for automatic deploys

1. In your App Service → **Deployment Center**
2. Source: **GitHub** → Authorize → select your repo and `main` branch
3. Save — Azure creates a GitHub Actions workflow and the first deploy starts automatically

### Step 4 — Set environment variables

1. In your App Service → **Environment variables → App settings**, click **+ Add** for each:
   ```
   ConnectionStrings__Default = Host=<server>.postgres.database.azure.com;Port=5432;Database=postgres;Username=pgadmin;Password=<your password>;Ssl Mode=Require
   ASPNETCORE_ENVIRONMENT    = Production
   ```
2. Click **Apply** and **Confirm** — the app restarts automatically

### Step 5 — Run the schema

1. Azure Portal → your PostgreSQL resource → **Query editor** (left sidebar)
2. Log in with your credentials
3. Paste the contents of `schema.sql` and click **Run**

**Your URL:** `https://your-app-name.azurewebsites.net`

---

## Environment Variables Reference

| Variable                     | Value                      | Required      |
| ---------------------------- | -------------------------- | ------------- |
| `ConnectionStrings__Default` | Postgres connection string | Yes           |
| `ASPNETCORE_ENVIRONMENT`     | `Production`               | Yes           |
| `CookieAuth__SecretKey`      | Random 32-char string      | If using auth |

---

## Custom Domain

All four platforms support custom domains.

| Platform          | Where to find it                                 |
| ----------------- | ------------------------------------------------ |
| Railway           | Service → Settings → Domains → Add Custom Domain |
| Render            | Service → Settings → Custom Domains              |
| AWS App Runner    | Service → Custom domains → Link domain           |
| Azure App Service | App Service → Custom domains → Add custom domain |

1. Buy a domain (Namecheap, Cloudflare Registrar, etc.)
2. Add it in your platform's domain settings and follow the DNS instructions
3. DNS propagation usually takes 5–30 minutes

---

## Continuous Deployment

Once set up, deployment is automatic:

```
git add .
git commit -m "update feature"
git push origin main
```

Your site updates in ~2 minutes.

---

## Troubleshooting Deployments

See [troubleshooting.md](troubleshooting.md) for common deploy errors and fixes.
