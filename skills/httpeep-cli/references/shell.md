<!-- GENERATED FILE: DO NOT EDIT DIRECTLY. -->
<!-- Source of truth: docs.httpeep.com/content/docs/cli/shell.mdx -->
<!-- Generate with: node scripts/generate-httpeep-cli-skill-reference.mjs <references-dir> -->

# Shell

`hp shell` opens an interactive terminal session with proxy environment variables already configured for HTTPeep. It is the fastest way to capture traffic from command-line tools, package managers, SDKs, and test commands that honor `HTTP_PROXY` and `HTTPS_PROXY`.

`hp` is the short alias for `httpeep-cli`; both command names work the same way.

## Start a capture shell

```bash
hp shell
```

When you run `hp shell`, the CLI:

1. Starts the HTTPeep proxy if it is not already running.
2. Reuses the running proxy when a live proxy instance is already available.
3. Enters a child shell with proxy variables such as `HTTP_PROXY`, `HTTPS_PROXY`, `ALL_PROXY`, `http_proxy`, and `https_proxy` set.

Run your commands inside that shell and exit when you are done.

```bash
hp shell
curl https://api.example.com/users
npm test
exit
```

## Inspect captured traffic

Open another terminal or run after leaving the shell:

```bash
hp sessions list --keyword api.example.com
hp --format json sessions list --fields id,method,url,status_code,timing
```

## How it chooses the proxy endpoint

`hp shell` uses the configured proxy host and port from the running HTTPeep runtime. If the proxy is configured to bind to an unspecified address such as `0.0.0.0`, `::`, or `[::]`, the shell uses `127.0.0.1` for client-side proxy environment variables.

This keeps local command-line tools connecting to the loopback interface while the proxy can still listen on a wider bind address.

## When to use shell

Use `hp shell` when you want to capture a sequence of terminal commands without editing each command individually.

```bash
hp shell
pnpm install
pnpm test
curl https://httpbin.org/get
exit
```

For launching desktop applications or browsers with capture enabled, use `hp launch` instead.

> **Tip:**
> `hp shell` is also used by `hp record start --shell` workflows so recorded terminal traffic goes through the same proxy setup path.
