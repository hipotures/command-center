## Security & Data Privacy

### Threat Model

Command Center is **local-first**, but there are optional networked features.

**Primary mode (core CLI + local DB):**
- Single-user desktop workflow
- Local filesystem + local SQLite only
- No prompt/response content persistence in analytics DB

**Optional networked surfaces:**
- Pricing refresh via LiteLLM dataset download (`--update-pricing` and fallback refresh in pricing helper)
- Telegram notifications in `scripts/cc_usage_web.py` when bot env vars are configured

**In scope:**
- Local data exposure from weak filesystem permissions
- Injection/misconfiguration in local developer tooling
- Leaks through logs and optional integrations

**Out of scope:**
- Physical compromise of host OS
- Compromised third-party services beyond basic transport/security controls

### Data Inventory

#### Core analytics database (`~/.claude/db/command_center.db`)

**Stored:**
- Session IDs, request IDs, message IDs
- UTC + local timestamps
- Model identifiers
- Token and cost counters
- Source file path
- Derived project ID
- Limit-event metadata (summary/reset fields)

**Not stored in core analytics DB:**
- Prompt text content
- Model response text content
- API keys/credentials

#### Project metadata (`~/.claude/db/command-center-projects.json`)

**Stored:**
- `project_id`
- User-facing name/description
- Reconstructed absolute path
- Visibility flag
- First/last seen timestamps

#### Pricing cache (`~/.claude/db/pricing_cache.json`)

**Stored:**
- Remote model pricing dataset downloaded from LiteLLM source

#### Optional usage accounts database (`~/.claude/db/cc_usage.db`)

Used by `scripts/cc_usage_logger.py` / `scripts/cc_usage_web.py`.

**Stored:**
- Account email
- Usage percentages and raw usage strings
- Reset times (local/UTC/epoch)
- Raw payload snapshot (`raw_json`)

This data includes personally identifiable account metadata and should be treated accordingly.

### Network Behavior & Security Impact

#### Outbound requests

1. **Pricing update**
- Destination: LiteLLM GitHub raw JSON URL
- Trigger: `--update-pricing` or missing-model fallback refresh
- Data sent: HTTP GET only (no local usage payload)

2. **Telegram alerts (optional script path)**
- Destination: `api.telegram.org`
- Trigger: scraper errors/recovery events when env vars are set
- Data sent: operational error messages (may include file names/paths)

### Sensitive Data Handling Recommendations

1. Restrict local file permissions:
```bash
chmod 700 ~/.claude ~/.claude/db
chmod 600 ~/.claude/db/command_center.db
chmod 600 ~/.claude/db/command-center-projects.json
chmod 600 ~/.claude/db/pricing_cache.json
chmod 600 ~/.claude/db/cc_usage.db
```

2. Avoid sharing raw DB files if `cc_usage.db` is enabled (contains account email).

3. Sanitize operational logs before external forwarding (especially Telegram alerts).

4. If strict offline mode is required:
- Do not run `--update-pricing`
- Disable optional web/Telegram scripts

### Data Retention

**Default policy:** Indefinite local retention.

**Deletion commands:**
```bash
rm ~/.claude/db/command_center.db
rm ~/.claude/db/command-center-projects.json
rm ~/.claude/db/pricing_cache.json
rm ~/.claude/db/cc_usage.db
rm cc-usage-report-*.png
```

**Selective deletion (analytics DB):**
```sql
DELETE FROM message_entries WHERE year < 2023;
VACUUM;
```

### Multi-User & Shared Systems

Command Center is not designed as a multi-tenant service. On shared systems:
- Use separate OS accounts per user
- Restrict home directory access
- Do not place DB files on shared/cloud-sync folders with concurrent writers

---
