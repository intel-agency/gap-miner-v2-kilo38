# Plan — Template-repo content strategy (template vs. cloned instances)

| | |
| --- | --- |
| **Plan** | Define how content in this template repo behaves when cloned into downstream app instances, and where the transfer/cleanup logic belongs |
| **Target repo** | `intel-agency/agent-context` (a GitHub **template** repo) |
| **Status** | Active |
| **Date** | 2026-07-18 |
| **Related** | [`gh-issue-tracking-init`](../../.agents/skills/gh-issue-tracking-init/SKILL.md) skill (no-arg defaults); `orchestrate-new-project` / `orchestrate-project-setup` skills (seeding flow) |

---

## Context

`agent-context` is a **generic GitHub template repo**. It is not an application —
it is the substrate. The intended lifecycle of an instance is:

1. **Clone** the template into a new repo (GitHub "Use this template" copies all
   files; it does **not** copy git history, issues, PRs, or Projects).
2. **Seed** the clone with a unique app plan and app-specific info (under
   `plan_docs/`, plus memory/decisions as the app takes shape).
3. **Build the issue hierarchy** from the seeded plan — this is exactly what
   `gh-issue-tracking-init` does, and its no-arg defaults (`current repo` +
   `plan_docs/`) are designed for this seeded state.
4. **Develop** the new app against that hierarchy.
5. **Back-flow** fixes and improvements validated in a clone **up** into the
   template so every future instance inherits them.

The flow is two-way: template → clone (seed), clone → template (fixes).

## Problem

GitHub template cloning copies the **file tree verbatim**. So everything we put in
this repo — including content that only makes sense *for the template itself* —
lands in every downstream instance, where it is **confusing or outright incorrect**.

This is **not solvable at the documentation/prose level**, because some content is
inherently project-specific (a project's memory *means* a specific project; it
cannot be reworded into neutrality). The fix must live in the **seeding workflow**
and in **back-flow discipline**, not in trying to neutralize every doc.

## Two content classes

Every file in this repo falls into one of two classes that behave differently
under template-clone:

### Class 1 — Reusable infrastructure (correct everywhere; should transfer)

Context-neutral by design. True in a seeded clone exactly as in the template.

- `.agents/rules/` — coding style, validation, source-control, delegation, tools,
  practices, scripts inventory, app-stacks.
- `.agents/skills/` — including `gh-issue-tracking-init` and its no-arg defaults
  (current-repo + `plan_docs/`), which are *more* useful downstream, not less.
- Repo-root `scripts/` — generic GitHub CLI helpers.
- `AGENTS.md` — the agent instruction contract.
- `.markdownlint.json`, CI workflow definitions, app-stack definitions.

**Test for Class 1:** the statement is equally true before and after cloning. If
yes → Class 1.

### Class 2 — Template-self-referential state (correct here; wrong downstream)

Inherently project-specific. Must **not** be ported down, and must be **reset** on
clone.

- `.agents/memory.md` — Current Activity / Completed Work / Decisions describe
  *agent-context's own* development, not the clone's app.
- This repo's own `docs/plans/.completed/*` — e.g.
  `agent-context-fix-plan.md`, `app-stacks-rules-plan.md`,
  `rules-scripts-and-legacy-migration.md` (plans about improving the template).
- This repo's own `docs/plans/.deferred/*` — e.g. `defect-level-plan.md`.
- **Foreign/contaminating artifacts** — downstream run reports that leaked up into
  the template. Concrete case:
  [`docs/plans/.completed/run-issues-review/gh-issue-tracking-init-run-review.md`](.completed/run-issues-review/gh-issue-tracking-init-run-review.md)
  reviews a run against `intel-agency/gap-miner-v2-oscar32` ("Gap Mining Platform
  v1.0") with plan source `plan_docs/development-plan.md` — a downstream app's
  review report sitting in the generic template, referencing a repo and a plan doc
  that do not exist here.

**Test for Class 2:** the content names a specific project/repo/plan that is not
"the template itself, by design." If yes → Class 2.

## Principle

> Class 1 travels freely and benefits every clone. Class 2 must be **reset on the
> way down** (seeding flow) and **filtered on the way up** (back-flow discipline).

The boundary is enforced by **workflow**, not by doc wording.

## GitHub template-repo capabilities

Researched against the official GitHub docs (2026-07-18;
<https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-repository-from-a-template>).
This is *why* the strategy must be workflow-driven.

**No exclusion mechanism.** "Use this template" copies the *entire default branch
verbatim* — every tracked file/dir. The only filter is one checkbox: default
branch vs. **"Include all branches."** Consequences:

- `.gitignore` does **not** help — it only governs untracked files; template
  content is tracked, so it all transfers.
- No allow/deny list, no per-path toggle, no `.templateignore`, **no post-clone
  hook.**
- Template repos cannot contain Git LFS objects.

**What it provides beyond a plain file copy:**

- `/generate` endpoint + "Use this template" button, plus
  `gh repo create --template` for scripting the clone.
- **Single-commit, unrelated history** — the clone starts as one commit with no
  link to the template's history (unlike a fork); template-derived branches can't
  be PR'd/merged across.
- No fork parent linkage — the clone is fully independent and commits count on the
  contributor's graph.
- Special `.github/` files transfer *and activate* in the clone: `CODEOWNERS`,
  `ISSUE_TEMPLATE/`, `FUNDING.yml`, `workflow-templates/` (starter Actions
  workflows). Org-level `/.github/` default community-health files are a separate
  fallback.
- Classroom integration (template as assignment starter code).

**What does NOT transfer** (only the file tree copies — nothing else): settings,
branch protection / rulesets, **labels**, milestones, **Issues**, **PRs**,
**Projects (v2)**, releases, wiki, Actions secrets/variables, webhooks, deploy
keys. A clone therefore starts with *no* labels, *no* issues, and *no* Project
board — all rebuilt by `gh-issue-tracking-init`.

**Implication.** GitHub gives zero content exclusion and zero post-clone hooks, so
keeping Class-2 out of clones has only two paths:

- **(a) Don't store Class-2 in the default branch** — push the template's own
  planning/memory to a separate branch (excluded unless "Include all branches" is
  checked) or a separate repo. This is the *only* path the platform helps with.
- **(b) Delete it in our own seeding workflow** (W1). The platform provides nothing
  here — it is entirely on us.

## Current state (verified 2026-07-18)

- No reset/cleanup step exists in the project-creation flow today; a clone inherits
  the template's full `memory.md` and `docs/plans/` as-is.
- `.agents/rules/practices.md:15` already orients the agent to glob `plan_docs/`,
  `docs/plans/`, and `docs/` — so the seeding convention (`plan_docs/` first) is
  consistent with existing orientation.
- `gh-issue-tracking-init` no-arg defaults were made explicit on 2026-07-18 and are
  Class 1 (context-neutral).
- The `run-issues-review` foreign artifact (above) is a documented instance of
  Class-2 contamination already present in the template.

## Work items

### W1 — Post-clone reset step in the seeding workflow

Add an explicit **reset/seed stage** to the project-creation dynamic workflow
(`orchestrate-new-project` / `orchestrate-project-setup`), run **after** cloning
the template and **before** building the issue hierarchy:

1. **Reset memory** — overwrite `.agents/memory.md` with a blank skeleton (section
   headers only, empty Current Activity). The populated template memory is
   Class 2.
2. **Clear template plans** — remove the template's own
   `docs/plans/.completed/*.md` and `docs/plans/.deferred/*.md` (plans *about the
   template*). Preserve the lifecycle directory structure for the new app's own
   plans.
3. **Remove foreign artifacts** — delete any run-review / report that references a
   downstream repo (e.g. `run-issues-review/`).
4. **Seed the app plan** — write the unique app plan into `plan_docs/`.
5. **Build the hierarchy** — invoke `gh-issue-tracking-init` with **no args**; its
   defaults resolve to the current repo and `plan_docs/`, exactly the seeded state.

Open: decide whether the template ships a blank `memory.md` skeleton under
`assets/` for the reset step to copy, or the reset script emits one inline.

### W2 — Back-flow discipline (clone → template)

When porting a fix or improvement from a downstream clone up into the template:

- **Port only Class 1** — rules, skills, scripts, `AGENTS.md`, config, app-stacks.
- **Never port Class 2** — the clone's `memory.md`, its `docs/plans/`, its run
  reports, or any app-specific decisions.
- The `run-issues-review` Gap Mining doc is the cautionary example of Class-2
  leakage that slipped through; add a check to the back-flow review.

### W3 — Marker / locating convention (optional)

So the reset step (W1) and back-flow review (W2) know deterministically what to
clear/port, adopt one of:

- **(a) Reserved paths** — keep template-self-referential planning under a clearly
  self-named location, or document that the entire `docs/plans/` tree is
  template-instance-scoped and is reset on clone.
- **(b) Context-neutral AGENTS.md note** — a short note describing the reset
  behavior. **Bootstrap caveat:** any note written in the template transfers too,
  so it must read correctly in *both* contexts (e.g. "The project-creation flow
  clears `memory.md` and `docs/plans/` after cloning this template" — true whether
  read in the template or in a fresh clone).

W3 is enabling work for W1/W2; pick the convention before scripting W1.

## Out of scope / open questions

- **Defect level** for `gh-issue-tracking-init` is tracked separately in
  [`docs/plans/.deferred/defect-level-plan.md`](.deferred/defect-level-plan.md) —
  unrelated to this strategy.
- Whether the template's *own* development planning should live in a separate
  branch/repo entirely (so it never enters the cloneable tree) is an open
  architectural question, not decided here.
- **No platform support for exclusion/hooks** (confirmed 2026-07-18 — see
  *GitHub template-repo capabilities* above): GitHub provides no content exclusion
  and no post-clone hook, so the reset **must** be a step our own seeding workflow
  performs explicitly. Remaining open question: which workflow owns it — confirm
  `orchestrate-new-project` / `orchestrate-project-setup` is the correct home.
  Path (a) above (store Class-2 off the default branch) is the only GitHub-native
  alternative and overlaps the next bullet.

## Validation

- Docs-only change: run `npx --no-install markdownlint-cli2` on this file and the
  edited skill docs.
- Once W1 is scripted: verify against a throwaway clone that (a) `memory.md` is
  blank, (b) template `docs/plans/` is cleared, (c) the foreign `run-issues-review`
  is gone, (d) `gh-issue-tracking-init` with no args resolves to the clone + its
  seeded `plan_docs/`.
