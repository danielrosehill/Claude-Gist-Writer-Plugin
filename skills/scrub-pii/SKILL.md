---
name: scrub-pii
description: Pre-flight PII / sensitive-data scrub for content about to be posted to a public GitHub gist. Identifies IPs, hostnames, MACs, emails, tokens, secrets, and home-directory usernames; proposes redactions; confirms with the user before any public publish. Invoked automatically by claude-solution-gist and debug-writeup-gist when visibility=public; can also be run standalone.
---

# Scrub PII Before Public Publish

This skill runs before any **public** gist is created. For private gists it can be skipped (private gists are still on GitHub and indexable, but the bar is the user's call).

## What to look for

Two passes — regex first (fast, catches the obvious), then a structured review pass (catches context-sensitive leaks).

### Pass 1 — Regex

Flag every match. Don't auto-replace; surface them all and let the user decide per category.

| Category | Patterns / examples |
|---|---|
| **IPv4** (private/public) | `\b(?:\d{1,3}\.){3}\d{1,3}\b` — but ignore `0.0.0.0`, `127.0.0.1`, `255.255.255.0`, common docs ranges (`192.0.2.*`, `198.51.100.*`, `203.0.113.*`) |
| **IPv6** | standard IPv6 patterns; ignore `::1` |
| **MAC** | `\b(?:[0-9A-Fa-f]{2}[:-]){5}[0-9A-Fa-f]{2}\b` |
| **Emails** | `[\w.+-]+@[\w-]+\.[\w.-]+` |
| **Hostnames** (likely-private) | `*.local`, `*.lan`, `*.home`, `*.internal`, `*.corp` |
| **Home-dir usernames** | `/home/<user>/`, `/Users/<user>/`, `/root/` (root is usually fine but flag for review) |
| **API keys / tokens** | `sk-[A-Za-z0-9]{20,}`, `ghp_[A-Za-z0-9]{36}`, `github_pat_[A-Za-z0-9_]{82}`, `xox[baprs]-[A-Za-z0-9-]+` (Slack), `AKIA[0-9A-Z]{16}` (AWS), JWTs (`eyJ[A-Za-z0-9_-]+\.eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+`), generic `Bearer [A-Za-z0-9._-]+` |
| **Passwords / secrets** in env-style lines | `(?i)(password|passwd|pwd|secret|token|api[_-]?key)\s*[=:]\s*\S+` |
| **SSH private keys** | `-----BEGIN (OPENSSH|RSA|DSA|EC|PGP) PRIVATE KEY-----` (block & abort if found) |
| **AWS account IDs** | `\b\d{12}\b` in AWS-context lines |

### Pass 2 — Structured review

After regex, re-read the document and look for:

- **Hostnames that look custom** (not in a public docs/RFC example set). E.g. `cloudpi.local`, `ops-vps-01`, `prod-db-eu`.
- **Project / org names** the user may not want public.
- **Filesystem layouts** that fingerprint the user's setup (e.g. `~/Documents/Clients/<RealClient>/`).
- **Internal URLs** — anything not public.
- **Commit hashes / branch names** referencing private repos.
- **Real names / handles** other than the gist author's intended attribution.

## Output

Produce a redaction report **before** the gist is created:

```
=== PII / Sensitive Data Scan ===

🔴 BLOCKING (must redact or abort):
  - SSH private key at line 42

🟡 LIKELY SENSITIVE (recommend redact):
  - IPv4 10.0.0.42 at line 17 (private RFC1918 → recommend `10.0.0.X`)
  - IPv4 91.123.45.67 at line 19 (public; could fingerprint host → recommend `<public-vps-ip>`)
  - Hostname `daniel-desktop.local` at line 23 → recommend `<workstation>.local`
  - Path `/home/daniel/repos/...` at line 31 → recommend `~/repos/...`

🟢 LOW RISK (review):
  - Email `public@example.com` at line 5 — looks intentional?

Apply redactions? [Y / select-categories / abort]
```

## Redaction conventions

When the user accepts a redaction, replace with a clearly-marked placeholder, not deletion:

- IPs → `<lan-ip>` / `<public-ip>` / `<vps-ip>` (descriptive when possible)
- Hostnames → `<workstation>` / `<server>` / `<host>.local`
- Paths → collapse to `~/...` for home dirs
- Tokens / keys → `<REDACTED-TOKEN>`
- Emails → `<email>` or keep if it's the public author email

Markers help future readers understand *what kind* of value belonged there.

## When to abort

Abort and refuse to publish (don't just warn) if:

- An SSH private key, GPG private key, or `.pem` body is present.
- A live-looking API token is present and the user declines to redact.
- The user's `git remote -v` output for a private repo is embedded.

## Standalone use

When invoked directly (not as a pre-flight), run the same two passes against a file or pasted content and produce the same report. Don't modify the source unless asked.
