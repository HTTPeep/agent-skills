<!-- GENERATED FILE: DO NOT EDIT DIRECTLY. -->
<!-- Source of truth: docs.httpeep.com/content/docs/cli/dns.mdx -->
<!-- Generate with: node scripts/generate-httpeep-cli-skill-reference.mjs <references-dir> -->

# DNS

`hp dns` manages DNS override rules used by the HTTPeep proxy. Use it to redirect selected hostnames to local IPs, switch between environment-specific mappings, and import or replace DNS configuration from JSON or YAML.

`hp` is the short alias for `httpeep-cli`; both command names work the same way.

## dns list

Show the full DNS override configuration.

```bash
hp dns list
hp --format json dns list
```

## dns replace

Replace the entire DNS override configuration from a JSON or YAML file.

```bash
hp dns replace --file ./dns.yaml
hp dns replace --file ./dns.json
```

Read the replacement payload from stdin with `-`:

```bash
cat ./dns.yaml | hp dns replace --file -
```

Example YAML:

```yaml
enabled: true
active_environment: local
global_hosts:
  api.example.com:
    ip: 127.0.0.1
    enabled: true
environments:
  local:
    hosts:
      db.example.com:
        ip: 127.0.0.1
        enabled: true
```

> **Warning:**
> `dns replace` overwrites the existing DNS override configuration. Export or copy the current configuration first if you may need to restore it.

## dns enable and disable

Enable or disable DNS override resolution without deleting any mappings.

```bash
hp dns enable
hp dns disable
```

## dns active-env

Set the active DNS environment. Environment-scoped host entries from the active environment are applied in addition to global host entries.

```bash
hp dns active-env set --name local
```

## dns env

List, create, replace, or delete named DNS environments.

```bash
# List environments
hp dns env list

# Create an empty environment
hp dns env upsert --name local

# Replace an environment from YAML or JSON
hp dns env upsert --name local --file ./dns-env.yaml

# Read the environment payload from stdin
cat ./dns-env.yaml | hp dns env upsert --name local --file -

# Delete an environment
hp dns env delete --name local
```

Example environment payload:

```json
{
  "hosts": {
    "api.internal.example.com": {
      "ip": "127.0.0.1",
      "enabled": true
    }
  }
}
```

## dns upsert

Add or update DNS host entries. Omit `--env` for global mappings; pass `--env <name>` for environment-scoped mappings.

```bash
# Add or update a global host entry
hp dns upsert \
  --domain api.example.com \
  --ip 127.0.0.1

# Add or update an environment-scoped host entry
hp dns upsert \
  --env local \
  --domain api.example.com \
  --ip 127.0.0.1

# Add a disabled environment-scoped entry
hp dns upsert \
  --env local \
  --domain staging.example.com \
  --ip 127.0.0.1 \
  --enabled false
```

## dns global-host

Manage host entries that apply globally, regardless of the active environment. Prefer `hp dns upsert --domain ... --ip ...` for creating and updating entries; `global-host` remains available for listing and deleting global mappings.

```bash
# List global host entries
hp dns global-host list

# Delete a global host entry
hp dns global-host delete --pattern api.example.com
```

## dns env-host

Manage host entries inside a named DNS environment. Prefer `hp dns upsert --env ... --domain ... --ip ...` for creating and updating entries; `env-host` remains available for listing and deleting environment-scoped mappings.

```bash
# List host entries in an environment
hp dns env-host list --env local

# Delete an environment-scoped host entry
hp dns env-host delete --env local --pattern api.example.com
```

## Output formats

Use `--format json` when scripting DNS updates or checking state in CI.

```bash
hp --format json dns list
hp --format json dns env list
hp --format json dns global-host list
```

> **Note:**
> DNS overrides are enforced by the proxy runtime. Existing long-lived connections may need to reconnect before a changed DNS mapping affects traffic.
