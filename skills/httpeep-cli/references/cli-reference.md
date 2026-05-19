# HTTPeep CLI Reference

## Installation Check

Before using `httpeep-cli`, verify it is installed:

```bash
httpeep-cli --version
```

If the command is not found, install it first:

- **macOS** (Homebrew):
  ```bash
  brew install --cask httpeep/httpeep/httpeep
  ```
- **Linux / macOS** (curl):
  ```bash
  curl -fsSL https://s1.httpeep.com/install-cli.sh | bash
  ```
- **Windows**: command-line installation is not supported yet. Please download and install from the website:
  <https://httpeep.com/download>

---

This reference summarizes the HTTPeep CLI documentation under `content/docs/cli`. Load it when a task needs command details, flag names, examples, or troubleshooting steps beyond the main skill workflow.

Verified against `httpeep-cli` with `rules create`/`rules update` matcher and pipeline flags, `dns` subcommands, and the `shell` command. Refresh this reference when CLI subcommands, flags, or output shapes change.

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

### proxy start

```bash
httpeep-cli proxy start --port 8800
```

| Flag | Description | Default |
|---|---|---|
| `--port <port>` | Listen port for the proxy | — |
| `--capture-pid <pid>` | Only capture traffic from the given process ID (repeatable) | — |
| `--watch` | Start watching new sessions immediately after the proxy starts | — |

Capture traffic from specific processes only:

```bash
httpeep-cli proxy start --capture-pid 1234 --capture-pid 5678
```

Start and immediately watch new traffic:

```bash
httpeep-cli proxy start --watch
```

### proxy pause / resume / stop / restart

```bash
httpeep-cli proxy pause
httpeep-cli proxy resume
httpeep-cli proxy stop
httpeep-cli proxy restart
httpeep-cli --format json proxy status
httpeep-cli proxy info
```

### proxy logs

```bash
# Show the last 50 lines (default)
httpeep-cli proxy logs

# Show the last 100 lines
httpeep-cli proxy logs --lines 100

# Follow new log output
httpeep-cli proxy logs --follow
```

| Flag | Description | Default |
|---|---|---|
| `--lines <n>` | Number of lines to show | 50 |
| `--follow` / `-f` | Follow log output continuously | — |

### proxy system

Configure the system proxy settings so that all applications route traffic through HTTPeep automatically:

```bash
httpeep-cli proxy system on
httpeep-cli proxy system off
httpeep-cli proxy system status
```

### capture alias

`capture` is a direct alias for `proxy`. All of the above commands work identically with `capture`:

```bash
httpeep-cli capture start --port 8800
httpeep-cli capture status
httpeep-cli capture pause
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

Manage traffic manipulation rules from the terminal. Rules follow a four-part pipeline: **Matcher** → **Resolve action** → **Request pipeline** → **Response pipeline**. Rules are evaluated top-to-bottom; use `rules reorder` when one rule should take priority over another.

### Command overview

| Command | Purpose |
|---|---|
| `rules create` | Create one persistent rule from flags or `--from-file` |
| `rules list` | List rules, optionally filtered by group or enabled state |
| `rules show` | Show one rule by ID or unique name |
| `rules update` | Patch one existing rule by ID or unique name |
| `rules delete` | Delete one rule |
| `rules enable` / `rules disable` | Toggle one rule, or every rule in a group |
| `rules reorder` | Move a rule before/after another rule or to top/bottom |
| `rules export` / `rules import` | Back up or restore rules |
| `rules validate` | Validate a rule payload before applying it |
| `rules test` | Test whether a request matches current rules |
| `rules run` | Run a command with temporary rules and automatic rollback |

### Matcher flags

At least one matcher is required for `rules create`. Pattern type is inferred automatically: contains `*` or `?` → wildcard, contains regex markers → regex, otherwise → exact. Comma-separated values are OR alternatives.

| Flag | Description |
|---|---|
| `--match-url <pattern>` | Match a full URL |
| `--match-host <pattern>` | Match the request host |
| `--match-path <pattern>` | Match the path without query string |
| `--match-method <method>` | Match one or more HTTP methods, e.g. `GET,POST` |
| `--match-header <key:value>` | Match request headers. Repeatable; comma-separated pairs accepted. |
| `--match-header-op <and\|or>` | Combine multiple header matchers. Default: `and`. |
| `--match-query <key=value>` | Match query parameters. Repeatable; comma-separated pairs accepted. |
| `--match-query-op <and\|or>` | Combine multiple query matchers. Default: `and`. |

```bash
# Wildcard host and path
httpeep-cli rules create "mock v1 or v2 users" \
  --match-host "*.example.com" \
  --match-path "/api/v*/users*" \
  --res-set-header "X-Matched:true"

# Header OR logic
httpeep-cli rules create "any debug header" \
  --match-host "api.example.com" \
  --match-header "X-Debug:true,X-Test:1" \
  --match-header-op or \
  --res-set-header "X-Debug-Rule:hit"
```

### Resolve actions

| Flag | Description |
|---|---|
| `--action pass-through` | Do not replace the upstream target; still allows pipeline actions. |
| `--action map-remote` | Forward matching requests to another remote URL. Requires `--map-remote-url`. |
| `--action map-local` | Serve a local file. Requires `--map-local-file`. |
| `--action block` | Reject the request or simulate a network error. |

```bash
# Map remote
httpeep-cli rules create "redirect API to staging" \
  --match-host "api.example.com" \
  --action map-remote \
  --map-remote-url "https://staging.example.com"

# Map local
httpeep-cli rules create "mock payment" \
  --match-path "/api/payment/*" \
  --action map-local \
  --map-local-file ./mocks/payment.json \
  --map-local-status 200

# Block (HTTP response)
httpeep-cli rules create "silent block analytics" \
  --match-host "*.analytics.com,*.tracking.io" \
  --action block \
  --block-mode reject \
  --block-status 200 \
  --block-body '{}'

# Block (network error)
httpeep-cli rules create "analytics network error" \
  --match-host "*.analytics.com" \
  --action block \
  --block-mode network-error
```

### Request pipeline

| Flag | Description |
|---|---|
| `--req-dns-override <domain:ip>` | Use a rule-level DNS override. Repeatable. |
| `--req-proxy <url>` | Use a rule-level upstream proxy, e.g. `http://host:8080` or `socks5://127.0.0.1:1080`. |
| `--req-delay <ms>` | Delay before sending the upstream request. |
| `--req-throttle <kbps>` | Throttle request upload speed. |
| `--req-set-header <key:value>` | Set request headers. Repeatable; comma-separated pairs accepted. |
| `--req-remove-header <key>` | Remove request headers. Repeatable. |
| `--req-set-query <key=value>` | Set query parameters. Repeatable; comma-separated pairs accepted. |
| `--req-remove-query <key>` | Remove query parameters. Repeatable. |
| `--req-set-body <body\|@file>` | Replace the request body with a literal string or file content. |
| `--req-breakpoint` | Pause matching requests before upstream delivery. |

```bash
# Inject auth and remove cookies
httpeep-cli rules create "inject test auth" \
  --match-host "api.example.com" \
  --req-remove-header "Cookie" \
  --req-set-header "Authorization:Bearer test-token" \
  --req-set-header "X-Debug:true"

# Replace request body from a file
httpeep-cli rules create "replace checkout body" \
  --match-path "/api/checkout" \
  --match-method POST \
  --req-set-body @./payloads/checkout.json
```

### Response pipeline

| Flag | Description |
|---|---|
| `--res-delay <ms>` | Delay before returning the response. |
| `--res-throttle <kbps>` | Throttle response download speed. |
| `--res-status <code>` | Rewrite the response status code. |
| `--res-set-header <key:value>` | Set response headers. Repeatable; comma-separated pairs accepted. |
| `--res-remove-header <key>` | Remove response headers. Repeatable. |
| `--res-set-body <body\|@file>` | Replace the response body with a literal string or file content. |
| `--res-breakpoint` | Pause matching responses before returning them to the client. |

```bash
# Force a 500 response
httpeep-cli rules create "simulate 500" \
  --match-host "api.example.com" \
  --res-status 500 \
  --res-set-header "Content-Type:application/json" \
  --res-set-body '{"error":"internal server error"}'

# Add breakpoints
httpeep-cli rules create "break checkout" \
  --match-path "/api/checkout" \
  --match-method POST \
  --req-breakpoint \
  --res-breakpoint \
  --group "debug"
```

### Create from file

Use `--from-file` when a rule is too complex for one command:

```bash
httpeep-cli rules create --from-file ./payment-rule.yaml
```

```yaml
# payment-rule.yaml
name: "mock payment API"
enabled: true
group: "mock"
matcher:
  url: "https://api.example.com/payment/*"
  method: ["POST", "GET"]
resolve:
  action: map-local
  map_local:
    file: ./mocks/payment.json
    status: 200
request_pipeline:
  delay_ms: 500
  set_headers:
    X-Debug: "true"
response_pipeline:
  delay_ms: 200
  set_headers:
    X-Mock: "true"
```

JSON works too. After creation, rules are stored in the standard `ForwardRuleConfig` format.

### List and show rules

```bash
httpeep-cli rules list
httpeep-cli --format json rules list
httpeep-cli rules list --enabled
httpeep-cli rules list --group "mock"
httpeep-cli rules show cli-rule-mock-users-api
httpeep-cli rules show "mock users API"
httpeep-cli --format json rules show "mock users API"
```

### Update rules

`rules update` patches only the fields you provide. Existing matchers, resolve action, and pipeline actions remain unchanged:

```bash
# Change only the response status and body
httpeep-cli rules update "simulate 500" \
  --res-status 200 \
  --res-set-body '{"ok":true}'

# Rename and regroup
httpeep-cli rules update cli-rule-mock-users-api \
  --name "mock users API v2" \
  --group "mock-v2"

# Patch from file
httpeep-cli rules update cli-rule-mock-payment-api --from-file ./payment-rule.yaml
```

### Enable, disable, delete

```bash
# Toggle one rule
httpeep-cli rules disable "mock users API"
httpeep-cli rules enable cli-rule-mock-users-api

# Toggle a whole group
httpeep-cli rules disable --group "debug"
httpeep-cli rules enable --group "mock"

# Delete
httpeep-cli rules delete "mock users API"
httpeep-cli rules delete --id cli-rule-mock-users-api --force
```

### Reorder rules

```bash
httpeep-cli rules reorder "silent block analytics" --to-top
httpeep-cli rules reorder "slow checkout" --to-bottom
httpeep-cli rules reorder "mock users API" --before "staging API"
httpeep-cli rules reorder "inject test auth" --after "staging API"
```

Exactly one of `--before`, `--after`, `--to-top`, or `--to-bottom` is required.

### Import and export

```bash
# Export
httpeep-cli rules export
httpeep-cli rules export --output ./rules-backup.json

# Import — replace all non-builtin rules (default)
httpeep-cli rules import ./rules-backup.json

# Import — merge with existing rules
httpeep-cli rules import ./rules-backup.json --mode merge
```

| Flag | Description | Default |
|---|---|---|
| `--mode replace` | Replace all non-builtin rules with imported rules | `replace` |
| `--mode merge` | Upsert imported rules into the current ruleset | — |

### Validate and test

```bash
# Validate an upsert payload
httpeep-cli rules validate --rule-file ./rule.yaml

# Validate as a full replacement
httpeep-cli rules validate --replace --rule-file ./rules-full.json

# Test whether a request would match
httpeep-cli rules test \
  --url "https://api.example.com/v1/users?debug=true" \
  --method GET \
  -H "X-Debug: true"

httpeep-cli --format json rules test \
  --url "https://api.example.com/v1/users" \
  --method POST
```

### Temporary rules (rules run)

`rules run` executes a command with temporary rules that roll back automatically:

```bash
httpeep-cli rules run \
  --map-remote "api.example.com=http://127.0.0.1:3000" \
  -- httpeep-cli request --method GET --url "https://api.example.com/users"

httpeep-cli rules run \
  --json \
  --reject "api.example.com/orders=503" \
  -- httpeep-cli replay --id s1
```

Exit code convention: if rule parsing fails, `rules run` exits non-zero. If the executed command fails, it exits with the command's exit code. With `--json`, the output includes `exit_code` and `success` fields.

### Legacy shortcut parameters

These shortcut flags remain available for `rules upsert`, `rules replace`, `rules validate`, `rules run`, `request`, and `replay --id`:

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

## DNS Override

Manage DNS Override settings from the terminal. DNS Override only affects traffic routed through HTTPeep — it does not edit `/etc/hosts` or require system-wide DNS changes. Global DNS host entries are available to all users; environment-scoped DNS groups and active environment switching require Pro entitlement.

### Command overview

| Command | Purpose |
|---|---|
| `dns list` | Show the full DNS Override configuration |
| `dns enable` / `dns disable` | Toggle DNS Override resolution globally |
| `dns replace` | Replace the full DNS configuration from JSON or YAML |
| `dns global-host list` | List global host mappings |
| `dns global-host upsert` | Create or update one global host mapping |
| `dns global-host delete` | Delete one global host mapping |
| `dns env list` | List environment groups |
| `dns env upsert` | Create or replace one environment group |
| `dns env delete` | Delete one environment group |
| `dns env-host list` | List host mappings in an environment |
| `dns env-host upsert` | Create or update one environment-scoped mapping |
| `dns env-host delete` | Delete one environment-scoped mapping |
| `dns active-env set` | Select the active environment |

### Mental model

DNS Override has three layers:

1. **Global switch** — `enabled` turns DNS Override on or off.
2. **Environment hosts** — mappings under the selected `activeEnv`.
3. **Global hosts** — fallback mappings that apply regardless of environment.

When a host exists in both the active environment and `globalHosts`, the environment mapping wins. Exact host matches are evaluated before wildcard matches.

### Global host mappings

Global hosts apply no matter which DNS environment is active:

```bash
httpeep-cli dns global-host upsert \
  --pattern api.example.com \
  --ip 127.0.0.1

httpeep-cli dns global-host upsert \
  --pattern "*.internal.example.com" \
  --ip 10.0.0.5
```

Disable a mapping without deleting it:

```bash
httpeep-cli dns global-host upsert \
  --pattern api.example.com \
  --ip 127.0.0.1 \
  --enabled false
```

List or delete global mappings:

```bash
httpeep-cli dns global-host list
httpeep-cli dns global-host delete --pattern api.example.com
```

### Toggle DNS Override

```bash
httpeep-cli dns disable
httpeep-cli dns enable
```

### Environment groups

Environment groups let you switch between dev, staging, and production mappings without editing each host one by one:

```bash
# Create an empty environment
httpeep-cli dns env upsert --name dev

# Add host entries
httpeep-cli dns env-host upsert \
  --env dev \
  --pattern api.example.com \
  --ip 127.0.0.1

httpeep-cli dns env-host upsert \
  --env staging \
  --pattern api.example.com \
  --ip 10.0.1.50

# Activate an environment (auto-creates if missing)
httpeep-cli dns active-env set --name dev

# List and delete
httpeep-cli dns env list
httpeep-cli dns env-host list --env dev
httpeep-cli dns env-host delete --env dev --pattern api.example.com
httpeep-cli dns env delete --name staging
```

### Replace full config

Use `dns replace` to apply a complete DNS configuration from a checked-in file:

```yaml
# dns.yaml
enabled: true
activeEnv: dev
globalHosts:
  internal-tool.example.com:
    ip: 10.0.0.5
    enabled: true
environments:
  dev:
    hosts:
      api.example.com:
        ip: 127.0.0.1
        enabled: true
      "*.dev.example.com":
        ip: 127.0.0.1
        enabled: true
  staging:
    hosts:
      api.example.com:
        ip: 10.0.1.50
        enabled: true
```

```bash
httpeep-cli dns replace --file ./dns.yaml
cat ./dns.yaml | httpeep-cli dns replace --file -
```

Replace a single environment:

```bash
httpeep-cli dns env upsert --name dev --file ./dev-dns.yaml
```

### JSON output

```bash
httpeep-cli --format json dns list
httpeep-cli --format json dns global-host list
httpeep-cli --format json dns env-host list --env dev

# List only enabled global host mappings
httpeep-cli --format json dns global-host list | \
  jq 'to_entries[] | select(.value.enabled) | "\(.key) -> \(.value.ip)"'
```

### Common workflows

Route a production API to localhost:

```bash
httpeep-cli dns global-host upsert \
  --pattern api.myapp.com \
  --ip 127.0.0.1
httpeep-cli dns enable
```

Switch a test run to staging DNS:

```bash
httpeep-cli dns env-host upsert \
  --env staging \
  --pattern api.myapp.com \
  --ip 10.0.1.50
httpeep-cli dns active-env set --name staging
```

Keep team DNS mappings in source control:

```bash
httpeep-cli --format json dns list > httpeep-dns.json
httpeep-cli dns replace --file ./httpeep-dns.json
```

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

Manage the root CA for HTTPS interception.

Before capturing traffic, verify that the root certificate is trusted. Without a trusted root CA, HTTPS traffic cannot be decrypted:

```bash
hp cert status
hp cert install
```

Other certificate commands:

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
