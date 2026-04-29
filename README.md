# Gist Writer

Publish content to GitHub gists with clear AI authorship and a PII pre-flight scrub for public publishes.

## Skills

- `configure` — set default visibility (public/private), GitHub username, environment-block auto-fill, and model attribution. Run this once.
- `claude-solution-gist` — Claude figured something out solo; publish with model + date attribution and a no-warranty disclaimer.
- `debug-writeup-gist` — collaborative debug post-mortem (symptom → environment → investigation → root cause → fix → verification → notes for agents).
- `scrub-pii` — pre-flight scan for IPs, hostnames, MACs, emails, tokens, secrets, home-dir usernames. Auto-invoked by the publish skills on `visibility=public`; usable standalone too.
- `private-gist` — raw `gh gist create` reference for private gists with no template.

## Templates

`templates/claude-solution.md` and `templates/debug-writeup.md` — GitHub-rendered markdown templates with placeholders for title, model, date, environment, and the AI-attribution banner. Edit them in place to suit your style.

## Visibility model

Every publish skill takes a visibility parameter. Resolution order:

1. Explicit argument from the user this turn.
2. `default_visibility` in `$CLAUDE_USER_DATA/gist-writer/config.json`.
3. Fall back to `private` if no config exists.

The shipped configure-prompt default is `public`, but the safe fallback when nothing is configured is `private`. Public publishes always confirm once before posting and always run through `scrub-pii`.

## Requirements

- `gh` CLI installed and authenticated (`gh auth status`), **or**
- `GITHUB_TOKEN` available to Claude Code's environment (PAT with `gist` scope) for the bundled MCP server. The `configure` skill will write this to `$CLAUDE_USER_DATA/gist-writer/.env` (chmod 600) and offer to auto-source it from your shell rc.

## Bundled MCP

This plugin bundles [github-gist-mcp-minimal](https://github.com/danielrosehill/Github-Gist-MCP-Minimal) — 4 tools (create-private / create-public / update / delete) to keep context overhead low. Set `GITHUB_TOKEN` (PAT with `gist` scope) before invoking the publish skills.

## Installation

```bash
claude plugins install gist-writer@danielrosehill
```

After install, run the `configure` skill once to set your defaults.

## License

MIT
