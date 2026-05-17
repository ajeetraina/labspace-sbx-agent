# Credential Proxy & Secrets

Agents often need credentials — a GitHub token to push a branch, an API key
to call a service. The naïve approach is to set an env var: `OPENAI_API_KEY=sk-...`
and pass it through. Now the agent can read the literal value, paste it into
a log, exfiltrate it via a prompt injection, or commit it by mistake.

`sbx secret` solves this with a **credential proxy**. The secret value lives
on your workspace; the sandbox only ever sees a localhost proxy URL it can
call. If the agent tries to dump the env, all it gets is a harmless proxy
endpoint.

## Add a secret

Use a fake "API key" so you can see the flow without burning a real one.

1. Pick a name and a value. The Run button below will store the value
   securely on the workspace:

    ```bash
    sbx secret add MY_API_KEY --value "sk-fake-demo-1234567890"
    ```

2. List your secrets — you'll see the name but not the value:

    ```bash
    sbx secret ls
    ```

## Use the secret inside a sandbox

Wire the secret into a sandbox at launch. The agent inside gets an env var
pointing at the proxy, not the raw secret.

```bash
sbx exec --secret MY_API_KEY -- sh -c 'echo "Agent sees: $MY_API_KEY"'
```

Notice the value isn't `sk-fake-demo-...` — it's a proxy URL like
`http://sbx-creds/secret/MY_API_KEY`. The agent can use it, but can't read
it directly.

## Hit the proxy from inside the sandbox

When the agent (or its code) needs the actual credential to make a request,
it asks the proxy. The proxy decides whether to allow it.

```bash
sbx exec --secret MY_API_KEY -- sh -c 'curl -s "$MY_API_KEY"'
```

The proxy returns the real value *only* into the curl process. The agent's
chat context and command history never see it.

## A realistic example

Most APIs want the credential in an `Authorization` header. The proxy makes
this a one-liner:

```bash
sbx exec --secret MY_API_KEY -- sh -c '
  TOKEN=$(curl -s "$MY_API_KEY")
  echo "Would call: curl -H \"Authorization: Bearer $TOKEN\" https://api.example.com/v1/things"
'
```

In real use, that `TOKEN=$(...)` line lives inside a script the agent runs —
never in something the model itself emits as plain text.

> [!WARNING]
> The proxy protects against accidental leakage and basic exfiltration. It
> does **not** stop a determined sandbox process that's allowed to make
> outbound requests. Combine secrets with `--network locked-down` or
> `balanced` whenever you can.

## Clean up

Remove the demo secret so it doesn't hang around:

```bash
sbx secret rm MY_API_KEY
```

Next: giving each agent its own git branch so they don't trample each other.
