---
name: munchfile
description: Share a local markdown or HTML file as a live URL using the munchfile CLI. Use when the user asks to share, publish, live-link, or get a URL for a local .md, .markdown, .html, or .htm file. Runs `munchfile watch <path>` and returns the live URL printed to stdout.
allowed-tools: "Bash(munchfile:*)"
version: 0.1.0
license: MIT
---

# MunchFile — Share a Local File as a Live URL

Use the `munchfile` CLI to get a persistent live URL for a local file. The URL stays current every time the user saves the file.

## Step 0 — Binary check

```bash
which munchfile
```

If exit code ≠ 0: tell the user munchfile is not installed and stop.
Install: `npm install -g @munchfile/cli`

## Step 1 — Session check

```bash
munchfile status
```

If the output tells the user to run `munchfile login` (i.e. they are not authenticated): relay that and stop. Do NOT read `~/.munchfile/session.json` directly — it holds the auth token, and there is no reason to pull a secret into the conversation.

## Step 2 — Extension guard

Normalize the file extension to lowercase before comparing. munchfile supports: `.md`, `.markdown`, `.html`, `.htm`

If the file has a different extension (after lowercasing), say so and stop.

## Step 3 — Watch the file

Resolve the absolute path of the file. Then:

```bash
munchfile watch <absolute-path>
```

## Step 4 — Parse the URL

Split stdout on newlines. Test each line against: `^✅ Live: (https?://\S+)$`

The first matching line's capture group is the live URL — share it with the user.

**If a line contains "Still munching":** the upload is in progress. Tell the user their URL will be ready shortly — they can run `munchfile watch <path>` again in 30 seconds to retrieve it once the upload completes.

**If a line mentions "the MunchFile app" (e.g. "hasn't minted a link yet — open it to check"):** the desktop app, not the CLI, owns the upload. Tell the user to open the MunchFile desktop app to complete or check the share; do NOT re-run `munchfile watch`.

**If a line contains "Already watching" but NO line matches the `✅ Live:` regex:** the daemon was already running from a previous session. The file IS being tracked and was uploaded then. Tell the user: "Your file is already live — the URL was printed when you first ran `munchfile watch`. Check your previous terminal output, or open https://www.munchfile.com/dashboard to see all your live links." Do NOT tell the user to re-run `munchfile watch <path>` — that will produce the same "Already watching" output with no URL and leave them stuck.

**If a line contains "Session expired" or references `munchfile login`:** tell the user their session has expired and they need to run `munchfile login` to refresh it.

**On any other error:** relay the first non-empty stdout line and suggest the user check `munchfile status` or re-run `munchfile login`.
