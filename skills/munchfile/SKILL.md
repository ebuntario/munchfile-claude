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

## Step 3 — Choose visibility (map the user's intent)

A munchfile link has three visibility levels. With **no flag the link is private**, and a private link can only be opened by the user themselves — anyone they send it to hits a "request access" wall. Most people who ask to "share" a file actually want **unlisted**. Pick the flag from what the user said:

| The user's ask sounds like… | Flag | Who can open the link |
|---|---|---|
| "just for me", "private", "keep it personal", or gives no audience at all | *(none — private)* | Only the user; recipients must request access |
| "share with", "send to", "give someone", "anyone with the link", "unlisted" | `--unlisted` | Anyone with the link; not shown in search |
| "publish", "make it public", "put it on my profile" | `--public` | Anyone; indexable in search and listed on the profile |

- If the ask clearly implies unlisted or public, pass that flag.
- If it's a bare "share this file" with no audience cue, use the private default (safe) — but you MUST surface the unlisted option when you report the link (Step 5). Do not silently hand back a private link the recipient can't open.

## Step 4 — Watch the file

Resolve the absolute path of the file. Then (add `--unlisted` or `--public` only when Step 3 selected it):

```bash
munchfile watch <absolute-path> [--unlisted|--public]
```

## Step 5 — Parse the URL and report visibility

Split stdout on newlines. Test each line against: `^✅ Live: (https?://\S+)$`

The first matching line's capture group is the live URL. When you share it, say what its visibility means in one plain sentence so the user isn't surprised:

- **private:** "This link is private — only you can open it. Want it unlisted so anyone with the link can view? I can switch it and the URL stays the same."
- **unlisted:** "This link is unlisted — anyone with the link can open it, and it won't show up in search."
- **public:** "This link is public — anyone can open it, and it may appear in search and on your profile."

**Changing visibility later:** re-running `munchfile watch` with a different flag does NOT change an already-watched file (the CLI ignores the flag and says so). To switch, unwatch then re-watch with the flag — the live URL is preserved:

```bash
munchfile unwatch <absolute-path>
munchfile watch <absolute-path> --unlisted
```

**If a line contains "Still munching":** the upload is in progress. Tell the user their URL will be ready shortly — they can run `munchfile watch <path>` again in 30 seconds to retrieve it once the upload completes.

**If a line mentions "the MunchFile app" (e.g. "hasn't minted a link yet — open it to check"):** the desktop app, not the CLI, owns the upload. Tell the user to open the MunchFile desktop app to complete or check the share; do NOT re-run `munchfile watch`.

**If a line contains "Already watching" but NO line matches the `✅ Live:` regex:** the daemon was already running from a previous session. The file IS being tracked and was uploaded then. Tell the user: "Your file is already live — the URL was printed when you first ran `munchfile watch`. Check your previous terminal output, or open https://www.munchfile.com/dashboard to see all your live links." Do NOT tell the user to re-run `munchfile watch <path>` — that will produce the same "Already watching" output with no URL and leave them stuck.

**If a line contains "Session expired" or references `munchfile login`:** tell the user their session has expired and they need to run `munchfile login` to refresh it.

**On any other error:** relay the first non-empty stdout line and suggest the user check `munchfile status` or re-run `munchfile login`.
