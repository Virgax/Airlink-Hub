# Security Policy — Airlink Platform Repo

## Rule #1 — No secrets in committed files. Ever.

Every secret value (API key, password, bearer token, signed URL, connection string with credentials) lives in a `.env` file that is gitignored. Docs and code reference secrets by their **variable name**, never by the literal value.

✅ Correct: `OpenWeatherMap key is loaded from \`${OPENWEATHERMAP_API_KEY}\` (see .env.example).`
❌ Wrong:   `OpenWeatherMap key is \`abc123actualvalue456\`.`

If you find yourself typing a literal credential into any file inside this repo, stop and reach for `.env`.

---

## Where secrets live

| Location | Purpose | Committed? |
|---|---|---|
| `.env.example` | Canonical list of variable names with empty values | ✅ Yes |
| `.env` (root) | Real local-dev values for orchestration scripts | ❌ Never |
| `airlink-desktop/.env` | Hub client local-dev secrets | ❌ Never |
| `airlink-fb-svc/.env` | Fishbowl bridge service secrets | ❌ Never |
| `airlink-shopify-sync/.env` | Shopify polling service secrets | ❌ Never |
| Power Automate portal | PA flow connection secrets (Shopify, OWM, SQL) | n/a (PA-side) |
| `D:\airlink-fb-svc\.env` on `192.168.181.246` | Production `airlink-fb-svc` secrets | ❌ Never |

`.gitignore` enforces this — see the `.env` patterns there.

---

## Historical exposure tracking

The following secrets were committed in earlier iterations of this repo (or to Claude.ai project memory before migration on 2026-05-26) and **have been rotated**. Each row identifies the credential by a **fingerprint** (first 12 hex chars of its SHA-256) so we can track rotations without storing any fragment of the literal value:

| Secret | Status | Rotation date | Notes |
|---|---|---|---|
| `OPENWEATHERMAP_API_KEY` — fingerprint `069c13d9760b` | 🔴 → ✅ rotated | 2026-05-26 | Detected by GitHub Secret Scanning during migration. New key lives in `.env` and PA connection only. |
| `SHOPIFY_ADMIN_TOKEN` — fingerprint `8ea11bc17ce1` | ✅ rotated | (pre-2026-05-01) | Originally rotated when discovered; string removed from repo on 2026-05-26. |
| `SHOPIFY_API_SECRET` — fingerprint `7cb48f31c8a1` | ✅ rotated | (pre-2026-05-01) | Same as above. |

If you discover another exposed credential:

1. **Rotate the credential at the provider first.** Cleaning the repo does not un-expose it — scanners and archives keep copies.
2. Replace the literal value in all files with the `${VAR_NAME}` reference.
3. Add the variable to `.env.example`.
4. Add a row to the table above.
5. Clean git history (see `tools/clean-git-history.sh`) and force-push.
6. Notify the team — the rotation may break flows that hardcoded the old value.

---

## What counts as a secret

- **Definitely a secret:** API keys, passwords, bearer tokens, OAuth tokens, signed URLs (PA flow URLs with `?sig=...`, SAS URLs, ngrok agent tokens), connection strings containing credentials, JWT signing keys, certificate private keys (`.pem`, `.key`).
- **Not a secret (but operational):** internal IP addresses (192.168.x.x), public DNS hostnames (`airlink-fb-svc.ngrok.app`), employee codes (DR0002), PA workflow IDs (without the SAS), database table/SP names, Fishbowl `customerId=4`. These are fine in committed docs.
- **Borderline:** internal email addresses, Fishbowl `appId=1013`. Default to keeping them out if no good reason to include them.

---

## Detection layers

We rely on the following to keep secrets out:

1. **`.gitignore`** — first line of defense. Prevents `.env`, `*.pem`, `*.key`, `secrets/` from being staged.
2. **GitHub Secret Scanning** — runs automatically on push to detect known patterns (Shopify, OpenAI, AWS, etc.). It found the exposures listed in the history table above.
3. **Pre-commit hook (recommended setup):**
   ```bash
   # In .git/hooks/pre-commit (not committed — set up locally)
   if git diff --cached | grep -E "(shpat_|sk-ant-|AKIA|[a-f0-9]{32,})" > /dev/null; then
     echo "❌ Possible secret in staged changes. Aborting commit."
     echo "   Move the value to .env and reference by name."
     exit 1
   fi
   ```
4. **Manual review** — when in doubt, ask: "Would I be OK if this commit were public on the open internet?" If the answer involves "as long as nobody sees …" — it's a secret.

---

## Loading `.env` in each subproject

### Node.js (`airlink-fb-svc`, future `airlink-shopify-sync`)

```js
import 'dotenv/config';
const key = process.env.OPENWEATHERMAP_API_KEY;
```

In `package.json`: add `"dotenv": "^16.x"` to dependencies.

### Electron (`airlink-desktop`)

In `main.js`, load `.env` early — before any IPC handlers or window creation:

```js
require('dotenv').config();
const sqlConfig = {
  user: process.env.SQL_USER,
  password: process.env.SQL_PASSWORD,
  server: process.env.SQL_SERVER_HOST,
  port: parseInt(process.env.SQL_SERVER_PORT, 10),
  database: 'AIRLINK',
  options: { trustServerCertificate: true, encrypt: false },
};
```

Do **not** read `.env` from the renderer process — keep secrets in main and expose only what's needed via the preload bridge.

### Power Automate

PA flows store their secrets in **Connection** objects (SQL Server connection, HTTP Authorization, etc.). The variables in `.env.example` with `PA_*_URL` names are the URLs of the flows themselves, which include signed query strings — those are secrets, never committed.

---

## Anthropic API key — current security debt

The Anthropic API key is currently hardcoded in the Hub client at `airlink-desktop/index.html` (around line 668), base64-obfuscated. **Obfuscation is not encryption** — anyone with the client binary can recover it. The plan:

1. Stand up a thin Power Automate HTTP flow that proxies Anthropic calls (`PA_Hub_Claude_Proxy`).
2. Move the API key to the PA connection only.
3. Replace the client's direct `fetch('https://api.anthropic.com/...')` with a call to the proxy URL.
4. Remove the base64 blob from `index.html`.
5. Rotate the Anthropic key.

Tracked as a P-high backlog item in `CLAUDE.md` §11.

---

*Maintained by Jaime Estevez (DR0002). Last updated: 2026-05-26.*

---

## If history cleaning is ever needed in the future

If a secret slips into history on the remote (it didn't this time — the migration started from an empty GitHub repo), the standard tool is [`git-filter-repo`](https://github.com/newren/git-filter-repo). The pattern is:

```bash
# 1. Write the literal secret(s) to a local file that is NEVER committed:
echo "literal-secret-value==>***REMOVED***" > /tmp/replace.txt
# 2. Rewrite history:
git filter-repo --replace-text /tmp/replace.txt --force
# 3. Force-push:
git push --force --all
# 4. Delete /tmp/replace.txt and rotate the credential at the provider.
```

Do **not** check the replacement file into the repo — it would itself trigger Push Protection.
