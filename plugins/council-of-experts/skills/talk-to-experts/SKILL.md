---
name: talk-to-experts
description: Consults a council of AI experts using multiple models in parallel. Activate when user says "ask the experts", "consult the experts", "get expert opinions", or requests multi-model expert opinions on complex decisions, architecture, or strategic questions.
---

# Talk to Experts

Launches 6 subagents IN PARALLEL - each consults one expert and returns the response. Your main context stays clean.

## How It Works

1. You prepare the question + context
2. Launch 6 Task subagents in ONE message (all parallel, each with `run_in_background: true`)
3. Each subagent runs its Bash command to consult one expert
4. Collect results from all 6 subagents
5. Synthesize the expert opinions

## Expert Council

| Expert | CLI | Model |
|--------|-----|-------|
| Grok | opencode | xai/grok-4.3 (high reasoning) |
| Kimi | opencode | openrouter/moonshotai/kimi-k2.7-code |
| Gemini | opencode | google/gemini-3.5-flash (high reasoning = max) |
| GLM | opencode | openrouter/z-ai/glm-5.2 |
| Qwen | opencode | openrouter/qwen/qwen3.7-max |
| GPT | codex | gpt-5.5 (xhigh reasoning) |

## Workflow

### 1. Prepare Context

**CRITICAL: Experts need sufficient context.**

Gather:
- Language/Framework versions (PHP 8.4, Laravel 12, Livewire 3)
- Database type and version
- Relevant file paths (absolute!)
- Problem description

### 2. Launch 6 Subagents in ONE Message

Launch ALL 6 Task calls in a SINGLE message. Each subagent runs ONE Bash command.

**Task 1 - Grok:**
```
subagent_type: "expert-consultant"
run_in_background: true
prompt: |
  Run this Bash command and return the output:

  opencode run "@expert [QUESTION_WITH_CONTEXT]" -m xai/grok-4.3 --variant high -f [FILES] --format json 2>&1 | jq -r 'select(.type == "text") | "response: \(.part.text)\nsessionid: \(.sessionID)"'
```

**Task 2 - Kimi:**
```
subagent_type: "expert-consultant"
run_in_background: true
prompt: |
  Run this Bash command and return the output:

  opencode run "@expert [QUESTION_WITH_CONTEXT]" -m openrouter/moonshotai/kimi-k2.7-code -f [FILES] --format json 2>&1 | jq -r 'select(.type == "text") | "response: \(.part.text)\nsessionid: \(.sessionID)"'
```

**Task 3 - Gemini:**
```
subagent_type: "expert-consultant"
run_in_background: true
prompt: |
  Run this Bash command and return the output:

  opencode run "@expert [QUESTION_WITH_CONTEXT]" -m google/gemini-3.5-flash --variant high -f [FILES] --format json 2>&1 | jq -r 'select(.type == "text") | "response: \(.part.text)\nsessionid: \(.sessionID)"'
```

**Task 4 - GLM:**
```
subagent_type: "expert-consultant"
run_in_background: true
prompt: |
  Run this Bash command and return the output:

  opencode run "@expert [QUESTION_WITH_CONTEXT]" -m openrouter/z-ai/glm-5.2 -f [FILES] --format json 2>&1 | jq -r 'select(.type == "text") | "response: \(.part.text)\nsessionid: \(.sessionID)"'
```

**Task 5 - Qwen:**
```
subagent_type: "expert-consultant"
run_in_background: true
prompt: |
  Run this Bash command and return the output:

  opencode run "@expert [QUESTION_WITH_CONTEXT]" -m openrouter/qwen/qwen3.7-max -f [FILES] --format json 2>&1 | jq -r 'select(.type == "text") | "response: \(.part.text)\nsessionid: \(.sessionID)"'
```

**Task 6 - GPT:**
```
subagent_type: "expert-consultant"
run_in_background: true
prompt: |
  Run this Bash command and return the response + session ID:

  codex exec --profile expert --sandbox read-only --skip-git-repo-check "[QUESTION_WITH_CONTEXT]

  Files to analyze:
  [FILE_PATHS]" -m gpt-5.5 -c model_reasoning_effort=\"xhigh\" 2>&1
```

**IMPORTANT: codex MUST use `--sandbox read-only` to prevent any file modifications!**

### 3. Collect Results

Use TaskOutput to collect all 6 responses. Each subagent returns:
- The expert's response
- The session ID for follow-ups

### 4. Synthesize Results

Present to user:

```markdown
## Expert Council Results

### Individual Responses
**Grok:** [summary]
**Kimi:** [summary]
**Gemini:** [summary]
**GLM:** [summary]
**Qwen:** [summary]
**GPT:** [summary]

### Consensus
- [Points where experts agree]

### Divergent Views
| Expert | Position |
|--------|----------|

### Recommendation
[Best solution combining insights]

### Session IDs
- Grok: [id]
- Kimi: [id]
- Gemini: [id]
- GLM: [id]
- Qwen: [id]
- GPT: [id]
```

## Follow-Up Conversations

**opencode:**
```bash
opencode run "Follow-up" -s SESSION_ID -m [MODEL] 2>&1
```

**codex:**
```bash
codex exec resume SESSION_ID "Follow-up" 2>&1
```

## File Handling

| CLI | Method |
|-----|--------|
| opencode | `-f /absolute/path/file1 -f /absolute/path/file2` |
| codex | List files in prompt text (codex reads them itself) |

## opencode JSON Output Format

When using `--format json`, opencode outputs one JSON object per line (NDJSON). Example:

```json
{"type":"step_start","timestamp":1765763743412,"sessionID":"ses_xxx",...}
{"type":"text","timestamp":1765763744943,"sessionID":"ses_xxx","part":{"type":"text","text":"The actual response",...}}
{"type":"step_finish","timestamp":1765763744961,"sessionID":"ses_xxx",...}
```

**Parsing with jq (recommended):**

Extract both response and session ID in one clean command:
```bash
opencode run "@expert [QUESTION]" -m [MODEL] --format json 2>&1 | jq -r 'select(.type == "text") | "response: \(.part.text)\nsessionid: \(.sessionID)"'
```

**Output format:**
```
response: The expert's answer here
sessionid: ses_xxxxxxxxxxxxx
```

This filters for the `type="text"` line and extracts both the response text and session ID in a clean, parseable format.

## Rules

- **Launch ALL 6 Tasks in ONE message** - critical for parallelism
- **Use `run_in_background: true`** for each Task
- **Use `subagent_type: "expert-consultant"`** - custom agent with Bash(opencode *), Bash(codex *) permissions
- Each subagent runs ONE Bash command to consult ONE expert
- **Always use absolute paths** for files
- **Always use `--format json`** for opencode commands to get session IDs
- **codex MUST use `--sandbox read-only`** - prevents any file modifications!
- **opencode expert agent has write/edit/bash disabled** - already read-only
- Bash commands run INSIDE subagents, not in main thread
- Main context only receives final synthesized results

## Required: Custom Agent

This skill requires the `expert-consultant` agent at `agents/expert-consultant.md` with these permissions:
```
tools: Bash(opencode *), Bash(codex *)
```
