---
name: council-of-llms
description: "Real multi-model council deliberation for OpenClaw subagents. Spawns 3 parallel subagents with different LLMs (kimi-k2.6, deepseek-v4-pro, gemma4:31b) and distinct analytical perspectives (Strategy, Analysis, Creativity), then synthesizes their independent outputs into a unified verdict with consensus points, disagreements, and action items. Fixes the single-model roleplay anti-pattern that causes context overflow and shallow analysis. Requires the subagent-orchestration skill for base spawning patterns. Triggers on: council, deliberate, debate, review, stress-test, multi-model, decision, verdict, analysis, perspectives."
tags:
  - council
  - multi-model
  - deliberation
  - analysis
  - decision-making
  - subagent
  - orchestration
  - llm
  - review
  - stress-test
---

# Council of LLMs

## Overview

A real council spawns **3 parallel subagents**, each with a different model and perspective, then synthesizes their outputs into a unified verdict. This is NOT one model roleplaying 3 experts — it's genuinely different models providing independent analysis.

## Models

Read from `~/.openclaw/council-config.json`:

```json
{
  "council_models": [
    "ollama/kimi-k2.6:cloud",
    "ollama/deepseek-v3.2:cloud",
    "ollama/gemma4:31b-cloud"
  ],
  "default_timeout": 900,
  "max_tokens": 8192
}
```

## Perspectives

Each model gets a different lens:

| Model | Perspective | Role |
|-------|------------|------|
| kimi-k2.6 | **Strategos** | Big-picture strategy, business impact, feasibility |
| deepseek-v4-pro | **Analyticos** | Data quality, technical correctness, edge cases |
| gemma4:31b | **Creativos** | Creative alternatives, user experience, novel approaches |

## How to Run a Council

### Step 1: Prepare the Context

Gather all relevant data BEFORE spawning. Council agents cannot browse the web or access your conversation history. Paste everything they need inline.

### Step 2: Spawn 3 Parallel Subagents

```
sessions_spawn(
  runtime: "subagent",
  mode: "run",
  model: "ollama/kimi-k2.6:cloud",
  label: "Council-Strategos",
  lightContext: true,
  runTimeoutSeconds: 900,
  task: "You are Strategos, a strategic analyst. [PASTE CONTEXT HERE]
  
  Analyze from a STRATEGIC perspective:
  - Business impact and feasibility
  - Market positioning and competitive advantage
  - Resource requirements and ROI
  - Strategic risks and opportunities
  
  Return your analysis as a structured review with: verdict, conditions, risks, recommendations."
)

sessions_spawn(
  runtime: "subagent",
  mode: "run",
  model: "ollama/deepseek-v4-pro:cloud",
  label: "Council-Analyticos",
  lightContext: true,
  runTimeoutSeconds: 900,
  task: "You are Analyticos, a data and logic analyst. [PASTE CONTEXT HERE]
  
  Analyze from an ANALYTICAL perspective:
  - Data quality and completeness
  - Technical correctness and edge cases
  - Statistical validity and sample sizes
  - Logical consistency and contradictions
  
  Return your analysis as a structured review with: verdict, conditions, risks, recommendations."
)

sessions_spawn(
  runtime: "subagent",
  mode: "run",
  model: "ollama/gemma4:31b-cloud",
  label: "Council-Creativos",
  lightContext: true,
  runTimeoutSeconds: 900,
  task: "You are Creativos, a creative and UX thinker. [PASTE CONTEXT HERE]
  
  Analyze from a CREATIVE perspective:
  - User experience and usability
  - Novel alternatives and unconventional approaches
  - Design and presentation improvements
  - What's missing that no one else would think of
  
  Return your analysis as a structured review with: verdict, conditions, risks, recommendations."
)
```

### Step 3: Synthesize

When all 3 return, merge their verdicts:

1. **Consensus points** — where all 3 agree
2. **Disagreements** — where they differ and why
3. **Blind spots** — what none of them caught
4. **Final verdict** — weighted synthesis with conditions
5. **Action items** — concrete next steps

Write the synthesis to `council-review-[topic].md`.

## Critical Rules

1. **Paste ALL context inline** — agents have no conversation history
2. **Keep task descriptions under 2000 words** — longer = context overflow = failure
3. **Use `lightContext: true`** — always, to prevent context bloat
4. **Set `runTimeoutSeconds: 900`** — councils need time
5. **Don't spawn with too much data** — if pasting 10k+ words, summarize first
6. **Wait for ALL 3 to complete** — don't synthesize with 2/3 results
7. **Never re-spawn** — if one model times out, note it in the synthesis

## Common Failure Modes

| Symptom | Cause | Fix |
|---------|-------|-----|
| All 3 return empty | Gateway overload | Kill zombie subagents, wait, retry |
| One model times out | Slow model + complex task | Increase timeout or simplify task |
| Context overflow (300k+ tokens) | Too much data pasted | Summarize to <2000 words |
| Shallow analysis | Vague task description | Be specific about what to analyze |
| All 3 say the same thing | Not enough perspective differentiation | Make perspective prompts more distinct |

## Security & Safety

This skill is **read-only and sandbox-safe**:
- Spawns 3 text-in/text-out subagents via `sessions_spawn` — no filesystem access, no arbitrary commands, no network calls
- Subagents receive a text prompt and return a text analysis — that's it
- No `exec`, no shell commands, no file reads/writes, no API calls
- Models are configured locally via `~/.openclaw/council-config.json` — you control which models run
- All output is a markdown synthesis file written to your workspace

**Why ClawHub may flag this:** The skill mentions `sessions_spawn` and model names, which can look like command execution. In reality, `sessions_spawn` is an OpenClaw primitive that creates an isolated text conversation — equivalent to opening 3 chat windows and pasting a prompt into each.

## Anti-Patterns

- ❌ Spawning one subagent and asking it to "be 3 experts" — that's roleplay, not a council
- ❌ Pasting 10k+ words of raw data — summarize first
- ❌ Using the same model for all 3 perspectives — defeats the purpose
- ❌ Synthesizing before all 3 complete — wait for everyone
- ❌ Ignoring disagreements — disagreements are the most valuable output