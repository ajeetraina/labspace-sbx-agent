# Welcome to Docker Sandboxes

👋 AI agents need a place to run code. Giving an agent your laptop shell is a
bad idea — one wrong `rm` and your morning disappears. **Docker Sandboxes**
(the `sbx` CLI) give agents a fresh, throwaway container with a controlled
network and a safe credential proxy. You stay in charge; the agent gets just
enough room to work.

In this lab you will:

- ✅ Install and verify the `sbx` CLI
- 🐚 Launch your first shell sandbox and run commands inside it
- 🌐 Switch between **open**, **balanced**, and **locked-down** network policies
- 🔑 Hand a sandbox a credential through the proxy without leaking the value
- 🌿 Use **branch mode** with git worktrees to give each agent its own checkout
- 🤖 Run a fleet of sandboxes in parallel — multiple agents, one workspace

## Verify the environment

First make sure the workspace can reach Docker. If the next command prints
container output, you're good to go.

```bash
docker run --rm hello-world
```

## Install the sbx CLI

The `sbx` binary is the entry point for everything you'll do in this lab.
Install it with the official installer:

```bash
curl -fsSL https://sandboxes.docker.com/install.sh | sh
```

> [!NOTE]
> The installer drops the binary into `/usr/local/bin/sbx`. If it asks for a
> password, run with `sudo sh` instead — but in this lab's workspace
> container it should install without prompting.

Confirm the install:

```bash
sbx version
```

You should see a version string like `sbx 0.x.x`. Now look at the top-level
help so you know what's in the toolbox:

```bash
sbx --help
```

Take a moment to scan the subcommands — `shell`, `exec`, `secret`, `branch`,
`ls`, `rm`. You'll meet each of them in the next five sections.

> [!TIP]
> A sandbox is just a container under the hood. You can always inspect what's
> running with `docker ps` if you're curious. The `sbx` CLI is the safe,
> opinionated front door; `docker` is the back door.

Ready? Head to the next section to launch your first sandbox.
