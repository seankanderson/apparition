# Deployment Guide

You'll have your app live in about 10 minutes following this guide. Railway is the recommended path. Render is the fallback.

---

## Before You Deploy

Make sure you have:
- [ ] A GitHub account
- [ ] Your project code pushed to a GitHub repository
- [ ] A Railway or Render account (free tier is fine to start)

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

1. Click on the PostgreSQL service
2. Go to the **Connect** tab
3. Copy the **DATABASE_URL** value

Then click on your app service, go to **Variables**, and add:

```
ConnectionStrings__Default = <paste the DATABASE_URL here>
ASPNETCORE_ENVIRONMENT    = Production
```

### Step 4 — Run the schema

1. In your Railway PostgreSQL service, click **Data → Query**
2. Paste the contents of your `schema.sql` file and run it

### Step 5 — Deploy

Railway deploys automatically on every push to your `main` branch. Your app is now live.

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

### Step 4 — Run the schema

Use the **Render Shell** tab on your PostgreSQL service to run `schema.sql`.

---

## Environment Variables Reference

| Variable                     | Value                      | Required      |
| ---------------------------- | -------------------------- | ------------- |
| `ConnectionStrings__Default` | Postgres connection string | Yes           |
| `ASPNETCORE_ENVIRONMENT`     | `Production`               | Yes           |
| `CookieAuth__SecretKey`      | Random 32-char string      | If using auth |

---

## Custom Domain

Both Railway and Render allow you to add a custom domain for free.

1. Buy a domain (Namecheap, Cloudflare Registrar, etc.)
2. In Railway/Render, go to your service → **Settings → Domains → Add Custom Domain**
3. Follow the DNS instructions — usually takes 5–30 minutes to propagate

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
