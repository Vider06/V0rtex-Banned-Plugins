# V0RTEX Banned Plugin List

> **Private repository — D.V.V. access only.**
> Maintained exclusively by Vider_06 (D.V.V. — Dipartimento di Verifica V0RTEX).

---

## Purpose

This repository stores the official banned plugin and banned account list for the V0RTEX plugin ecosystem.

V0RTEX fetches `banned.json` at every startup to perform ban checks against locally installed plugins. The check runs automatically and cannot be disabled by the user.

---

## How V0RTEX uses this repository

1. On startup, V0RTEX downloads `banned.json` from this repository
2. For every plugin found in `plugins/`, V0RTEX computes three hashes and an AST token count locally
3. It checks against `banned_plugins` (by id, by all three hashes) and `banned_accounts` (by author field)
4. If no exact match is found, V0RTEX runs a **similarity check** combining hash distance and token count
5. Any exact match → immediate block, no override
6. Similarity ≥ 80% → local ban, blocked like a remote ban, no override
7. Similarity 60–80% → warning shown to user, user can choose to continue
8. If offline: falls back to local `banned_cache.json` from last successful fetch
9. If no cache exists (first run, no internet): online ban check skipped with warning — local malware scan still runs

---

## File structure

```
V0rtex-Banned-Plugins/
├── banned.json          ← machine-readable ban list, fetched by V0RTEX
├── README.md            ← this file
└── ENFORCEMENT_LOG.md   ← internal D.V.V. enforcement history
```

---

## banned.json format — version 1.1

```json
{
  "_info": {
    "description": "V0RTEX Banned Plugin List — D.V.V. private repository",
    "repository": "https://github.com/Vider06/V0rtex-Banned-Plugins",
    "maintainer": "Vider_06",
    "last_updated": "YYYY-MM-DD",
    "format_version": "1.1"
  },
  "banned_plugins": [
    {
      "id":               "plugin_id_snake_case",
      "name":             "Plugin Name",
      "author":           "GitHubUsername",
      "hash_full":        ["sha256_of_exact_file", "sha256_of_repost"],
      "hash_structural":  ["sha256_code_no_comments_no_docstrings"],
      "hash_ast":         ["sha256_of_normalized_ast"],
      "ast_token_count":  342,
      "reason_category":  "malicious_code",
      "level":            3,
      "banned_date":      "YYYY-MM-DD",
      "notes":            "Internal D.V.V. note — not shown to users"
    }
  ],
  "banned_accounts": [
    {
      "github_username":  "GitHubUsername",
      "reason_category":  "signature_forgery",
      "level":            4,
      "banned_date":      "YYYY-MM-DD"
    }
  ]
}
```

### Hash fields explained

| Field | What it hashes | Purpose |
|---|---|---|
| `hash_full` | Entire file, byte for byte | Catches exact copies |
| `hash_structural` | Code with all `#` comments and docstrings removed | Catches reposts that only change header/comments |
| `hash_ast` | Normalized AST (variable and function names stripped) | Catches reposts with superficial refactoring |
| `ast_token_count` | Number of AST nodes in the structural code | Used in similarity scoring alongside hash distance |

All three fields are arrays — add a new hash for each confirmed repost variant.

### Similarity scoring

When no exact hash match is found, V0RTEX computes a combined similarity score:

- **60% weight** — Hamming distance on `hash_structural` (bit-level similarity)
- **40% weight** — token count ratio: `min(local, banned) / max(local, banned)`

| Score | Action |
|---|---|
| < 60% | No action — plugin is considered different |
| 60–80% | Warning shown to user — user may choose to continue |
| > 80% | Plugin banned locally, written to `local_ban_cache.json`, blocked with no override |

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

### Before adding any entry

Compute the three hashes using V0RTEX's own hashing functions (available as a standalone utility — see `v0rtex_utils/`). Do not compute hashes manually.

Required values:
- `hash_full` — SHA-256 of the raw plugin `.py` file
- `hash_structural` — SHA-256 of the file after stripping `#` comments and docstrings via AST
- `hash_ast` — SHA-256 of the normalized AST dump (variable/function names replaced with placeholders)
- `ast_token_count` — integer node count from the normalized AST

### Plugin ban (Level 3)

1. Add an entry to `banned_plugins` with all four hash fields populated
2. Update `_info.last_updated`
3. Add a row to `ENFORCEMENT_LOG.md`

### Account ban (Level 4)

1. Add an entry to `banned_accounts`
2. Also add or update the plugin entry in `banned_plugins` if applicable
3. Update `_info.last_updated`
4. Add a row to `ENFORCEMENT_LOG.md`

### Handling reposts

If a banned plugin reappears under a different name or id:
- If the code is **identical** → same `hash_full`, add the new file's hash to the existing entry's arrays. V0RTEX catches it via hash match regardless of id.
- If the code is **lightly modified** → `hash_full` differs but `hash_structural` or `hash_ast` may still match. Add the new hashes to the arrays.
- If the code is **substantially rewritten** → add a new `banned_plugins` entry. If it is the same author, escalate to Level 4 (`ban_evasion`) and add to `banned_accounts`.

---

*V0RTEX Banned Plugin List — © 2024–2026 Vider_06. All rights reserved.*
