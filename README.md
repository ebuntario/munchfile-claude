# munchfile for Claude Code

Share a local markdown or HTML file as a live URL — just by asking Claude.

## What it does

Once installed, Claude knows how to run `munchfile watch <file>` and return the live URL. Say "share this file as a live link" and Claude handles the rest.

```
You: Share my meeting-notes.md as a live link.

Claude: Here's your live link: https://view.munchfile.com/x7k2p
        It updates automatically every time you save the file.
```

## Prerequisites

1. Install the munchfile CLI:
   ```
   npm install -g @munchfile/cli
   ```

2. Log in:
   ```
   munchfile login
   ```

## Install the plugin

In Claude Code, run these two commands:

```
/plugin marketplace add ebuntario/munchfile-claude
/plugin install munchfile
```

That's it. Ask Claude to share a file.

## Supported file types

`.md`, `.markdown`, `.html`, `.htm`

## Links

- [munchfile.dev](https://munchfile.dev) — product homepage
- [munchfile-cli](https://github.com/ebuntario/munchfile-cli) — open source CLI

## License

MIT
