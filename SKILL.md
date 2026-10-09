---
name: multi-model-council
description: >-
  Multi-model council — ask the SAME hard question to several DIFFERENT models, each in its own
  isolated subagent, then synthesize their independent answers into ONE result (consensus,
  disagreements, unique insights, prioritized next steps). Use this whenever a decision is
  high-stakes, ambiguous, or "we want a second opinion": architecture/approach reviews, "are we
  doing this right?", "is this the right path?", risk/strategy questions, what-to-do-next,
  research direction, sanity-checks of a plan, or any request to "ask other models", "get a
  consensus", "council of models", "compare model opinions", "research across models". Prefer it
  over answering alone for decisions that are expensive to reverse or where a single perspective
  could be blind. If the user names specific models, use exactly those.
---

# Multi-model council

Run a **panel** of independent models on one question and merge their answers into a single,
decision-ready result. The value is **independent perspectives**: each model answers alone (no
cross-talk, no shared context), so agreement between them is stronger evidence and disagreement
pinpoints the risky/uncertain parts.

## When to use
- High-stakes or hard-to-reverse decisions (architecture, approach, strategy, research direction).
- "Are we doing this right?", "is this the right path?", "what should we do next?".
- The user explicitly asks to consult several models / a council / other opinions.
- A sanity check before committing to a plan.

## Default roster (override if the user names models)
Ask 3–7 models. A good default mix (provider `opencode-go`):
`deepseek-v4-pro`, `glm-5.3`, `grok-4.7`, `qwen3.8-max`, `kimi-k3`.
(Confirm exact IDs with the `models` tool first — names/versions drift.)

## Procedure

1. **Sharpen the question.** Write ONE self-contained prompt: goal + context + the exact question
   + the desired answer shape. Every model gets the IDENTICAL prompt (comparable answers).

2. **Give full context.** Subagents start with a FRESH context — they don't see this conversation,
   the repo, or prior decisions. Pack in what they need: objective, constraints, current state,
   key numbers/measurements, what was already tried, and the explicit questions. If the answer
   would benefit from the real code/files, either quote the key parts or tell the agent which
   files/paths it may read (give it tool access via the `general` agent).

3. **Spawn one subagent per model — all in the same turn (parallel).** For each model:
   - `agent: general`
   - `model: <provider>/<model>` (e.g. `opencode-go/glm-5.3`)
   - the same prompt, plus: "Answer concisely in <language>. Structure: (1) verdict — is the path
     right, yes/no/with-caveats; (2) top risks or flaws you see; (3) concrete next steps,
     prioritized; (4) anything we're missing / novel ideas; (5) confidence 0-100 and why.
     Be specific and critical; do not just agree."

   Keep them independent — never forward one model's answer into another.

4. **Collect and synthesize into ONE result.** Do not just concatenate. Produce:
   - **Consensus** — what most/all agree on (strong signal).
   - **Disagreements** — where they conflict, and what each side's reasoning is (this is where the
     real uncertainty lives).
   - **Unique insights** — points only one model raised, that are actually valuable.
   - **Recommended path** — your synthesized recommendation, plus the single best next action.
   - **Minority/dissent** — flagged, not discarded.
   Attribute claims to models (e.g., "grok-4.7 і deepseek-v4-pro вважають X").

5. **(Optional) persist.** Save the full per-model answers + the synthesis to a file (e.g.
   `docs/council-<topic>-<date>.md`) so the decision is traceable.

## Output template
```markdown
# Council: <topic> (<date>)
Models: <list>
## 1. Verdict (consensus)
## 2. Where they disagree
## 3. Unique insights worth keeping
## 4. Recommended path + next action
## 5. Per-model summaries (verbatim gist)
### <model A> — confidence N/100
### <model B> — ...
```

## Notes
- **Independence is the point.** Same prompt, separate agents, no sharing.
- If a model fails/times out, report that and proceed with the rest (don't block the council).
- Weight by reasoning quality and specificity, not by model popularity.
- For very large inputs, have each agent read files itself (`general` agent has tools) instead of
  stuffing everything into the prompt.
