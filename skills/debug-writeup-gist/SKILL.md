---
name: debug-writeup-gist
description: Publish a collaborative debug write-up as a GitHub gist — symptom, environment (auto-detected), investigation, root cause, fix, verification, and notes for other agents. Use when the user and Claude debugged something together and want to share the post-mortem. Visibility is a parameter; defaults to the user's configured preference. Public publishes go through scrub-pii first.
---

# Debug Write-Up Gist

Use this for **collaborative debug post-mortems** — the user hit a problem, Claude helped work it out, and the goal is to publish a write-up that helps the next person (or agent) solve the same thing faster.

For Claude-authored solutions where the user wasn't really part of the debug, use `claude-solution-gist` instead.

## Inputs

- **Title** — concise problem statement (e.g. "Snapcast client drops audio when bluetoothd restarts on Ubuntu 25.10").
- **Symptom** — what the user observed.
- **Investigation** — what was tried; what each step ruled in/out. Trim dead ends that didn't inform the fix.
- **Root cause** — the actual underlying issue.
- **Fix** — what made it work, copy-pasteable.
- **Verification** — how the fix was confirmed.
- **Notes for other agents** *(optional but recommended)* — what an LLM/agent should know to apply this safely.
- **Visibility** — `public` or `private`. Same resolution as `claude-solution-gist`: explicit arg → config default → fall back to `private`. Confirm once before any public publish.

## Auto-detected environment block

If `include_environment_block: true` in config (default), gather these and pre-fill the **Environment** section. Only include rows that are relevant to the bug — don't pad it.

| Field | Command |
|---|---|
| OS + version | `lsb_release -ds` or `cat /etc/os-release` (`PRETTY_NAME`) |
| Kernel | `uname -r` |
| Desktop / session | `echo $XDG_CURRENT_DESKTOP $XDG_SESSION_TYPE` |
| Architecture | `uname -m` |
| Relevant package | `apt list --installed <pkg> 2>/dev/null` or `dpkg -l <pkg>` |
| Python / Node / etc. | `python3 --version`, `node --version` (only if relevant) |
| Hardware quirk | only if load-bearing (e.g. specific GPU / audio device) |

Render the block as a markdown table or fenced block — whichever reads cleaner for the bug class. Always **strip hostnames** from the auto-collected output before embedding.

## Workflow

1. **Pull together the structured fields** from the conversation. If a field is missing, ask the user — don't guess.
2. **Auto-detect environment** (if enabled) and **show the user what was detected before embedding**. They can trim or override.
3. **Render** `templates/debug-writeup.md`.
4. **If `visibility=public`**, invoke `scrub-pii`. Pay extra attention to:
   - Internal hostnames in log excerpts.
   - Real IPs from `ip a` / `journalctl` output.
   - Username paths in stack traces.
   - Any pasted error messages that leak environment specifics not relevant to the bug.
   Apply accepted redactions; abort on 🔴 BLOCKING findings.
5. **Show the rendered gist** for final review.
6. **Publish** via the bundled `github-gist` MCP (`create_public_gist` / `create_private_gist`), or fall back to `gh gist create`.
7. **Report the URL.** Offer to copy to clipboard.

## Format guidance for the body

- **Address it to humans and agents.** Use clear section headers, fenced code blocks for commands and config, and short paragraphs.
- **Investigation should be structured, not narrative.** Bullet points or numbered steps. Each step says what was tried and what it told us.
- **Code blocks must be runnable as written** (after the redaction placeholders are filled in). No prose interleaved inside fences.
- **Notes for other agents** (when included) should call out: which versions this is tied to, signals that mean "this is the same bug," what NOT to try, what to verify after applying.

## Don't

- Don't include full `journalctl` dumps or multi-screen log paste-ins. Trim to the lines that matter and link to the source if longer context is needed.
- Don't drop the AI-assisted banner — the disclaimer is part of the value.
- Don't bypass `scrub-pii` on public publishes.
- Don't fabricate the environment block — if a field can't be detected, leave it out rather than guessing.
