# Environment Variables

How to set environment variables — locally and on each hosting platform.

Every API key, database password, and secret your app uses is stored as an environment variable. They never go in source code files.

---

## How It Works

```
Local development          →  .env file in your app root
Railway / Render / AWS     →  the platform's Variables / Environment settings UI
```

Your app reads all of these the same way — via `IConfiguration` or `Environment.GetEnvironmentVariable()`. The source is transparent to the code.

---

## Setting Variables Locally

### Step 1 — Add the DotNetEnv package

ASP.NET Core does not read `.env` files automatically. Install this package once per project:

```
dotnet add package DotNetEnv
```

### Step 2 — Load the file in `Program.cs`

Add this as the first line of `Program.cs`, before anything else:

```csharp
DotNetEnv.Env.Load();
```

This reads your `.env` file and injects all values as environment variables. After this line, `Environment.GetEnvironmentVariable("MY_KEY")` and `builder.Configuration["MY_KEY"]` both work.

### Step 3 — Edit your `.env` file

Your `.env` file lives in your app root. It was created during scaffolding with placeholder values:

```
SEED_ADMIN_EMAIL=you@example.com
SEED_ADMIN_PASSWORD=CHANGE_ME
```

Open it and replace `CHANGE_ME` values with your real credentials. For file storage and other services, uncomment the lines for your chosen provider and fill them in.

**`.env` is gitignored** — it will never be uploaded to GitHub or committed to source control.

> If you're starting from scratch, copy `.env.example` (from the `.apparition/` folder) to `.env` and fill in only the variables you need.

---

## Setting Variables on Railway

Railway's Variables tab is where production environment variables live.

### Adding a variable

1. Open your project on [railway.app](https://railway.app)
2. Click on your **app service** (not the PostgreSQL service)
3. Go to the **Variables** tab
4. Click **New Variable**
5. Enter the variable name (e.g. `STRIPE_SECRET_KEY`) and its value
6. Click **Add** — Railway redeploys your app automatically

### Adding multiple variables at once

Railway supports raw text import:

1. Variables tab → click the **Raw Editor** button (top right)
2. Paste your variables in `KEY=value` format, one per line:
   ```
   STRIPE_SECRET_KEY=sk_live_...
   R2_ACCOUNT_ID=abc123
   R2_ACCESS_KEY_ID=...
   R2_SECRET_ACCESS_KEY=...
   R2_BUCKET_NAME=myapp-uploads
   R2_PUBLIC_URL=https://pub-xxx.r2.dev
   ```
3. Click **Update** — Railway deploys with the new values

### Checking existing variables

Variables tab shows all currently set variables. Values are hidden by default — click the eye icon to reveal.

---

## Setting Variables on Render

### Adding a variable

1. Open your service on [render.com](https://render.com)
2. Go to **Environment** in the left sidebar
3. Scroll to **Environment Variables**
4. Click **Add Environment Variable**
5. Enter the key and value — click **Save Changes**
6. Render redeploys automatically

### Adding multiple variables at once

1. Environment → click **Add from .env** (top right of the variables section)
2. Paste your variables in `KEY=value` format
3. Click **Add Variables**

---

## Setting Variables on AWS App Runner

1. Open your App Runner service in the AWS Console
2. Click **Edit** (top right)
3. Under **Configure service → Environment variables**, click **Add environment variable**
4. Enter the key and value for each variable
5. Click **Save and deploy**

---

## Setting Variables on Azure App Service

1. Open your App Service in the Azure Portal
2. Go to **Environment variables → App settings**
3. Click **+ Add** for each variable
4. Enter the name and value
5. Click **Apply** at the top, then **Confirm** — the app restarts

---

## Variables by Feature

### Always required

| Variable | Value | Where set |
|----------|-------|-----------|
| `DATABASE_URL` | PostgreSQL `postgresql://` URI | **Railway only:** injected automatically when a PostgreSQL service is linked to your app service — do not set this manually. |
| `ConnectionStrings__Default` | ADO.NET connection string (`Host=...;Port=...`) | **Render / AWS / Azure:** set manually. Not needed on Railway. |
| `ASPNETCORE_ENVIRONMENT` | `Production` | Set manually on all platforms |

> **Railway note:** Do not copy `DATABASE_URL` and paste it as `ConnectionStrings__Default`. The scaffolded `ResolveConnectionString()` helper in `Program.cs` reads `DATABASE_URL` directly and converts the URI format automatically. Setting both will not cause harm, but it is unnecessary and confusing.

### First-run admin seeding

| Variable | Value |
|----------|-------|
| `SEED_ADMIN_EMAIL` | Your email address |
| `SEED_ADMIN_PASSWORD` | Your chosen password |

After your first successful login, remove these from Railway/Render — they're no longer needed.

### Stripe

| Variable | Value |
|----------|-------|
| `STRIPE_SECRET_KEY` | Your Stripe secret key (starts with `sk_live_` or `sk_test_`) |
| `STRIPE_WEBHOOK_SECRET` | Your Stripe webhook signing secret |

### Cloudflare R2

| Variable | Value |
|----------|-------|
| `R2_ACCOUNT_ID` | Your Cloudflare account ID |
| `R2_ACCESS_KEY_ID` | R2 API token access key |
| `R2_SECRET_ACCESS_KEY` | R2 API token secret |
| `R2_BUCKET_NAME` | Name of your R2 bucket |
| `R2_PUBLIC_URL` | Your bucket's public URL (e.g. `https://pub-xxx.r2.dev`) |

### Cloudinary

| Variable | Value |
|----------|-------|
| `CLOUDINARY_CLOUD_NAME` | Your cloud name from the Cloudinary dashboard |
| `CLOUDINARY_API_KEY` | API key |
| `CLOUDINARY_API_SECRET` | API secret |

### AWS S3

| Variable | Value |
|----------|-------|
| `AWS_ACCESS_KEY_ID` | IAM user access key |
| `AWS_SECRET_ACCESS_KEY` | IAM user secret |
| `AWS_REGION` | e.g. `us-east-1` |
| `AWS_BUCKET_NAME` | Your S3 bucket name |

### Azure Blob Storage

| Variable | Value |
|----------|-------|
| `AZURE_STORAGE_CONNECTION_STRING` | Connection string from the storage account's Access Keys page |
| `AZURE_STORAGE_CONTAINER` | Container name (e.g. `uploads`) |

---

## Troubleshooting

### Variable set but app still can't read it

1. **Check the name exactly** — variable names are case-sensitive. `stripe_secret_key` ≠ `STRIPE_SECRET_KEY`.
2. **Redeploy after adding** — some platforms require a manual redeploy after adding variables. On Railway, variables trigger an automatic redeploy. On Azure, clicking Apply restarts the app.
3. **Check the right service** — on Railway, variables must be set on the **app service**, not the PostgreSQL service.

### Works locally but fails in production

The most common cause: the variable exists in `.env` but was never added to Railway/Render. Open the platform's Variables tab and confirm every key from your `.env` is also set there.

### `SEED_ADMIN_EMAIL` / `SEED_ADMIN_PASSWORD` — account not created

- Make sure both variables are set before the first deploy
- Check the app's startup logs — you should see: `✓ Admin account created for you@example.com`
- If you see nothing, the variables may be missing or blank
- Once the account exists, the seeding code skips silently on all future startups
