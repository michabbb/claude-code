# Council of Experts

A Claude Code plugin that consults a council of 5 AI experts in parallel to get diverse perspectives on complex technical and strategic questions.

## What is the Council of Experts?

When you need expert advice on architecture decisions, code reviews, or strategic questions, this plugin consults 5 different AI models simultaneously:

| Expert | Provider | Model |
|--------|----------|-------|
| **Grok** | xAI | grok-4-1-fast |
| **Kimi** | Moonshot | kimi-k2-thinking |
| **Gemini** | Google | gemini-3-pro-preview |
| **MiniMax** | OpenRouter | minimax-m2.1 |
| **GPT** | OpenAI | gpt-5.2 |

Each expert analyzes your question independently, and you receive a synthesized summary with consensus points, divergent views, and a final recommendation.

## Prerequisites

This plugin requires the following tools to be installed:

### 1. jq (JSON processor)

[jq](https://stedolan.github.io/jq/) is required to parse JSON output from opencode.

**Installation:**
```bash
# Ubuntu/Debian
sudo apt-get install jq

# macOS
brew install jq

# Arch Linux
sudo pacman -S jq
```

### 2. opencode CLI

[opencode](https://github.com/opencode-ai/opencode) is used for Grok, Kimi, Gemini, and MiniMax.

**IMPORTANT:** The `--format json` flag is required to get session IDs from opencode output.

**Installation:**
```bash
# Install opencode (check their docs for the latest method)
npm install -g opencode
# or
pip install opencode
```

**Configuration:**
- Configure API keys for the providers you want to use
- Copy the expert agent to your opencode config:

```bash
cp extras/opencode/expert.md ~/.config/opencode/agent/expert.md
```

### 3. codex CLI

[codex](https://github.com/openai/codex) is used for GPT-5.2.

**Installation:**
```bash
npm install -g @openai/codex
```

**Configuration:**

1. Copy the expert prompt:
```bash
mkdir -p ~/.codex/prompts
cp extras/codex/expert.md ~/.codex/prompts/expert.md
```

2. Add the expert profile to `~/.codex/config.toml`:
```toml
# Expert Profile - read-only advisor for strategic questions
[profiles.expert]
sandbox = "read-only"
model_reasoning_effort = "high"
```

**IMPORTANT:** The `sandbox = "read-only"` setting is critical! It prevents codex from making any file modifications when consulting the expert.

### API Keys Required

Make sure you have API keys configured for:
- xAI (for Grok)
- Moonshot (for Kimi)
- Google AI (for Gemini)
- OpenRouter (for MiniMax)
- OpenAI (for GPT)

## Installation

### Via Claude Code Plugin Marketplace

```bash
# Add the marketplace
/plugin marketplace add michabbb/claude-code

# Install the plugin
/plugin install council-of-experts
```

### Manual Installation

Clone this repository and copy the plugin folder to your Claude Code plugins directory.

## Claude Code Permissions (Required!)

**CRITICAL:** This plugin requires Bash permissions for `opencode` and `codex` commands. Without these permissions, the expert consultations will fail.

### Add to Settings

Add the following permissions to your Claude Code settings:

| Scope | File Location |
|-------|---------------|
| **Global** | `~/.claude/settings.json` |
| **Project** | `.claude/settings.json` (in project root) |

### Required Permissions

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

### Important Syntax Notes

| Syntax | Meaning |
|--------|---------|
| `Bash(opencode:*)` | ✅ Correct - allows `opencode` with any arguments |
| `Bash(opencode *)` | ❌ Wrong - space instead of colon causes validation error |
| `Bash(opencode)` | ❌ Insufficient - only allows `opencode` without arguments |

### Restart Required

After modifying `settings.json`, **restart Claude Code** to apply the new permissions.

## Usage

Once installed, activate the Council of Experts by saying:

- "ask the experts"
- "consult the experts"
- "get expert opinions"

### Example

```
User: Ask the experts - I need to decide whether to use a queue-based architecture
or synchronous processing for my Laravel project. It's an e-commerce platform
handling about 1000 orders per day.
```

Claude will then:
1. Prepare the context
2. Launch 5 parallel subagents (one for each expert)
3. Collect all responses
4. Present a synthesized summary with:
   - Individual expert responses
   - Consensus points
   - Divergent views
   - Final recommendation
   - Session IDs for follow-up questions

### Follow-Up Conversations

Each expert consultation provides a session ID. You can continue the conversation with a specific expert:

**opencode (Grok, Kimi, Gemini, MiniMax):**
```bash
opencode run "Follow-up question here" -s SESSION_ID -m MODEL
```

**codex (GPT):**
```bash
codex exec resume SESSION_ID "Follow-up question here"
```

## Important: Read-Only Safety

All experts are configured to be **read-only**:

- **opencode expert agent**: Has `write: false`, `edit: false`, `bash: false`
- **codex expert profile**: Uses `sandbox = "read-only"`

This ensures that the experts can analyze your code but cannot make any modifications.

## File Structure

```
council-of-experts/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── agents/
│   └── expert-consultant.md # Subagent for running CLI commands
├── skills/
│   └── talk-to-experts/
│       └── SKILL.md         # Main skill definition
├── extras/
│   ├── opencode/
│   │   └── expert.md        # opencode @expert agent
│   └── codex/
│       ├── expert.md        # codex expert prompt
│       └── config.toml.example  # codex profile example
├── LICENSE
└── README.md
```

## Troubleshooting

### "jq command not found"
Make sure jq is installed. See Prerequisites section above.

### "opencode command not found"
Make sure opencode is installed and in your PATH.

### "codex command not found"
Make sure codex is installed and in your PATH.

### "Session ID not found"
Make sure you're using the `--format json` flag with opencode commands. Without this flag, session IDs are not included in the output.

### "API key not configured"
Check that your API keys are properly set in the respective CLI tools.

### Expert makes file changes
This should not happen if configured correctly. Double-check:
- opencode: `~/.config/opencode/agent/expert.md` has `write: false`, `edit: false`, `bash: false`
- codex: `~/.codex/config.toml` has `[profiles.expert]` with `sandbox = "read-only"`

## License

MIT License - see [LICENSE](LICENSE)
