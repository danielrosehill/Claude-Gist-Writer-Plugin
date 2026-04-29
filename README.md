# Gist Writer

Write content to GitHub gists (public or private) via the `gh` CLI.

## Skills

- `private-gist` — create a private (secret) GitHub gist from a file or content.

## Requirements

- `gh` CLI installed and authenticated (`gh auth status`), **or**
- `GITHUB_TOKEN` environment variable set (PAT with `gist` scope) for the bundled MCP server.

## Bundled MCP

This plugin bundles [github-gist-mcp-minimal](https://github.com/danielrosehill/Github-Gist-MCP-Minimal) — a 4-tool MCP for create-private / create-public / update / delete. Set `GITHUB_TOKEN` in your environment (PAT with `gist` scope) before invoking the plugin's skills.

## Installation

```bash
claude plugins install gist-writer@danielrosehill
```

## License

MIT
