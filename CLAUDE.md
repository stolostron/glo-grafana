# CLAUDE.md — glo-grafana

AI assistant context for **glo-grafana** — the Red Hat fork of Grafana used by Multicluster Global Hub for observability dashboards backed by the global hub PostgreSQL database.

---

## Build, Test, and Lint Commands

```bash
# Install dependencies
make deps              # backend + frontend
make deps-go           # Go modules only
make deps-js           # yarn frontend only

# Local development build
make build             # backend + frontend
make build-go          # Go binaries only
make build-js          # frontend assets only
make run-go            # build and run server immediately

# Run Grafana locally (after build)
./bin/grafana server

# Tests
make test-go-unit
make test-js
make test

# Konflux / production container image
podman build -f Containerfile.konflux -t glo-grafana:local .
```

> Konflux builds use `Containerfile.konflux`: applies `stolostron-patches/`, builds with `-build-tags=strictfipsruntime` and CGO enabled for FIPS compliance, publishes multi-arch images via `.tekton/`.

---

## Repo Layout

```text
Containerfile.konflux     Production multi-stage container build (Konflux / brew)
stolostron-patches/       ACM-specific patches applied at image build time
.tekton/                  Konflux PipelineRuns (push + pull-request)
pkg/                      Grafana backend (services, plugins, APIs)
public/                   Frontend application
apps/                     Grafana App SDK apps
conf/                     Sample grafana.ini and defaults
packaging/docker/         Container entrypoint (run.sh)
```

---

## Architecture

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for Global Hub integration, auth-proxy header forwarding, datasource configuration, Konflux CI, and deployment context.

Upstream Grafana docs remain in [README.md](README.md) and [docs/](docs/).

---

## Personal Config

Read `.claude/user.local.md` at the start of any task that needs an assignee, email, or project key. If the file does not exist, fall back to Claude memory (`user-config`), then placeholders.

---

## Fleet Engineering Skills

Fetch and apply the relevant skill when the task matches its domain.

| Skill | When to use |
|---|---|
| [start-work](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/sdlc/start-work/SKILL.md) | Begin work on a Jira ticket — creates sub-task, transitions status |
| [finish-work](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/sdlc/finish-work/SKILL.md) | Commit, push, open PR, update Jira |
| [pr-review](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/sdlc/pr-review/SKILL.md) | Review a GitHub PR with worktree isolation and inline comments |
| [pr-fix](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/sdlc/pr-fix/SKILL.md) | Fix blocked PRs: merge conflicts, CI failures, review comments |
| [jira-specialist](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/jira/jira-specialist/SKILL.md) | Triage, search, link, or transition Jira issues |
| [bug-specialist](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/jira/bug-specialist/SKILL.md) | Create a well-structured bug report with reproduction steps |
| [story-specialist](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/jira/story-specialist/SKILL.md) | Create a user story with acceptance criteria |
| [spike-specialist](https://raw.githubusercontent.com/OpenShift-Fleet/agentic-sdlc/main/skills/jira/spike-specialist/SKILL.md) | Time-boxed research and PoC tickets |

---

## Key Dependencies

| Dependency | Version | Purpose |
|---|---|---|
| Go | 1.25.7 | Backend toolchain (see Makefile) |
| Node / Yarn | (see package.json) | Frontend build |
| UBI 9 minimal | latest | Runtime base image |
| openshift-golang-builder | rhel_9_1.25 | Konflux build stage |

---

## Environment Variables

Standard Grafana path variables (set in `Containerfile.konflux`):

| Variable | Default | Purpose |
|---|---|---|
| `GF_PATHS_CONFIG` | `/etc/grafana/grafana.ini` | Main config file |
| `GF_PATHS_DATA` | `/var/lib/grafana` | Data directory |
| `GF_PATHS_HOME` | `/usr/share/grafana` | Install root |
| `GF_PATHS_LOGS` | `/var/log/grafana` | Log directory |
| `GF_PATHS_PLUGINS` | `/var/lib/grafana/plugins` | Plugin directory |
| `GF_PATHS_PROVISIONING` | `/etc/grafana/provisioning` | Provisioning configs |

Container runs as user `grafana` (UID/GID 472). Exposes port **3000**.
