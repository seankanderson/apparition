# Patterns

Canonical solutions to recurring problems. Use these verbatim — they have been verified to work.

---

## Server → Alpine.js Data Handoff

**Problem:** You have a list or object fetched server-side (e.g., from a Dapper query in the PageModel), and you need Alpine.js to have access to it for reactive rendering — without making a separate API call.

**Pattern:** Serialize the server value into the `x-data` attribute using `@Html.Raw(JsonSerializer.Serialize(...))`.

```csharp
// Pages/Invoices/Index.cshtml.cs
public class IndexModel : PageModel
{
    public List<Invoice> Invoices { get; set; } = new();

    public async Task OnGetAsync()
    {
        Invoices = await _repo.GetAllAsync();
    }
}
```

```html
<!-- Pages/Invoices/Index.cshtml -->
@using System.Text.Json

<div x-data='{ invoices: @Html.Raw(JsonSerializer.Serialize(Model.Invoices)), selected: null }'>
    <template x-for="inv in invoices" :key="inv.id">
        <div x-text="inv.clientName"></div>
    </template>
</div>
```

**Notes:**
- Use single quotes around the `x-data` value so the embedded JSON's double quotes don't conflict
- `JsonSerializer.Serialize` produces camelCase property names by default when the source is a C# class — match your Alpine references accordingly, or set `JsonSerializerOptions` if you need exact casing
- For large datasets (hundreds of rows), prefer an API call (`fetch('/api/invoices')`) instead to avoid bloating the initial HTML
- Never serialize sensitive fields (passwords, hashes, tokens) into the page

---

## Bootstrap Dark Mode

**Problem:** The app needs a dark background and light text. There are several ways to do this in Bootstrap 5 — only one is correct.

**Pattern:** Set `data-bs-theme="dark"` on the `<html>` element. That's it.

```html
<!-- Pages/Shared/_Layout.cshtml -->
<html lang="en" data-bs-theme="dark">
```

**What this does:**
- Switches all Bootstrap components (cards, modals, navbars, tables, form inputs) to their dark variants automatically
- No separate dark stylesheet needed
- No CSS class toggling needed

**Also add body background to `wwwroot/css/site.css`** for a true dark background (Bootstrap's dark theme uses a dark grey, not near-black):

```css
/* site.css — dark background override */
:root {
  --bs-primary: #6366f1;
  --bs-primary-rgb: 99, 102, 241;
  --bs-body-bg: #0f172a;
  --bs-body-color: #f1f5f9;
}
body { background-color: #0f172a; color: #f1f5f9; }
.border-top { border-color: #334155 !important; }
```

**What NOT to do:**
- Do not add a `dark` CSS class to `<body>` — that's Bootstrap 4 and earlier
- Do not import a separate dark stylesheet
- Do not override Bootstrap colors with `!important` throughout — use `--bs-*` custom properties instead

---

## JSONB Round-Trip with Dapper

**Problem:** The documents table stores data as a PostgreSQL `jsonb` column. Dapper doesn't know how to deserialize that column into a C# object automatically.

**Pattern:** Deserialize manually after the query using `JsonSerializer.Deserialize<T>`.

### Reading a single document

```csharp
// Data/InvoiceRepository.cs
public async Task<Invoice?> GetByIdAsync(Guid id)
{
    const string sql = """
        SELECT data FROM documents
        WHERE id = @id AND type = 'invoice'
        """;

    var raw = await _db.QuerySingleOrDefaultAsync<string>(sql, new { id });
    return raw is null ? null : JsonSerializer.Deserialize<Invoice>(raw);
}
```

### Reading a list

```csharp
public async Task<List<Invoice>> GetAllAsync()
{
    const string sql = """
        SELECT data FROM documents
        WHERE type = 'invoice'
        ORDER BY created_at DESC
        """;

    var rows = await _db.QueryAsync<string>(sql);
    return rows.Select(r => JsonSerializer.Deserialize<Invoice>(r)!).ToList();
}
```

### Writing (insert)

```csharp
public async Task CreateAsync(Invoice invoice)
{
    invoice.Id = Guid.NewGuid();
    invoice.CreatedAt = DateTime.UtcNow;

    const string sql = """
        INSERT INTO documents (id, type, data)
        VALUES (@id, 'invoice', @data::jsonb)
        """;

    await _db.ExecuteAsync(sql, new
    {
        id = invoice.Id,
        data = JsonSerializer.Serialize(invoice)
    });
}
```

### Writing (update)

```csharp
public async Task UpdateAsync(Invoice invoice)
{
    invoice.UpdatedAt = DateTime.UtcNow;

    const string sql = """
        UPDATE documents SET data = @data::jsonb, updated_at = NOW()
        WHERE id = @id AND type = 'invoice'
        """;

    await _db.ExecuteAsync(sql, new
    {
        id = invoice.Id,
        data = JsonSerializer.Serialize(invoice)
    });
}
```

**Notes:**
- Always cast the parameter with `::jsonb` on insert/update — Npgsql won't infer the type automatically
- Use `QueryAsync<string>` (not a typed class) when selecting `data` — let `JsonSerializer` do the deserialization step
- The `id`, `type`, `created_at`, `updated_at` columns live outside `data` on the documents row — do not embed them redundantly inside your serialized object (or if you do, keep them in sync)
- Add a GIN index on `data` if you filter by JSON fields: `CREATE INDEX idx_invoices_data ON documents USING gin(data) WHERE type = 'invoice';`
