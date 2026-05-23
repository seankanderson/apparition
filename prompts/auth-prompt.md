# Auth Prompt — Add Login & Logout

Use this prompt to add cookie-based authentication to an existing Apparition app.

Paste into **GitHub Copilot Chat** (Agent mode) after your base app is working.

---

```
Add cookie-based authentication to my existing ASP.NET Core Razor Pages app.

## Requirements
- Use ASP.NET Core's built-in cookie authentication (Microsoft.AspNetCore.Authentication.Cookies)
- No third-party auth services (no Auth0, no Okta, no Azure AD)
- Store user accounts in the existing documents table with type = 'user'
- Passwords must be hashed with BCrypt (BCrypt.Net-Next NuGet package)
- Login page at /login
- Logout endpoint at /logout (POST)
- Redirect unauthenticated users to /login

## User data shape (store in JSONB)
{
  "email": "user@example.com",
  "passwordHash": "<bcrypt hash>",
  "displayName": "Jane Smith",
  "createdAt": "2024-01-01T00:00:00Z"
}

## Pages to create
- /Pages/Login/Index.cshtml — email + password form
- /Pages/Login/Index.cshtml.cs — handles POST, validates credentials, sets cookie

## Files to modify
- Program.cs — add AddAuthentication().AddCookie() and UseAuthentication() / UseAuthorization()
- Any pages that should require login — add [Authorize] attribute to the PageModel

## Data layer
- Add /Data/UserRepository.cs with:
  - FindByEmailAsync(string email) — returns user or null
  - CreateAsync(string email, string password, string displayName) — hashes password, inserts

## Constraints
- Do not use ASP.NET Core Identity (too heavy)
- Do not use JWT tokens
- Keep everything in the existing single project
- Follow the existing Dapper + JSONB pattern
- Email lookups must be case-insensitive: use `lower(data->>'email') = lower(@email)` in all SQL queries that find a user by email
```

---

## After Running This Prompt

1. Run your app locally and test login/logout
2. Add `[Authorize]` to any Razor Page that should require login
3. In `_Layout.cshtml`, show the logged-in user's name using `User.Identity.Name`
4. Deploy — no additional environment variables needed for cookie auth

---

## Creating the First Admin User

After deploying, you'll need to create the first user. Ask Copilot:

> "Add a one-time setup endpoint at /api/setup that creates an admin user if no users exist yet. Disable it after first use."
