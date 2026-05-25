<!-- GENERATED FILE: DO NOT EDIT DIRECTLY. -->
<!-- Source of truth: docs.httpeep.com/content/docs/cli/rules.mdx -->
<!-- Generate with: node scripts/generate-httpeep-cli-skill-reference.mjs <references-dir> -->

# Rules

The `rules` subcommand gives you full command-line control over HTTPeep's traffic routing rules. You can list active rules, add new forwarding rules, test whether a URL matches any rule, export your ruleset for version control, import rules on a new machine, and reset everything to a clean state.

## rules list

List all currently configured rules.

```bash
httpeep-cli rules list
httpeep-cli --format json rules list
```

## rules export

Export all rules to JSON. Use this to back up your ruleset or commit it to version control.

```bash
# Export to stdout
httpeep-cli rules export

# Export to a file
httpeep-cli rules export --output rules-backup.json
```

## rules import

Import rules from a JSON or YAML file.

```bash
# Replace all non-builtin rules (default)
httpeep-cli rules import ./rules.yaml

# Merge with existing rules
httpeep-cli rules import ./rules.yaml --mode merge
```

| Flag | Description | Default |
|---|---|---|
| `--mode <mode>` | Import mode: `replace` or `merge` | `replace` |

## rules validate

Validate a rule payload against the current store state before applying it.

```bash
# Upsert semantic validation
httpeep-cli rules validate --rule-file ./rule.yaml

# Replace semantic validation
httpeep-cli rules validate --replace --rule-file ./rules-full.json
```

## rules test

Test whether a given URL and method match any active rule, and see which rule would apply.

```bash
httpeep-cli rules test \
  --url "https://api.example.com/v1/users" \
  --method GET \
  -H "X-Debug: 1"
```

## rules upsert

Create or update a rule with merge (upsert) semantics. If a rule with the same ID exists, it is updated; otherwise a new rule is created.

```bash
httpeep-cli rules upsert --map-remote "api.example.com=http://127.0.0.1:3000"
```

You can also provide full rule JSON or a rule file:

```bash
httpeep-cli rules upsert --rule '{"id":"tmp-rule","description":"demo","enabled":true,"match":{}}'
httpeep-cli rules upsert --rule-file ./rule.yaml
```

## rules delete

Delete a rule by its ID.

```bash
httpeep-cli rules delete --id my-rule-id
```

## rules replace

Replace all non-builtin rules with the provided payload. This is a destructive operation — make sure to export your current rules first.

```bash
httpeep-cli rules replace --rule-file ./rules-full.json
```

> **Warning:**
> `rules replace` removes all non-builtin rules before importing the new set. Export your rules with `rules export` first if you may need them later.

## rules reset

Reset all rules to the builtin-only defaults. This removes every custom rule.

```bash
httpeep-cli rules reset
```

> **Warning:**
> `rules reset` permanently deletes all custom rules. Export your rules first with `rules export` if you may need them later.

## rules run

Run an arbitrary command with temporary rules applied. The rules are automatically rolled back when the command exits, so they never pollute the global ruleset.

```bash
httpeep-cli rules run \
  --map-remote "api.example.com=http://127.0.0.1:3000" \
  -- httpeep-cli request --method GET --url "https://api.example.com/users"
```

Agent-friendly JSON output:

```bash
httpeep-cli rules run \
  --json \
  --reject "api.example.com/orders=503" \
  -- httpeep-cli replay --id s1
```

Exit code convention:

- If rule parsing or validation fails, `rules run` exits non-zero.
- If the executed command fails, `rules run` exits with the command's exit code.
- With `--json`, the output includes `exit_code` and `success` fields.

## Shortcut rule parameters

Several commands (`rules upsert`, `rules replace`, `rules validate`, `rules run`, `request`, `replay`) accept shortcut parameters for common rule patterns without writing full JSON.

| Flag | Format | Description |
|---|---|---|
| `--rule` | Inline JSON string | Full rule JSON (repeatable) |
| `--rule-file` | File path | Rule file path, JSON or YAML (repeatable) |
| `--map-remote` | `<match>=<target>` | Redirect matching requests to another host |
| `--map-local-file` | `<match>=<file_path>` | Serve a local file for matching requests |
| `--inline-response` | `<match>=<status>:<mime>:<content>` | Return an inline response |
| `--reject` | `<match>=<status>` | Reject matching requests with a status code |
| `--delay` | `<match>=<ms>` | Add a delay in milliseconds |
| `--throttle` | `<match>=<spec>` | Throttle bandwidth (e.g. `100`, `req=100,res=200`) |

## Plan-gated response modification

Before creating rules that modify an upstream response body, status, or headers, check the current entitlement:

```bash
hp --format json license status
```

Response modification actions are a Pro capability. On a plan without that entitlement, a rule using response modification may fail with:

```text
Response modification actions are a Pro feature.
```

When you only need to mock an endpoint, use a resolve-based response such as `--inline-response` or `--map-local-file` instead of modifying an upstream response:

```bash
hp request --method GET --url "https://httpeep.com/api/welcome" \
  --inline-response "https://httpeep.com/api/welcome=200:application/json:{\"message\":\"Hello HTTPeep\"}"
```

> **Note:**
> If an attempted rule requires a plan-gated feature, report that limitation to the user before switching to an available alternative.

`<match>` supports:

- `host` — e.g. `api.example.com`
- `host/path` — e.g. `api.example.com/users`
- `/path` — e.g. `/api/health`
- Full URL — e.g. `https://api.example.com/v1/users`
- Wildcards — e.g. `*.example.com`

### Examples

**Map remote:**

```bash
httpeep-cli rules upsert --map-remote "api.example.com=http://127.0.0.1:3000"
```

**Map local file:**

```bash
httpeep-cli request --method GET --url "https://example.com/banner" \
  --map-local-file "example.com/banner=./fixtures/banner.json"
```

**Inline response:**

```bash
httpeep-cli request --method GET --url "https://api.example.com/health" \
  --inline-response "/health=503:text/plain:maintenance"
```

The `content` part supports `@file` to read from a file. If you need a literal `@` at the start, write `@@...`.

**Reject:**

```bash
httpeep-cli request --method GET --url "https://api.example.com/orders" \
  --reject "api.example.com/orders=503"
```

**Delay:**

```bash
httpeep-cli request --method GET --url "https://api.example.com/feed" \
  --delay "api.example.com/feed=300"
```

**Throttle:**

```bash
httpeep-cli request --method GET --url "https://api.example.com/feed" \
  --throttle "api.example.com/feed=req=100/200,res=150"
```

`--throttle` supports the shorthand `100` which means the same speed for both request and response.
