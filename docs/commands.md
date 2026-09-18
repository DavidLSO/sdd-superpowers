# Commands and the path of a new feature

Pocket guide for using the `sdd-superpowers` schema day to day: what to type, what each command does, which Superpowers skill fires at each point, and the step-by-step path of a feature from zero to merge.

---

## 1. Setup (once per machine, once per project)

### Per machine

```bash
npm i -g @fission-ai/openspec@latest            # OpenSpec ≥ 1.13
claude plugin install superpowers@claude-plugins-official

# enable ALL /opsx workflows (the "core" profile installs only 4 and the schema needs new/continue/ff/verify)
openspec config set profile custom
openspec config set workflows '["propose","explore","new","continue","apply","ff","sync","archive","bulk-archive","verify","update"]'
```

### Per project

```bash
cd my-project
openspec init                                    # creates openspec/ and the /opsx commands in your harness
mkdir -p openspec/schemas
cp -r /path/to/sdd-superpowers/openspec/schemas/sdd-superpowers openspec/schemas/
#   (or a relative symlink while the schema is still being developed:
#    ln -s ../../../sdd-superpowers/openspec/schemas/sdd-superpowers openspec/schemas/sdd-superpowers)
echo "schema: sdd-superpowers" > openspec/config.yaml
openspec schema validate sdd-superpowers
openspec update --force                          # regenerates the /opsx commands with the enabled workflows
cat openspec/schemas/sdd-superpowers/templates/adopters/CLAUDE.md.fragment.md >> CLAUDE.md
```

Check: `openspec schemas` must list `sdd-superpowers (project)` and `ls .claude/commands/opsx/` must show `apply archive bulk-archive continue explore ff new propose sync update verify`.

---

## 2. `/opsx:*` commands (inside Claude Code)

| Command | What it does | When to use |
|---|---|---|
| `/opsx:new <name>` | Creates `openspec/changes/<name>/` with the default schema | Start of any feature |
| `/opsx:continue` | Produces the **next** pending artifact in the graph (brainstorm → proposal → design → specs → tasks → plan → … → retrospective → finalize) | Step-by-step flow, reviewing each artifact |
| `/opsx:ff <name>` | Creates the change and produces **all** planning artifacts at once (brainstorm through plan) | Small, well-understood change |
| `/opsx:propose <name>` | Same as `ff`, with a free-form name/description | Alternative to `ff` |
| `/opsx:explore` | Investigation mode, no artifacts created | Before deciding whether a change is worth opening |
| `/opsx:apply` | Runs the schema's `apply` block: pre-flight → worktree → subagent-driven TDD → `apply.md` | Once `plan.md` exists |
| `/opsx:verify` | Runs the 7 checks and writes `verify.md` | After every `/opsx:apply` |
| `/opsx:sync` | Copies the change's delta specs into `openspec/specs/` without archiving | Rare; `archive` already does it |
| `/opsx:archive` | Syncs specs + moves the change to `openspec/changes/archive/` | After `finalize.md` |
| `/opsx:bulk-archive` | Archives several changes | Cleanup |
| `/opsx:update` | Updates an existing change (specs/tasks) when scope shifted | Change of direction mid-way |

Always use the slash command; the matching skill (`openspec-continue-change`, etc.) is invoked by it.

### `openspec` CLI (in the terminal)

```bash
openspec status --change <name>        # what is done, what is blocked, what comes next
openspec list                          # open changes
openspec list --specs                  # capabilities already specified
openspec validate --all                # validates changes + specs (verify runs this)
openspec show <name>                   # shows a change or spec
openspec view                          # interactive dashboard
openspec instructions <artifact> --change <name>   # the exact instruction the agent will receive
```

---

## 3. Superpowers skills and where each one fires

| Skill | Invoked by | Moment |
|---|---|---|
| `superpowers:brainstorming` | `brainstorm` artifact | 1st `/opsx:continue` — guided conversation, one question at a time, 2-3 alternatives |
| `superpowers:writing-plans` | `plan` artifact | breaks `tasks.md` into TDD micro-steps with file paths and test commands |
| `superpowers:using-git-worktrees` | `/opsx:apply` step 1 | isolated worktree per change (native tool on Claude Code) |
| `superpowers:subagent-driven-development` | `/opsx:apply` step 2 | fresh subagent per task; **activates on its own** `test-driven-development` and `requesting-code-review` |
| `superpowers:test-driven-development` | transitive | RED → GREEN → REFACTOR on every task; code written before a failing test is deleted |
| `superpowers:requesting-code-review` | transitive | subagent review after each task + final whole-branch review |
| `superpowers:finishing-a-development-branch` | `finalize` artifact | runs tests, detects the worktree, menu: merge locally / PR / keep |
| `superpowers:systematic-debugging` | you, outside the schema | bug fixes — no change is opened |
| `superpowers:verification-before-completion` | always | before declaring anything "done" |

Skills the schema does **not** use: `executing-plans` (does not transitively activate TDD/review) and `dispatching-parallel-agents` (SDD already parallelizes what it can).

---

## 4. Path of a new feature (step by step)

Example: "communities can publish products in the store".

### Phase 0 — Is a change worth opening?

Open a change for: new capability, behavior/contract change, breaking change, architecture, new external dependency, DB schema, cross-system integration.
Don't (branch + direct PR; Superpowers skills still apply): bug fix, typo, config, docs, non-breaking dependency bump.

When unsure: `/opsx:explore` to investigate first.

### Phase 1 — Planning (on the integration branch or a feature branch)

```text
git checkout -b feat/community-store          # recommended; the apply worktree branches off here

/opsx:new community-store
  → creates openspec/changes/community-store/

/opsx:continue                                 # → brainstorm.md
  skill: superpowers:brainstorming
  the agent explores the code, asks questions (one at a time), proposes 2-3 approaches,
  you pick one; it writes the raw capture to brainstorm.md and STOPS
  (it does not jump to writing-plans — the schema instructs it not to)

/opsx:continue                                 # → proposal.md
  Why / What Changes / Capabilities (new + modified) / Impact — 1-2 pages
  ⚠ the Capabilities list is the contract with the specs; review it here

/opsx:continue                                 # → design.md
  only if: cross-cutting, new dependency, migration, security/performance
  otherwise the agent writes "Not needed: <reason>" and moves on

/opsx:continue                                 # → specs/<capability>/spec.md
  one per capability; ADDED / MODIFIED / REMOVED; Requirement + Scenario (WHEN/THEN)
  ⚠ every Scenario becomes a test; review carefully — this is what TDD will implement

/opsx:continue                                 # → tasks.md
  coarse checklist "- [ ] 1.1 ... and verify that <test> passes"

/opsx:continue                                 # → plan.md
  skill: superpowers:writing-plans
  2-5 minute micro-steps per task, with file, snippet and test command;
  each ## Task cites the X.Y ids from tasks.md

git add openspec/changes/community-store && git commit -m "docs(openspec): scaffold community-store"
  (apply does this for you if you forget — step 0c)
```

Shortcut for a small change: `/opsx:ff community-store` runs all the steps above without stopping; review the result before apply.

### Phase 2 — Implementation

```text
/opsx:apply
  0. pre-flight: checks skills, tooling (npm/pytest/docker…), commits the change directory,
     records the current branch
  1. superpowers:using-git-worktrees → worktree + branch "community-store"
  2. superpowers:subagent-driven-development
       for each task in plan.md: a subagent implements with TDD → a subagent reviews (spec + quality)
       → fix rounds if needed → flips "- [x]" in tasks.md → commit
       final review of the whole branch
  3. writes apply.md (iteration 1, commit range, X of Y tasks)

/opsx:verify                                    # → verify.md
  1 validate --all   2 all tasks [x]   3 specs synced?   4 design ↔ specs
  5 everything committed   6 anything leaked into docs/superpowers/?   7 manual checks have a test equivalent?
  PASS → move on      FAIL → /opsx:apply again (iteration 2), then /opsx:verify
  more than 5 iterations → stop and talk
```

### Phase 3 — Closing

```text
/opsx:continue                                  # → retrospective.md
  evidence (commits, diff, tasks, iterations) → wins / misses / plan deviations /
  skills used or skipped (with justification) / surprises / memory candidates
  trivial change: one line "Skipped: <reason>"

/opsx:continue                                  # → finalize.md
  skill: superpowers:finishing-a-development-branch
  runs the test suite → menu:
    1. merge locally into the base branch
    2. push + Pull Request
    3. keep the branch as-is
  records the outcome in finalize.md

/opsx:archive
  syncs the change's specs/<capability>/spec.md into openspec/specs/
  moves the directory to openspec/changes/archive/YYYY-MM-DD-community-store/
  - chose merge locally → run on the base branch
  - chose PR → run on the feature branch, push, and the PR merge lands code + archive together
```

### One-screen summary

```text
/opsx:new X → /opsx:continue ×6 (brainstorm, proposal, design, specs, tasks, plan)
→ /opsx:apply → /opsx:verify (repeat until PASS)
→ /opsx:continue ×2 (retrospective, finalize) → /opsx:archive
```

---

## 5. Alternative paths

| Situation | Path |
|---|---|
| Bug fix | `git checkout -b fix/…` → `superpowers:systematic-debugging` → TDD → PR. No change. |
| Large refactor with no behavior change | Normal `/opsx:new` plus `skip_specs: true` in the change's `.openspec.yaml` (no delta specs) |
| Scope changed mid-apply | `/opsx:update` to adjust specs/tasks → commit in the worktree → `/opsx:apply` (iteration N+1) |
| Plain OpenSpec flow for one change | `/opsx:new quick-fix --schema spec-driven` |
| Lost context (compaction) | `openspec status --change X` says where you stopped; the ledger at `.superpowers/sdd/plan/progress.md` in the worktree says which tasks are done |
| Test the schema without side effects | `openspec new change smoke && openspec status --change smoke && rm -rf openspec/changes/smoke` |

---

## 6. Signs something went off the rails

- A file appeared under `docs/superpowers/specs/` or `docs/superpowers/plans/` → a skill wrote outside the change (verify check 6 flags it). Move the content into the change and delete it.
- `/opsx:continue` offered `verify` before you ran `apply` → it shouldn't; `verify` requires `apply.md`. Report it as a schema bug.
- The agent started implementing without a worktree or on `main` → stop; `/opsx:apply` step 1 is mandatory.
- Brainstorm jumped straight to a plan → the schema says to stop after `brainstorm.md`; ask for `/opsx:continue` to get the proposal.
