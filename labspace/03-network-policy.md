# Network Policy

By default an agent in a sandbox can talk to the whole internet. That's fine
when you trust the task — but a prompt-injected agent that can `curl` anywhere
is a data-exfiltration risk waiting to happen. `sbx` ships three network
policies you can pick from:

| Policy | What's allowed | When to use it |
|--------|----------------|----------------|
| `open` | Everything | Quick local experiments, fully trusted tasks |
| `balanced` *(default)* | Package managers + a curated allowlist (npm, PyPI, GitHub, etc.) | Most agent work — installs deps, clones repos, but blocks random egress |
| `locked-down` | Nothing | Reviewing untrusted code, processing sensitive data, paranoid mode |

You pick a policy per sandbox with `--network`.

## Try `open`

Full internet — anything goes.

```bash
sbx exec --network open -- curl -s -o /dev/null -w "HTTP %{http_code}\n" https://example.com
```

You should see `HTTP 200`. Now try a domain that *isn't* on the balanced
allowlist:

```bash
sbx exec --network open -- curl -s -o /dev/null -w "HTTP %{http_code}\n" https://httpbin.org/get
```

Also `HTTP 200` — because `open` allows everything.

## Try `balanced`

Package managers still work. Random domains don't.

1. Install a package — should succeed because PyPI is allowlisted:

    ```bash
    sbx exec --network balanced -- pip install --quiet cowsay
    ```

2. Now try the same off-allowlist domain from above. It should fail or time out:

    ```bash
    sbx exec --network balanced -- curl -s -o /dev/null -w "HTTP %{http_code}\n" --max-time 5 https://httpbin.org/get || echo "blocked — as expected"
    ```

3. GitHub is on the allowlist, so a clone works:

    ```bash
    sbx exec --network balanced -- git ls-remote https://github.com/docker/docs HEAD
    ```

> [!TIP]
> `balanced` is the right default for most agent tasks. It lets the agent
> install dependencies and pull source, but stops it from quietly POSTing
> your data to a domain it picked up from a prompt.

## Try `locked-down`

No network at all. Use this when you don't trust the input.

1. Confirm DNS itself is blocked:

    ```bash
    sbx exec --network locked-down -- curl -s -o /dev/null -w "HTTP %{http_code}\n" --max-time 5 https://example.com || echo "blocked — no network"
    ```

2. Local-only work still runs fine — the sandbox can still use its
   filesystem and CPU:

    ```bash
    sbx exec --network locked-down -- python3 -c "print(sum(range(1000000)))"
    ```

## Choose per task

A realistic agent loop might be:

1. **Plan** in a `locked-down` sandbox — agent reads code, no internet.
2. **Implement** in a `balanced` sandbox — agent can install deps.
3. **Smoke test** in `balanced` — run the test suite.
4. Only escalate to `open` when you have a deliberate reason.

> [!IMPORTANT]
> Network policy is per-sandbox, set at launch. You can't change it on a
> running sandbox. If you need a different policy, exit and start a new one.

Clean up before moving on:

```bash
sbx rm --all
```

Next: handing secrets to a sandbox without ever showing them to the agent.
