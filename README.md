# V0RTEX Banned Plugin List

> **Private repository — D.V.V. access only.**
> This repository is maintained exclusively by Vider_06 (D.V.V. — Dipartimento di Verifica V0RTEX).

---

## Purpose

This repository stores the official banned plugin and banned account list for the V0RTEX plugin ecosystem.

V0RTEX fetches `banned.json` at every startup to perform ban checks against locally installed plugins. The check runs automatically and cannot be disabled by the user.

---

## How V0RTEX uses this repository

1. On startup, V0RTEX downloads `banned.json` from this repository
2. It checks every plugin found in the local `plugins/` folder against `banned_plugins` (by plugin `id`) and `banned_accounts` (by `author` field in the plugin header)
3. Any match results in an immediate block — no override, no exception
4. If the system is offline, V0RTEX falls back to the local cache stored at the previous successful fetch
5. If no cache exists yet (first run, no internet), the online ban check is skipped with a warning — local malware scan still runs

---

## File structure

```
V0rtex-Banned-Plugins/
├── banned.json          ← machine-readable ban list, fetched by V0RTEX
├── README.md            ← this file
└── ENFORCEMENT_LOG.md   ← internal D.V.V. enforcement history
```

---

## banned.json format

```json
{
  "_info": { ... },
  "banned_plugins": [
    {
      "id": "plugin_id_snake_case",
      "name": "Plugin Name",
      "author": "GitHubUsername",
      "sha256_hashes": ["hash_of_original", "hash_of_repost_1"],
      "reason_category": "malicious_code",
      "level": 3,
      "banned_date": "YYYY-MM-DD",
      "notes": "Internal D.V.V. note — not shown to users"
    }
  ],
  "banned_accounts": [
    {
      "github_username": "GitHubUsername",
      "reason_category": "signature_forgery",
      "level": 4,
      "banned_date": "YYYY-MM-DD"
    }
  ]
}
```

### reason_category values

| Value | Meaning |
|---|---|
| `malicious_code` | Plugin contains confirmed malicious code |
| `signature_forgery` | Plugin forged a Verified / Elevated / V0RTEX-Made signature |
| `class_impersonation` | Plugin falsely claims a higher class than granted |
| `permission_abuse` | Plugin uses permissions not granted to its class |
| `policy_violation` | Repeated or severe policy violations |
| `ban_evasion` | Re-submitted a banned plugin under a different name or account |
| `redistribution` | Plugin repackages or wraps V0RTEX itself |

### Ban levels

| Level | Scope |
|---|---|
| 3 | Permanent plugin ban — plugin blocked, developer account still active |
| 4 | Permanent account ban — developer + all associated plugins blocked |

> Levels 1 and 2 (Warning, Suspension) are managed in the public plugin repository and do not appear here.

---

## Adding a ban

### Plugin ban (Level 3)

Add an entry to `banned_plugins`:

```json
{
  "id": "plugin_id",
  "name": "Plugin Name",
  "author": "GitHubUsername",
  "sha256_hashes": ["sha256_of_the_plugin_file"],
  "reason_category": "malicious_code",
  "level": 3,
  "banned_date": "2026-04-12",
  "notes": "Internal note"
}
```

Update `_info.last_updated` and add a row to `ENFORCEMENT_LOG.md`.

### Account ban (Level 4)

Add an entry to `banned_accounts` AND add the plugin entry to `banned_plugins` if applicable:

```json
{
  "github_username": "GitHubUsername",
  "reason_category": "signature_forgery",
  "level": 4,
  "banned_date": "2026-04-12"
}
```

Update `_info.last_updated` and add a row to `ENFORCEMENT_LOG.md`.

### Handling reposts

If a banned plugin is reposted under a different name or id, add the new sha256 to the existing entry's `sha256_hashes` array. V0RTEX checks hashes independently of the id, so a repost with a different name but identical code is caught automatically. If the code was modified, add a new `banned_plugins` entry with the new id and update the author's `banned_accounts` entry if a Level 4 is warranted.

---

*V0RTEX Banned Plugin List — © 2024–2026 Vider_06. All rights reserved.*
