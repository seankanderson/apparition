# File Storage

How to handle user file uploads — profile photos, attachments, documents, images.

---

## The Problem: Railway and Render Don't Keep Files

Railway and Render run your app in a container. That container is rebuilt and replaced every time you deploy new code, and can be restarted any time the platform needs to. When that happens, **any files written to the container's disk are deleted** — including anything in `wwwroot/uploads/` or any other folder you write to.

This means you cannot store uploaded files on the server itself. If you do, they will randomly disappear.

> **"But doesn't Railway have persistent Volumes?"**
> Yes — Railway and Render both offer a persistent disk you can attach to your service. But it is not blob storage:
> - Files are served through your app server, not a CDN — slow for users far away
> - The disk is attached to one instance only; if Railway ever runs two copies of your app to handle traffic, only one of them can see the files
> - There is no direct URL delivery, no image resizing, no access control
>
> Volumes are useful for things like SQLite databases or log files. They are not suitable for user-uploaded content.

---

## Choose a Provider

| Provider | Free tier | Best for | Complexity |
|----------|-----------|----------|------------|
| **Cloudflare R2** | 10 GB storage, no egress fees | General files, documents, images | Low |
| **Cloudinary** | 25 credits/month | Images — auto-resize, optimize, transform | Low |
| **AWS S3** | 5 GB for 12 months (then paid) | If you already have AWS | Medium |
| **Azure Blob Storage** | 5 GB for 12 months (then paid) | If you already have Azure | Medium |

**Not sure?** Pick Cloudflare R2 for documents and mixed files, or Cloudinary if your app is primarily image-focused (profile photos, product images, galleries).

---

## How It Works (Same for All Providers)

```
User selects file
       ↓
Browser submits form to your Razor Page (or fetch() to /api/uploads)
       ↓
Your server reads credentials from environment variables
       ↓
Your server uploads the file to the provider (API key never leaves the server)
       ↓
Provider returns a public URL
       ↓
You save that URL as a string field in your JSONB document
```

The file lives on the provider's CDN. Your database stores only a URL string.

---

## Option A — Cloudflare R2

**Best for:** general file storage. Genuinely free egress (you don't pay for downloads). Works with the standard S3 SDK so the code is portable.

### Setup

1. Sign up at https://cloudflare.com (free)
2. Go to **R2 Object Storage → Create bucket** — name it something like `myapp-uploads`
3. Go to **R2 → Manage R2 API Tokens → Create API Token**
   - Permissions: Object Read & Write
   - Copy the **Access Key ID** and **Secret Access Key**
4. On the bucket overview page, copy the **S3 API endpoint** — it looks like:
   `https://ACCOUNT_ID.r2.cloudflarestorage.com`
5. Set your bucket to allow public reads (R2 → bucket → Settings → Public access)

### NuGet package

```
dotnet add package AWSSDK.S3
```

### Environment variables

```
R2_ACCOUNT_ID=your-account-id
R2_ACCESS_KEY_ID=your-access-key
R2_SECRET_ACCESS_KEY=your-secret-key
R2_BUCKET_NAME=myapp-uploads
R2_PUBLIC_URL=https://pub-XXXX.r2.dev
```

### Register in `Program.cs`

```csharp
using Amazon.S3;

var s3Config = new AmazonS3Config
{
    ServiceURL = $"https://{builder.Configuration["R2_ACCOUNT_ID"]}.r2.cloudflarestorage.com",
    ForcePathStyle = true
};
var s3Client = new AmazonS3Client(
    builder.Configuration["R2_ACCESS_KEY_ID"],
    builder.Configuration["R2_SECRET_ACCESS_KEY"],
    s3Config);
builder.Services.AddSingleton<IAmazonS3>(s3Client);
```

### Upload endpoint — `/Api/Uploads.cs`

```csharp
using Amazon.S3;
using Amazon.S3.Model;

namespace YourApp.Api;

public static class UploadsApi
{
    public static void Map(WebApplication app)
    {
        app.MapPost("/api/uploads", async (IFormFile file, IAmazonS3 s3, IConfiguration config) =>
        {
            if (file == null || file.Length == 0) return Results.BadRequest("No file provided.");
            if (file.Length > 10 * 1024 * 1024) return Results.BadRequest("File too large. Maximum 10MB.");

            var key = $"uploads/{Guid.NewGuid()}{Path.GetExtension(file.FileName)}";
            using var stream = file.OpenReadStream();

            await s3.PutObjectAsync(new PutObjectRequest
            {
                BucketName = config["R2_BUCKET_NAME"],
                Key = key,
                InputStream = stream,
                ContentType = file.ContentType,
                CannedACL = S3CannedACL.PublicRead
            });

            return Results.Ok(new { url = $"{config["R2_PUBLIC_URL"]}/{key}" });
        })
        .RequireAuthorization()
        .DisableAntiforgery();
    }
}
```

---

## Option B — Cloudinary

**Best for:** images. Cloudinary automatically optimizes, resizes on-the-fly, and delivers via CDN. Choose this for profile photos, product images, or galleries.

### Setup

1. Sign up at https://cloudinary.com (free — no credit card)
2. From the **Dashboard**, copy your **Cloud Name**, **API Key**, and **API Secret**

### NuGet package

```
dotnet add package CloudinaryDotNet
```

### Environment variables

```
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-api-key
CLOUDINARY_API_SECRET=your-api-secret
```

### Register in `Program.cs`

```csharp
using CloudinaryDotNet;

var cloudinary = new Cloudinary(new Account(
    builder.Configuration["CLOUDINARY_CLOUD_NAME"],
    builder.Configuration["CLOUDINARY_API_KEY"],
    builder.Configuration["CLOUDINARY_API_SECRET"]
));
cloudinary.Api.Secure = true;
builder.Services.AddSingleton(cloudinary);
```

### Upload endpoint — `/Api/Uploads.cs`

```csharp
using CloudinaryDotNet;
using CloudinaryDotNet.Actions;

namespace YourApp.Api;

public static class UploadsApi
{
    public static void Map(WebApplication app)
    {
        app.MapPost("/api/uploads", async (IFormFile file, Cloudinary cloudinary) =>
        {
            if (file == null || file.Length == 0) return Results.BadRequest("No file provided.");
            if (file.Length > 10 * 1024 * 1024) return Results.BadRequest("File too large. Maximum 10MB.");

            await using var stream = file.OpenReadStream();
            var uploadParams = new ImageUploadParams
            {
                File = new FileDescription(file.FileName, stream),
                Folder = "uploads",
                UniqueFilename = true,
                Overwrite = false
            };

            var result = await cloudinary.UploadAsync(uploadParams);
            if (result.Error != null) return Results.Problem(result.Error.Message);

            return Results.Ok(new { url = result.SecureUrl.ToString() });
        })
        .RequireAuthorization()
        .DisableAntiforgery();
    }
}
```

For non-image files (PDFs, documents), replace `ImageUploadParams` with `RawUploadParams`.

---

## Option C — AWS S3

**Best for:** teams already using AWS. Requires creating an IAM user — do not use root credentials.

### Setup

1. Sign in to https://aws.amazon.com
2. **S3 → Create bucket** — choose a region close to your users; uncheck "Block all public access" if files should be publicly readable
3. **IAM → Users → Create user** — attach policy `AmazonS3FullAccess` (or a least-privilege policy scoped to your bucket) — create an **Access Key** and copy the ID and secret

### NuGet package

```
dotnet add package AWSSDK.S3
```

### Environment variables

```
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_REGION=us-east-1
AWS_BUCKET_NAME=myapp-uploads
```

### Register in `Program.cs`

```csharp
using Amazon.S3;
using Amazon;

var s3Client = new AmazonS3Client(
    builder.Configuration["AWS_ACCESS_KEY_ID"],
    builder.Configuration["AWS_SECRET_ACCESS_KEY"],
    RegionEndpoint.GetBySystemName(builder.Configuration["AWS_REGION"]));
builder.Services.AddSingleton<IAmazonS3>(s3Client);
```

### Upload endpoint — `/Api/Uploads.cs`

```csharp
using Amazon.S3;
using Amazon.S3.Model;

namespace YourApp.Api;

public static class UploadsApi
{
    public static void Map(WebApplication app)
    {
        app.MapPost("/api/uploads", async (IFormFile file, IAmazonS3 s3, IConfiguration config) =>
        {
            if (file == null || file.Length == 0) return Results.BadRequest("No file provided.");
            if (file.Length > 10 * 1024 * 1024) return Results.BadRequest("File too large. Maximum 10MB.");

            var key = $"uploads/{Guid.NewGuid()}{Path.GetExtension(file.FileName)}";
            using var stream = file.OpenReadStream();

            await s3.PutObjectAsync(new PutObjectRequest
            {
                BucketName = config["AWS_BUCKET_NAME"],
                Key = key,
                InputStream = stream,
                ContentType = file.ContentType,
                CannedACL = S3CannedACL.PublicRead
            });

            var url = $"https://{config["AWS_BUCKET_NAME"]}.s3.{config["AWS_REGION"]}.amazonaws.com/{key}";
            return Results.Ok(new { url });
        })
        .RequireAuthorization()
        .DisableAntiforgery();
    }
}
```

---

## Option D — Azure Blob Storage

**Best for:** teams already using Azure.

### Setup

1. Sign in to https://portal.azure.com
2. Create a **Storage Account** (LRS redundancy is fine to start)
3. Inside the storage account: **Containers → + Container** — name it `uploads`, set public access to **Blob**
4. Go to **Access keys** and copy the **Connection string**

### NuGet package

```
dotnet add package Azure.Storage.Blobs
```

### Environment variables

```
AZURE_STORAGE_CONNECTION_STRING=DefaultEndpointsProtocol=https;AccountName=...
AZURE_STORAGE_CONTAINER=uploads
```

### Register in `Program.cs`

```csharp
using Azure.Storage.Blobs;

builder.Services.AddSingleton(new BlobServiceClient(
    builder.Configuration["AZURE_STORAGE_CONNECTION_STRING"]));
```

### Upload endpoint — `/Api/Uploads.cs`

```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;

namespace YourApp.Api;

public static class UploadsApi
{
    public static void Map(WebApplication app)
    {
        app.MapPost("/api/uploads", async (IFormFile file, BlobServiceClient blobService, IConfiguration config) =>
        {
            if (file == null || file.Length == 0) return Results.BadRequest("No file provided.");
            if (file.Length > 10 * 1024 * 1024) return Results.BadRequest("File too large. Maximum 10MB.");

            var container = blobService.GetBlobContainerClient(config["AZURE_STORAGE_CONTAINER"]);
            var blobName = $"uploads/{Guid.NewGuid()}{Path.GetExtension(file.FileName)}";
            var blob = container.GetBlobClient(blobName);

            using var stream = file.OpenReadStream();
            await blob.UploadAsync(stream, new BlobHttpHeaders { ContentType = file.ContentType });

            return Results.Ok(new { url = blob.Uri.ToString() });
        })
        .RequireAuthorization()
        .DisableAntiforgery();
    }
}
```

---

## All Providers — Enable Large Uploads in `Program.cs`

Add this regardless of which provider you use:

```csharp
builder.Services.Configure<FormOptions>(options =>
{
    options.MultipartBodyLengthLimit = 10 * 1024 * 1024; // 10MB
});
```

And register the endpoint:
```csharp
UploadsApi.Map(app);
```

---

## Upload Form (Razor Page — Same for All Providers)

```html
<form method="post" enctype="multipart/form-data">
    <div class="mb-3">
        <label class="form-label">Attach file</label>
        <input type="file" name="file" class="form-control" />
    </div>
    <button type="submit" class="btn btn-primary">Upload</button>
</form>
```

In the `.cshtml.cs` page model — call your own `/api/uploads` endpoint and store the returned URL:

```csharp
public async Task<IActionResult> OnPostAsync(IFormFile file)
{
    string? fileUrl = null;

    if (file != null && file.Length > 0)
    {
        using var content = new MultipartFormDataContent();
        using var stream = file.OpenReadStream();
        content.Add(new StreamContent(stream), "file", file.FileName);

        var http = HttpContext.RequestServices
            .GetRequiredService<IHttpClientFactory>()
            .CreateClient();
        var response = await http.PostAsync("/api/uploads", content);
        var result = await response.Content.ReadFromJsonAsync<UploadResult>();
        fileUrl = result?.Url;
    }

    // Include fileUrl when saving your JSONB document
    // e.g.  new { name = customerName, photoUrl = fileUrl }
}

record UploadResult(string Url);
```

Register `IHttpClientFactory` in `Program.cs`:
```csharp
builder.Services.AddHttpClient();
```

---

## Storing and Displaying the URL

The URL is just a string field in your JSONB document — no different from any other text:

```csharp
var record = new {
    clientName = "Acme Corp",
    logoUrl = "https://pub-xxx.r2.dev/uploads/abc.jpg"
};
```

Display it:
```html
@if (!string.IsNullOrEmpty(Model.LogoUrl))
{
    <img src="@Model.LogoUrl" alt="Logo" style="max-width:200px;" />
}
```

Download link:
```html
<a href="@Model.DocumentUrl" target="_blank">Download document</a>
```

---

## Environment Variables — Where to Set Them

File storage credentials must be set in two places: your local `.env` file for development, and your hosting platform's variables UI for production. **The app will crash on startup if these are missing.**

| Provider | Variables needed |
|----------|-----------------|
| Cloudflare R2 | `R2_ACCOUNT_ID`, `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_BUCKET_NAME`, `R2_PUBLIC_URL` |
| Cloudinary | `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, `CLOUDINARY_API_SECRET` |
| AWS S3 | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`, `AWS_BUCKET_NAME` |
| Azure Blob | `AZURE_STORAGE_CONNECTION_STRING`, `AZURE_STORAGE_CONTAINER` |

### Locally (development)

Open your app's `.env` file and uncomment the lines for your chosen provider. Fill in the real values from your provider's dashboard.

`.env` is gitignored — it never gets committed.

> `.env` requires the `DotNetEnv` NuGet package and `DotNetEnv.Env.Load()` at the top of `Program.cs`. See [environment-variables.md](environment-variables.md) for setup.

### On Railway

1. Open your project → click the **app service** (not the PostgreSQL service)
2. Go to **Variables** tab → click **New Variable**
3. Add each variable name and value from the table above
4. Railway redeploys automatically after you save

For adding many at once: click **Raw Editor**, paste all `KEY=value` lines, click **Update**.

### On Render

1. Open your service → **Environment** (left sidebar)
2. Click **Add Environment Variable** for each variable
3. Click **Save Changes** — Render redeploys automatically

For adding many at once: click **Add from .env**, paste your variables, click **Add Variables**.

### On AWS App Runner / Azure App Service

See [environment-variables.md](environment-variables.md) for step-by-step instructions on all four hosting platforms.

---

> **After deploying:** visit the upload page in your live app and test with a real file. If the upload fails, check the platform logs first — the error message will name the missing variable.
