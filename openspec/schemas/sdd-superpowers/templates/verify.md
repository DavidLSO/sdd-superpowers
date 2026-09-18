# Verification Report

> Produced by `openspec-verify-change` after apply. Overwritten each run.

**Change**: `<change-name>`
**Verified at**: `YYYY-MM-DDTHH:mm`
**Iteration**: `<copy from apply.md>`
**Verifier**: `<agent>`

## 1. Structural validation (`openspec validate --all --json`)

- [ ] All items `"valid": true`

| Item | Type | Issues |
|---|---|---|
| — | — | — |

## 2. Task completion (`tasks.md`)

- [ ] Every checkbox is `- [x]`

| Task | Reason incomplete | Blocks archive? |
|---|---|---|
| — | — | — |

## 3. Delta spec sync state

| Capability | Status (✓ synced / ✗ needs sync / N/A) | Notes |
|---|---|---|
| — | — | — |

## 4. Design / specs coherence (N/A if no design.md)

| design.md decision | specs counterpart | Gap |
|---|---|---|
| — | — | — |

**Drift warnings** (non-blocking): <list or "None">

## 5. Implementation signal

- [ ] No unstaged files in the worktree
- **Commit range**: `<base>..<head>` (`<n>` commits)

## 6. Front-door routing leak

- [ ] No files under `docs/superpowers/specs/` or `docs/superpowers/plans/`

<list leaked files or "None">

## 7. Deferred checks coverage

| Deferred item | Automated equivalent | Gap? |
|---|---|---|
| — | — | — |

## Overall decision

- [ ] ✅ PASS — `/opsx:continue` → retrospective → finalize → `/opsx:archive`
- [ ] ⚠️ PASS WITH WARNINGS — proceed; note: `<explanation>`
- [ ] ❌ FAIL — fix (code → `/opsx:apply`; artifact → fix then `/opsx:apply`), re-run verify

**Next step**: <action>
