# Claude Code Plugins by michabbb

A collection of Claude Code plugins for enhanced AI-assisted development.

## Available Plugins

### Council of Experts

Consult a council of 5 AI experts (Grok, Kimi, Gemini, MiniMax, GPT) in parallel to get diverse perspectives on complex technical and strategic questions.

**Features:**
- 5 AI experts from different providers
- Parallel execution for fast results
- Synthesized summaries with consensus and divergent views
- Session IDs for follow-up conversations
- Read-only safety (experts cannot modify files)

[Read more](plugins/council-of-experts/README.md)

## Installation

### Add the Marketplace

In Claude Code, run:
```
/plugin marketplace add michabbb/claude-code
```

### Install a Plugin

```
/plugin install council-of-experts
```

### List Available Plugins

```
/plugin marketplace list
```

## Required Permissions

**IMPORTANT:** After installing any plugin, you must configure the required Bash permissions in Claude Code.

### Where to Configure

Add permissions to your Claude Code settings file:

| Scope | Location |
|-------|----------|
| **Global** (all projects) | `~/.claude/settings.json` |
| **Project-specific** | `.claude/settings.json` in your project root |

### Permission Syntax

Permissions use prefix matching with `:*` syntax:

```json
{
  "permissions": {
    "allow": [
      "Bash(opencode:*)",
      "Bash(codex:*)"
    ]
  }
}
```

**Syntax Rules:**
- Use `Bash(command:*)` for prefix matching (allows `command` with any arguments)
- Use `Bash(command)` for exact match (only allows `command` without arguments)
- Do NOT use `Bash(command *)` with a space - this is invalid syntax

### Example Full Settings

```json
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(opencode:*)",
      "Bash(codex:*)"
    ],
    "deny": [],
    "ask": []
  }
}
```

### Per-Plugin Requirements

| Plugin | Required Permissions |
|--------|---------------------|
| **council-of-experts** | `Bash(opencode:*)`, `Bash(codex:*)` |

See individual plugin READMEs for detailed permission requirements.

## License

MIT License - see individual plugin licenses for details.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## Author

[michabbb](https://github.com/michabbb)
