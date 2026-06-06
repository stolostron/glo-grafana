# Architecture — glo-grafana (Global Hub)

**glo-grafana** is a Red Hat–maintained fork of upstream Grafana, packaged for Multicluster Global Hub. The Global Hub **operator** deploys this image on the global hub cluster alongside PostgreSQL, Kafka, and the manager. Dashboards query fleet-wide policy compliance and inventory data stored in the global hub database.

Related: [multicluster-global-hub docs/ARCHITECTURE.md](https://github.com/stolostron/multicluster-global-hub/blob/release-5.0/docs/ARCHITECTURE.md) for the full operator/manager/agent data flow.

---

## Role in Global Hub

| Aspect | Detail |
|---|---|
| **Deployed by** | `MulticlusterGlobalHub` operator reconciler (`operator/pkg/controllers/grafana`) |
| **Runs on** | Global hub cluster, namespace `multicluster-global-hub` |
| **Data source** | PostgreSQL (`multicluster-global-hub-postgresql`) — schemas `local_spec`, `local_status`, `status`, `history` |
| **Access** | OpenShift OAuth proxy in front of Grafana; auth headers forwarded to datasources |
| **Image** | `registry.redhat.io/multicluster-globalhub/grafana-rhel9` (prod) / Konflux dev workload quay path |

Grafana is **not** a sync participant — it is a read-only observability UI over data the manager persists.

---

## ACM Customizations

### `stolostron-patches/`

Patches are applied at **image build time** in `Containerfile.konflux` (`git apply ./stolostron-patches/*`).

| Patch | Purpose |
|---|---|
| `0001-Forward-headers-from-auth-proxy-to-datasource.patch` | Forwards OpenShift OAuth-proxy authentication headers from the browser session to configured PostgreSQL (and other) datasources, enabling SSO-backed queries without separate Grafana login |

Datasource JSON can list headers to forward via `jsonData.forwardHeaders`.

### FIPS / Konflux build

Production builds use:

```bash
go run build.go -cgo-enabled -build-tags=strictfipsruntime build
```

CGO is required for Konflux FIPS binary scanning (`fbc-fips-check-oci-ta`).

---

## Container Image

`Containerfile.konflux` is a two-stage build:

1. **Builder** — `brew.registry.redhat.io/rh-osbs/openshift-golang-builder:rhel_9_1.25`, applies patches, compiles Grafana binaries.
2. **Runtime** — `registry.access.redhat.com/ubi9/ubi-minimal`, non-root `grafana` user, port 3000, entrypoint `packaging/docker/run.sh`.

Labels identify component `multicluster-globalhub-grafana-rhel9`, CPE `cpe:/a:redhat:multicluster_globalhub:5.0::el9`, version `release-5.0`.

Local build:

```bash
podman build -f Containerfile.konflux -t glo-grafana:local .
```

---

## CI/CD (Konflux)

| Pipeline | Trigger | File |
|---|---|---|
| Push | `release-5.0` branch push | `.tekton/glo-grafana-globalhub-5-0-push.yaml` |
| Pull request | PRs to `release-5.0` | `.tekton/glo-grafana-globalhub-5-0-pull-request.yaml` |

- **Application:** `release-globalhub-5-0`
- **Component:** `glo-grafana-globalhub-5-0`
- **Output:** `quay.io/redhat-user-workloads/acm-multicluster-glo-tenant/glo-grafana-globalhub-5-0:<sha>`
- **Platforms:** linux/x86_64, ppc64le, s390x, arm64
- **Pipeline:** `konflux-build-catalog/pipelines/common-base.yaml` (hermetic, prefetch gomod)

---

## Branch Strategy

| Branch | Purpose |
|---|---|
| `release-5.0` | Current Global Hub 5.0 development and Konflux builds |
| `release-1.x` | Prior GH release tracks (maintenance / backport source) |

Feature work for the 5.0 line lands on `release-5.0`. Sync branches (e.g. `sync/release-1.8-into-5.0`) merge prior-track fixes forward.

---

## Local Development

Typical workflow for backend-only changes:

```bash
make deps-go
make build-go
./bin/grafana server
```

Frontend changes require `make deps-js` and `make build-js` (or `make run` for watch mode). Full upstream Grafana contributor docs apply for deep frontend/plugin work.

---

## Testing in Global Hub E2E

The multicluster-global-hub repo runs Grafana-focused E2E via:

```bash
make e2e-test-grafana
make e2e-log/grafana
```

See [acmqe-hoh-e2e](https://github.com/stolostron/acmqe-hoh-e2e) for Polarion-tagged regression suites that exercise dashboard availability post-install.
