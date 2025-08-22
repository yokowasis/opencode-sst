# Syncing Your Fork with Upstream

This repository tracks the upstream project at: https://github.com/sst/opencode (branch: `dev`).

Your fork can drift in two ways:

1. Upstream has new commits you don't have (you are "behind").
2. You have local commits not yet in upstream (you are "ahead").

## Typical Workflow (One-Time Setup + Repeated Sync)

### 1. One-time: add the upstream remote

```bash
git remote add upstream https://github.com/sst/opencode.git
```

### 2. Fetch latest refs (does not modify working tree)

```bash
git fetch upstream
```

### 3. Update your local tracking branch of upstream/dev

```bash
git checkout dev
git fetch upstream
```

### 4. Merge or rebase upstream changes

**Option A (merge, preserves history):**

```bash
git merge upstream/dev
```

**Option B (rebase, linear history):**

```bash
git rebase upstream/dev
```

### 5. Resolve conflicts if prompted

- Edit conflicted files
- `git add <file>` ...
- Continue:
  - For merge: `git commit`
  - For rebase: `git rebase --continue`

When a conflict involves a file that was deleted locally but modified upstream (modify/delete conflict), prefer keeping the local deletion for files such as workflow or CI configs that are intentionally removed in the fork. To accept the local deletion and finish the merge, run:

```bash
git rm <path-to-file>                # remove upstream-modified file from the index
git commit                           # finish the merge
```

(If you instead want to restore the upstream version, add the file and commit.)

### 6. Push updated dev back to your fork

```bash
git push origin dev
```

## Keeping Feature Branches Updated

If working on a feature branch (e.g. feature/x) branched from dev:

```bash
git checkout dev
git fetch upstream
git merge upstream/dev   # or git rebase upstream/dev
git checkout feature/x
git merge dev            # or git rebase dev
```

## Fast-Forward Only Strategy

If you prefer a clean linear history and have no local divergent commits:

```bash
git fetch upstream
git checkout dev
git reset --hard upstream/dev
git push origin dev --force-with-lease
```

## Checking Divergence

```bash
git fetch upstream
git status
git rev-list --left-right --count upstream/dev...dev
```

The output `A	B` means: A commits only upstream, B commits only local.

## Viewing Commit Differences

```bash
git log --oneline --decorate --graph --boundary upstream/dev...dev
```

## Pushing After a Rebase

If you rebased your local `dev` and remote has the old history, a regular push will be rejected. Use:

```bash
git push origin dev --force-with-lease
```

`--force-with-lease` protects against overwriting someone else's newer work.

### Safe Force Push Pattern

```bash
git fetch origin dev
git push origin dev --force-with-lease
```

If push is still rejected, re-check divergence:

```bash
git rev-list --left-right --count origin/dev...dev
```

And refetch before retrying.

## Avoiding Common Pitfalls

1. Always fetch before merge/rebase.
2. Prefer --force-with-lease over --force when rewriting history.
3. Do not rebase public branches collaborators already pulled unless agreed.
4. Resolve conflicts carefully; run tests after conflict resolution.

## Using Merge vs Rebase

**Use merge when:**

- You want to preserve exact historical context.

**Use rebase when:**

- You want a linear, bisect-friendly history.

## Aborting Operations

```bash
git merge --abort     # Abort an in-progress merge
git rebase --abort    # Abort an in-progress rebase
```

## Troubleshooting

**Detached HEAD after fetching?**

```bash
git checkout dev
```

**Accidentally committed to dev instead of a feature branch?**

```bash
git checkout -b feature/fix-stuff
git reset --hard HEAD~1  # (IF you want to remove the commit from dev)
git checkout dev
git merge feature/fix-stuff  # (later when ready)
```

**Need to sync fork on GitHub web UI only?**

- Use: GitHub -> Your fork -> Sync fork (if available) OR run the commands locally above.

## Quick Commands

**Minimal One-Liner (merge strategy)**

```bash
git fetch upstream && git checkout dev && git merge upstream/dev && git push origin dev
```

**Minimal One-Liner (rebase strategy)**

```bash
git fetch upstream && git checkout dev && git rebase upstream/dev && git push origin dev --force-with-lease
```

## License Note

All contributions remain under the repository's existing license.
