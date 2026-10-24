# Limrun auth.md

How agents authenticate against the Limrun API (`api.limrun.com`).

## Summary

- The control plane authenticates with API keys, sent as `Authorization: Bearer <key>`. A human must approve every key: either by creating it in the console, or by approving a `lim login` request. Agents cannot mint org API keys on their own.
- Keys have the prefix `lim_`. Create them at [console.limrun.com](https://console.limrun.com) under Settings, API Keys.
- The `lim` CLI and every SDK (TypeScript, Python, Go) read the key from the `LIM_API_KEY` environment variable by default:

```bash
export LIM_API_KEY=lim_...
curl https://api.limrun.com/v1/ios_instances \
  -H "Authorization: Bearer $LIM_API_KEY"
```

## Agent setup

If you are an agent operating on a user's behalf and `LIM_API_KEY` is not set, sign in with `lim login`. It prints a URL and a phrase, then waits until the user approves in a browser on any device (a phone works). The CLI stores the key in `~/.lim/config.yaml`, so the key never appears in your transcript.

```bash
lim login --json    # first line: {"status":"pending","verificationUrl":"...","phrase":"...","expiresAt":"..."}
                    # after the user approves: {"status":"signed-in"}
lim whoami --json   # {"signedIn":true,"organizationId":"org_...",...}; never prints the key
```

Show the user `verificationUrl` and `phrase` as soon as the first line appears, and ask them to approve. The command waits until `expiresAt` (about 10 minutes), so run it in the background or with a long timeout. If your shell stops it, run `lim login` again: it continues the same login with the same URL and phrase.

Older `lim` versions reject `--json` for `lim login`. Run it without `--json`: it prints the same URL and phrase and waits up to 5 minutes. When no human can approve (for example in CI), ask the user to create an API key in the console and set it as `LIM_API_KEY`.

## Other credentials you will encounter

| Credential | Purpose |
|---|---|
| Org API key (`lim_...`) | REST control plane on `api.limrun.com`: create, list, and delete instances. |
| Per-instance token (`instance.status.token`) | Everything scoped to one running instance: device endpoints (`apiUrl`, WebSocket endpoints) and the instance's MCP server (`status.mcpUrl`), as `Authorization: Bearer`. Returned in the instance status when it is ready. |
| Scoped registry token (`lim_st_...`) | Short-lived, scope-limited token minted by your backend for browser use. |

Full details, including which credential each endpoint expects, are in the [SDK reference](https://docs.limrun.com/docs/reference/sdk.md).
