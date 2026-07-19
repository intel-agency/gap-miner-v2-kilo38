# Delegation & Orchestration

## Delegation

- Delegate work to the appropriate subagent type when possible.
- Prefer to delegate work if you are the top-level agent, esp. if your agent type is not relevant to the current task.
- Delegate to parallel agents to speed up work and reduce implementation time.

## Orchestration

Use orchestration agents to **decompose and delegate** work instead of implementing it all yourself. Pick the **smallest layer** that fits the scope — do not spawn a higher layer for work a lower one (or you directly) can handle.

- `orchestrator` — top-level coordinator for multi-step, multi-agent tasks. Breaks the work into a dependency graph and dispatches units to specialists (`planner`, `developer`, `code-reviewer`, `qa-tester`, `researcher`) in parallel batches. Use as the default for non-trivial, multi-part work.
- `team-lead` — owns a **single workstream** (one feature/epic/fix) end-to-end: reviews the plan, assigns specialists, and enforces the definition of done. Use when the work fits within one accountable owner.
- `team-orchestrator` — runs a **program of multiple parallel workstreams** by delegating each to a `team-lead` and managing cross-team dependencies. Use only for efforts too large for one `team-lead`; otherwise delegate straight to a `team-lead`.
