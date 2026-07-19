---
name: gh-issue-tracking-init
description: Use when initializing (or re-syncing) a GitHub issue-based planning hierarchy in a repo — a plan → epic → story → task tree of linked sub-issues plus a Projects v2 board, milestones, and labels. Composes the idempotent PowerShell operation scripts in this skill's own scripts/ directory (self-contained) to build the structure from a plan document and the issue templates. Trigger when the user asks to "set up GH issue tracking", "create the plan/issue hierarchy", "scaffold epics/stories/tasks as issues", or names a target repo ($ghrepo) to initialize.
---

# gh-issue-tracking-init

Initialize the GitHub issue-tracking hierarchy — defined in the design plan
at [`references/gh-issue-tracking-plan.md`](./references/gh-issue-tracking-plan.md)
— for a target repository. The heavy lifting is done by idempotent PowerShell
operation scripts in this skill's own
[`scripts/`](./scripts/) directory (see its
[`README.md`](./scripts/README.md)); this skill decides
**what** to create from the plan and composes those scripts to do it.

This skill is **fully self-contained**: its `scripts/` directory vendors the
small set of general-purpose GitHub CLI helpers it depends on
(`common-auth.ps1`, `import-labels.ps1`, `create-milestones.ps1` — also kept at
the repository root as generic utilities), its issue body templates live in
[`assets/templates/`](./assets/templates/), and the canonical label taxonomy
ships as [`assets/labels.json`](./assets/labels.json). The skill has no
dependency on anything outside its own directory — copy
`gh-issue-tracking-init/` into another repo and it works unmodified.

## Inputs

Both inputs have defaults and may be omitted — the skill then runs against the repo
it is invoked from.

- `$ghrepo` — target repository as `owner/repo` (or a URL you normalize to that).
  **Default when omitted:** the repository the skill is running from — the GitHub
  repo of the current working directory, resolved with
  `gh repo view --json nameWithOwner -q .nameWithOwner` (equivalently, the `origin`
  remote of the enclosing git repo). Use this default unless the user names a
  different repo.
- **Plan source** — a description of the epics/stories/tasks (a filled plan doc, or
  the user's description) that you map onto the four levels and the numbering scheme.
  **Default when omitted:** all plan/app-plan documents under `plan_docs/` in the
  workspace root (`glob plan_docs/**/*.md`). If `plan_docs/` is missing or empty, the
  user must supply a plan source explicitly. When multiple files are found, confirm
  the selection (or merge them in filename order) with the user before proceeding.

## Prerequisites

- `pwsh` 7+ and `gh` authenticated with the `project` scope (`$GITHUB_TOKEN` is honored).
- Run everything from the repository root. Preview with `-DryRun` before applying.

## Conventions (must follow)

- **Sub-issues only** for hierarchy (plan→epic→story→task). Never use issue-linked
  task lists for parent/child. Plain checklists inside a body are fine for
  acceptance criteria / sub-steps only.
- **Numbered titles:** `Plan: <Name>`, `Epic <N>: <Name>`, `Story <N>.<M>: <Name>`,
  `Task <N>.<M>.<K>: <Name>`. Keep numbering stable across re-runs.
- **Level labels:** `plan` / `epic` / `story` / `task`; plus `P0..P3`, `area/*`.
  Workflow state lives in the Project **Status** field, not labels.
- **Milestones** = conceptual work groups (e.g. `POC`, `MVP`, `UI`, `Server`),
  assigned to the epic **and all its descendants**.
- **Phases** are optional; only create the Phase field/values if the plan uses them.
- Bodies come from the templates in
  [`assets/templates/`](./assets/templates/)
  (`application-plan.md`, `epic.md`, `story.md`, `task.md`) — fill placeholders, then
  pass the result to `ensure-issue.ps1` via `-BodyFile`.

## Orchestration steps

Work top-down. Always do a `-DryRun` pass first, show the plan, then apply.
All scripts below live in this skill's `scripts/` directory.

**Resolve inputs first.** If `$ghrepo` is not given, derive it from the current
repo (see [Inputs](#inputs)). If no plan source is named, read every doc under
`plan_docs/`. Only proceed once both are known — if either can't be resolved and
the user hasn't supplied it, ask before doing anything.

1. **Parse the plan** into a tree of nodes (plan, epics, stories, tasks) with the
   numbered titles above, plus per-node labels, milestone, phase, priority, estimate,
   and any blocking dependencies.

   **Canonical node schema** (assert presence before rendering — an omitted field
   produces empty titles/milestones silently otherwise):
   - `Level` (`plan`/`epic`/`story`/`task`)
   - `Title` (the full numbered title, e.g. `Story 1.1: Bootstrap`) **or** the parts to
     build it (`N`/`M`/`K`, `Name`)
   - `Labels`, `Milestone`, `Priority`, `Phase` (if used)
   - `BodyFile` (after rendering), `Prereqs` (array of sibling keys, story/task level)
2. **Labels:** `ensure-labels.ps1 -Repo $ghrepo`.
3. **Milestones:** for each conceptual group, `create-milestones.ps1 -Repo $ghrepo -Titles <name> -SkipExisting`.
4. **Project + fields:** `ensure-project.ps1 -Owner <owner> -Repo $ghrepo [-Phases <p1,p2>]`.
   Capture the **project number** from stdout (`$Proj = & ensure-project.ps1 ...`); in
   `-DryRun` it is `$null` (the project is not created, so nothing is emitted).
5. **Create issues** (top-down) with `ensure-issue.ps1`, filling the matching template
   into a temp file and passing `-BodyFile`. Capture each printed issue **number**:
   - plan → epics → stories → tasks; apply the level label and (for epics + descendants) the milestone.
6. **Link sub-issues** with `link-sub-issue.ps1 -ParentNumber <parent> -ChildNumber <child>`
   for every parent/child edge.
7. **Set board fields** with `set-project-fields.ps1` for each issue: `-Level`,
   `-Priority`, `-Phase` (if used), `-Estimate`, `-Status` (default `Todo`).
8. **Dependencies:** for each "blocked by" edge, `set-dependency.ps1 -IssueNumber <blocked> -BlockedByNumber <blocker>`.
9. **Views:** `ensure-project.ps1` prints the four views to add once in the Project UI
   (not automatable). Relay that to the user.

## Delegation performance (batching)

When composing these scripts, do **not** invoke each op script as its own LLM turn
(one script per reasoning step). That pattern has been **measured at ~77 minutes** for
roughly the first 40% of a 30-issue hierarchy — the per-turn overhead dominates the
actual REST calls.

Instead, **compose a single PowerShell orchestration script** that builds an issue map
once, then loops through the phases in dependency order:

```pwsh
# build the issue map (title -> number) as ensure-issue.ps1 returns each number,
# then drive the remaining ops in tight loops with a small sleep between
# REST-mutating calls to avoid GitHub's secondary rate limits.
$map = @{}
foreach ($n in $nodes) {
    $map[$n.title] = & "$Skill/ensure-issue.ps1" -Repo $ghrepo -Title $n.title -BodyFile $n.bodyFile -Labels $n.labels -Milestone $n.milestone
    Start-Sleep -Milliseconds 500
}
foreach ($e in $edges) { & "$Skill/link-sub-issue.ps1"   -Repo $ghrepo -ParentNumber $map[$e.parent] -ChildNumber $map[$e.child]; Start-Sleep -Milliseconds 500 }
foreach ($n in $nodes) { $ht = $n.fields; & "$Skill/set-project-fields.ps1" @ht; Start-Sleep -Milliseconds 500 }
foreach ($d in $deps)  { & "$Skill/set-dependency.ps1"  -Repo $ghrepo -IssueNumber $map[$d.blocked] -BlockedByNumber $map[$d.blocker]; Start-Sleep -Milliseconds 500 }
```

### `set-project-fields.ps1` must use hashtable splatting

`set-project-fields.ps1` binds parameters **by name**. When you compose calls, use
**hashtable splatting** (`@ht`), never array splatting (`@a` of a `string[]`):

```pwsh
$ht = @{ Owner=$Owner; ProjectNumber=$Proj; Repo=$ghrepo; IssueNumber=$n.number; Level='story'; Status='Todo'; Priority='P1'; Phase='X' }
& "$Skill/set-project-fields.ps1" @ht
```

**Important:** array splatting (`@a` of a `string[]`) is **positional** and will misbind
parameter names — this was the one call-site bug observed in the forensic run. Always
splat a hashtable for this script.

### Avoid integer-keyed lookup dictionaries in the driver

PowerShell integer-keyed `[ordered]@{}` / `@{}` indexing is **unreliable for some keys**
(silent empty returns; reproduced on PS 7.6 — keys 1–6 resolve while key 7 returns empty
even though `.Keys` and `.Values` confirm it is present). When composing a driver, **do
not** build milestone/phase lookup maps as integer-keyed dictionaries. Prefer, in order:

1. a `switch` function (single source of truth, fails loudly on unknown keys), or
2. a `[string]`-keyed `@{}` accessed via `$map["7"]`.

```pwsh
function Milestone-For {
    param([int]$N)
    switch ($N) {
        1 { 'Gate 1: Foundation' }
        2 { 'Gate 1: Foundation' }
        3 { 'Gate 2: Pipelines' }
        default { throw "Unknown epic number for milestone: $N" }
    }
}
```

### DryRun must assert completeness

Before printing the DryRun preview, assert every node renders a non-empty title and (for
epics/stories) a non-empty milestone. Throw on the first empty value so omissions fail
loudly in preview rather than creating empty-titled issues on apply:

```pwsh
foreach ($n in $nodes) {
    if ([string]::IsNullOrWhiteSpace($n.Title)) { throw "Node missing title: $($n | ConvertTo-Json -Compress -Depth 2)" }
    if ($n.Level -in 'epic','story' -and [string]::IsNullOrWhiteSpace($n.Milestone)) { throw "Node $($n.Title) missing milestone" }
}
```

## Idempotency & re-runs

The scripts match existing labels/milestones/fields/issues (by numbered title)/links
and skip or update rather than duplicate. Re-run this skill after editing the plan to
sync additions and changes. Keep titles/numbers stable so matches hold.

## Definition of done

Completion = **closing** a sub-issue (parent progress rolls up automatically). Use the
Project **Status** field for in-flight state. In-issue checklists are informational only.

## Out of scope

The separate issue-implementation skill (current-issue selection / ordering) is
deferred — see the plan's "Out of Scope". **Defects/bugs** are also deferred:
adding a `defect` level would require extending the Project `Level` single-select
*after* field creation, which `gh`/API cannot do (options are only settable at
field creation); shipping it half-working would force a manual UI step on every
re-run, so it is intentionally omitted until that can be fully automated.

## Known limitations

Decide with these in mind up front (not only at script-reference time):

- **Project views are not automatable.** `gh`/API has no supported "create view" operation; `ensure-project.ps1` prints the four views to add once in the Project UI: **By Phase, By Status, By Epic, Current work**.
- **Built-in `Status` options are limited.** `gh` cannot add options to an existing single-select field, so the initial set is `Todo / In Progress / Done`. If your workflow needs `In Review` or `Blocked`, add those once in the UI.
- **GitHub database IDs exceed Int32.** Global issue DB IDs exceed `Int32.MaxValue` (2,147,483,647); `Get-IssueDbId` casts to `[long]`, so `link-sub-issue.ps1` and `set-dependency.ps1` work against modern repos (fixed in this release).

## Validation

Run the Pester suite for the scripts, and the user-run end-to-end smoke test against a
throwaway repo — both documented in
[`scripts/README.md`](./scripts/README.md).
