# Hermes MSYS HERMES_HOME Normalize Patch

A patch kit for [Hermes Agent](https://hermes-agent.nousresearch.com) that fixes `HERMES_HOME` path mangling when Hermes runs under **git-bash / MSYS** on Windows.

## The bug

Under git-bash, MSYS converts Win32 env vars to POSIX form when spawning native Python — e.g. `HERMES_HOME` arrives as `/c/Users/<user>/AppData/Local/hermes` instead of `C:/Users/<user>/...`. A bare `Path(value)` then mangles the leading `/c` into a rooted relative path (`\c\Users\...`), silently pointing at a non-existent shadow tree. Consequences:

- board enumeration breaks (looks in the wrong tree)
- junk `C:\c\...` directories can get created
- sessions/profiles resolve to the wrong location

## The fix

Adds `_normalize_hermes_home_env()` to `hermes_constants.py`, converting MSYS-style `/x/...` → `X:/...` on win32 only (POSIX systems pass through untouched). It's applied at the two places `HERMES_HOME` is read into a `Path`:

- `_hermes_home_from_env()`
- `get_default_hermes_root()`

## Files

| File | Notes |
|------|-------|
| `2026-08-04-msys-hermes-home-normalize.patch` | Full patch (large — includes full `hermes_constants.py` context) |
| `2026-08-04-msys-hermes-home-normalize-minimal.patch` | **Preferred** — minimal, only the `_normalize_hermes_home_env` addition + 2 call-site changes |

## Apply

From the Hermes Agent source root (`hermes-agent/`):

```bash
git apply /path/to/2026-08-04-msys-hermes-home-normalize-minimal.patch
# or
patch -p1 < /path/to/2026-08-04-msys-hermes-home-normalize-minimal.patch
```

Then restart Hermes. Verify: under git-bash, `hermes` now resolves `HERMES_HOME` to the correct `C:\Users\...\AppData\Local\hermes` tree.

> Apply the **minimal** patch. The full patch is retained for reference/audit but carries far more diff context.

## Related

- Windows-specific tooling is collected under the `windows` / `hermes` skill families in Hermes Agent.

## License

MIT
