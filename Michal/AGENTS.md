# AGENTS.md (Michal)

Global rules I give my coding agents for DevOps/infra work. Copy into `~/.config/opencode/AGENTS.md` (or `~/.claude/CLAUDE.md`) and adapt.

## Kubernetes
- Always pass the context explicitly: `kubectl --context <ctx>`, `helm --kube-context <ctx>`. Never rely on the default context.
- **Prod: read-only** (get, describe, logs, top, events, `helm list/get/status`) unless I explicitly ask for a specific prod change in the current conversation. That covers apply, patch, edit, delete, scale, rollout restart, exec, helm upgrade and rollback.
- Stage/shared clusters: say what a change will do before running it.
- Preview first: `--dry-run=server`, `kubectl diff`, `helm diff`.
- Pick the restart or change with the smallest blast radius.

## Azure
- Don't run `az login`, `az logout` or `az account set` (they change the shared login). Pass `--subscription` on each command.

## Git
- Don't run `git config --global`. Use `git -c` or repo-local config.

## Terraform
- Only run targeted applies (`-target=...`) in shared stacks unless I ask otherwise. An untargeted apply can provision resources nobody expected.
- I run commits, applies and VM changes myself. The agent verifies read-only and proposes the commands.

## Incident hygiene
- Servers run UTC and workstations often don't. Pin every incident timeline to one clock, and make monitoring scripts print the remote `date -u`.
- Sample for minutes, not seconds, before calling a trend ("stuck", "frozen", "recovered").

## Context
- OpusCapita is part of GEP. GEP APIs and platforms are intra-group services, not external vendors.
