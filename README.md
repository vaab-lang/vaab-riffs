# vaab-riffs

Official and community **riffs** for [Vaab](https://github.com/vaab-lang/vaab): plain-English packages that wrap APIs and utilities.

A riff is a directory with:

- `riff` — manifest (name, version, dependencies)
- `lib.vaab` — the library source

Use one from your app:

```vaab
need supabase from ../vaab-riffs/supabase

print(supabase.rest_get("https://xyz.supabase.co", "service-role-key", "/rest/v1/tasks"))
```

Then:

```bash
vaab gather
vaab run main.vaab
```

## Riffs

| Riff | Description |
|------|-------------|
| [supabase](./supabase/) | Supabase REST helpers via `http.get` / `http.post` |

Registry fetch (`need json from ada`) is not wired yet — use path dependencies for now.
