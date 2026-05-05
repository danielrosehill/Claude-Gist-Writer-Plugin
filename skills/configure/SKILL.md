---
name: configure
description: One-time configuration for gist-writer — sets default gist visibility (public/private), the GitHub username used in author footers, and the model-name placeholder behaviour. Writes to $CLAUDE_USER_DATA/gist-writer/config.json.
---

# Configure Gist-Writer

Resolve the data root:

```
DATA_ROOT="${CLAUDE_USER_DATA:-${XDG_DATA_HOME:-$HOME/.local/share}/claude-plugins}/gist-writer"
mkdir -p "$DATA_ROOT"
```

Config lives at `$DATA_ROOT/config.json`. Read existing values if present and offer them as defaults.

## Settings to capture

```json
{
  "default_visibility": "public",
  "github_user": "<auto-detected from `gh api user --jq .login`>",
  "include_environment_block": true,
  "model_attribution": "auto"
}
```

### `default_visibility`

`public` or `private`. **Shipped default for the prompt: `public`** (the user opted into public-by-default). If the user skips configuration entirely and a skill needs a value, fall back to `private` (safe default — never publish without an explicit opt-in).

### `github_user`

The handle that appears in the template footer (`Posted by [@user]`). Auto-detect via `gh api user --jq .login`; let the user override.

### `include_environment_block`

For `debug-writeup-gist`, whether to auto-fill the **Environment** section from the host (OS, kernel, desktop session, relevant package versions). Default: `true`.

### `model_attribution`

How the model name appears in the AI-authored banner.

- `auto` (default) — read from `$CLAUDE_MODEL` if set, otherwise leave a placeholder for the user to confirm at publish time.
- A literal string (e.g. `"Claude Sonnet 4.6"`) — use this string verbatim.

## Workflow

1. Read existing `config.json` if present.
2. Walk through each setting, showing the current/proposed default; accept Enter to keep.
3. Validate: `default_visibility ∈ {public, private}`; `github_user` non-empty; booleans actually booleans.
4. Write the file with `chmod 600` (no secrets, but it's user-scoped).
5. Confirm the path: "Wrote `$DATA_ROOT/config.json`."

## GITHUB_TOKEN — write a `.env`

The bundled `github-gist` MCP needs `GITHUB_TOKEN` (PAT with `gist` scope). Rather than asking the user to manage shell rc exports, write it to a per-plugin `.env`:

1. Prompt for the token (or accept an `op://` reference if 1Password is available).
2. Write it to `$DATA_ROOT/.env`:
   ```
   GITHUB_TOKEN=<your-github-pat-with-gist-scope>
   ```
3. `chmod 600 "$DATA_ROOT/.env"`.
4. Tell the user how to make Claude Code see it. The `.mcp.json` `${GITHUB_TOKEN}` interpolation reads from the shell environment Claude Code was launched in, so the user needs **one** of:

   **Option A — auto-source in shell rc** (recommended for daily use):
   ```bash
   # Append to ~/.bashrc or ~/.zshrc
   set -a; [ -f "$HOME/.local/share/claude-plugins/gist-writer/.env" ] && . "$HOME/.local/share/claude-plugins/gist-writer/.env"; set +a
   ```
   (Use the actual `$DATA_ROOT` path resolved above.)

   **Option B — wrap the launch** (if the user prefers not to pollute global env):
   ```bash
   set -a; source "$DATA_ROOT/.env"; set +a; claude
   ```

   Offer to append Option A to the user's shell rc with their confirmation. Don't do it silently.

5. Confirm: "Wrote token to `$DATA_ROOT/.env`. Restart your shell (or `source` the file) before the next Claude Code session."

If the user already has `GITHUB_TOKEN` set in their environment, skip the .env write and just note that the existing value will be used.

## Notes

- `config.json` holds no secrets. `.env` holds the token only.
- Re-running this skill is safe — it merges over existing values for the JSON config and rewrites the `.env` if the token changed.
