---
name: httpeep-cli
description: Use HTTPeep from the terminal with httpeep-cli for proxy lifecycle control, HTTP/HTTPS traffic capture, session inspection, rule injection, request replay, recording flows, certificate troubleshooting, CI scripting, and agent-driven network debugging. Use when a task mentions HTTPeep, httpeep-cli, proxy debugging, captured HTTP sessions, traffic rules, request replay, HTTPS interception certificates, or terminal-based traffic monitoring.
---

# HTTPeep CLI

## Purpose

Use `httpeep-cli` or its `hp` alias to investigate and control HTTPeep from the terminal. This file defines agent operating policy. For concrete subcommands, flags, schemas, and examples, load the specific file under `references/` that matches the task. Reference files are generated one-to-one from the HTTPeep documentation site.

## Operating Strategy

1. Establish context before changing state.
   - Verify that the CLI is available, inspect proxy status, and check relevant recent logs.
   - Check the current license entitlement before attempting capabilities that may be gated.

2. Scope capture and query output aggressively.
   - Start with filters relevant to the requested app, domain, process, status, or time range.
   - Prefer bounded, field-selected JSON results for analysis rather than dumping every captured body and header.
   - Inspect full session detail only for representative or suspicious requests.

3. Separate failure analysis from slowness analysis.
   - For failures, group transport errors and status classes, then inspect representative responses and timing breakdowns.
   - For latency, distinguish ordinary API calls from streams, long polling, downloads, and intentionally delayed rules.

4. Prefer reversible changes.
   - Use temporary rules or request/replay-level overrides for experiments.
   - Export existing state before persistent replacement or reset operations.
   - Dry-run destructive session cleanup when supported.

5. Respect entitlement boundaries and disclose them.
   - If an attempted capability is unavailable on the user's plan, state the unavailable feature and the evidence from the CLI.
   - When the requested outcome can be reached with an available alternative, switch to it and tell the user what changed in the implementation approach.
   - For mock responses on non-Pro plans, prefer resolve-based mock responses over modification of an upstream response.

6. Treat capture data as sensitive.
   - Redact authorization values, cookies, tokens, API keys, and sensitive bodies from reports.
   - Report only the fields needed to support conclusions.

7. Keep the installed skill current without interrupting user work.
   - Do not run routine update checks in the middle of an active investigation or command sequence.
   - If a requested HTTPeep capability appears unavailable, incomplete, or inconsistent with the current skill guidance, check for a newer skill before concluding the capability is unsupported:
     `npx skills check HTTPeep/agent-skills --skill httpeep-cli`
   - If the check reports that a newer skill is available, update it with:
     `npx skills update HTTPeep/agent-skills --skill httpeep-cli`
   - After completing each user task, run the same check as a background maintenance step. If a newer skill is available, update it after the task outcome has been reported or otherwise avoid blocking the user's requested operation.
   - If the update command fails, report the failure briefly and continue with the task result that was already produced.

## Reference Directory

Start with `references/index.md` when you are unsure which reference file to load. Otherwise load only the file needed for the task:

- `references/overview.md` for the CLI overview and common workflows.
- `references/basics.md` for installation, aliases, global flags, JSON output, and troubleshooting.
- `references/proxy.md` for proxy lifecycle, logs, system proxy, and capture status.
- `references/sessions.md` for listing, filtering, pagination, field selection, watch, delete, and clear.
- `references/dns.md` for DNS override configuration.
- `references/shell.md` for interactive terminal capture.
- `references/license.md` for `hp license status`, activation, and entitlement checks.
- `references/launch.md` for launching browsers, terminals, and desktop apps with capture enabled.
- `references/rules.md` for rules, shortcuts, validation, temporary rules, and plan-gated response modification.
- `references/request.md` for sending requests through HTTPeep.
- `references/replay.md` for replaying captured sessions.
- `references/record.md` for recording traffic flows.
- `references/cert.md` for HTTPS interception certificates.
- `references/import.md` for cURL, HAR, and raw HTTP imports.
- `references/monitor.md` for the terminal traffic monitor.

## Investigation Output

For multi-step work, report the important commands or actions performed, the session or rule identifiers used, relevant statuses and timings, and any entitlement limitation or fallback applied. Do not expose secrets in that trace.
