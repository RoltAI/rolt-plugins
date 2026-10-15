# Rolt plugins

Rolt's plugin for Claude Code and Codex. Rolt reviews the change on your
branch in its cloud and brings the result back to your laptop. The plugin's
one skill, `rolt-review`, lets your agent run that review through the `rolt`
command and act on its findings. The plugin holds no review logic and no
keys: everything goes through `rolt`.

## Install

First install the command and sign in, in a terminal:

```
npm install -g @rolt/cli     # Node 22.12 or later
rolt login
```

Claude Code:

```
claude plugin marketplace add RoltAI/rolt-plugins
claude plugin install rolt@rolt
```

Then ask for a Rolt review, or run `/rolt:rolt-review` (with a pull request
URL, or `--uncommitted`).

Codex:

```
codex plugin marketplace add RoltAI/rolt-plugins
codex plugin add rolt@rolt
```

Then ask Codex for a Rolt review; the skill is `rolt:rolt-review`.

## Files

This repository is both the marketplace and the plugin: Claude Code and
Codex read the same folder.

- `.claude-plugin/marketplace.json`: the marketplace, listing the plugin
  `rolt` at this folder.
- `skills/rolt-review/SKILL.md`: the skill.
