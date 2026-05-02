# MCP Setup Guide

This is a guide for **Preply PMs** to connect Claude Code to the tools you use every day — Jira, Slack, Figma, Amplitude, Snowflake, and more.

There are **three different methods** Claude Code uses to talk to external services. Each has its own setup, and your tools are split across all three:

| Method                      | What it is                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | Tools that use it                                                       |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **`mcp.json`**              | A config file that lists MCP servers. Claude Code reads it on startup and connects directly to each. Think of it as a phonebook of which APIs to call.                                                                                                                                                                                                                                                                                                                       | Atlassian (Jira/Confluence), Amplitude, Datadog, Figma, Granola, Gosset |
| **claude.ai MCP connector** | Managed connections you set up via your claude.ai account. Anthropic handles the auth, and Claude Code picks them up automatically. More reliable when the local OAuth flow is flaky.                                                                                                                                                                                                                                                                                        | Slack                                                                   |
| **CLI tools**               | Command-line tools (like `gh`, `gws`, `snow`) installed on your machine. Claude runs them the same way an engineer would — write a command, run it, read the result. **TL;DR — why CLI beats MCP here:** an MCP dumps the whole response into Claude's context (slow, expensive, can hit the limit on big SQL results); a CLI lets Claude write a query/command, save the output to a file, and read back only what it needs. Faster, cheaper, more reliable on big results. | Snowflake, Google Workspace, GitHub                                     |

This guide is split into three parts, one per method:
- **Part 1** — `mcp.json` (the bulk of your tools)
- **Part 2** — claude.ai MCP connector (Slack)
- **Part 3** — CLI tools (Snowflake, Google Workspace, GitHub)

Time required: **~30 minutes**. You'll do most of this by asking Claude to set things up for you.

---

## Part 1 — `mcp.json`

Paste this prompt into Claude Code:

> Look at this repo: https://github.com/shanicekhoo-git/preply-pm-os-starter — can you download the `mcp.json` file into my `.claude` settings and walk me through connecting to all the MCPs in it?

Follow Claude's instructions. It might take some back and forth — that's part of the fun of working with AI :)

**What you'll end up authenticating:**
- `atlassian` — Jira + Confluence (OAuth)
- `amplitude` — analytics + experiments (OAuth)
- `datadog` — logs, metrics, dashboards (OAuth)
- `figma` — designs, screenshots, metadata (OAuth)
- `granola` — meeting transcripts and summaries (OAuth)
- `gosset` — experiment data (VPN required, no OAuth)

### A few setup notes Claude will (probably) walk you through

**Gosset.** Connect to Preply VPN first — this MCP only resolves on the internal network. First use prompts for browser auth.

---

## Part 2 — claude.ai MCP connector (Slack)

Slack uses the **claude.ai-hosted connector** instead of `mcp.json`. The local OAuth flow for Slack is unreliable (silent token-refresh failures), so we route it through claude.ai instead — same Slack functionality, more stable.

**To set it up:** inside Claude Code, run `/mcp`, find **Slack** under the claude.ai connectors, and follow the browser sign-in.

Once connected, ask Claude things like *"draft a Slack message to my squad # squad_channel_name about today's standup"*.

---

## Part 3 — CLI tools

Three of the most-used tools are set up as CLIs instead of MCPs: **Snowflake**, **Google Workspace**, and **GitHub**. CLIs let Claude write commands to a file, run them, and read back only what it needs — same way a human would. Faster, cheaper, more reliable on big results.

### Snowflake

Paste this prompt to Claude:

> Help me install and authenticate the Snowflake CLI so I can run queries against the data warehouse.

If you'd rather do it yourself:
```bash
brew install snowflake-cli
snow connection add  # follow prompts
```

Then in Claude Code, ask things like *"run this query against Snowflake"* and Claude uses the CLI directly.

### Google Workspace

Preply uses the official Google Workspace CLI (`gws`) for Drive, Gmail, Calendar, Sheets, Docs, Chat, Admin, and more.

> ⚠️ **Access prerequisite**: only `dev@preply.com` and `data@preply.com` group members can authenticate. If you're in another group (e.g. Marketing, Product), ask **IT Helpdesk** to share the credentials with you first.

Paste this prompt to Claude:

> Help me install and authenticate the Google Workspace CLI (`gws`) for Drive, Gmail, Calendar, Sheets, Docs, Chat. I'll grab the client-id and client-secret from `1Password → Engineering → GoogleWorkspace CLI credentials` when you ask for them.

If you'd rather do it yourself:

1. Install the **Google Cloud CLI** — [Google's quickstart guide](https://cloud.google.com/sdk/docs/install)
2. Install the **Google Workspace CLI** (`gws`) — [GitHub repo](https://github.com/googleworkspace/cli)
3. Authenticate:
   ```bash
   gcloud auth login
   gws auth setup
   ```
   When prompted, use:
   - **project-id**: `preply-gworkspace-cli`
   - **client-id** + **client-secret**: from `1Password → Engineering → GoogleWorkspace CLI credentials`

Once setup is done, ask Claude things like *"check my calendar for tomorrow"* or *"find the latest version of the SEM strategy doc in Drive"*.

### GitHub

Paste this prompt to Claude:

> Help me install and authenticate the `gh` CLI so I can use it for repos, PRs, issues, and gists.

If you'd rather do it yourself:
```bash
brew install gh
gh auth login   # follow the browser flow
```

Verify with `gh auth status`. No env vars needed — `gh` handles auth via keychain.

Once authenticated, ask Claude things like *"create an issue in shanicekhoo-git/preply-pm-os-starter"* or *"show me my open PRs"*.

---

## Verify it works

After launching `claude`, try the different tools that you've just installed:

```
show me my Jira tickets in the squad_name project
```
or
```
create a fijam for a start-stop-continue retro
```

Claude will prompt to authenticate the relevant MCP on first use. If a server isn't responding, run `/mcp` inside Claude Code to check status.

---

## Troubleshooting

| Symptom                                            | Likely fix                                                                                   |
| -------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| `gh: command not found`                            | Ask Claude to install it, or run `brew install gh && gh auth login`                          |
| Gosset returns connection refused                  | VPN not connected                                                                            |
| `gws` says "permission denied" or "no credentials" | You may not be in `dev@` or `data@` groups — ping IT Helpdesk for credentials                |



