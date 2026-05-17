# Your First Sandbox

A sandbox is an ephemeral container with your project mounted inside it. Two
ways to use one:

- `sbx shell` — drop into an interactive shell. Good for exploring.
- `sbx exec <cmd>` — run one command and exit. Good for scripting and for
  agents that just need to run a build or a test.

Most of this lab uses `sbx exec` because the buttons on the right have to
return — but you'll try `sbx shell` first to feel how it works.

## Look at what you're starting with

Before you sandbox anything, take a look at the sample project you'll be
working with. It's a tiny Python calculator with a failing test — perfect
fodder for an AI agent.

Open :fileLink[calculator.py]{path="calculator.py"} and
:fileLink[test_calculator.py]{path="test_calculator.py"} in the editor.

```bash
ls
```

## Launch a shell sandbox

This one is for you to try yourself in the terminal — the interactive shell
would block the Run button.

```bash no-run-button
sbx shell
```

Inside the sandbox try a few things, then `exit` when you're done:

```bash no-run-button
whoami
hostname
ls
cat /etc/os-release
exit
```

Notice the hostname is something like `sbx-abc123`. You were in a fresh
container the whole time, with your project mounted read-write but isolated
from the host workspace's other state.

## Run one command, no shell

For the rest of the lab you'll use `sbx exec` so the Run buttons work. Same
sandbox semantics, just non-interactive.

1. Run a command inside a sandbox:

    ```bash
    sbx exec -- ls -la
    ```

2. Show the kernel and OS — proving you're inside a container, not the
   workspace host:

    ```bash
    sbx exec -- uname -a
    ```

3. Install something *inside* the sandbox and confirm it doesn't leak out.
   First, install Python's `pytest` in a sandbox and run the test suite:

    ```bash
    sbx exec -- sh -c "pip install --quiet pytest && pytest -v"
    ```

   You should see one failing test. (That's the bug you'll fix later.)

4. Now check that `pytest` is *not* installed in your workspace — the install
   only existed inside the sandbox:

    ```bash
    which pytest || echo "pytest not on workspace — confirmed isolated"
    ```

## See what's running

List active sandboxes:

```bash
sbx ls
```

If any are still around, clean them up:

```bash
sbx rm --all
```

> [!TIP]
> Sandboxes are cheap. Spin one up for every task, throw it away when done.
> Treat them like browser tabs, not like pet servers.

Next up: locking down what the sandbox can reach on the network.
