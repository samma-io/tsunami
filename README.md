# Samma scanner: Tsunami

![Samma-io](assets/samma_logo.png)

[Google Tsunami](https://github.com/google/tsunami-security-scanner) wrapped as a Samma scanner. It
is one of the open-source scanners that the [Samma operator](https://github.com/samma-io/operator)
can run in your Kubernetes cluster, against the hosts in your annotated Ingresses.

## What it checks

Tsunami is a plugin-based scanner for **high-severity** vulnerabilities, with few false positives:

- remote code execution in common software
- admin and management interfaces exposed without authentication
- weak credentials on exposed services

**Use it for** finding the critical issues first. It is slower and heavier than the other scanners,
and it is not in the `default` profile. Only point it at hosts you are allowed to test.

## How it is built

The scan runs in two steps:

1. Tsunami scans the target and writes `/out/tsunami-output.json`. It needs a different flag for an
   IP (`--ip-v4-target`) and a hostname (`--hostname-target`). The operator picks the right one.
2. The Samma logger (`sammascanner/logger`) reads that file and emits one Samma finding per
   vulnerability, to stdout and optionally to NATS.

`tsunami-security-scanner/` is the upstream Tsunami source. `set_tsunami_repos.sh` sets up its
plugin repositories.

## Run it locally

Edit the target in the `command:` line of `docker-compose.yaml`, then:

```sh
docker compose up --build
```

## Settings

The logger is configured with environment variables. The values shown are the ones in
`docker-compose.yaml`:

| Variable | Value | Description |
|---|---|---|
| `FILE` | `tsunami-output.json` | Tsunami output file to read from `/out/` |
| `SAMMA_IO_SCANNER` | `tsunami` | Scanner label on every finding |
| `SAMMA_IO_ID` | `1234` | Id added to every finding |
| `SAMMA_IO_TAGS` | `scanner,prod` | Comma-separated tags added to every finding |
| `SAMMA_IO_JSON` | `{}` | Extra JSON added to every finding |
| `TARGET_ID` | — | samma.io target id, so findings show on that target |
| `WRITE_TO_FILE` | `true` | `true` writes findings to `/out/<PARSER>.json` |
| `NATS_ENABLED` | `False` | `true` publishes every finding to NATS |
| `NATS_URL` | `nats://localhost:4222` | NATS server |
| `NATS_SUBJECT` | `scans` | NATS subject (the operator uses `samma-io.scan`) |

## In Kubernetes

Don't deploy this image by hand. Install the [Samma operator](https://github.com/samma-io/operator)
and pick a profile that includes Tsunami (`classic` or `all`) on an Ingress:

```yaml
metadata:
  annotations:
    samma-io.alpha.kubernetes.io/enable: "true"
    samma-io.alpha.kubernetes.io/profile: "classic"
```

The operator runs the scan once straight away and then weekly. When you delete the Ingress, the
scanners are removed. The [Samma guide](https://github.com/samma-io/guide) covers the whole setup.
