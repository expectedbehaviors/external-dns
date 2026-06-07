# external-dns

Helm chart wrapping [Bitnami external-dns](https://github.com/bitnami/charts/tree/main/bitnami/external-dns). One provider per Helm release.

## Features

- Bitnami external-dns 7.5.6 (app 0.15.0 via image tag override)
- Optional OnePasswordItem integration for provider credentials
- Configurable sources, domain filters, and sync policy

## Requirements

- Kubernetes cluster with ingress (or service) sources you want published to DNS
- Provider credentials in a Secret (name configurable via values)
- DNS provider API access (e.g. DigitalOcean, Pi-hole, Cloudflare — set `provider` in values)

## Configuration

Key values under `external-dns`:

| Key | Purpose |
|-----|---------|
| `provider` | DNS provider (e.g. `digitalocean`, `pihole`, `cloudflare`) |
| `domainFilters` | Domains ExternalDNS manages |
| `excludeDomains` | Records never created, updated, or deleted |
| `sources` | `[ingress]` and/or `[service]` |
| `policy` | `sync`, `upsert-only`, or `create-only` |
| `digitalocean.secretName` | Secret containing provider API token (when using DigitalOcean) |

Optional `onepassworditem` block syncs credentials from 1Password when enabled.

## Baseline values

Placeholder values only (`example.com`, generic secret names). Override provider, domains, secrets, and credential sources in your deployment values.

## Upstream documentation

| Resource | Link |
|----------|------|
| External DNS | https://kubernetes-sigs.github.io/external-dns/ |
| Bitnami chart | https://github.com/bitnami/charts/tree/main/bitnami/external-dns |
