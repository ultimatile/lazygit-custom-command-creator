# lazygit-custom-command-creator

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill that helps you create [lazygit](https://github.com/jesseduffield/lazygit) custom commands with conflict-aware keybinding selection.

## What it does

When you ask Claude Code to create a lazygit custom command, this skill:

1. **Fetches the latest lazygit docs** directly from the repository — custom command syntax, available keys, default keybindings, and template variable definitions
2. **Reads your existing config** to understand your style and avoid key conflicts
3. **Selects a non-conflicting key** by checking the target context defaults, global keybindings, and your existing custom commands
4. **Designs the command** — decides whether to keep it inline or extract to a shell script based on complexity
5. **Generates the YAML snippet** (and shell script if needed) ready to add to your `config.yml`

## Install

```
claude install ultimatile/lazygit-custom-command-creator
```

## Usage

Just ask Claude Code to create a lazygit custom command. The skill triggers on mentions of "lazygit custom command", "lazygit keybinding", or lazygit config editing.

Examples:

```
"Create a lazygit custom command to copy the selected commit's short hash to clipboard"
"Add a keybinding in the submodules context to run git submodule update --remote"
"I want a lazygit command to create a PR with gh, prompting for title and body"
```

## Key features

- **Always up-to-date**: Fetches lazygit documentation on every invocation via `curl`, so it stays accurate even when lazygit updates its keybindings or template API
- **Conflict-aware key selection**: Checks against context-specific defaults, global keybindings, and your existing custom commands — proactively suggests available keys
- **Scope awareness**: Questions whether template variables are actually needed and suggests single-item / bulk variants when appropriate
- **Script extraction**: Automatically determines when command logic should be extracted to a shell script, following best practices (absolute paths, `set -euo pipefail`, executable permissions)
