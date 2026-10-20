---
name: helm-uninstall-stuck-finalizers
description: Fix `helm uninstall` hanging with a Deployment stuck in Terminating because of stale operator finalizers (autovpa / kopf). Use when removing legacy (non-lolly) helm releases or when any resource stays in Terminating.
---

# helm uninstall hangs: stale finalizers

Legacy (non-lolly) Deployments can carry finalizers from the autovpa operator, which no longer runs:
- `autovpa.autoscaling.k8s.io/finalizer`
- `kopf.zalando.org/KopfFinalizerMarker`

No controller removes them, so `helm uninstall` leaves the Deployment stuck in Terminating. Lolly-managed deployments don't have them.

## Steps
1. Before uninstalling, check: `kubectl --context <ctx> -n <ns> get deploy <name> -o jsonpath='{.metadata.finalizers}'`
2. Confirm the operator that owns the finalizer really isn't running.
3. With approval, clear them:
   `kubectl --context <ctx> -n <ns> patch deploy <name> --type=merge -p '{"metadata":{"finalizers":null}}'`
4. Re-run or wait for the uninstall.

Note: the CD trigger (`list_services.py` in andariel-ci-jenkins-jobs) skips services not present in the env, so removing a service doesn't need `[skip ci]`.
