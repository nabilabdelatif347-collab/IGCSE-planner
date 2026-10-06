# Claude tooling setup

What's configured, why, and how to install it on each surface.

## The toolset

| Need | Tool | Where it comes from |
|---|---|---|
| File edits, codebase search | Built-in Read/Edit/Grep (ripgrep) + `ast-grep` CLI | Built into Claude Code; `npx @ast-grep/cli` |
| Version control / PRs | Git + **GitHub MCP** | Built into cloud sessions; add it yourself locally (see below) |
| Run code and tests | Built-in Bash + Vitest | Built in |
| Up-to-date library docs | **context7** MCP | `.mcp.json` (project) |
| Web pages / docs fetch | **fetch** MCP (+ built-in WebFetch/WebSearch) | `.mcp.json` (project) |
| Browser verification | **playwright** MCP | `.mcp.json` (project) |

Deliberately left out: **filesystem MCP** in Claude Code (redundant with the built-in tools; Desktop
only) and "sequential-thinking"-style servers (the model already reasons and the server just adds
context overhead). Every MCP server adds tool definitions to each request, so keep the list short.

Versions are pinned to ones verified to start on 2026-10-06. Bump deliberately.

## Claude Code (CLI): project servers

`.mcp.json` and `.claude/settings.json` are committed, so cloning the repo and running `claude`
is enough: approve the three servers at the prompt, or they're pre-enabled via
`enabledMcpjsonServers`. Prerequisites: Node ≥ 18 (`npx`) and [uv](https://docs.astral.sh/uv/)
(`uvx`).

To add the same servers by hand instead (e.g. to another repo):

```bash
claude mcp add playwright --scope project -- npx -y @playwright/mcp@0.0.83 --headless --isolated
claude mcp add context7   --scope project -- npx -y @upstash/context7-mcp@4.1.1
claude mcp add fetch      --scope project -- uvx mcp-server-fetch==2026.8.18
```

Don't pass `-e KEY=secret` with `--scope project`: it writes the value into the committed
`.mcp.json`. The committed file reads `CONTEXT7_API_KEY` from your shell via `${CONTEXT7_API_KEY:-}`.

### GitHub MCP (local CLI only; user scope, never committed)

Create a fine-grained PAT (Contents, Pull requests, Issues: read/write, scoped to this repo), then:

```bash
claude mcp add --scope user --transport http github https://api.githubcopilot.com/mcp/ \
  -H "Authorization: Bearer $GITHUB_PAT"
```

Cloud sessions on claude.ai/code already have a GitHub MCP, so skip this there.

Check everything with `claude mcp list` (or `/mcp` inside a session).

### Cloud sessions (claude.ai/code)

The cloud container has Chromium pre-installed but no Google Chrome, and runs as root, so
Playwright MCP needs three variables. Add them in the environment's settings (**Environment
variables**), not in `.mcp.json`, so local checkouts are unaffected:

```
PLAYWRIGHT_MCP_BROWSER=chromium
PLAYWRIGHT_MCP_EXECUTABLE_PATH=/opt/pw-browsers/chromium
PLAYWRIGHT_MCP_NO_SANDBOX=true
```

Optional: `CONTEXT7_API_KEY` (higher rate limits; free at context7.com).

## Claude Desktop

Config file:
- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

Desktop has no built-in file tools, so it also gets the filesystem server. Replace the path
placeholder and tokens, then fully quit and reopen Claude Desktop.

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem@2026.8.31", "/ABSOLUTE/PATH/TO/IGCSE-planner"]
    },
    "github": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "-e", "GITHUB_PERSONAL_ACCESS_TOKEN", "ghcr.io/github/github-mcp-server"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "<YOUR_GITHUB_PAT>" }
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@playwright/mcp@0.0.83"]
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@4.1.1"],
      "env": { "CONTEXT7_API_KEY": "<OPTIONAL_CONTEXT7_KEY>" }
    },
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch==2026.8.18"]
    }
  }
}
```

Notes:
- Windows: if `npx` isn't found, use `"command": "cmd", "args": ["/c", "npx", "-y", ...]`.
- If Desktop can't find `npx`/`uvx` (common on macOS with nvm or Homebrew), use the absolute
  path from `which npx` / `which uvx`.
- No Docker? Remove the `github` entry and use the remote server via a connector in
  Settings → Connectors instead.
