# Command Center UI

React + TypeScript + Vite frontend for the Command Center Tauri desktop app.

## Development

From repository root:

```bash
npm --prefix desktop/ui install
npm --prefix desktop/ui run dev
```

## Quality Gates

```bash
npm --prefix desktop/ui run lint
npm --prefix desktop/ui run build
```

Both commands should pass before merge/release.

## Runtime Integration

- Desktop mode uses Tauri `invoke` API to call Rust commands.
- Rust commands call Python API module: `python -m command_center.tauri_api`.
- Browser dev mode uses a Vite middleware adapter under `/api/*`.

## Security Notes

- Do not expose Vite dev server to untrusted networks.
- `/api` dev middleware executes local Python commands and should be treated as trusted-local tooling only.
- Avoid logging sensitive local paths or payloads in production builds.

## Useful Smoke Checks

```bash
# UI build
npm --prefix desktop/ui run build

# End-to-end backend path used by UI
PYTHONPATH=src python -m command_center.tauri_api dashboard --from 2025-01-01 --to 2025-12-31 --refresh 0 --granularity month
```
