# StackGen Power — Setup and Configuration Guide

This guide covers initial PAT configuration, reconfiguration after setup, and troubleshooting MCP connection issues.

---

## Initial Setup (First Time)

### Step 1 — Check Current State

Inspect the `mcp.json` for this power. If either server URL or Authorization header still contains `not-configured`, setup is incomplete.

```json
{
  "mcpServers": {
    "stackgen-admin": {
      "type": "sse",
      "url": "https://not-configured/api/mcp/admin",
      "headers": { "Authorization": "Bearer not-configured" }
    },
    "stackgen-user": {
      "type": "sse",
      "url": "https://not-configured/api/mcp/user",
      "headers": { "Authorization": "Bearer not-configured" }
    }
  }
}
```

### Step 2 — Collect Credentials

Ask the user for:

1. **StackGen Workspace URL** — the base URL of their StackGen instance (e.g. `https://app.stackgen.com` or a self-hosted URL). Normalize to `https://<hostname>` — strip any trailing slashes and path segments.

2. **Personal Access Token (PAT)** — from their StackGen workspace settings under **API Tokens** or **Personal Access Tokens**. The PAT must have **Read + Write** scope to create and modify appStacks.

> Never display the full PAT value once entered. When confirming, show only the last 4 characters.

### Step 3 — Write Configuration

Update `mcp.json` with the collected values:

```json
{
  "mcpServers": {
    "stackgen-admin": {
      "type": "sse",
      "url": "https://<workspace-url>/api/mcp/admin",
      "headers": { "Authorization": "Bearer <PAT>" }
    },
    "stackgen-user": {
      "type": "sse",
      "url": "https://<workspace-url>/api/mcp/user",
      "headers": { "Authorization": "Bearer <PAT>" }
    }
  }
}
```

Both servers must use the same base URL and PAT.

### Step 4 — Reload Kiro

Instruct the user to reload Kiro (equivalent of Cursor's "Reload Window"):
- Use the **Command Palette** → **Reload Window**, or restart Kiro entirely

### Step 5 — Verify

After reload, both MCP servers should be active. Ask Kiro to call `get_stackgen_projects` on `stackgen-admin` to confirm connectivity.

If tools still don't appear, double-check for typos in the URL and ensure no trailing slashes remain.

---

## Reconfiguration (After Initial Setup)

Use this section when the user needs to update an existing working configuration.

### Updating the Workspace URL

If the StackGen instance URL has changed:

1. Read the current `mcp.json` and report the existing base URL
2. Ask for the new URL and normalize it
3. Update both `stackgen-admin` and `stackgen-user` URL entries:
   - `https://<new-url>/api/mcp/admin`
   - `https://<new-url>/api/mcp/user`
4. Instruct the user to reload Kiro

### Rotating the PAT

If the PAT has expired or been revoked:

1. Read current `mcp.json` and confirm the last 4 chars of the existing token
2. Ask the user to generate a new PAT with Read + Write scope from their workspace settings
3. Replace the token in both `Authorization` headers
4. Instruct the user to reload Kiro

---

## Troubleshooting Reference

| Symptom | Likely Cause | Resolution |
|---------|-------------|------------|
| StackGen tools not available | Setup incomplete or reload not done | Check `mcp.json` for `not-configured`; guide through setup; reload |
| 401 Unauthorized | PAT expired, revoked, or wrong scope | Rotate PAT with Read + Write scope |
| Connection timeout | Wrong workspace URL | Verify the base URL is reachable; check for typos |
| Write operations fail (403) | PAT is Read-only | Re-issue PAT with Write scope |
| Tools appear then disappear | Conflicting MCP config in global settings | Remove duplicate `stackgen-admin`/`stackgen-user` entries from other MCP config files |
| `not-configured` URL in responses | `mcp.json` not saved before reload | Save file, then reload |

---

## Security Notes

- The PAT is stored only in `mcp.json` on the local filesystem
- It is never sent to Anthropic, Claude, or any external AI provider
- Kiro's MCP client reads the PAT directly and injects it as an HTTP header when connecting to StackGen's SSE endpoints
- Rotate PATs periodically or immediately if compromised
