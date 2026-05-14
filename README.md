# Council of LLMs

Real multi-model council deliberation for OpenClaw. Spawns 3 parallel subagents with different LLMs and distinct analytical perspectives, then synthesizes their independent outputs into a unified verdict.

## Why This Exists

Single-model "councils" — where one subagent roleplays 3 experts — fail repeatedly. They produce context overflow (300-600k tokens), shallow analysis, and empty outputs. Real deliberation requires genuinely different models providing independent perspectives.

## How It Works

1. **Spawn 3 parallel subagents**, each with a different model and analytical lens:
   - **Strategos** — strategy, business impact, feasibility
   - **Analyticos** — data quality, technical correctness, edge cases
   - **Creativos** — creative alternatives, UX, novel approaches

2. **Each agent analyzes independently** with their specific perspective

3. **Synthesize** — merge verdicts into consensus points, disagreements, blind spots, and action items

## Key Principle

The models are **yours to choose**. This skill doesn't hardcode any specific LLM — you configure which models to use based on what's available to you. The more diverse the models, the better the council output.

## Quick Start

### 1. Configure your models

Create `~/.openclaw/council-config.json`:

```json
{
  "council_models": [
    "your-strategic-model",
    "your-analytical-model",
    "your-creative-model"
  ],
  "default_timeout": 900,
  "max_tokens": 8192
}
```

Pick models with different strengths:
- **Strategos:** Long-context reasoning, strategic thinking
- **Analyticos:** Data analysis, technical precision, logical reasoning
- **Creativos:** Creative thinking, novel perspectives, user empathy

**Example (Ollama cloud):**
```json
{
  "council_models": [
    "ollama/kimi-k2.6:cloud",
    "ollama/deepseek-v4-pro:cloud",
    "ollama/gemma4:31b-cloud"
  ]
}
```

**Example (local models):**
```json
{
  "council_models": [
    "ollama/qwen3:32b",
    "ollama/llama3.1:70b",
    "ollama/mistral:7b"
  ]
}
```

### 2. Spawn a council

```
sessions_spawn(
  model: <first model from config>,
  label: "Council-Strategos",
  lightContext: true,
  runTimeoutSeconds: 900,
  task: "You are Strategos... [PASTE CONTEXT]"
)

sessions_spawn(
  model: <second model from config>,
  label: "Council-Analyticos",
  lightContext: true,
  runTimeoutSeconds: 900,
  task: "You are Analyticos... [PASTE CONTEXT]"
)

sessions_spawn(
  model: <third model from config>,
  label: "Council-Creativos",
  lightContext: true,
  runTimeoutSeconds: 900,
  task: "You are Creativos... [PASTE CONTEXT]"
)
```

### 3. Synthesize

When all 3 return, merge their verdicts:

1. **Consensus points** — where all 3 agree
2. **Disagreements** — where they differ and why
3. **Blind spots** — what none of them caught
4. **Final verdict** — weighted synthesis with conditions
5. **Action items** — concrete next steps

Write the synthesis to `council-review-[topic].md`.

## Critical Rules

- **Paste ALL context inline** — agents have no conversation history
- **Keep task descriptions under 2000 words** — longer = context overflow = failure
- **Use `lightContext: true`** — always, to prevent context bloat
- **Set `runTimeoutSeconds` from config** — default 900, increase for complex topics
- **Wait for ALL 3 to complete** — don't synthesize with 2/3 results
- **Never re-spawn** — if one model times out, note it in the synthesis

## Common Failure Modes

| Symptom | Cause | Fix |
|---------|-------|-----|
| All 3 return empty | Gateway overload | Kill zombie subagents, wait, retry |
| One model times out | Slow model + complex task | Increase timeout or simplify task |
| Context overflow (300k+ tokens) | Too much data pasted | Summarize to <2000 words |
| Shallow analysis | Vague task description | Be specific about what to analyze |
| All 3 say the same thing | Not enough perspective differentiation | Use more diverse models |

## Anti-Patterns

- Spawning one subagent and asking it to "be 3 experts" — that's roleplay, not a council
- Pasting 10k+ words of raw data — summarize first
- Using the same model for all 3 perspectives — defeats the purpose
- Synthesizing before all 3 complete — wait for everyone

## Install

```bash
# Via ClawHub (recommended)
clawhub install council-of-llms

# Requires subagent-orchestration
clawhub install subagent-orchestration

# Or from GitHub
openclaw skills install wahajahmed010/council-of-llms
```

## ClawHub

Published at: https://clawhub.ai/skills/council-of-llms

## Companion Skills

- **[Subagent Orchestration](https://github.com/wahajahmed010/subagent-orchestration)** — Core delegation patterns, sandbox constraints, and timeout strategy.
- **[Open Source Contributor](https://github.com/wahajahmed010/open-source-contributor)** — Uses Council of LLMs for Level 3 (advanced) issue review before implementing complex fixes.

## License

MIT-0