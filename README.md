# sdd-superpowers

Custom [OpenSpec](https://github.com/Fission-AI/OpenSpec) schema that wires the spec-driven artifact pipeline into the execution skills of [Superpowers](https://github.com/obra/superpowers) (brainstorming, writing-plans, worktrees, subagent-driven TDD, code review, finishing-a-development-branch).

It is a hybrid of two MIT projects, taking the best of each:

| From | What |
|---|---|
| [danielhanold/superspec](https://github.com/danielhanold/superspec) v4 | Workflow mechanics: pre-flight commit of the artifacts before the worktree is created, `apply.md` as a receipt (verify only after apply exists), optional `design.md`, apply → verify convergence loop with an iteration counter |
| [JiangWay/openspec-schemas](https://github.com/JiangWay/openspec-schemas) `superpowers-bridge` v1 | Skill PRECHECK before every invocation, `brainstorm.md` as raw capture and `design.md` as its restructuring, `retrospective` artifact (evidence first, memory candidates), "when NOT to open a change" guidance, `CLAUDE.md` fragment |
| This repo | `finalize` invokes `superpowers:finishing-a-development-branch` instead of embedding 350 lines of bash; base branch resolved without depending on `origin/main`; templates 100% in English; aligned with OpenSpec 1.13 (`skip_specs`, `[x]` checkbox rule, `## Purpose` on new capabilities) |

Validated with `openspec schema validate` on OpenSpec **1.13.1**, written against Superpowers **6.3.0**.

## Flow

```text
PLANNING
  brainstorm.md ──┬─→ proposal.md ──→ specs/**/*.md ──→ tasks.md ──→ plan.md
                  └─→ design.md (optional; "Not needed: <reason>" to skip)

APPLY  (/opsx:apply — requires: plan, tracks: tasks.md)
  0. pre-flight: skills, tooling, commit of openspec/changes/<name>/
  1. superpowers:using-git-worktrees
  2. superpowers:subagent-driven-development (+ TDD + code-review, transitive)
  3. apply.md (receipt, iteration N)
  4. /opsx:verify → verify.md ──FAIL──→ back to step 2

CLOSING
  retrospective.md → finalize.md (superpowers:finishing-a-development-branch) → /opsx:archive
```

Each artifact becomes available only when the previous one exists; `verify` requires `apply.md`, so it is impossible to verify before implementing.

## Installation

Prerequisites: OpenSpec ≥ 1.13 and the Superpowers plugin installed in your harness (Claude Code: `claude plugin install superpowers@claude-plugins-official`).

```bash
# once per machine: the "core" profile installs only 4 /opsx commands; the schema needs new/continue/ff/verify
openspec config set profile custom
openspec config set workflows '["propose","explore","new","continue","apply","ff","sync","archive","bulk-archive","verify","update"]'

# at the root of the project that will use the schema
openspec init                       # if there is no openspec/ yet (or `openspec update --force` to regenerate the commands)
mkdir -p openspec/schemas
cp -r /path/to/sdd-superpowers/openspec/schemas/sdd-superpowers openspec/schemas/
echo "schema: sdd-superpowers" > openspec/config.yaml   # or use --schema per change

openspec schema validate sdd-superpowers
openspec schemas                    # sdd-superpowers must be listed
```

Optional but recommended: append `openspec/schemas/sdd-superpowers/templates/adopters/CLAUDE.md.fragment.md` to the project's `CLAUDE.md`. It teaches the agent *when* to open a change (feature, contract, architecture) and when to go straight to a PR (bug fix, typo, config).

## Usage

> Full command reference and the step-by-step path of a feature: [docs/commands.md](docs/commands.md).

Inside the harness (Claude Code etc.), with the `/opsx:*` commands OpenSpec installs:

```text
/opsx:new my-feature      # creates the change
/opsx:continue            # → brainstorm (skill-guided conversation)
/opsx:continue            # → proposal
/opsx:continue            # → design (or "Not needed: ...")
/opsx:continue            # → specs
/opsx:continue            # → tasks
/opsx:continue            # → plan (writing-plans)
/opsx:apply               # worktree + subagent-driven TDD → apply.md
/opsx:verify              # 7 checks → verify.md
/opsx:continue            # → retrospective
/opsx:continue            # → finalize (menu: merge locally / PR / keep)
/opsx:archive             # syncs delta specs and archives the change
```

For small, well-understood changes, `/opsx:ff my-feature` produces all planning artifacts at once; from `/opsx:apply` on it is the same.

To bypass the schema for one change: `/opsx:new quick-fix --schema spec-driven`.

## Design decisions

- **`design.md` optional but always "complete"**: when it is not needed, the agent writes one line (`Not needed: <reason>`). The graph never has a permanently pending artifact and `/opsx:continue` moves linearly.
- **`verify` requires `apply`, not `plan`**: the bridge declares `requires: [plan]` and patches it with prose; here the `apply.md` receipt solves it structurally.
- **Pre-flight commit before the worktree**: fixes bridge bug [#6](https://github.com/JiangWay/openspec-schemas/issues/6) (untracked artifacts do not exist inside the worktree).
- **Base branch without `origin/main`**: fixes bridge bug [#14](https://github.com/JiangWay/openspec-schemas/issues/14); resolves via `origin/HEAD`, then local `main`/`master`/`develop`, then the root commit.
- **`finalize` delegates to the skill**: `finishing-a-development-branch` already does verify tests → detect worktree → menu → merge/PR → cleanup. Re-implementing that in YAML (as superspec v4 does) is brittle and assumes GitHub + `gh` + a fixed branch topology.
- **Retrospective before finalize**: it lands in the same PR as the code, written while context is still hot.
- **No `executing-plans` fallback**: it does not transitively activate TDD or code review; without subagents, use the built-in `spec-driven` schema.

## Note: subagent-driven-development ledger (Superpowers ≥ 6.x)

The skill stores progress at `.superpowers/sdd/<plan-basename>/progress.md` in the worktree root, with the plan's full path on the first line. Since every plan in this schema is named `plan.md`, the directory is always `.superpowers/sdd/plan/`; the skill tells changes apart by the path recorded in the ledger but never deletes another plan's ledger. Always run `/opsx:apply` in one worktree per change (apply step 1 instructs the agent to clear a ledger belonging to another change, if any).

## License

MIT. Derived from [danielhanold/superspec](https://github.com/danielhanold/superspec) (MIT © 2026 Daniel Hanold) and [JiangWay/openspec-schemas](https://github.com/JiangWay/openspec-schemas) (MIT). The `specs`/`tasks`/`design` instructions are adapted from OpenSpec's `spec-driven` schema (MIT).
