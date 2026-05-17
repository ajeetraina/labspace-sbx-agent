# Branch Mode with Git Worktrees

When you run several agents on the same project, they'll fight over files.
One edits `calculator.py`, the other reverts it, neither knows what happened.
**Branch mode** solves this with **git worktrees** — each sandbox gets its
own checkout, on its own branch, in its own directory. They share the same
repo history but never collide on the filesystem.

## Set up a git repo

The sample project isn't a git repo yet. Make it one.

1. Initialize and make the first commit:

    ```bash
    git init -q -b main && git add . && git -c user.email=you@example.com -c user.name=you commit -q -m "initial commit"
    ```

2. Confirm:

    ```bash
    git log --oneline
    ```

## Launch a branch-mode sandbox

`sbx branch <name>` creates a git worktree for the named branch in
`.sbx/branches/<name>` and starts a sandbox rooted there. Anything the
sandbox edits stays on that branch.

```bash
sbx branch fix-bug --exec "pytest -v || true"
```

That ran the tests on a brand-new `fix-bug` branch in its own worktree.

Have a look at what got created:

```bash
ls -la .sbx/branches/
```

You'll see one directory per active branch. Your main checkout is untouched.

## Edit on the branch, not on main

Open a shell in the branch-mode sandbox — you're inside the worktree, so
edits land on `fix-bug`, not `main`. (Try this in your terminal.)

```bash no-run-button
sbx branch fix-bug
```

Inside the sandbox, fix the bug:

```bash no-run-button
sed -i 's/a - b/a + b/' calculator.py
pytest -v
git add calculator.py
git -c user.email=agent@example.com -c user.name=agent commit -m "fix: add() should add"
exit
```

Back on your workspace, confirm `main` is untouched:

```bash
git log --oneline main
```

And the branch has the fix:

```bash
git -C .sbx/branches/fix-bug log --oneline
```

## Merge when you're happy

Once a branch passes review, fast-forward `main`:

```bash
git fetch .sbx/branches/fix-bug fix-bug:fix-bug && git merge --ff-only fix-bug
```

```bash
git log --oneline
```

> [!TIP]
> Treat each agent like a junior developer with their own feature branch.
> You review, you merge. The worktree gives you full `git diff` between
> what they did and `main` — no surprises.

## Tear down a branch

When you're done with a branch, remove its worktree:

```bash
git worktree remove .sbx/branches/fix-bug --force || true
git branch -D fix-bug || true
```

Next: running several of these branches at the same time.
