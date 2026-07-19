# Project Memory

## Current Activity

Current project and its sub-work items that we are working on actively. This section is a placeholder for the current project and its work items, which will be updated as we progress. Once completed, the work items will be moved to the "Completed Work Items" section below.

### Project: Template-repo content strategy

- **`gh-issue-tracking-init` no-arg defaults** (2026-07-18): Made defaults explicit in the skill — `$ghrepo` defaults to the current repo (`gh repo view --json nameWithOwner`); plan source defaults to all docs under `plan_docs/`. Updated `SKILL.md` (Inputs + orchestration preface) and `README.md`. Defaults are Class 1 (context-neutral) — see plan below.
- **Template vs. clone content strategy** (2026-07-18): Authored [`docs/plans/template-content-strategy.md`](../docs/plans/template-content-strategy.md) — two-class model (Class 1 reusable infra travels; Class 2 template-self-referential state must be reset on clone / filtered on back-flow). Three work items: W1 post-clone reset step in the project-creation flow, W2 back-flow discipline (port Class 1 only), W3 marker convention. Not yet implemented; `run-issues-review/` Gap Mining doc flagged as existing Class-2 contamination. Added a *GitHub template-repo capabilities* section (researched vs. official docs): no exclusion mechanism, no post-clone hook, only the file tree copies (no labels/issues/Projects) → reset must be our own seeding workflow; only platform-native alternative is storing Class-2 off the default branch.

#### Work Items

## Completed Work Items

### Project: gh-issue-tracking-init driver fixes + Gap Mining hierarchy recovery (2026-07-16)

Implemented `docs/plans/gh-issue-tracking/run-issues-review/gh-issue-tracking-fix-trace.md` against `intel-agency/gap-miner-v2-oscar32` (Project #88). Driver: `/tmp/kilo/gh-init-driver.ps1`.

- **Driver fixes (all tasks T1–T8):** trace logging (`Trace-Write`/`Invoke-ScriptWithTrace`, per-run file `gh-init-trace-*.log`); renamed `$num`/`$Num` → `$issueNum`/`$NumberByKey` (case-collision root cause); normalized all epics to `Parent='P'` and every story to `Level='story'`; pre-flight schema check (Level/Title/Parent/AC); DryRun body-generation parity; post-run verification block. Added `-ProjectNumber` and `-SkipFields` recovery switches.
- **Skill fixes discovered in-run** (`.agents/skills/gh-issue-tracking-init/scripts/common.ps1`): (1) `Get-IssueDbId` cast changed `[int]`→`[long]` — GitHub issue DB IDs now exceed Int32.MaxValue, breaking sub-issue linking + dependencies; (2) `Find-IssueNumberByTitle` rewritten to use the REST issues endpoint (paginated) instead of `gh issue list --search`, which routes through the GraphQL Search API and fails under its separate rate limit.
- **Result:** hierarchy complete and verified in a clean trace (0 ERR): 30 issues, Plan #1 → 7 epic sub-issues, each epic → its stories (22 total), board = 30 items, 32 dependency edges. `ALL COUNTS MATCH`.
- **Operational note:** heavy GraphQL usage (Projects v2 field stage) exhausts the 5000-point/hour GraphQL budget separately from the REST core limit; recovery switches let re-runs proceed REST-only while GraphQL resets.

### Project: Rules and Memory Consolidation

- **App Stacks section added** (2026-07-15): Added an `App Stacks` entry to the AGENTS.md Rules list, referencing `.agents/rules/app-stacks/` as the home for pre-defined language/tech stack profiles named by slug ID. All 3 existing stacks listed inline. Fixed pre-existing issues: renamed `dotnet-avalonia-xplatform-desktop` to add `.md` extension, corrected wrong H1 (`# python-uv-fastapi-vite` → `# dotnet-aspire-aspnet-blazor`) in the Aspire stack file. Decided against a separate `app-stacks.md` rules file — the directory is self-describing.
- **Tool rules consolidation** (2026-07-10): Moved all tool-related information (Sequential-Thinking, Memory, Semantic Search, Z.AI MCP, Exa MCP) from `AGENTS.md` and the two `docs/tool-*.md` guides into a single new `.agents/rules/tools.md`. AGENTS.md `## Tool Usage` section is now a 2-line pointer; `docs/tool-memory.md` and `docs/tool-sequential-thinking.md` were deleted (content merged, not lost). AGENTS.md rules list now includes a `Tools` entry.
- **Additional rules file extraction** (2026-07-10): Created three more rules files from AGENTS.md content — `.agents/rules/delegation.md` (Delegation + Orchestration), `.agents/rules/validation.md` (Validation + Testing + TDD), `.agents/rules/source-control.md` (Committing/Safe Commit/Monitor/Branching/PRs). Each AGENTS.md section slimmed to a brief pointer. Rules list updated: added `Delegation` entry, removed `Testing` entry (merged into validation.md). CI and Scripts entries remain stub references (files not yet created).
- **Practices rules file + relocation directive** (2026-07-10): Combined the remaining three AGENTS.md sections (Planning, Investigation, Making Changes) into `.agents/rules/practices.md`, framed as the 3-stage engineering lifecycle. Added a written directive to AGENTS.md `#### Files` section: when relocating content to a rules file, the section brief's heading/summary must describe what the reader finds, not just name the subject. All AGENTS.md content sections now follow the brief-with-link pattern.
- **Memory and Rules section cleanup** (2026-07-10): Condensed the AGENTS.md `## Memory and Rules` section from 75 to 32 lines. Removed redundant "consult before starting" (was 3×) and "update as you go" (was 3×) restatements. Removed stale Rules#Examples list (had duplicate "validation" entry) and Memory#Examples subsection. Removed non-existent `ci.md` and `scripts.md` from rules file list. Fixed typos ("informatin", "assumptions-", double "is"). Concentrated uppercase/bold emphasis on 3 distinct critical directives. Full file is now 60 lines.

## Decisions

- **Rules restructuring of `local_ai_instruction_modules/`** (2026-07-18): Consolidated the legacy module directory. Added `.agents/rules/scripts.md` (repo-root `scripts/` brief, built from each script's header/`param()` block — fills the long-standing stub referenced in the 2026-07-10 notes) and `.agents/rules/ai-instructions-modules.md` (pointer to the remote canonical repo `nam20485/agent-instructions`). Deleted four stale/contradictory modules — `ai-custom-agents.md` (25-subagent taxonomy, conflicts with `delegation.md`), `ai-delegation-mandate.md` (75% coverage / FORBIDDEN direct execution, conflicts with `delegation.md`), `ai-development-instructions.md` (references nonexistent `AGENTS.md mandatory_tool_protocols`, stale `mcp_*` tooling), `ai-terminal-commands.md`. Kept `ai-workflow-assignments.md` and `ai-dynamic-workflows.md` in place — their path is hard-coded by `scripts/update-remote-indices.ps1:123-124` and the `/commands` that resolve shortIds. Salvaged two valid facts: SHA-Pinned Actions rule → `ci-cd.md`; CLI error-triage pattern → `practices.md`.

## Remember To Do

Things to plan, add, or change when we are done with the current activity.
