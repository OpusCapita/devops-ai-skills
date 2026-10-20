# BN_Infra workspace notes

This folder is a meta-workspace with several related repos. Many tasks need checking more than one repo.

## Prometheus target cheat sheet

- Use Prometheus MCP with `target` parameter when asking for metrics/query help.
- Preferred form:
  - `{"query":"up","target":"bnp-prod-prometheus"}`

- Available targets:
  - BNP (Azure VMs):
    - `bnp-dev-prometheus`
    - `bnp-dev-prometheus-archive`
    - `bnp-dev-vmetrics`
    - `bnp-stage-prometheus`
    - `bnp-stage-vmetrics`
    - `bnp-prod-prometheus`
    - `bnp-prod-vmetrics`
  - WM / messaging (Azure VMs):
    - `wm-stage-prometheus`
    - `wm-prod-prometheus`
  - K8s HAProxy (AKS Kebernetes):
    - `k8s-dev`
    - `k8s-stage`
  - `k8s-prod`

## Jenkins instances and CLI

- Three isolated Jenkins instances are available: `dev`, `stage`, and `prod`.
- The `dev` instance is responsible for CI builds. Each environment's instance is responsible for its own deployment and operations jobs.
- Automatic deploy triggers: As soon as you push changes to Lolly service YAML files in `andariel-platform-jenkins-jobs`, deployment Jenkins jobs are automatically triggered within ~1 minute. You do not need to manually trigger Jenkins deploy jobs after pushing new versions/configs.
- Use `andariel-devops-infra/scripts/jenkins-cli` to inspect Jenkins folders, projects, runs, run details, and console output, or to execute jobs when manual invocation is needed.
- The CLI defaults to `dev`; select another instance with `--env stage` or `--env prod`.
- No port-forward is required: the wrapper executes inside the corresponding Jenkins controller pod through `kubectl`.
- Run `andariel-devops-infra/scripts/jenkins-cli help` for command and usage examples.

## Read local repo instructions too

- After entering any repo under this root, always check whether that repo has its own `AGENTS.md` and read it before running commands there.
- Root `AGENTS.md` gives cross-repo navigation hints; subrepo `AGENTS.md` may contain command safety rules, preferred runners, environment setup, and repo-specific workflow details.
- When asked any questionas about infra read them before doing anything:
  - `andariel-devops-infra/AGENTS.md`
  - `andariel-platform-jenkins-jobs/AGENTS.md`

## Repo map

In large mixed Terraform/Terragrunt folders with many unrelated resources, prefer scoped -target apply for user-requested narrow changes.

- `consul-mock/`
  - App source code.
  - Truth for runtime behavior, config format, CLI flags, Docker image contents.
  - If question about `ENV` vs YAML config, check here first.

- `andariel-platform-jenkins-jobs/`
  - Truth for lolly-based app deployments.
  - If service exists under `_helm/lolly/services/<service>/`, then lolly is configured.
  - If service exists under `infra/lolly/services/<service>/`, treat it as infra-owned lolly config.
  - In this setup, lolly + `andariel-ci-library` generate/apply ArgoCD `Application` resources, so app may use ArgoCD even when no `argocd.yaml` exists in this repo.
  - Old-style per-env `dev/deploy_*`, `stage/deploy_*`, `prod/deploy_*` folders still exist for some services.
  - Important nuance: `lolly.version: 0.0.0` is a deliberate sentinel used to make deployment Jenkins job skip image deployment.

- `andariel-ci-library/`
  - Jenkins shared library.
  - Truth for how lolly/Helm deployment pipeline works.
  - Important file: `src/lib/utils/Platform/Helm.groovy`
    - `lollyArgocd(data)` means lolly writes `argocdapp.yaml` and applies ArgoCD app dynamically.
    - Do not assume missing checked-in ArgoCD manifest means “not using ArgoCD”.

- `andariel-devops-infra/`
  - Infra/Terragrunt repo.
  - Truth for old Terraform/Terragrunt-managed platform resources.
  - If service still exists in `terragrunt/bnp/<env>/15-kubernetes-services/` (or similar), that may be legacy deployment, partially migrated deployment, or stale code after migration.
  - Presence here alone does **not** prove service is still active if lolly migration already happened elsewhere.

- `andariel-ci-jenkins-jobs/`
  - CI jobs repo.
  - Usually relevant for image build/publish jobs, less for runtime deployment state.

### Service branch to environment mapping

- Service repository branch deployments handled by CI/CD pipeline (`andariel-ci-library`):
  - `develop` (and `v*.* /develop*`) → `dev`
  - `release/*` (and `v*.* /release*`) → `stage`
  - `master` (and `v*.* /master*`) → `prod`

## Fast investigation rules

### 1. “Is service using ArgoCD/lolly?”
Check in this order:

1. `andariel-platform-jenkins-jobs/_helm/lolly/services/<service>/`
2. `andariel-ci-library/src/lib/utils/Platform/Helm.groovy`
3. Only then inspect `andariel-platform-jenkins-jobs/infra/argo-apps/` or `tools/argo-apps/`

Reason: ArgoCD app may be generated dynamically by pipeline, not stored as static manifest.

### 2. “Was service migrated from Terraform/Terragrunt?”
Check both:

1. New location: lolly service config in `andariel-platform-jenkins-jobs/_helm/lolly/services/<service>/`
2. Old location: infra code in `andariel-devops-infra/terragrunt/...`

Migration may be partial:
- dev migrated, stage/prod not migrated
- new lolly config present while old infra code still remains in repo

### 3. “Did app switch from ENV vars to YAML config?”
Check app repo first:

1. source code / README in `consul-mock/`
2. deployment config in lolly or Terraform repo

Do not infer app behavior only from deployment manifests.

## Known nuance: consul-mock

- App repo: `consul-mock/`
  - App refactored to YAML config file.
  - `consul_mock.py` states old `CM_*` env vars removed.
  - Default config path: `/etc/consul-mock/config.yaml`

- Lolly/ArgoCD deployment:
  - Dev exists in `andariel-platform-jenkins-jobs/_helm/lolly/services/consul-mock/`
  - If `dev.yaml` exists there, dev uses lolly-generated ArgoCD app.
  - Lack of `stage.yaml` means stage likely not yet migrated.

- Legacy infra:
  - Old Terraform/Terragrunt deployment still exists in `andariel-devops-infra/terragrunt/bnp/<env>/15-kubernetes-services/consul_mock.tf`
  - This may remain after migration; verify latest branch state and lolly files before concluding.

## Practical hints

- Maintain strict structural parity across dev/stage/prod files (terragrunt modules, lolly configs, argo-apps manifests). Verify parity via `diff` when introducing or changing configuration.
- When results look contradictory, `git pull --ff-only` in relevant repos before concluding.
- For cross-repo deployment questions, usually check at least:
  - `andariel-platform-jenkins-jobs/`
  - `andariel-ci-library/`
- sometimes `andariel-devops-infra/`
- Prefer exact service folder searches over broad `argo`/`consul` greps; broad searches create noise.
- When you need explore github use `gh` for any search and lookups (repos, files, issues, discussions, PRs, releases).

## Large platform activities

- Use a persistent runbook for every large or complicated platform change. Record the current state, ownership boundaries, exact commands, observed evidence, gates, rollback, and next action as the work progresses.
- Execute migrations in environment order: dev, then stage, then production. Inventory each environment independently; do not copy names, versions, secrets, or assumptions blindly.
- Treat each environment and resource group as an explicit approval gate. Before every sync, apply, or cutover, capture read-only state and diffs, classify unrelated changes, verify ownership handoff, and obtain explicit authorization for the mutating action.
- Do not advance to the next environment until the previous environment's health, ownership, rollback position, and post-change evidence are recorded in the runbook.
- On any unexpected deletion, replacement, ownership conflict, secret change, degraded dependency, or unexplained plan change: stop, preserve the last known-good state, and update the runbook before proceeding.

## Upstream OSS PRs

- Many upstream OSS repos enforce DCO. For commits that may go upstream, use `git commit -s`.
- If a PR fails only because of missing DCO sign-off on your last local commit, fix with:
  - `git commit --amend -s --no-edit`
  - `git push --force-with-lease origin <branch>`

## Helm validation for external repos

- If `helm` is not available on host, use `andariel-devops-infra/workspace/run.sh`.
- Pattern:
  - `./workspace/run.sh --cwd /absolute/path/to/target/repo -- helm template ...`

## Application removal playbook

- For ArgoCD-managed app removal, prefer this order:
  1. Detect all owning ArgoCD `Application` objects first. There may be more than one app owning same workload/resources.
  2. If an owning app has auto-sync enabled, remove `spec.syncPolicy.automated` from checked-in manifest and push first.
  3. Wait until live app shows manual sync.
  4. Delete owning ArgoCD app with foreground propagation so ArgoCD prunes resources using its own logic.
  5. If app-of-apps recreates empty/manual app from Git, remove app code from Git and push again so app-of-apps removes final app object.
  6. Verify leftovers. Mixed ownership/history drift can still leave orphan resources; delete those manually if needed.
- For Terraform/Terragrunt-managed app removal, remove code and apply with `-target` against exact resource(s). No Git automation is assumed for Terraform repo.
- Always verify across all relevant layers after removal:
  - live `Application` objects
  - live Kubernetes resources in target namespace
  - Terraform/Terragrunt state when legacy infra existed

## ArgoCD CLI usage

- `argocd` login writes to shared `~/.config/argocd/config` and changes `current-context`.
- To work with multiple BNP environments safely, login once per environment (credentials: admin/admin), then run commands with explicit `--argocd-context`.
- Context names currently follow ArgoCD hostnames:
  - dev: `argo-cd.haproxy.dev.bnp.it.opuscapita.com`
  - stage: `argo-cd.haproxy.stage.bnp.it.opuscapita.com`
  - prod: `argo-cd.haproxy.prod.bnp.it.opuscapita.com`
- Example pattern:
  - `./workspace/run.sh --env adev -- argocd --grpc-web login argo-cd.haproxy.dev.bnp.it.opuscapita.com --username admin --password admin`
  - `./workspace/run.sh --env adev -- argocd --grpc-web --argocd-context argo-cd.haproxy.dev.bnp.it.opuscapita.com app list`
  - `./workspace/run.sh --env astage -- argocd --grpc-web --argocd-context argo-cd.haproxy.stage.bnp.it.opuscapita.com app get <app>`
- For parallel work across environments, do not rely on shared `current-context`; always pass `--argocd-context` explicitly on every command.
- Avoid running multiple `argocd login` commands in parallel against same shared config file.

## New app install shortcuts (learned)

- For new Helm/ArgoCD installs in `andariel-platform-jenkins-jobs`, read `HELM-OC.md` early.
- Default ArgoCD namespace is `argo-cd`.
- For infra apps, usual path is `andariel-platform-jenkins-jobs/infra/argo-apps/<app>/1_dev/`.
- If app has separate UI/dashboard, prefer separate neighboring app directory, for example:
  - `infra/argo-apps/<app>/1_dev/`
  - `infra/argo-apps/<app>-dashboards/1_dev/`
- Good examples for Helm-based Argo apps:
  - `infra/argo-apps/pdfinbound/1_dev/argocd.yaml`
  - `infra/argo-apps/consul/1_dev/argocd.yaml`

### Ingress defaults for new installs

- Internal/backend/API endpoint usually goes through `haproxy` ingress class.
- External browser/UI endpoint usually goes through `nginx-external` ingress class.
- Common host patterns by environment:
  - HAProxy: `<app>.haproxy.<env>.bnp.it.opuscapita.com`
  - External UI: `<app>.<env>.bnp.it.opuscapita.com`
- For `nginx-external` TLS, existing secret is usually `cert-wildcard-ext`.
- Standard oauth2-proxy annotations pattern for external UI ingress:
  - `nginx.ingress.kubernetes.io/auth-url: https://<env>.bnp.it.opuscapita.com/oauth2-proxy/auth`
  - `nginx.ingress.kubernetes.io/auth-signin: https://<env>.bnp.it.opuscapita.com/oauth2-proxy/start?rd=$scheme%3A%2F%2F$host$escaped_request_uri`
  - often also `nginx.ingress.kubernetes.io/auth-cache-key: $cookie_oauth2proxy`
- Example oauth2-proxy protected ingress:
  - `andariel-platform-jenkins-jobs/infra/argo-apps/proxy-mfa/3_prod/nagios/ingress.yaml`

### 1Password conventions

- Prefer 1Password over ExternalSecret when user asks for it explicitly.
- Vault used in this workspace is often `Andariel`.
- Common item title pattern for app admin credentials:
  - `bnp-<env>-<app>-admin`
  - example: `bnp-dev-opensearch-logs-admin`
- `OnePasswordItem` manifests commonly use:
  - `apiVersion: onepassword.com/v1`
  - `kind: OnePasswordItem`
  - `spec.itemPath: vaults/Andariel/items/<item-title-or-id>`
- Example patterns:
  - `andariel-platform-jenkins-jobs/dev/deploy_json-router/helm/templates/secret.yaml`
  - `andariel-platform-jenkins-jobs/lolly-base/templates/secret.yaml`

### K8s scheduling defaults often expected for stateful apps

- Persistent/stateful apps that keep data usually belong on data nodes, for example:
  - RabbitMQ
  - SQL databases
  - similar data-storing infra components
- If such workload should run on data nodes, use:
  - `nodeSelector: node.kubernetes.io/role: data`
  - toleration for `DataOnly=true:NoSchedule`
- Storage class defaults by environment:
  - dev: `retain-standard-hdd`
  - stage/prod: `retain-standard-ssd`
  - use premium SSD when user explicitly asks for it or when it is known to help significantly for the workload/use case
- Cost-efficient disk sizes: Azure Managed Disks tier at \(2^n\) boundaries (8, 16, 32, 64, 128, 256, 512, 1024...). Using unaligned sizes (e.g. 5 GiB) incurs charges for the next tier (8 GiB). Always align VM disks and persistent volume requests to power-of-2 numbers.

## BNP troubleshooting

- check logs on `(dev|stage|prod)-logs` VM.
  - Directories under /data/logs/<date>:
    - container - docker containers hosted with docker compose on VMs
    - kubernetes - all pods that are not services
    - services - logs from applications that are developed by our organization (hosted in k8s)
    - nodes - logs from VMs from OS and workloads
- check andariel namespace in kubernetes - all services are deployed there
- check core services (they are dependencies for all other services):
  - auth
  - acl
  - user
  - business-partner
  - kong
- check prometheus for VMs metrics and k8s prometheus for services pods and other pods.

### Elasticsearch & Logs Investigation

Query clusters via `curl -s "http://<ip-or-host>:9200/<index>/_search" | jq .`.

#### Cluster Access Endpoints (dev / stage / prod)
- **Logs Cluster (`elasticsearch-logs`):**
  - VM direct: `(dev|stage|prod)-elasticsearch-logs-01:9200` (e.g. prod: `10.26.19.203:9200`)
- **TNT Cluster (`elasticsearch-tnt`):**
  - HAProxy / k8s: `haproxy-elasticsearch.andariel:9200` (e.g. prod LB: `10.26.32.108:9200`)
  - VM direct: `(dev|stage|prod)-elasticsearch-tnt-01..05:9200` (e.g. prod: `10.26.19.144:9200`)
- **Track Cluster (`elasticsearch-track`):**
  - HAProxy / k8s: `haproxy-elasticsearch-track.andariel:9200` (e.g. prod LB: `10.26.32.117:9200`)
  - VM direct: `(dev|stage|prod)-elasticsearch-track-01..04:9200` (e.g. prod: `10.26.19.148:9200`)
- **Archive Cluster (`elasticsearch-archive`):**
  - HAProxy / k8s: `haproxy-elasticsearch-archive.andariel:9200` (e.g. prod LB: `10.26.32.112:9200`)
  - VM direct: `(dev|stage|prod)-elasticsearch-archive-01..03:9200` (e.g. prod: `10.26.19.132:9200`)

#### `elasticsearch-logs` Index Conventions
- `aks-YYYY.MM.DD`: Kubernetes platform & infra pods (all pods except business services).
- `services-YYYY.MM.DD`: Business service pods deployed with label `tag:<service-name>`.
- `ingress-YYYY.MM.DD`: Ingress traffic logs from AKS.
- `aks-events-YYYY-MM-DD`: Kubernetes cluster events (from event-exporter).
- `containers-YYYY.MM.DD`: Docker container logs from Azure VMs (Docker Compose).
- `filebeat-YYYY.MM.DD` / `journalbeat-YYYY.MM.DD`: VM OS, systemd, and host workload logs.
- `wm-logs-YYYY.MM.DD`, `wm-filebeat-YYYY.MM.DD`, `wm-journalbeat-YYYY.MM.DD`: webMethods (Sirius) platform logs.

#### Specialized Elasticsearch Clusters (for investigations)
- **`tnt`** (`elasticsearch-tnt`):
  - Index pattern: `bn_tx_logs-YYYY.MM.DD`
  - Content: Raw TNT event stream with `event.transactionId`, `event.eventId`, `event.stepType`, `event.severity`, etc.
- **`track`** (`elasticsearch-track`):
  - Index pattern: `bn_tx_agg_event_sel_fields-YYYY.MM.DD`
  - Content: Aggregated transaction records indexed by `_id: <transactionId>` containing full chronological `events[]` array and document metadata.
- **`archive`** (`elasticsearch-archive`):
  - Index patterns: `archive_invoice_receiving-YYYY[-new]`, `archive_invoice_sending-YYYY[-new]`, `archive_ipa_invoice-YYYY`, `bn_dayreports-YYYY`, `bn_dayreports_sla-YYYY`
  - Content: Long-term archived transaction and event records.

### Quick note: Kafka lag for k8s service backed by MySQL

- Example: `track-stats` consumer lag on topic `pub.tnt.stats`
- First find env and workload:
  - `kubectl --context bnp-<env> -n andariel get deploy,rs,pods | grep 'track-stats'`
  - `kubectl --context bnp-<env> -n andariel top pod | grep '^track-stats-'`
  - `kubectl --context bnp-<env> -n andariel get events --sort-by=.lastTimestamp | grep 'track-stats'`
- Check k8s metrics in `k8s-<env>`:
  - `up{namespace="andariel",pod=~"track-stats-.*"}`
  - `kube_pod_status_ready{namespace="andariel",pod=~"track-stats-.*",condition="true"}`
  - `container_memory_working_set_bytes{namespace="andariel",pod=~"track-stats-.*",container="track-stats"}`
  - `rate(container_cpu_usage_seconds_total{namespace="andariel",pod=~"track-stats-.*",container="track-stats"}[5m])`
- Find related MySQL VMs in `andariel-devops-infra/terragrunt/bnp/<env>/mysql-track-stats/terragrunt.hcl` and via `./workspace/run.sh --env a<env> -- azvms | grep 'mysql-track-stats'`
- Check VM/MySQL metrics in `bnp-<env>-prometheus`:
  - `up{instance=~".*mysql-track-stats.*"}`
  - `mysql_up{instance=~".*mysql-track-stats.*"}`
  - `100 - (avg by(instance) (irate(node_cpu_seconds_total{instance=~".*mysql-track-stats.*",mode="idle"}[5m])) * 100)`
  - `100 * (1 - (node_memory_MemAvailable_bytes{instance=~".*mysql-track-stats.*"} / node_memory_MemTotal_bytes{instance=~".*mysql-track-stats.*"}))`
  - `rate(mysql_global_status_queries{instance=~".*mysql-track-stats.*"}[5m])`
  - `rate(mysql_global_status_slow_queries{instance=~".*mysql-track-stats.*"}[5m])`
  - `mysql_global_status_wsrep_ready{instance=~".*mysql-track-stats.*"}`
  - `mysql_global_status_wsrep_cluster_size{instance=~".*mysql-track-stats.*"}`
- If pods healthy now but events show recent recreate / readiness failures, suspect rollout churn as lag cause.
- If one MySQL node shows much higher CPU or disk throughput than peers, suspect DB-side bottleneck on writer/primary.


### Jira

- Author Jira descriptions and comments as Markdown.
- Markdown is not supported by jira directly, so use `jira-adf.sh` wrapper to convert Markdown (code is in ~/.local/bin):
  - `jira-adf.sh create --project BND --type Task --summary "..." --description-file [description.md|-]`
  - `jira-adf.sh comment --key BND-123 --body-file [comment.md|-]`
- Use `-` to read Markdown from stdin.
- Pass extra `acli` flags after `--`.
When you need Jira or Confluence access use `acli`. Examples: `acli jira workitem view [key]`; to list assigned tickets: `acli jira workitem search -f "key,status,summary" --csv --jql "assignee = currentUser() AND resolution = Unresolved order by updated DESC"`.
I have additional scripts, prefer them over acli directly: `jira-workitem.sh <key>` and `jira-workitem-links.sh <key>`. Use this to get all help needed instantly: `acli jira workitem create --help; acli jira board --help; acli jira workitem edit --help`. When asked to create a ticket, assign it to `@me`, add it to the board, and set TODO status. Default project: BND, default board id: 441. Create it and capture the response: `issue_json=$(jira-adf.sh create --project BND --type Task --summary "Title..." --description-file description.md -- --assignee @me --json)`. Extract the key: `issue_key=$(printf '%s' "$issue_json" | jq -r '.key // .id // empty')`; verify it is non-empty before continuing. Then run `jira-board-ticket.sh to-board 441 "$issue_key"` and `acli jira workitem transition --key "$issue_key" --status "To Do" --yes --json`. In user-facing output, render BND issue keys as links, for example `[BND-1928](https://opuscapita.atlassian.net/browse/BND-1928)`.
