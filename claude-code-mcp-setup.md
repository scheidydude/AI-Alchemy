# Claude Code MCP Setup

MCP (Model Context Protocol) connects Claude Code to external tools — GitHub, databases, browsers, file systems, and more. This guide covers the fastest path to getting common servers running.

---

## What MCP Does

Without MCP, Claude Code can only run shell commands and read/write local files. MCP adds structured integrations with real APIs and services — no shell scripting required.

---

## Configuration

Add MCP servers to `~/.claude/settings.json` (global) or `.claude/settings.json` (project):

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-name"],
      "env": {
        "API_KEY": "your-key-here"
      }
    }
  }
}
```

Restart Claude Code after editing.

---

## Common Servers

### GitHub

Read issues, PRs, repos — no `gh` CLI needed.

```json
"github": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-github"],
  "env": {
    "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_..."
  }
}
```

Get a token: GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens. Scopes needed: `repo`, `read:org`.

---

### Filesystem (Extended)

Give Claude access to directories outside the current project.

```json
"filesystem": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/you/Documents", "/Users/you/Desktop"]
}
```

---

### PostgreSQL

Query your database directly.

```json
"postgres": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://user:password@localhost/dbname"]
}
```

Use read-only credentials in production environments.

---

### Puppeteer (Browser)

Let Claude control a browser — scrape, test, screenshot.

```json
"puppeteer": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
}
```

---

### Brave Search

Web search without leaving Claude Code.

```json
"brave-search": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-brave-search"],
  "env": {
    "BRAVE_API_KEY": "BSA..."
  }
}
```

Get a key at brave.com/search/api.

---

### Slack

Read channels, post messages.

```json
"slack": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-slack"],
  "env": {
    "SLACK_BOT_TOKEN": "xoxb-...",
    "SLACK_TEAM_ID": "T..."
  }
}
```

---

### Memory (Persistent KV Store)

Give Claude a persistent key-value store across sessions.

```json
"memory": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-memory"]
}
```

---

## Project-Level vs Global

| Config location | When to use |
|----------------|-------------|
| `~/.claude/settings.json` | Servers you want everywhere (GitHub, search, memory) |
| `.claude/settings.json` | Servers specific to this project (database, internal APIs) |

---

## Verify It's Working

After restarting Claude Code, ask:

```
What MCP tools do you have available?
```

Claude will list all connected servers and their capabilities.

---

## Security Notes

- Store API keys in environment variables, not hardcoded in settings files
- Don't commit `.claude/settings.json` if it contains secrets
- Add `.claude/settings.local.json` to `.gitignore` for sensitive local config
- Use read-only DB credentials unless write access is explicitly needed

---

## Troubleshooting

**Server not appearing:** Check `npx -y @modelcontextprotocol/server-name` runs without error in your terminal.

**Auth errors:** Verify the token has the right scopes and hasn't expired.

**Timeout:** Some servers are slow to start. Wait 10-15 seconds after Claude Code launches before testing.
