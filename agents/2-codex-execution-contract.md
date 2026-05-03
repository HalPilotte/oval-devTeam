# Dev Team — Codex Execution Contract

This guide defines how the Oval dev team runs inside Codex. It is the runtime layer for skills, tools, and delegated sub-agents. It does not replace the lane, tier, or gate rules in `1-orchestration-guide.md`; it tells the team how to execute those rules efficiently in Codex.

---

## Core Model

- Named roles such as **Bob**, **Ava**, and **Orion** are logical personas used for routing, authority, and artifact ownership.
- Named roles are **not** literal Codex sub-agent types and must not be treated as directly spawnable roles.
- Actual spawned sub-agents must use Codex-native types only: `explorer`, `worker`, or `default`.
- `Bob` owns routing and delegation decisions by default unless the user explicitly overrides the flow.
- Specialists may recommend delegation, but they do not fan out by default without trigger conditions.
- The team should use the minimum execution surface that will complete the task correctly. Do not simulate a large cast conversationally when direct execution is cheaper and clearer.

---

## Execution Decision Order

After `Bob` completes intake, choose the execution path in this order:

`classify -> choose role(s) -> decide direct vs skill vs tool vs sub-agent -> execute -> synthesize`

Use this precedence:

1. **Direct execution**
   Use when the task is small, obvious, or blocked on the next fact.
2. **Skill**
   Use when an installed skill clearly matches the task workflow.
3. **Tool**
   Use when the work is short, local, mechanical, or connector-driven.
4. **Sub-agent**
   Use when the work is bounded, parallelizable, or context-heavy enough to justify delegation.

Installed skills and tools vary by session. Prefer capability categories first and named skills second.

---

## Delegation Policy

### Default Posture

- Delegation default is `selective`.
- Spawn sub-agents only for bounded side work, independent analysis, or disjoint implementation slices that materially shorten the critical path.
- Default parallelism cap is **2 active sub-agents** unless the task is clearly `T3 major` and decomposes cleanly.

### Token-Efficiency Rules

- Never spawn for trivial work or single-step answers.
- Never spawn for work smaller than the delegation overhead.
- Never spawn when the immediate next action is blocked on the delegated result and can be done directly faster.
- Never duplicate full team context unless the delegated task genuinely requires it.

### Time-Efficiency Rules

- Spawn when independent work can run in parallel and materially reduces wall-clock time.
- Spawn when repository exploration, evidence gathering, or draft generation can happen off the critical path while the parent continues working.
- Prefer `worker` only when the delegated write scope is bounded and does not overlap another executor.

### Context-Hygiene Rules

- Pass only the minimum context needed for the delegated task.
- Prefer explicit task framing over forwarding long transcripts.
- Define a stop condition so the sub-agent does not continue past the requested scope.

### Result-Shape Rules

- Every delegated task must request a compact answer or artifact.
- Do not request transcript-style reasoning dumps.
- Results should return as one of: findings, draft artifact, scoped implementation summary, or yes/no gate evidence.

### No-Delegation List

- Single-step answers
- Work whose next action is blocked on the delegated result
- Tightly coupled edits in the same file set
- Work where delegation would duplicate context instead of reducing it

---

## Codex Sub-Agent Matrix

|Type|Use For|Avoid When|Expected Return|
|---|---|---|---|
|`explorer`|Repo discovery, codebase questions, evidence gathering, contract tracing|The parent can inspect directly faster|Short findings with file references, risks, or answers|
|`worker`|Bounded implementation, disjoint write-scope changes, test fixes, draft artifacts|Write scope overlaps another executor, or task is mostly exploratory|Change summary, touched files, verification status, open blockers|
|`default`|Synthesis, comparison, sidecar reasoning, or non-code delegated analysis|A more specific type clearly fits|Compact memo, ranked options, or summarized recommendation|

---

## Skill And Tool Policy

### Skills

- Prefer skills when the task clearly matches an installed workflow.
- Examples by category: `github:*` for PR/CI workflows, `browser-use:browser` for local browser inspection, `figma-implement-design` for Figma-driven UI work, `documents`, `spreadsheets`, or `presentations` for artifact generation, and `openai-docs` for OpenAI platform questions.
- If a named skill is unavailable in the current session, fall back to direct tools and keep the same artifact expectations.

### Tools

- Prefer direct tools for short, local, or mechanical actions.
- Prefer tool discovery only when the needed capability is not already available.
- Prefer parallel tool usage for independent read-only discovery.
- Use connectors only when the task truly depends on external system state.

---

## Shared Contracts

### Bob Intake Record

`Request | Goal | Success criteria | Lane | Tier | Required roles now | Execution mode | Skipped roles + reason | Hard gates | Artifacts | Next checkpoint`

### Delegation Handoff

`Objective | Why delegated | Required inputs | Allowed tools/skills | Expected output | Stop condition | Route-back rule`

`Execution mode` should be one of:

- `direct`
- `skill`
- `tool`
- `sub-agent`
- `hybrid`

---

## Role Group Defaults

- Coordination and governance roles (`Bob`, `Ava`, `Orion`, `Hermes`, `Sable`, `Quinn`, `Finch`) prefer direct reasoning plus targeted `explorer` delegation for evidence gathering. They should rarely use `worker`.
- Delivery roles (`Nova`, `Lyra`, `Doc`, `Kira`) may use `worker` for bounded implementation or drafting and `explorer` for repo tracing.
- Design, data, and AI roles (`Pixel`, `Atlas`, `Vega`, `Aster`) should prefer matching skills first when available, then direct tools, and delegate only when analysis or implementation can be cleanly split.

---

## Validation Scenarios

- Small `T1 quick` UI tweak: `Bob` routes to `Lyra`; execution stays direct or uses one `explorer`.
- Standard UI + API feature: `Bob` routes to `Ava`, `Quinn`, `Lyra`, `Nova`, and `Pixel`; exploration may parallelize, implementation stays bounded.
- New AI feature: `Bob` routes to `Vega`, `Aster`, and `Quinn` first; `Sable` and `Hermes` remain hard gates before production release.
- Sev-1 incident: `Bob` routes to `Finch` and `Kira`; delegation stays minimal and scoped.
- Release-only change: `Hermes`, `Quinn`, and `Sable` stay centralized; any delegation is evidence gathering or scoped verification only.
