---
name: claude-solution-gist
description: Publish a Claude-authored solution as a GitHub gist with clear AI-attribution, model name, date, and a "no warranty" disclaimer. Use when Claude solved a problem solo (not collaborative debug — that's debug-writeup-gist) and the user wants to share the validated answer. Visibility (public/private) is a parameter; defaults to the user's configured preference. Public publishes go through scrub-pii first.
---

# Claude-Authored Solution Gist

Use this when **Claude figured something out** and the user wants to share the result. The gist is clearly attributed to Claude with a model + date header so it's transparent that the content is AI-authored, not the user's own writeup.

For collaborative debugging, use `debug-writeup-gist` instead.

## Inputs

- **Title** — short headline.
- **Problem / question** — what was being solved.
- **Solution** — the working answer, fenced and copy-pasteable.
- **Why it works** — short rationale (not a wall of theory).
- **Caveats** — versions it's tied to, things to watch for.
- **Visibility** — `public` or `private`. Resolve in this order:
  1. Explicit argument from the user this turn.
  2. `default_visibility` in `$CLAUDE_USER_DATA/gist-writer/config.json`.
  3. Fall back to `private` if no config exists. **Never publish public without an explicit opt-in path** — if visibility resolves to `public` from config (not from this turn), confirm once before posting.

## Workflow

1. **Render the template** at `templates/claude-solution.md` with the fields above. Fill `{{model}}` from `$CLAUDE_MODEL` (or the configured `model_attribution`) and `{{date}}` from `date -u +%Y-%m-%d`. Pull `{{user}}` from config.
2. **Environment block** — keep brief. Only include what's actually load-bearing for the solution (e.g. "Ubuntu 25.10, Python 3.12"). If the solution is environment-agnostic, write `Environment-agnostic.` and move on.
3. **If `visibility=public`**, invoke `scrub-pii` on the rendered content. Do not proceed past a 🔴 BLOCKING finding without explicit user override. Apply 🟡 redactions the user accepts.
4. **Show the rendered gist to the user** (file path or inline) and confirm. Last chance to edit before publish.
5. **Publish.** Prefer the bundled MCP (`github-gist` server, tools `create_public_gist` / `create_private_gist`). If the MCP is unavailable, fall back to:
   ```bash
   gh gist create -d "<short description>" /tmp/<slug>.md         # private
   gh gist create -d "<short description>" --public /tmp/<slug>.md # public
   ```
6. **Report the gist URL** to the user and offer to copy it to the clipboard (`wl-copy` on Wayland, `xclip` on X11).

## Template fields → render mapping

| Placeholder | Source |
|---|---|
| `{{title}}` | user-provided headline |
| `{{problem}}` | user-described question / failure mode |
| `{{solution}}` | the working answer |
| `{{rationale}}` | "Why this works" |
| `{{environment}}` | minimal env note, or `Environment-agnostic.` |
| `{{caveats}}` | known constraints |
| `{{model}}` | `$CLAUDE_MODEL` or `model_attribution` |
| `{{date}}` | UTC date `YYYY-MM-DD` |
| `{{user}}` | configured `github_user` |

## Don't

- Don't dress this up as the user's own writeup — the AI-attribution banner is non-negotiable.
- Don't auto-publish public gists without confirmation.
- Don't include the user's full path / hostname / private repo references in the body. The scrub-pii pre-flight is there for this; don't bypass it on public publishes.
- Don't append a verification suffix to the body — the bundled MCP already handles its own; double-stamping looks spammy.
