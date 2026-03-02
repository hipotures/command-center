## Deployment & Configuration

### Installation Methods

#### Method 1: Global Installation (Recommended)

```bash
# From the project root directory
uv tool install -e .
```

**Advantages:**
- Available system-wide
- No environment activation needed
- Command: `command-center`

**Location:** Installed in `~/.local/share/uv/tools/`

#### Method 2: Virtual Environment

```bash
# From the project root directory
uv sync
source .venv/bin/activate
command-center --verbose
```

**Advantages:**
- Isolated dependencies
- Development-friendly

#### Method 3: Direct Execution

```bash
# From the project root directory
uv run command-center --verbose
```

### Configuration Model

#### Core runtime configuration

Core CLI behavior is configured in `src/command_center/config.py` (paths, batch size, canvas, fonts).

#### Environment variables (optional subsystems)

The **core** `command-center` CLI does not require env vars for normal usage. Optional scripts and integrations do.

| Variable | Used By | Purpose |
|----------|---------|---------|
| `CC_USAGE_DB_PATH` | usage scripts | Override usage accounts DB path (`cc_usage.db`) |
| `CC_USAGE_LOG_DB` | usage scripts | Enable/disable usage snapshot DB writes (`0` disables) |
| `CC_USAGE_LOGGER` | usage scripts | Override path to `cc_usage_logger.py` |
| `CDP_ENDPOINT` | `cc_usage_web.py` / Playwright flow | Browser CDP endpoint |
| `TARGET_URL` | web usage scripts | Usage page URL |
| `EMAIL_URL` | web usage scripts | Settings/email page URL |
| `EMAIL_LABEL` | web usage scripts | Email field label selector |
| `TIMEOUT_MS` | web usage scripts | Timeout override |
| `OUTDIR` | web usage scripts | Output snapshot/log directory |
| `CHROME_PROFILE` | `cc_usage_web.py` | Selenium profile directory |
| `TELEGRAM_BOT_TOKEN` | `cc_usage_web.py` | Telegram bot token for alerts |
| `TELEGRAM_CHAT_ID` | `cc_usage_web.py` | Telegram destination chat |

### CLI Usage

```bash
command-center [OPTIONS]
```

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--from DATE` | String | Jan 1 of current year | Start date (YYYY-MM-DD or YYYYMMDD) |
| `--to DATE` | String | Today | End date (YYYY-MM-DD or YYYYMMDD) |
| `--verbose` | Boolean | False | Show detailed progress |
| `--force-rescan` | Boolean | False | Reprocess all files |
| `--rebuild-db` | Boolean | False | Delete and rebuild database |
| `--db-stats` | Boolean | False | Show database statistics and exit |
| `--update-pricing` | Boolean | False | Update pricing cache from LiteLLM and exit |
| `--list-projects` | Boolean | False | List discovered projects with metadata |
| `--update-project` | String[3] | - | Update project metadata |

### Testing & Operations Procedures

Run these checks before merge/release:

```bash
# Python tests (current repo layout expects src on path)
PYTHONPATH=src pytest

# CLI smoke test
PYTHONPATH=src python -m command_center --db-stats

# Tauri API smoke test
PYTHONPATH=src python -m command_center.tauri_api dashboard --from 2025-01-01 --to 2025-12-31 --refresh 0 --granularity month

# Frontend quality gates
npm --prefix desktop/ui run lint
npm --prefix desktop/ui run build

# Rust/Tauri backend compile check
cargo check --manifest-path desktop/src-tauri/Cargo.toml
```

### Operational Runbook

1. Daily/regular use:
```bash
command-center --verbose
```

2. Force full rescan after source log structure changes:
```bash
command-center --force-rescan --verbose
```

3. Full rebuild after schema/backfill-impacting changes:
```bash
command-center --rebuild-db --verbose
```

4. Manual pricing refresh:
```bash
command-center --update-pricing
```

### Release Checklist (Manual)

No repository CI workflow is configured currently, so these checks are expected to run locally before merge/release.

1. Run all checks from "Testing & Operations Procedures".
2. Validate representative date ranges (small + large) and project-filtered dashboard output.
3. Confirm docs are updated when data/network behavior changes.
4. Verify no secrets/PII are introduced in logs or generated artifacts.

### Dependencies

Runtime dependencies are declared in `pyproject.toml`; exact lock versions in `uv.lock`.

---
