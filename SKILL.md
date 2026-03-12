---
name: lazygit-custom-command-creator
description: Create lazygit custom commands and keybindings. Use when the user wants to add, modify, or debug lazygit custom commands, including YAML config entries and associated shell scripts. Triggers on mentions of "lazygit custom command", "lazygit keybinding", or lazygit config.yml custom command editing.
---

# lazygit Custom Command Creator

This skill helps create lazygit custom commands by generating YAML configuration entries and, when needed, companion shell scripts.

## Step 1: Fetch Latest Documentation

Before generating any custom command, fetch the latest lazygit documentation using `curl -s` via Bash. These are the authoritative source of truth — do NOT rely on memorized or cached versions, as lazygit updates may change available options, template variables, and default keybindings.

Fetch ALL FOUR in parallel (use separate Bash calls so they run concurrently):

1. **Custom command syntax and all available options**:
   `https://raw.githubusercontent.com/jesseduffield/lazygit/master/docs/Custom_Command_Keybindings.md`

2. **Keys available for custom keybindings**:
   `https://raw.githubusercontent.com/jesseduffield/lazygit/master/docs/keybindings/Custom_Keybindings.md`

3. **Default keybindings by context**:
   `https://raw.githubusercontent.com/jesseduffield/lazygit/master/docs/keybindings/Keybindings_en.md`

4. **Template variable field definitions** (Go struct shims — the stable API for custom commands):
   `https://raw.githubusercontent.com/jesseduffield/lazygit/master/pkg/gui/services/custom_commands/models.go`

## Step 2: Read User's Existing Config

Locate and read the user's lazygit `config.yml`. Custom commands are defined under the top-level `customCommands:` key as a YAML list.

Config path: `$XDG_CONFIG_HOME/lazygit/config.yml`

- macOS default: `~/Library/Application Support/lazygit/config.yml`
- Linux default: `~/.config/lazygit/config.yml`

Read existing entries to determine:

- Already-bound custom command keys and their contexts (for conflict avoidance in Step 3)
- The user's style conventions (description language, script locations, naming patterns)

## Step 3: Choose a Keybinding

When selecting a key for the new custom command:

1. Identify the **target context** (e.g., `files`, `commits`, `localBranches`, `remoteBranches`, `global`)
2. Check for conflicts against ALL of these:
   - Default keybindings for the **target context** (from fetched doc #3)
   - Default **global/universal** keybindings (from fetched doc #3) — these apply in every context and must not be shadowed
   - The user's **existing custom commands** for the same context (from Step 2)
3. The chosen key must not conflict with any of the above three sources
4. **Proactively present available keys**: List candidate keys in the target context with mnemonic reasoning, so the user can make an informed choice — don't wait until asked
5. If the user requests a specific key, verify it and warn clearly about any conflicts found

## Step 4: Design the Command

### Scope: does the command need a selected item?

Before using context-specific template variables (e.g., `{{.SelectedSubmodule.Name}}`), consider whether the command actually requires targeting a specific item. Many git commands work on all items by default and specifying a target may not match the user's intent. When both variants (single-item and all-items) are useful, proactively suggest adding both as separate keybindings.

### Inline vs. script extraction

**Keep inline** when: Simple, single-purpose command (1-2 lines, no conditionals/loops, under ~80 chars)

**Extract to shell script** when ANY of these apply:

- Conditional logic (if/else, case)
- Loops or iteration
- Structured data parsing (regex, jq, etc.)
- Exceeds ~80 characters
- Needs error handling beyond basic `set -eu`
- Reusable across multiple commands

### Shell script conventions

When creating a shell script:

- `#!/usr/bin/env bash` shebang with `set -euo pipefail`
- Accept lazygit parameters via positional arguments (`$1`, `$2`, etc.)
- **Use absolute paths** in the YAML `command` field to reference scripts — lazygit's working directory is the project directory, so relative paths are unreliable
- Ensure the script has executable permissions (`chmod +x`), or prefix the command with `bash` as a fallback
- Place alongside the user's existing scripts or in their preferred script directory

## Step 5: Generate the Custom Command

Produce:

1. **YAML snippet** — a complete `customCommands` entry following the syntax from the fetched documentation, including all necessary fields (`key`, `context`, `command`, `description`, and optionally `prompts`, `loadingText`, `output`, `stream`, `showOutput`)
2. **Shell script(s)** — if extracted in Step 4
3. **Explanation** — brief note on the key choice, any conflict considerations, and where to place files

Use Go template syntax as documented in the fetched custom command documentation for dynamic values (selected items, form inputs, filters like `| quote`).
