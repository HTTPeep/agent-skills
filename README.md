# HTTPeep Agent Skills

Agent Skills for HTTPeep.

## Install

After this repository is published to GitHub and is public or accessible to the installer, install the HTTPeep CLI skill with:

```bash
npx skills add HTTPeep/agent-skills --skill httpeep-cli
```

To install from this local checkout while developing:

```bash
npx skills add /Users/chris/workspaces/agent-skills --skill httpeep-cli
```

To list available skills without installing:

```bash
npx skills add HTTPeep/agent-skills --list
npx skills add /Users/chris/workspaces/agent-skills --list
```

## Skills

- `httpeep-cli`: Use HTTPeep from the terminal with `httpeep-cli` for proxy lifecycle control, HTTP/HTTPS traffic capture, session inspection, rule injection, request replay, recording flows, certificate troubleshooting, CI scripting, and agent-driven network debugging.

## Repository Layout

```text
skills/
  httpeep-cli/
    SKILL.md
    references/
      cli-reference.md
    agents/
      openai.yaml
```

## Maintain

- Keep each skill under `skills/<skill-name>/SKILL.md`.
- Keep detailed command references in `references/` and load them from `SKILL.md` only when needed.
- Refresh verification notes when HTTPeep CLI flags, commands, or output shapes change.
- Validate locally before publishing:

```bash
npx skills add /Users/chris/workspaces/agent-skills --skill httpeep-cli --list
```
