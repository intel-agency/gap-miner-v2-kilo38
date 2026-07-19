# AGENTS.md

## Memory and Rules

This project uses a dynamic memory and rules system under the `.agents/` directory — the **single source of truth** for project-specific decisions, conventions, and history.

**ALWAYS consult `.agents/` before starting work** — do not assume; look it up.

### Memory

Memory is the project's durable context, located at `.agents/memory.md`. It has four sections:

- **Current Activity** — the active project and its in-progress work items. Move items to Completed when done.
- **Completed Work Items** — finished work, organized by project.
- **Decisions** — design decisions, trade-off choices, and their rationale.
- **Remember To Do** — deferred tasks and future changes to plan when current work is done.

### Rules

Rules dictate coding conventions, tools, validation, testing, source control, delegation, and work practices. Files live under `.agents/rules/`, one per subject:

- **Tools**: `.agents/rules/tools.md`
- **Validation**: `.agents/rules/validation.md`
- **Source Control**: `.agents/rules/source-control.md`
- **Delegation**: `.agents/rules/delegation.md`
- **Practices**: `.agents/rules/practices.md`
- **Coding Style**: `.agents/rules/coding-style.md`
- **Scripts**: [`.agents/rules/scripts.md`](.agents/rules/scripts.md) — repo-root `scripts/` inventory (auth helpers, `import-labels.ps1` for label sync, `query.ps1` as the canonical PR review-thread manager, dispatch-issue creator, permission verifier, remote-index regenerator). Read each script's header + `param()` block for authoritative docs.
- **Agent-Instructions Modules**: [`.agents/rules/ai-instructions-modules.md`](.agents/rules/ai-instructions-modules.md) — lookup tables for `nam20485/agent-instructions` workflow assignments and dynamic workflows, stored at `local_ai_instruction_modules/` (hard-coded path; refreshed by `scripts/update-remote-indices.ps1`).
- **App Stacks**: pre-defined language and tech stack profiles in `.agents/rules/app-stacks/`. Each file is a stack definition named by slug ID, referenced from app development/implementation plans to specify the language, tech stack, tools, and packages to use. Available stacks:
  - `dotnet-aspire-aspnet-blazor` — .NET Aspire + ASP.NET Core + Blazor WASM
  - `dotnet-avalonia-xplatform-desktop` — .NET Avalonia cross-platform desktop
  - `python-uv-fastapi-vite` — Python (uv) + FastAPI + Vite frontend

**IMPORTANT:** Check the relevant rules file before working on any file or performing implementation.

When relocating content into a rules file, the AGENTS.md section that replaces it should become a two-part brief: a title describing the type of content the linked file holds (not just its subject name), and a summary previewing the 1–2 most important actual items from the file, abbreviated in-context.

### Updating Memory and Rules

**ALWAYS update as you go — never wait until you are done with all work.**

- Update the memory file's **Current Activity** section as you progress; move completed items to **Completed Work Items**, move the **Project** header down when you complete the **entire** project.
- Update the relevant rules file when you learn new conventions, requirements, or guidelines. If a rules file does not exist for a subject, create it.

---

## Validation

All non-trivial changes must be validated using build, scan, and test steps after making them and always before committing. Detailed validation, testing, and TDD rules live in [`.agents/rules/validation.md`](.agents/rules/validation.md). Test coverage must be maintained > 85%.

## CI/CD Pipeline Requirements

CI/CD pipeline requirements — mandatory test, coverage, and scanning steps plus strict version pinning — live in [`.agents/rules/ci-cd.md`](.agents/rules/ci-cd.md). Pipelines must enforce > 85% coverage and generate an HTML coverage report.

## Committing

Source control rules — safe commits, workflow monitoring, branching, and pull requests — live in [`.agents/rules/source-control.md`](.agents/rules/source-control.md). Always run the `/safe-commit` skill before committing.

## Delegation & Orchestration

Delegate work to the appropriate subagent and orchestrate multi-agent tasks using the smallest layer that fits the scope. Detailed delegation and orchestration rules live in [`.agents/rules/delegation.md`](.agents/rules/delegation.md).

## Orientation

Always orient to the project's history and current state before starting any work — memory, plans, pending changes, and recent commits. Detailed orientation steps live in [`.agents/rules/practices.md`](.agents/rules/practices.md).

## Planning, Investigation & Making Changes

Always plan non-trivial tasks before starting, investigate root causes using first-hand sources, and make the smallest surgical changes possible. Detailed rules for the full lifecycle live in [`.agents/rules/practices.md`](.agents/rules/practices.md).

## Coding Style Discipline

**ALWAYS load and apply these guidelines whenever making or planning code changes.** Write minimum code that solves the problem (no speculative features or unnecessary abstractions), define verifiable success criteria before implementing, and loop until verified. Detailed rules live in [`.agents/rules/coding-style.md`](.agents/rules/coding-style.md).

## Tool Usage

Detailed tool guidance and decision points live in [`.agents/rules/tools.md`](.agents/rules/tools.md). Always use `sequentialthinking` and the Memory knowledge-graph for all non-trivial tasks.

- **Sequential-Thinking** — externalizes reasoning into discrete, revisable thought steps for non-trivial, multi-step problems.
- **Memory** — persistent knowledge-graph store for durable, reusable context (entities, observations, relations) across sessions.
- **Semantic Search (Codebase Indexing)** — finds code by meaning via AI embeddings; prefer as the first probe in unfamiliar code areas.
- **Web & Repository Research (Z.AI MCP)** — remote web search (`webSearchPrime`), URL reader (`webReader`), and public GitHub repo reader (`zread`).
- **Exa Search (MCP)** — neural web search, code-context lookup, and site crawling; complement to Z.AI.
