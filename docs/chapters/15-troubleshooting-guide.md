## Troubleshooting Guide

### Common Issues

#### 1. No Activity Found for Date Range

**Symptom:**
```
No activity found for date range ...
```

**Diagnosis:**
```bash
find ~/.claude/projects -name "*.jsonl" -ls
command-center --db-stats
```

**Resolution:**
```bash
command-center --force-rescan --verbose
```

#### 2. Database Integrity Check Failed

**Symptom:**
```
Database integrity check failed!
```

**Resolution:**
```bash
cp ~/.claude/db/command_center.db ~/command-center-backup.db
command-center --rebuild-db --verbose
```

#### 3. PNG Not Displaying in Terminal

**Diagnosis:**
```bash
echo "TERM=$TERM"
echo "TERM_PROGRAM=$TERM_PROGRAM"
```

**Resolution:**
- Use a supported terminal (Kitty, WezTerm, iTerm2, Ghostty, Konsole, VS Code).
- Open saved file directly if inline protocols are unavailable.

#### 4. Pricing Update Fails

**Symptom:**
- `--update-pricing` fails with timeout/network error.

**Diagnosis:**
- Check outbound connectivity to LiteLLM source.
- Confirm local permissions for `~/.claude/db/pricing_cache.json`.

**Resolution:**
- Retry later; core analytics continues with cached/stale pricing or `None` cost for unknown models.

#### 5. Python Tests Fail at Collection (`ModuleNotFoundError: command_center`)

**Symptom:**
```
ModuleNotFoundError: No module named 'command_center'
```

**Cause:**
- Current test invocation requires `src` on `PYTHONPATH`.

**Resolution:**
```bash
PYTHONPATH=src pytest
```

#### 6. Frontend Build or Lint Fails

**Diagnosis:**
```bash
npm --prefix desktop/ui run lint
npm --prefix desktop/ui run build
```

**Resolution:**
- Fix reported TypeScript/ESLint issues before release.
- Re-run both commands until clean.

#### 7. Tauri Backend Check Fails

**Diagnosis:**
```bash
cargo check --manifest-path desktop/src-tauri/Cargo.toml
```

**Resolution:**
- Resolve Rust compile errors/warnings in `desktop/src-tauri`.

### Operational Debugging Commands

```bash
# CLI verbose run
command-center --verbose

# DB stats
command-center --db-stats

# Tauri API smoke
PYTHONPATH=src python -m command_center.tauri_api dashboard --from 2025-01-01 --to 2025-12-31 --refresh 0 --granularity month

# Manual DB inspection
sqlite3 ~/.claude/db/command_center.db
```

### SQL Checks

```sql
SELECT * FROM schema_version;
SELECT year, COUNT(*) FROM message_entries GROUP BY year;
SELECT COUNT(*) FROM file_tracks;
SELECT COUNT(*) FROM limit_events;
```

### Pre-Release Verification

Run full local quality gate:

```bash
PYTHONPATH=src pytest
npm --prefix desktop/ui run lint
npm --prefix desktop/ui run build
cargo check --manifest-path desktop/src-tauri/Cargo.toml
```

---
