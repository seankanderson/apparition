# Database Patterns

PostgreSQL with JSONB is the heart of Apparition's data layer. This guide shows you the patterns AI will use — and that you can reuse for any feature.

---

## The One Table Pattern

Every Apparition app starts with a single table:

```sql
CREATE TABLE documents (
    id         UUID      PRIMARY KEY DEFAULT gen_random_uuid(),
    type       TEXT      NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT now(),
    updated_at TIMESTAMP NOT NULL DEFAULT now(),
    data       JSONB     NOT NULL
);

CREATE INDEX idx_documents_type        ON documents(type);
CREATE INDEX idx_documents_data_gin    ON documents USING gin(data);
```

You store everything in `data`. The `type` column tells you what kind of thing it is.

---

## JSON Key Casing — Always Use CamelCase

**This is the single most common source of silent bugs in Apparition apps.**

PostgreSQL JSONB key lookups are case-sensitive. `data->>'email'` and `data->>'Email'` are different keys.
`System.Text.Json` serializes named C# records and classes with **PascalCase** keys by default (`Email`, `PasswordHash`),
but camelCase (`email`, `passwordHash`) is the convention used in SQL queries and anonymous objects.

**Always declare a shared options instance in every repository and use it for all serialization and deserialization:**

```csharp
private static readonly JsonSerializerOptions _json = new()
{
    PropertyNamingPolicy        = JsonNamingPolicy.CamelCase,
    PropertyNameCaseInsensitive = true   // tolerant on read
};
```

Then use it consistently:
```csharp
// Writing
var json = JsonSerializer.Serialize(myRecord, _json);

// Reading
var obj = JsonSerializer.Deserialize<MyDto>(json, _json);
```

SQL queries must use the camelCase key names that result from this policy:
```sql
-- correct — matches camelCase JSON key
WHERE data->>'email' = @email

-- WRONG — PascalCase only matches if you serialized without CamelCase policy
WHERE data->>'Email' = @email
```

If you use anonymous objects (`new { email = ..., status = ... }`) the keys are already camelCase,
but **named records must use the options above** or the keys will be PascalCase and queries will return nothing.

---

## Storing Data

### Example: Save an order

```csharp
private static readonly JsonSerializerOptions _json = new()
{
    PropertyNamingPolicy        = JsonNamingPolicy.CamelCase,
    PropertyNameCaseInsensitive = true
};

var order = new OrderData("cust-123", "pending", 39.98);
var json = JsonSerializer.Serialize(order, _json);

await db.ExecuteAsync(@"
    INSERT INTO documents (type, data)
    VALUES ('order', @data::jsonb)",
    new { data = json });
```

---

## Reading Data

### Fetch one document by id

```csharp
var json = await db.QuerySingleAsync<string>(@"
    SELECT data FROM documents
    WHERE id = @id AND type = 'order'",
    new { id });

var order = JsonSerializer.Deserialize<OrderDto>(json);
```

### List all documents of a type

```csharp
var rows = await db.QueryAsync<string>(@"
    SELECT data FROM documents
    WHERE type = 'order'
    ORDER BY created_at DESC",
    new { });

var orders = rows.Select(r => JsonSerializer.Deserialize<OrderDto>(r));
```

### Query inside JSONB

```csharp
// Find all pending orders for a customer
var rows = await db.QueryAsync<string>(@"
    SELECT data FROM documents
    WHERE type = 'order'
      AND data->>'status' = 'pending'
      AND data->>'customerId' = @customerId",
    new { customerId = "cust-123" });
```

---

## Updating Data

```csharp
// Merge a partial update into existing JSONB
await db.ExecuteAsync(@"
    UPDATE documents
    SET data = data || @patch::jsonb,
        updated_at = now()
    WHERE id = @id",
    new {
        id,
        patch = JsonSerializer.Serialize(new { status = "shipped" })
    });
```

---

## The DbExtensions Helper

Always include this helper — it reduces repetition and keeps AI output clean:

```csharp
public static class DbExtensions
{
    public static async Task<T?> QueryJsonAsync<T>(
        this IDbConnection db, string sql, object param)
    {
        var json = await db.QuerySingleOrDefaultAsync<string>(sql, param);
        return json is null ? default : JsonSerializer.Deserialize<T>(json);
    }

    public static async Task<IEnumerable<T>> QueryJsonListAsync<T>(
        this IDbConnection db, string sql, object param)
    {
        var rows = await db.QueryAsync<string>(sql, param);
        return rows.Select(r => JsonSerializer.Deserialize<T>(r)!);
    }
}
```

---

## When to Add More Tables

Start with one table. Add a second table only when you genuinely need:

- A foreign key relationship you query by JOIN
- A column you need a unique constraint on (e.g. email)
- Full-text search on a specific field at scale

**Signal to add a table:** you find yourself writing complex JSONB path queries that are hard to index.

---

## Connection String

**Local development** (`appsettings.Development.json`):
```json
{
  "ConnectionStrings": {
    "Default": "Host=localhost;Database=myapp;Username=postgres;Password=postgres"
  }
}
```

**Production — Railway:**
Railway automatically injects `DATABASE_URL` as a `postgresql://` URI when a PostgreSQL service is linked. The `ResolveConnectionString()` helper in `Program.cs` converts it to ADO.NET format at startup. You do not need to set `ConnectionStrings__Default` on Railway.

**Production — Render / AWS / Azure:**
Set `ConnectionStrings__Default` to ADO.NET format:
```
ConnectionStrings__Default=Host=...;Port=5432;Database=myapp;Username=postgres;Password=...
```

---

## Migrations

Apparition does not use a migration framework. Instead:

1. Create a `schema.sql` file at the project root
2. Add each `CREATE TABLE` and `CREATE INDEX` statement there
3. Run it once on a new database: `psql $DATABASE_URL -f schema.sql`

AI will generate the correct `schema.sql` for your app when you use the master prompt.
