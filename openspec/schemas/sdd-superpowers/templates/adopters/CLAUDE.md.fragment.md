<!-- Source: openspec/schemas/sdd-superpowers/templates/adopters/CLAUDE.md.fragment.md -->
<!-- Append this section to your project's CLAUDE.md so the agent routes work through the schema. -->

## Workflow routing (read on session start)

This repo uses the `sdd-superpowers` OpenSpec schema (`openspec/schemas/sdd-superpowers/`) to bridge OpenSpec (what) and Superpowers (how). The schema's artifact instructions are authoritative for each phase; this section only decides *whether* and *when* to enter it.

### Entry routing

| Trigger | What to do |
|---|---|
| User starts a narrative "let's discuss / brainstorm X" | Run `superpowers:brainstorming` verbally, but **do NOT** write to `docs/superpowers/specs/`. When the 5 promotion criteria below hold, suggest `/opsx:new <name>` — wait for the user's ack |
| User invokes `/opsx:new`, `/opsx:ff`, `/opsx:propose` | Follow the schema flow; each artifact's instruction says which skill to invoke |
| User is mid-change | Advance with `/opsx:continue`, `/opsx:apply`, `/opsx:verify`, `/opsx:archive` |
| Bug fix / typo / config tweak / docs / non-breaking dep bump | Direct branch + PR — **do NOT** open a change. Superpowers skills (TDD, systematic-debugging) still apply |

### When to open a change

Process ceremony scales with risk. Open a change for: new capability, behavior/contract change, breaking change, architecture change, new external dependency, DB schema change, cross-system integration. Everything else is a direct PR. For a pure refactor that still deserves a change, set `skip_specs: true` in the change's `.openspec.yaml`.

### Brainstorm → change promotion criteria (all 5 must hold)

1. **Scope locked** — one sentence says what is in / out
2. **Major design forks resolved** — alternatives weighed, one chosen; remaining TBDs have an owner and impact statement
3. **Cross-system dependencies mapped** — each is ready / mockable / genuinely unknown
4. **Acceptance criteria stateable** — concrete pass conditions (test command + N deliverables)
5. **Conversation converging** — recent turns are confirmations, not new alternatives

### Anti-patterns

- Letting `brainstorming` write to `docs/superpowers/specs/` or `writing-plans` to `docs/superpowers/plans/` (verify check 6 flags this)
- Running `/opsx:apply` outside a worktree
- Promoting to a change with unresolved blocking TBDs
- Opening a change for a bug fix or typo
