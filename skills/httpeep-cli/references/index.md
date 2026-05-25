# HTTPeep CLI References

<!-- GENERATED FILE: DO NOT EDIT DIRECTLY. -->
<!-- Source of truth: docs.httpeep.com/content/docs/cli/meta.json -->
<!-- Generate with: node scripts/generate-httpeep-cli-skill-reference.mjs <references-dir> -->

These files are generated one-to-one from `docs.httpeep.com/content/docs/cli/*.mdx`. Load only the reference file needed for the current task.

| File | Source | Use When |
|---|---|---|
| `overview.md` | `content/docs/cli/overview.mdx` | Understand what httpeep-cli can do and jump to command-specific guides. |
| `basics.md` | `content/docs/cli/basics.mdx` | httpeep-cli gives full programmatic access to the HTTPeep proxy engine — capture traffic, query sessions, manage rules, and replay requests from a terminal. |
| `proxy.md` | `content/docs/cli/proxy.mdx` | Control the HTTPeep proxy lifecycle from the terminal: start, pause, resume, stop, restart, and inspect the proxy engine. |
| `sessions.md` | `content/docs/cli/sessions.mdx` | Query and inspect captured traffic. List, filter, watch, and delete captured HTTP sessions from the command line. Combine with jq for powerful scripting and CI traffic analysis. |
| `dns.md` | `content/docs/cli/dns.mdx` | Manage HTTPeep DNS override configuration from the command line, including global host mappings and environment-scoped overrides. |
| `shell.md` | `content/docs/cli/shell.mdx` | Enter an interactive shell with HTTP_PROXY and HTTPS_PROXY configured for HTTPeep capture. |
| `license.md` | `content/docs/cli/license.mdx` | Activate an HTTPeep license from the CLI and inspect the current license runtime status. |
| `launch.md` | `content/docs/cli/launch.mdx` | Launch browsers, terminals, Electron apps, and desktop applications with HTTPeep proxy capture enabled. |
| `rules.md` | `content/docs/cli/rules.mdx` | Manage HTTPeep traffic manipulation rules from the command line: list, validate, test, upsert, delete, import, export, and reset rulesets. |
| `request.md` | `content/docs/cli/request.mdx` | Send HTTP requests through the HTTPeep proxy from the command line. Supports headers, body, proxy settings, HTTP version selection, and temporary rules. |
| `replay.md` | `content/docs/cli/replay.mdx` | Replay captured HTTP sessions from the command line. Replay by session ID with retry, replay from a script file, or replay the most recently captured request. |
| `record.md` | `content/docs/cli/record.mdx` | Record HTTP traffic flows into reusable script files for later replay or regression testing. |
| `cert.md` | `content/docs/cli/cert.mdx` | Manage the HTTPS interception root CA certificate: install to the system trust store, check status, or export for manual installation. |
| `monitor.md` | `content/docs/cli/monitor.mdx` | Watch live HTTP traffic in a real-time terminal dashboard. |

