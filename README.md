# Akto MCP Registry

This repository is the source of truth for the MCP server allowlist used by [Akto](https://akto.io).  
On every GitHub release, a workflow automatically syncs `mcp-allowlist.csv` into Akto's MCP Registry.

---

## How it works

```
Edit mcp-allowlist.csv → merge to main → cut a release → GitHub Action → Akto syncs allowlist
```

1. `mcp-allowlist.csv` — single column (`mcp_server_name`), one server per row.
2. GitHub Actions workflow fires on every published release (or manually via workflow_dispatch).
3. The workflow calls Akto's `POST /api/syncMcpRegistry` with an API key stored in GitHub Secrets.
4. Akto fetches the raw CSV from `main`, diffs it against the stored allowlist, and applies changes.

---

## Managing the allowlist

**Add a server** — append a row to `mcp-allowlist.csv` and merge to `main`.

**Remove a server** — delete its row and merge to `main`.

Changes take effect on the next release (or via manual workflow run).

---

## One-time setup

### 1. Fork / create this repo

Push this repo to GitHub and set the default branch to `main`.

### 2. Register the registry in Akto

Call the Akto API once to register the raw CSV URL:

```bash
curl -X POST https://<YOUR_AKTO_URL>/api/addMcpRegistry \
  -H "X-API-KEY: <YOUR_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{
    "url": "https://raw.githubusercontent.com/<ORG>/<REPO>/refs/heads/main/mcp-allowlist.csv"
  }'
```

Save the `registryId` from the response — you need it in the next step.

### 3. Generate an Akto External API token

```bash
curl -X POST https://<YOUR_AKTO_URL>/api/addApiToken \
  -H "Cookie: <admin-session-cookie>" \
  -H "Content-Type: application/json" \
  -d '{"tokenUtility": "EXTERNAL_API", "name": "mcp-registry-github-action"}'
```

Copy the returned API key.

### 4. Add GitHub Secrets

In your GitHub repo → **Settings → Secrets and variables → Actions**, add:

**Secrets** (Settings → Secrets):

| Secret name    | Value                   |
|----------------|-------------------------|
| `AKTO_API_KEY` | The API key from step 3 |

**Variables** (Settings → Variables):

| Variable name      | Value                            |
|--------------------|----------------------------------|
| `AKTO_URL`         | `https://your-akto-instance.com` |
| `AKTO_REGISTRY_ID` | The `registryId` from step 2     |

### 5. Trigger a sync

Either publish a GitHub release, or run the workflow manually:

```
GitHub → Actions → "Sync MCP Registry" → Run workflow
```

---

## CSV format

```csv
mcp_server_name
api.githubcopilot.com
docs.akto.io
filesystem-local
```

- Header must be exactly `mcp_server_name`.
- One server name per row, no extra columns needed.
- Blank lines are ignored by Akto's parser.
