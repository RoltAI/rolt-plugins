# Rolt plugins

The marketplace for Rolt's plugin for Claude Code and Codex. Rolt reviews the
change on your branch in the cloud and brings the result back to your
laptop; the plugin lets your agent ask for that review and act on what it
finds.

This repository holds only the two marketplace files. The plugin itself
ships inside the `rolt` npm package, beside the `rolt` command it runs, so
the plugin and the command are always the same version. Each marketplace
pins that version exactly.

## Install

Claude Code:

```
claude plugin marketplace add RoltAI/rolt-plugins
claude plugin install rolt@rolt
```

Codex:

```
codex plugin marketplace add RoltAI/rolt-plugins
```

then install `rolt` from the `rolt` marketplace.

Then sign in once from a terminal (`npx rolt login`) and ask your agent to
review your branch (`/rolt:review`).

## Files

- `.claude-plugin/marketplace.json`: Claude Code's marketplace.
- `.agents/plugins/marketplace.json`: Codex's marketplace.
