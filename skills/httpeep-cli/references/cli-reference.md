# HTTPeep CLI Reference

## Installation Check

Before using `httpeep-cli`, verify it is installed:

```bash
httpeep-cli --version
```

If the command is not found, install it first:

- **Linux / macOS**: run the following command
  ```bash
  curl -fsSL https://s1.httpeep.com/install-cli.sh | bash
  ```
- **Windows**: command-line installation is not supported yet. Please download and install from the website:
  <https://httpeep.com/download>

---

This reference summarizes the HTTPeep CLI documentation under `content/docs/cli`. Load it when a task needs command details, flag names, examples, or troubleshooting steps beyond the main skill workflow.

Verified against `httpeep-cli` 0.8.6 plus the `shell` command update. Refresh this reference when CLI subcommands, flags, or output shapes change.

## Global Usage

`httpeep-cli` is bundled with the HTTPeep desktop app. Verify availability with:

```bash
httpeep-cli --version
```

`hp` is a visible alias for `httpeep-cli`. Prefer the full `httpeep-cli` form in agent-authored commands unless the user specifically requests short commands.

Global flags:

| Flag | Purpose |
|---|---|
| `--format <fmt>` | Output format: `human`, `json`, or `table`; default is `human` |
| `--quiet`, `-q` | Suppress informational messages |
| `--verbose`, `-v` | Enable verbose output |
| `--color <mode>` | Color mode: `auto`, `always`, or `never` |
| `-h`, `--help` | Print help |
| `-V`, `--version` | Print version |

Prefer `--format json` for automation. `sessions watch --format json` emits NDJSON, one JSON object per line.

## Proxy And Capture

The `proxy` subcommand controls the proxy engine. `capture` is an alias for `proxy`.

```bash
httpeep-cli proxy start --port 8800
httpeep-cli proxy start --capture-pid 1234 --capture-pid 5678
httpeep-cli proxy start --watch
httpeep-cli proxy pause
httpeep-cli proxy resume
httpeep-cli proxy stop
httpeep-cli proxy restart
httpeep-cli --format json proxy status
httpeep-cli proxy info
```

Proxy logs:

```bash
httpeep-cli proxy logs
httpeep-cli proxy logs --lines 100
httpeep-cli proxy logs --follow
```

System proxy:

```bash
httpeep-cli proxy system on
httpeep-cli proxy system off
httpeep-cli proxy system status
```

When the proxy is stopped, traffic no longer flows through HTTPeep. Existing captured sessions remain until cleared.

## Shell Capture

Use `shell` to enter an interactive child shell with HTTPeep terminal capture enabled:

```bash
httpeep-cli shell
hp shell
```

Behavior:

- starts or reuses the HTTPeep proxy
- writes setup artifacts under `~/.httpeep/automatic-setup/`
- loads `HTTP_PROXY`, `HTTPS_PROXY`, `ALL_PROXY`, certificate variables, and runtime hooks for Node.js, Python, Ruby, and Java/JVM tooling
- enters a child shell; exiting returns to the original terminal
- exposes `httpeep_intercept_off` inside the child shell to remove the capture environment without closing the shell

Inside the shell, verify with:

```bash
echo "$HTTPEEP_INTERCEPT_ACTIVE"
echo "$HTTP_PROXY"
curl -v https://httpbin.org/get
```

Agent guidance: suggest `hp shell` for human interactive debugging. For unattended automation, prefer explicit proxy environment variables, `request`, `rules run`, or `proxy start --capture-pid <pid>` because `shell` intentionally waits inside an interactive session.

## Sessions

List captured sessions:

```bash
httpeep-cli sessions list
httpeep-cli --format json sessions list --keyword login
```

Common filters:

| Flag | Purpose |
|---|---|
| `--id <id>` | Exact session ID |
| `--method <method>` | HTTP method |
| `--status-code <code>` | Status code |
| `--process-id <pid>` | Process ID |
| `--domain <domain>` | Exact domain |
| `--client-ip <ip>` | Client IP |
| `--is-pinned` | Pinned sessions only |
| `--is-important` | Important sessions only |
| `--from-ts <ts>` | Captured after timestamp |
| `--to-ts <ts>` | Captured before timestamp |
| `--url-like <pattern>` | Fuzzy URL match |
| `--path-like <pattern>` | Fuzzy path match |
| `--domain-like <pattern>` | Fuzzy domain match |
| `--process-name-like <pattern>` | Fuzzy process name match |
| `--keyword <keyword>` | Keyword across URL, domain, method, status, process name |

Watch live sessions:

```bash
httpeep-cli sessions watch
httpeep-cli sessions watch --domain api.example.com
httpeep-cli --format json sessions watch --keyword login
```

Delete or clear sessions:

```bash
httpeep-cli sessions delete --id abc123 --id def456
httpeep-cli sessions delete --keyword login --dry-run
httpeep-cli sessions delete --keyword login
httpeep-cli sessions clear --all --yes --dry-run
httpeep-cli sessions clear --all --yes
```

Useful `jq` patterns:

```bash
httpeep-cli sessions list --format json | jq '.[] | select(.status >= 400)'
httpeep-cli sessions list --format json | jq '.[] | select(.duration_ms > 500) | {url, status, duration_ms}'
httpeep-cli sessions list --format json | jq '[.[].host] | unique | sort[]'
httpeep-cli sessions list --format json | jq 'group_by(.method) | map({method: .[0].method, count: length})'
```

## Rules

Inspect, validate, test, and persist traffic manipulation rules:

```bash
httpeep-cli rules list
httpeep-cli --format json rules list
httpeep-cli rules export
httpeep-cli rules export --output rules-backup.json
httpeep-cli rules import ./rules.yaml
httpeep-cli rules import ./rules.yaml --mode merge
httpeep-cli rules validate --rule-file ./rule.yaml
httpeep-cli rules validate --replace --rule-file ./rules-full.json
httpeep-cli rules test --url "https://api.example.com/v1/users" --method GET -H "X-Debug: 1"
httpeep-cli rules upsert --map-remote "api.example.com=http://127.0.0.1:3000"
httpeep-cli rules delete --id my-rule-id
httpeep-cli rules replace --rule-file ./rules-full.json
httpeep-cli rules reset
```

Run a command with temporary rules and automatic rollback:

```bash
httpeep-cli rules run \
  --map-remote "api.example.com=http://127.0.0.1:3000" \
  -- httpeep-cli request --method GET --url "https://api.example.com/users"

httpeep-cli rules run \
  --json \
  --reject "api.example.com/orders=503" \
  -- httpeep-cli replay --id s1
```

Rule shortcut flags accepted by `rules upsert`, `rules replace`, `rules validate`, `rules run`, `request`, and `replay --id`:

| Flag | Format | Purpose |
|---|---|---|
| `--rule` | Inline JSON string | Full rule JSON, repeatable |
| `--rule-file` | File path | JSON or YAML rule file, repeatable |
| `--map-remote` | `<match>=<target>` | Redirect matching requests |
| `--map-local-file` | `<match>=<file_path>` | Serve a local file |
| `--inline-response` | `<match>=<status>:<mime>:<content>` | Return an inline response |
| `--reject` | `<match>=<status>` | Reject with status code |
| `--delay` | `<match>=<ms>` | Add latency |
| `--throttle` | `<match>=<spec>` | Throttle bandwidth, e.g. `100` or `req=100,res=200` |

`<match>` supports hosts, host/path, paths, full URLs, and wildcards such as `*.example.com`.

## Request

Send a request through HTTPeep so it appears in sessions:

```bash
httpeep-cli request --method GET --url "https://api.example.com/v2/users"
httpeep-cli --format json request --method GET --url "https://api.example.com/v2/users"
```

Headers and body:

```bash
httpeep-cli request \
  --method POST \
  --url "https://api.example.com/v2/users" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer token123" \
  --body '{"name": "Alice"}'
```

Core flags:

| Flag | Purpose |
|---|---|
| `--method <method>` | HTTP method; default is `GET` |
| `--url <url>` | Target URL |
| `-H`, `--header <header>` | Header in `Key: Value` format, repeatable |
| `--body <body>` | Inline request body |
| `--body-file <path>` | Read request body from file |
| `--no-save` | Do not save request and response as a session |

HTTP version:

```bash
httpeep-cli request --method GET --url "https://www.google.com" --http-version http2
httpeep-cli request --method GET --url "https://example.com" --http-version http1
```

Redirects:

```bash
httpeep-cli request --method GET --url "https://google.com" --follow-redirect --max-redirects 10
```

Upstream proxy:

```bash
httpeep-cli request --method GET --url "https://httpbin.org/get" \
  --proxy-url "http://user:pass@127.0.0.1:8800"

httpeep-cli request --method GET --url "https://httpbin.org/get" \
  --proxy-protocol socks5h \
  --proxy-host 127.0.0.1 \
  --proxy-port 1080 \
  --proxy-username user \
  --proxy-password pass
```

Temporary rules during request:

```bash
httpeep-cli request --method GET --url "https://api.example.com/users" \
  --map-remote "api.example.com=http://127.0.0.1:3000"

httpeep-cli request --method GET --url "https://api.example.com/health" \
  --inline-response "/health=503:text/plain:maintenance"
```

## Replay

Replay a captured session:

```bash
httpeep-cli replay --id <session_id>
httpeep-cli replay --id <session_id> --retry-times 3 --retry-interval-ms 800
```

Apply temporary rules during replay by ID:

```bash
httpeep-cli replay --id <session_id> --rule-file ./rule.yaml
httpeep-cli replay --id <session_id> --map-remote "api.example.com=http://127.0.0.1:3000"
```

Replay from file or latest capture:

```bash
httpeep-cli replay file ./recording.httpeep
httpeep-cli replay last
```

Temporary rule parameters are valid with `--id` mode, not with `replay file` or `replay last`.

## Record

Record a reusable traffic flow:

```bash
httpeep-cli record start
httpeep-cli record status
httpeep-cli record stop --output test-flow.httpeep
httpeep-cli replay file test-flow.httpeep
```

Typical regression flow:

```bash
httpeep-cli record start
npm run integration-tests
httpeep-cli record stop --output baseline.httpeep
httpeep-cli replay file baseline.httpeep
```

## Certificate

Manage the root CA for HTTPS interception:

```bash
httpeep-cli cert status
httpeep-cli cert install
httpeep-cli cert uninstall
httpeep-cli cert export --output ./httpeep-ca.crt
```

If HTTPS sessions are missing or unreadable, verify `cert status` and restart browsers or apps after installing trust because some apps cache the trust store.

## Import

Import external traffic formats:

```bash
httpeep-cli import curl "curl -X POST https://api.example.com/users -H 'Content-Type: application/json' -d '{\"name\":\"Alice\"}'"
httpeep-cli import har ./network.har
httpeep-cli import http ./request.http
```

These commands are not reliable for automation unless verified first. In the current standalone CLI implementation, import commands may return that they require a running HTTPeep instance and are not yet fully implemented. Prefer `request`, `record`, or `replay` for automated flows until an import smoke test succeeds in the target environment.

## MCP

Diagnose local MCP prerequisites, runtime paths, and port conflicts:

```bash
httpeep-cli mcp doctor
httpeep-cli --format json mcp doctor
```

Serve HTTPeep's MCP tools over stdio or streamable HTTP:

```bash
httpeep-cli mcp serve
httpeep-cli mcp serve --transport streamable-http --bind 127.0.0.1:8765 --path /mcp
```

MCP serve modes:

| Flag | Purpose |
|---|---|
| `--transport <mode>` | `stdio` or `streamable-http`; default is `stdio` |
| `--bind <host:port>` | Bind address for streamable HTTP |
| `--path <path>` | Route path for streamable HTTP; default is `/mcp` |

Use `mcp doctor` before debugging agent integration issues. Use `mcp serve` only when the task is specifically about running HTTPeep as an MCP server or wiring an agent to HTTPeep.

## Monitor

Launch an interactive terminal dashboard:

```bash
httpeep-cli monitor
```

The monitor shows live sessions, request rate, error rate, top hosts, and slowest endpoints. It requires the proxy engine to be running.

Keyboard controls:

| Key | Action |
|---|---|
| `q` | Quit |
| `f` | Change filters |
| `?` | Show help |

## Troubleshooting

CLI not found:

```bash
httpeep-cli --version
```

If missing, repair from HTTPeep desktop Settings -> MCP -> Repair CLI / PATH Installation, then restart the terminal.

Proxy not reachable:

```bash
httpeep-cli --format json proxy status
httpeep-cli proxy start
httpeep-cli proxy logs --lines 100
```

Sessions not appearing:

```bash
HTTP_PROXY=http://localhost:8080 HTTPS_PROXY=http://localhost:8080 your-app
httpeep-cli proxy system status
httpeep-cli cert status
```

Permission denied during cert install:

```bash
sudo httpeep-cli cert install
```

Garbled CI output:

```bash
httpeep-cli --format json sessions list
```

## Trace Log Template

For complex investigations, preserve enough evidence to replay the reasoning:

```text
Step:
Command:
Time or order:
Session IDs:
Rule IDs or shortcuts:
Key JSON fields:
Proxy log summary:
Outcome:
Redactions:
```
