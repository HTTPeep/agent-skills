---
name: httpeep-cli
description: Use HTTPeep from the terminal with httpeep-cli for proxy lifecycle control, HTTP/HTTPS traffic capture, session inspection, rule injection, request replay, recording flows, certificate troubleshooting, CI scripting, and agent-driven network debugging. Use when a task mentions HTTPeep, httpeep-cli, proxy debugging, captured HTTP sessions, traffic rules, request replay, HTTPS interception certificates, or terminal-based traffic monitoring.
---

# HTTPeep CLI

## Overview

Use `httpeep-cli` to operate HTTPeep from the terminal for local debugging, automation, CI checks, and agent workflows. Prefer `--format json` for commands that Codex must parse or summarize.

For detailed flags, examples, and command-specific notes, read `references/cli-reference.md`.

## Operating Workflow

1. Verify the CLI and proxy state before deeper debugging:

```bash
httpeep-cli --version
httpeep-cli --format json proxy status
httpeep-cli proxy logs --lines 50
```

2. Start or repair capture depending on the task:

```bash
httpeep-cli proxy start --port 8800
httpeep-cli proxy system status
httpeep-cli cert status
```

Use `proxy system on` only when the user wants system-wide proxying. For scoped capture, prefer explicit app proxy environment variables or `proxy start --capture-pid <pid>`.

3. Capture and inspect traffic:

```bash
httpeep-cli --format json sessions list --keyword login
httpeep-cli --format json sessions watch --domain api.example.com
```

Use filters before destructive cleanup. Always dry-run deletes when possible:

```bash
httpeep-cli sessions delete --keyword login --dry-run
httpeep-cli sessions clear --all --yes --dry-run
```

4. Apply temporary rules for reproducible tests before changing the global ruleset:

```bash
httpeep-cli rules run \
  --map-remote "api.example.com=http://127.0.0.1:3000" \
  -- httpeep-cli request --method GET --url "https://api.example.com/users"
```

Use `rules upsert`, `rules import`, `rules replace`, or `rules reset` only when persistent rules are required. Export existing rules first before destructive changes:

```bash
httpeep-cli rules export --output rules-backup.json
```

5. Send, replay, or record requests:

```bash
httpeep-cli --format json request --method GET --url "https://api.example.com/v2/users"
httpeep-cli replay --id <session_id> --retry-times 3 --retry-interval-ms 800
httpeep-cli record start
httpeep-cli record stop --output baseline.httpeep
httpeep-cli replay file baseline.httpeep
```

## Troubleshooting Priority

Check failures in this order:

1. CLI availability: `httpeep-cli --version`
2. Proxy engine reachability: `httpeep-cli --format json proxy status`
3. Recent proxy logs: `httpeep-cli proxy logs --lines 100`
4. App routing: `HTTP_PROXY`, `HTTPS_PROXY`, or `httpeep-cli proxy system status`
5. HTTPS trust: `httpeep-cli cert status`, then `httpeep-cli cert install` if needed
6. Output parsing: rerun relevant commands with `--format json`; remember `sessions watch --format json` emits NDJSON

If `httpeep-cli` is not on PATH, instruct the user to open HTTPeep desktop settings and use Settings -> MCP -> Repair CLI / PATH Installation, or call the MCP repair tool when available.

## Trace Evidence

For complex debugging or multi-step capture/replay work, record a concise trace log in the final answer or task notes:

- Commands executed, with important flags
- Timestamp or sequence order for each major step
- Session IDs used or produced
- Relevant JSON fields from `sessions list`, `request`, `rules run`, or `replay`
- Summary from `httpeep-cli proxy logs --lines <n>` when proxy behavior is involved
- Rule IDs or temporary rule shortcuts applied

Avoid logging secrets from headers, cookies, Authorization values, or request bodies. Redact sensitive values before reporting.

## Safety Defaults

- Prefer temporary rules with `rules run`, `request`, or `replay --id` before persistent rule edits.
- Use `--format json` for machine parsing and CI logs.
- Dry-run destructive session cleanup first.
- Export rules before `rules replace` or `rules reset`.
- Run `cert install`, `cert uninstall`, `proxy system on`, and `proxy system off` only when the user explicitly asks for certificate trust or system-wide proxy changes.
- Run `rules replace`, `rules reset`, or `sessions clear --all --yes` only when the user explicitly asks for persistent replacement, reset, or full cleanup. Show or run the backup/dry-run command first when possible.
- Treat `import curl`, `import har`, and `import http` as version-dependent because some CLI builds may report that these commands are not yet implemented.
