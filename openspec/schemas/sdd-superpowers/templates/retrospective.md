# Retrospective: <change-name>

> Written: <YYYY-MM-DD> (after verify passed)
> Commit range: `<base-sha>..<head-sha>`
> Worktree: <path or "merged">

<!-- Trivial single-commit change? Replace this whole file with one line:
     "Skipped: <reason>" -->

## 0. Evidence

<!-- Quantitative front-matter. Later sections cite these instead of
     repeating [evidence: ...] on every line. Should be reconstructible
     later from `git log` + tasks.md + apply.md alone. -->

- **Commit range**: `<base-sha>..<head-sha>` (<n> commits)
- **Diff size**: <+X / -Y lines across N files>
- **Tasks done**: <x>/<y>
- **Apply iterations**: <n> (from apply.md)
- **Subagent dispatches**: <count or "n/a">
- **New external dependencies**: <name, version, license — or "none">
- **OpenSpec validate at verify**: <pass / fail>
- **Test signal**: <test count / coverage % / "n/a">

Commit chain:

```
<base-sha> <one-line summary>
...
<head-sha> <one-line summary>
```

## 1. Wins

- [evidence: <commit/file/test>] <description>

## 2. Misses

- 🔴 [blocking | evidence: ...] <description>
- 🟡 [painful  | evidence: ...] <description>
- 📌 [nit      | evidence: ...] <description>

## 3. Plan deviations

| Plan task | What changed | Why |
|-----------|--------------|-----|
| 1.2       | ...          | ... |

## 4. Skill / workflow compliance

| Skill | Used |
|---|---|
| superpowers:brainstorming | |
| superpowers:writing-plans | |
| superpowers:using-git-worktrees | |
| superpowers:subagent-driven-development | |
| (transitive) superpowers:test-driven-development | |
| (transitive) superpowers:requesting-code-review | |
| superpowers:finishing-a-development-branch | |

<!-- Default expectation: all ✓. Every ✗ must be explained below. -->

### Deliberately Skipped Skills

<!-- Empty (all green) is the expected state. Per skipped skill: -->

- **`<skill name>`**
  - **What was skipped**: <whole skill or which sub-step>
  - **Why this cycle**: <concrete trigger: commit / log line / observed behavior. Not "not needed", "too small", "no time">
  - **How to prevent recurrence**: <one of: schema graph fix (which section) / skill description tightening (which skill) / CLAUDE.md trigger (which rule) / scope-judgment rule / one-off — schema boundary case (state why it is a boundary)>

## 5. Surprises

- <assumption that turned out wrong>

## 6. Promote candidates → long-term learning

<!-- `- [ ]` checklist. Unchecked items carry forward to the next cycle's retro.
     Carry-forward: grep -A 5 '^- \[ \]' openspec/changes/archive/*/retrospective.md -->

- [ ] 🔴 **<short rule>** → **Promote to memory** (type: feedback)
  > **Why**: <past incident or strong preference that motivated this rule>
  > **How to apply**: <file / phase / decision moment where this kicks in>

- [ ] 🟡 **<candidate>** → **Promote to project CLAUDE.md** (`<section>`)
  > **Why**: ...
  > **How to apply**: ...

- [ ] 📌 **<candidate>** → **One-off** (record only)
  > **Why**: <why it does not generalize>
