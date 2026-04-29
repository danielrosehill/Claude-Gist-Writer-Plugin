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

## Notes

- This file holds **no secrets**. The `GITHUB_TOKEN` for the bundled MCP comes from the environment, not from here.
- Re-running this skill is safe — it merges over existing values.
