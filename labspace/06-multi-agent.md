# Multi-Agent Fleets

You've seen one sandbox at a time. The real payoff of `sbx` is running
**many at once** — each on its own branch, each with its own network policy
and secrets — and then picking the best result. This is how agent fleets,
parallel refactors, and "try three approaches and compare" workflows are
built.

## Three agents, one task

The task: try three different fixes for a hypothetical bug, run the test
suite for each, see which passes. In real life each branch would be driven
by an actual model — here you'll just run `pytest` to simulate the agent
loop.

1. Make sure you're starting clean:

    ```bash
    sbx rm --all && git worktree prune
    ```

2. Kick off three branch-mode sandboxes in parallel. The `&` runs each in
   the background; `wait` blocks until all three finish:

    ```bash
    sbx branch attempt-1 --exec "pytest -v" &
    sbx branch attempt-2 --exec "pytest -v" &
    sbx branch attempt-3 --exec "pytest -v" &
    wait
    echo "✅ all three attempts finished"
    ```

3. See the three worktrees side by side:

    ```bash
    ls .sbx/branches/
    git branch
    ```

Each branch is independent. Agent #1 can edit `calculator.py` while agent #2
edits `test_calculator.py` and they will never see each other's work until
you choose to merge.

## Different policy per agent

A common pattern: most agents work in `balanced`, but one runs `locked-down`
to handle data you don't want phoned home. You set policy per sandbox.

```bash
sbx branch trusted --network balanced --exec "echo balanced agent" &
sbx branch untrusted --network locked-down --exec "echo locked-down agent" &
wait
```

## Watch them with `sbx ls`

While agents are running, you can list active sandboxes from another
terminal:

```bash
sbx ls
```

You can also kill a runaway agent by id:

```bash no-run-button
sbx rm <sandbox-id>
```

## Pick a winner, throw the rest away

Once the fleet finishes you'll usually keep one branch and discard the
others. The pattern:

```bash
# Keep `attempt-1`, drop the rest
for b in attempt-2 attempt-3 trusted untrusted; do
  git worktree remove ".sbx/branches/$b" --force 2>/dev/null || true
  git branch -D "$b" 2>/dev/null || true
done
git branch
```

Then merge the winner:

```bash no-run-button
git merge --ff-only attempt-1
```

## What you've learned

You can now:

- ✅ Install and verify the `sbx` CLI
- 🐚 Launch sandboxes with `sbx shell` and `sbx exec`
- 🌐 Choose between `open`, `balanced`, and `locked-down` per task
- 🔑 Hand agents credentials through the proxy without leaking them
- 🌿 Use `sbx branch` and git worktrees so agents don't clobber each other
- 🤖 Run a fleet of sandboxes in parallel and merge the winner

## Where to go next

- 📚 Read the [Docker Sandboxes docs](https://docs.docker.com/sandboxes/)
- 🧪 Wire `sbx exec` into your own agent framework — anywhere you'd
  shell out to run untrusted code is a place a sandbox belongs
- 🛡 Tighten your defaults: make `balanced` the floor and only opt up
  when a task truly needs it

🎉 Nice work — go build something with agents you can actually trust.
