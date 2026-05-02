# MCP Setup Guide

This folder contains a starter `mcp.json` — a config file that connects Claude Code to the tools you use every day (Slack, Jira, Figma, etc.). After setup, you'll be able to ask Claude to query Jira issues, search Slack, read Granola transcripts, and more, directly from your terminal.

For **Snowflake** and **Google Workspace** (Drive / Gmail / Calendar / Sheets / Docs), we use CLIs instead of MCPs — see the "CLI tools" section below.

Time required: **15–30 minutes**, depending on how many MCPs you connect.

---

## Which Claude is this for?

This guide is for **Claude Code** (the terminal / IDE version). If you're using a different surface, here's where to go:

| You use...                               | Where to add MCPs                                                                                                                               |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Claude Code** (terminal or IDE plugin) | This guide — paste into `~/.claude.json` (global config, available in every Claude Code session)                                                |
| **Claude Desktop** (Mac / Windows app)   | Same JSON, different path: `~/Library/Application Support/Claude/claude_desktop_config.json` (Mac) — you can copy this `mcp.json` content there |
| **claude.ai** (web)                      | Settings → Connectors. Click each one, sign in via browser. No file needed.                                                                     |

The recommendation for Preply PMs is **Claude Code** because it gives you slash commands, project files, agents, and full access to your local file system — which is what most PM workflows need.

---

## Before you start

You'll need:
- **Claude Code installed** — [install guide](https://docs.claude.com/en/docs/claude-code/quickstart)
- **VPN connected** — required for Gosset
- **A GitHub Personal Access Token** — required for the GitHub MCP ([generate one here](https://github.com/settings/personal-access-tokens), fine-grained, read access to your own repos)
- **For Google Workspace access**: see "Google Workspace (CLI, not MCP)" section below — uses Preply's `gws` CLI tool, requires IT Helpdesk to grant credentials if you're not in `dev@` or `data@` groups

---

## Quick start

Let Claude Code do the work — clone this repo and ask Claude to set everything up.

1. **Install Claude Code** if you haven't yet — [install guide](https://docs.claude.com/en/docs/claude-code/quickstart)

2. **Clone this repo** to your machine:
   ```bash
   git clone <repo-url> ~/preply-pm-os
   cd ~/preply-pm-os/ai-enablement
   ```

3. **Launch Claude Code** from this folder:
   ```bash
   claude
   ```

4. **Paste this prompt** to Claude:

   > Read `mcp.json` in this folder. Back up `~/.claude.json` to `~/.claude.json.bak`, then merge the `mcpServers` block into `~/.claude.json` while preserving all other fields. After saving, tell me what env vars I need to set (and the exact commands), then instruct me to `/quit` and restart. After I restart, walk me through authenticating each MCP one by one — tell me what to expect (browser flow, VPN, plan tier, etc.) and wait for me to confirm each is working before moving to the next.

5. **Follow Claude's instructions.** It will:
   - Edit `~/.claude.json` for you (with backup)
   - Tell you which env vars to set (e.g. `GITHUB_PAT`)
   - Ask you to `/quit` and restart so the new servers load
   - After restart, walk you through authenticating each of the 9 MCPs

> ⚠️ **Why `/quit` immediately after the edit**: Claude Code writes its own state to `~/.claude.json` during a session, which can clobber Claude's edit if you keep using the same session. Quit and restart cleanly to lock in the change.

> ⚠️ **If Claude Code won't start after the edit**: your JSON has a syntax error. Restore the backup: `cp ~/.claude.json.bak ~/.claude.json` and re-run the prompt above.

---

## Authenticating each MCP

Once Claude Code is running, type `/mcp` to see all servers and their status.

```
/mcp
```

You'll see something like:
```
✓ atlassian       connected
⚠ slack           needs authentication
⚠ figma           needs authentication
✗ gosset          failed (VPN?)
...
```

**For each server marked "needs authentication":**
1. Run `/mcp` and select the server, OR trigger a use case (e.g. *"list my Slack channels"*) — Claude Code will open a browser window
2. Sign in with your Preply account (or personal account where applicable)
3. Approve the requested permissions
4. Done — Claude can now use that MCP

**Order I'd suggest** (least to most fiddly):
1. `atlassian`, `slack`, `figma`, `amplitude`, `granola`, `dovetail` — pure OAuth, click-through
2. `datadog` — OAuth, check region (US vs EU)
3. `github` — needs `GITHUB_PAT` env var set first (see below)
4. `gosset` — needs VPN connected first

Re-run `/mcp` at any time to check what's still pending.

---

## What's included

| MCP | What it does | Auth |
|---|---|---|
| `slack` | Search messages, read channels, send drafts | OAuth |
| `atlassian` | Jira issues + Confluence pages | OAuth |
| `gosset` | Query experiment data (richer than the UI) | none — uses VPN |
| `figma` | Read designs, screenshots, metadata | OAuth |
| `amplitude` | Query analytics + experiments | OAuth |
| `datadog` | Query logs, metrics, dashboards | OAuth |
| `granola` | Pull meeting transcripts and summaries | OAuth |
| `dovetail` | Query user research insights | OAuth |
| `github` | Manage your personal repos | PAT (env var) |

**Not in this file** (use CLIs instead — see "CLI tools" section):
- **Snowflake** — query the data warehouse
- **Google Workspace** — Drive, Gmail, Calendar, Sheets, Docs, Chat

---

## Setup details for the trickier ones

### GitHub
The GitHub MCP needs a Personal Access Token. The remote OAuth flow isn't supported in Claude Code yet.

1. Go to [GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens](https://github.com/settings/personal-access-tokens)
2. Generate a token with **read access to your own repositories**
3. Add to your shell config:
   ```bash
   export GITHUB_PAT=ghp_xxxxx
   ```
4. Restart your terminal so the env var loads

### Gosset
1. Connect to Preply VPN (required — this MCP only resolves on the internal network)
2. First use will prompt for browser auth
3. If you get connection errors, double-check VPN is connected

### Datadog & Amplitude — region check
Default URLs in this config point to **US tenant**. If your Preply account is on EU:
- Datadog: change URL to `https://mcp.datadoghq.eu/...`
- Amplitude: change URL to `https://mcp.eu.amplitude.com/mcp`

(Most Preply users are US — only change if you hit auth errors.)

---

## CLI tools (separate setup, not MCPs)

Two of the most-used tools are set up via CLI instead of MCP: **Snowflake** and **Google Workspace**. Same principle in both cases — CLIs let Claude run commands, write/read files, and iterate naturally instead of dumping everything into the context window. Faster, cheaper, more reliable on big results.

### Snowflake

**TL;DR — why CLI:** PM queries return a lot of rows. An MCP dumps the whole result into Claude's context (slow, expensive, sometimes hits the limit). The CLI lets Claude write SQL to a file, run it, and read back only what it needs — same way a human analyst would.

```bash
brew install snowflake-cli
snow connection add  # follow prompts
```

Then in Claude Code, ask things like *"run this query against Snowflake"* and Claude uses the CLI directly.

### Google Workspace

Preply uses the official Google Workspace CLI (`gws`) for Drive, Gmail, Calendar, Sheets, Docs, Chat, Admin, and more.

> ⚠️ **Access prerequisite**: only `dev@preply.com` and `data@preply.com` group members have permission to authenticate. If you're in another group (e.g. Marketing, Product), ask **IT Helpdesk** to share the credentials with you first.

**Setup:**

1. Install the **Google Cloud CLI** — [Google's quickstart guide](https://cloud.google.com/sdk/docs/install)

2. Install the **Google Workspace CLI** (`gws`) — [GitHub repo](https://github.com/googleworkspace/cli)

3. Authenticate in GCP:
   ```bash
   gcloud auth login
   ```

4. Authenticate in Google Workspace:
   ```bash
   gws auth setup
   ```
   When prompted, use these values:
   - **project-id**: `preply-gworkspace-cli`
   - **client-id**: from `1Password → Engineering → GoogleWorkspace CLI credentials`
   - **client-secret**: same 1Password entry
   - If you don't have access to 1Password Engineering, ping IT Helpdesk

Once setup is done, ask Claude things like *"check my calendar for tomorrow"* or *"find the latest version of the SEM strategy doc in Drive"* — it'll use `gws` directly.

---

## Verify it works

After launching `claude`, try:

```
list my recent Slack DMs
```
or
```
show me my Jira tickets in the MO project
```

Claude will prompt to authenticate the relevant MCP on first use. If a server isn't responding, run `/mcp` inside Claude Code to check status.

---

## Troubleshooting

| Symptom | Likely fix |
|---|---|
| `MCP server "github" failed to start` | Check `echo $GITHUB_PAT` returns your token. Restart terminal if not. |
| Gosset returns connection refused | VPN not connected |
| `gws` says "permission denied" or "no credentials" | You may not be in `dev@` or `data@` groups — ping IT Helpdesk for credentials |
| MCP is missing from `/mcp` list | Check the `mcpServers` block in `~/.claude.json` includes that server (typo? missing entry?) |
| Claude Code won't start after editing config | JSON syntax error — restore backup: `cp ~/.claude.json.bak ~/.claude.json` |

Still stuck? Drop into the AI enablement office hours or post in the channel.
