# Finalize Receipt

> Written after superpowers:finishing-a-development-branch returns.
> Overwritten if finalize is re-entered (e.g. kept-as-is → PR later).

**Change**: `<change-name>`
**Finalized at**: `YYYY-MM-DDTHH:mm`
**Outcome**: `merge-locally` | `pr-created` | `kept-as-is` | `discarded`

## Branch state

- **Worktree branch**: `<change-name>`
- **Base branch**: `<main / feature branch>`
- **Final state**: `merged` | `pr-open` | `kept-open` | `deleted`
- **PR URL**: `<url or N/A>`

## Workspace

- **Worktree**: `<path or "removed">`
- **Cleanup**: `removed` | `preserved` | `N/A`

## Tests

- **Status at finish**: `passing` | `N/A (no test runner)`

## Next step

- `merge-locally` → run `/opsx:archive` on `<base branch>`.
- `pr-created` → after review approval, run `/opsx:archive` on the feature branch, push, then merge the PR.
- `kept-as-is` → resume later; re-enter finalize when ready.
- `discarded` → no further action.
