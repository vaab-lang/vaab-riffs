# vaab-riffs

Official **riffs** for [Vaab](https://github.com/vaab-lang/vaab): plain-English packages for vendor APIs and shared utilities.

Riffs that have grown into their own repos (for example [tape](https://github.com/vaab-lang/tape)) still install with `riff install <name>` — the toolchain checks `github.com/vaab-lang/<name>` first, then this catalog.

## Install a riff

In your app's `main.vaab`:

```vaab
choice HttpError {
    Failed(message: Text)
}

need supabase from "../vaab-riffs/supabase"
```

In your app's `riff` manifest:

```
need supabase from "../vaab-riffs/supabase"
```

Then:

```bash
vaab gather
vaab run main.vaab
```

## supabase

PostgREST client over Vaab's `http.send` builtin.

**Environment**

| Variable | Purpose |
|----------|---------|
| `SUPABASE_URL` | Project URL, e.g. `https://xyz.supabase.co` |
| `SUPABASE_ANON_KEY` | Anon or service-role key |

**Example**

```vaab
choice HttpError {
    Failed(message: Text)
}

need supabase from "../vaab-riffs/supabase"

to main() {
    let client = match supabase.connect_with(
        "https://your-project.supabase.co",
        "your-anon-key",
    ) {
        when success value then value
        when failure error then match error {
            when Missing(name) then {
                print("missing {name}")
                return
            }
            when Request(message) then {
                print("request failed: {message}")
                return
            }
        }
    }

    match supabase.fetch(client, "tasks") {
        when success rows then print(rows)
        when failure error then print("could not load tasks")
    }
}

main()
```

**API**

| Call | Purpose |
|------|---------|
| `supabase.connect()` | Read `SUPABASE_URL` + `SUPABASE_ANON_KEY` from env |
| `supabase.connect_with(url, key)` | Explicit credentials |
| `supabase.fetch(client, table)` | `GET /rest/v1/{table}` |
| `supabase.fetch_where(client, table, query)` | GET with query string, e.g. `id=eq.abc` |
| `supabase.insert_row(client, table, json)` | POST row JSON |
| `supabase.patch_where(client, table, query, json)` | PATCH rows |
| `supabase.remove_where(client, table, query)` | DELETE rows |

## Other official riffs

| Riff | Repo |
|------|------|
| [tape](https://github.com/vaab-lang/tape) | Static file helpers for Vaab HTTP servers |

Install with `riff install tape`, then `need tape` in your app.

Registry installs (`need supabase from vaab`) land when `riffs.vaab.dev` is live. Path deps work today.
