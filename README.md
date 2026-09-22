# vaab-riffs

Official **riffs** for [Vaab](https://github.com/vaab-lang/vaab): plain-English packages for vendor APIs and shared utilities.

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

## tape

Static file helpers for Vaab HTTP servers. Path resolution with traversal guards and MIME hints for `reply file` routes.

**Example**

```vaab
need tape

serve on port 8787 {
    route get "/{*filepath}" {
        match tape.resolve("web/dist", filepath) {
            when success target then reply file target
            when failure error then match error {
                when Unsafe(requested) then {
                    reply text "forbidden" as "text/plain; charset=utf-8" status 403
                }
                when NotFound(requested) then {
                    reply text "not found" as "text/plain; charset=utf-8" status 404
                }
            }
        }
    }
}
```

**API**

| Call | Purpose |
|------|---------|
| `tape.for_path(root, requested)` | Build a path under `root`; returns `{root}/__forbidden__` when unsafe |
| `tape.resolve(root, requested)` | Same, but returns `success` or `failure StaticError` |
| `tape.mime_for(path)` | Guess a content type from the file extension |

Install with `riff install tape`, then `need tape` in your app.

Registry installs (`need supabase from vaab`) land when `riffs.vaab.dev` is live. Path deps work today.
