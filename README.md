# external-dns (Helm wrapper)

Wrapper chart that deploys **two** Bitnami [external-dns](https://github.com/bitnami/charts/tree/main/bitnami/external-dns) subcharts:

- **pihole** — Internal DNS (Pi-hole API); sources: ingress + service; uses Pi-hole nameservers and `pihole-external-dns` secret.
- **digitalocean** — Public DNS (DigitalOcean API); sources: ingress only; uses `digitalocean` secret.

Core installations (including this chart) are covered by the setup script for now. Argo CD / nginx may be used later.

## Policy: Pi-hole and DigitalOcean

Both use **policy: sync** — **create**, **update**, and **delete** records. When you remove an ingress or service, ExternalDNS removes the corresponding DNS record (cleanup). Root and manual records are in `excludeDomains` so they are never touched. When adding new static/manual records in Pi-hole or DO, add them to `excludeDomains` so ExternalDNS never deletes them.

## Values layout

- **defaults** — Shared settings: `domainFilters`, `clusterDomain`, `logLevel`, `sources`, `policy: sync`, etc.
- **pihole** — `excludeDomains` (root + manual), Pi-hole server/secret, ingress + service sources.
- **digitalocean** — `excludeDomains` (root + manual), DO secret, ingress-only sources.

## Key values

| Key | Purpose |
|-----|--------|
| `domainFilters` | Domains ExternalDNS manages (e.g. `expectedbehaviors.com`). |
| `excludeDomains` | Root + manual/static records; never created, updated, or deleted by ExternalDNS. |
| `policy` | **sync** (both): create/update/delete when source goes away. |
| `sources` | `[ingress]` or `[ingress, service]` depending on provider. |

## Version (what’s updated and what isn’t)

- **Is 7.5.6 latest?** **No.** 7.5.6 is the latest on the **7.x** line. The **overall latest** Bitnami external-dns chart is **9.x** (e.g. 9.0.3), which ships **ExternalDNS app 0.18.0** and uses the Bitnami **common** OCI subchart (different structure and values).
- **What we use:** Bitnami **external-dns chart 7.5.6** (both subcharts). We override **image.tag** in values to **0.15.0-debian-12-r1** so we run **ExternalDNS app 0.15.0** (newer than the 0.14.2 the chart ships; compatible, no chart change required).
- **Why we’re not further (why not 9.x)?** Moving to 9.x would mean: (1) a **new dependency** on the Bitnami common OCI chart (`oci://registry-1.docker.io/bitnamicharts/common`), (2) **values and template changes** (common subchart renames/restructures some keys), and (3) **app jump** to 0.18.0. We’d need to run the 9.x chart, diff values, and test both providers (Pi-hole + DigitalOcean) before switching. We stayed on **7.5.6** for **minimal risk**: same 7.x major, no breaking change, no migration. To go further you’d upgrade to 9.x in a branch, adapt values to the new structure, and validate.
- **Summary:** Chart = 7.5.6 (latest on 7.x). App = 0.15.0 (pinned in values). Overall latest = 9.x (0.18.0); we’re on 7.x to avoid 9.x breaking changes until we’re ready to test and migrate.

## Upgrading the Bitnami dependency

1. In `Chart.yaml`, set both `external-dns` dependencies to the desired version (e.g. `7.5.7` when available).
2. Run `helm dependency update` in this directory.
3. Commit `Chart.lock` and the updated `charts/` (if not gitignored).
4. Update the wrapper `Chart.yaml` `images` and `appVersion` to match the locked Bitnami chart.

## Future: Hardening

Once the current deployment is stable, consider:

- **Resource requests/limits:** Set `defaults.resources` (or per-provider under `pihole.resources` / `digitalocean.resources`) with `requests` and `limits` for CPU/memory so scheduling is predictable and usage is bounded. The Bitnami chart defaults to no resources; we expose `resources: {}` in defaults so you can override.
- **Security context:** The Bitnami chart already sets a hardened container security context (non-root, read-only root filesystem, drop all capabilities, seccomp). If you need to tighten further (e.g. pod security standards on the namespace), apply that at the cluster level; no chart change required for the current defaults.

## Future: Metrics (Prometheus)

When Prometheus (and optionally the Prometheus Operator) is available:

- Set **`defaults.metrics.enabled: true`** (or per-provider) to expose the ExternalDNS metrics endpoint.
- If using the Prometheus Operator, set **`defaults.metrics.serviceMonitor.enabled: true`** and configure **`metrics.serviceMonitor.labels`** so your Prometheus instance selects these ServiceMonitors (e.g. `release: prometheus` or your operator’s selector).
- Redeploy or sync the release. No change is required until you want to scrape ExternalDNS metrics; options are already in `values.yaml` (disabled by default).
