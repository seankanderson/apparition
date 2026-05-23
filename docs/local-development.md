# Local Development

How to run your Apparition app on your own computer before deploying it.

---

## What You Need

Two things must be installed:

| Tool       | Why          | Version       |
| ---------- | ------------ | ------------- |
| .NET 8 SDK | Runs the app | 8.x or higher |
| PostgreSQL | The database | 14 or higher  |

If you'd rather skip local setup entirely, jump to [Deploy first, test in production](#deploy-first) at the bottom of this page.

---

## Step 1 — Install .NET 8 SDK

**Check if it's already installed:**
```
dotnet --version
```
If you see `8.x.x` or higher, you're done. Skip to Step 2.

**Install it:**
- Windows: open a terminal and run `winget install Microsoft.DotNet.SDK.8`
- Mac: download from https://dot.net/download or run `brew install --cask dotnet-sdk`
- After installing, close your terminal and open a new one.

---

## Step 2 — Install PostgreSQL

**Check if it's already installed:**
```
psql --version
```
If you see a version number, skip to Step 3.

**Install it:**
- Windows: download the installer from https://www.postgresql.org/download/windows/
  - During install, set a password for the `postgres` user — write it down
  - Port 5432 is the default — leave it
- Mac: `brew install postgresql@16` then `brew services start postgresql@16`

---

## Step 3 — Create a Local Database

Open a terminal and run:

```bash
psql -U postgres -c "CREATE DATABASE myappname;"
```

Replace `myappname` with whatever your app folder is called (lowercase, no spaces).

Then run your schema to create the tables:

```bash
psql -U postgres -d myappname -f schema.sql
```

Run this from your app's root folder (where `schema.sql` lives).

---

## Step 4 — Set Up the Connection String

Open `appsettings.Development.json` in your app. It should look like:

```json
{
  "ConnectionStrings": {
    "Default": "Host=localhost;Database=myappname;Username=postgres;Password=YOUR_PASSWORD"
  }
}
```

Replace `myappname` with your database name and `YOUR_PASSWORD` with the password you set during PostgreSQL install.

> **This file is gitignored** — it will never be uploaded to GitHub. It's safe to put your local password here.

---

## Step 5 — Run the App

Open a terminal in your app folder and run:

```
dotnet run
```

Then open your browser to: **http://localhost:5000**

The first time it runs, `SeedAdmin.cs` will create your admin account using the values in your `.env` file. You should see a confirmation in the terminal:

```
✓ Admin account created for you@example.com. You can now log in.
```

---

## Common Problems

### "Cannot connect to database"

PostgreSQL isn't running. Start it:
- Windows: search for "Services" → find PostgreSQL → Start
- Mac: `brew services start postgresql@16`

### "relation 'documents' does not exist"

You haven't run `schema.sql` yet. Run:
```
psql -U postgres -d myappname -f schema.sql
```

### "Port 5000 is already in use"

Something else is already running on port 5000. Kill it:
```bash
# Windows
netstat -ano | findstr :5000
taskkill /PID <pid_number> /F

# Mac/Linux
lsof -ti:5000 | xargs kill
```

Or open `Properties/launchSettings.json` and change `5000` to any other port (e.g. `5001`).

### "Command 'dotnet' not found"

The .NET SDK isn't installed, or you need to reopen your terminal after installing it.

---

## Deploy First, Test in Production {#deploy-first}

If local setup feels like too much right now, you can skip it entirely. Railway gives you a free PostgreSQL database automatically — you can deploy and test your app live in a browser in about 10 minutes.

See `docs/deployment.md` for the step-by-step guide.

---

## Hot Reload During Development

When the app is running, you can use:

```
dotnet watch
```

instead of `dotnet run`. This watches for file changes and automatically recompiles and refreshes the browser when you save a `.cshtml` or `.cs` file. Useful for fast iteration.
