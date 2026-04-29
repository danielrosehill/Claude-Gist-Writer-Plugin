---
name: private-gist
description: Create a private (secret) GitHub gist from a file or content using the gh CLI. Use when the user asks to save something as a private gist, secret gist, or "gist this privately".
---

# Creating a Private GitHub Gist

`gh gist create` creates **secret (private) gists by default**. There is no
`--secret` or `--private` flag — passing one will fail. The `--public` flag is
what makes a gist public; omit it for private.

## Commands

**From an existing file:**
```bash
gh gist create -d "Short description" /path/to/file.md
```

**From multiple files (same gist):**
```bash
gh gist create -d "Description" file1.md file2.sh
```

**From stdin (requires `--filename`):**
```bash
echo "content" | gh gist create -d "Description" --filename notes.md -
```

**Open in browser after creation:**
```bash
gh gist create -d "Description" -w file.md
```

## Flags Reference

- `-d, --desc` — description
- `-f, --filename` — filename when reading from stdin
- `-p, --public` — make it public (DO NOT pass this for private gists)
- `-w, --web` — open in browser after creation

## Workflow

1. If the content doesn't exist as a file yet, write it to a temp path (e.g. `/tmp/<name>.md`) first.
2. Run `gh gist create -d "..." /tmp/<name>.md`.
3. Report the returned gist URL to the user.

## Common Mistakes to Avoid

- Do NOT use `--secret` — it doesn't exist and will error.
- Do NOT use `--private` — same.
- Private is the default; just omit `--public`.
