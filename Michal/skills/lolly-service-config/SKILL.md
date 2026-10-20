---
name: lolly-service-config
description: Where to put configuration for a new lolly-deployed service (helm `_helm/lolly/services/<svc>`). Use when adding a new lolly service, migrating a service to lolly, or deciding between Consul keys and env vars for service config.
---

# Config for new lolly services: envs, not Consul

- Put service-specific config (ES host/port, feature settings, etc.) in `lolly.commonEnv` in `_helm/lolly/services/<svc>/all.yaml`, and env-specific values in `<env>.yaml`.
- Don't add custom keys under `autodeploy.consul`. Keep `autodeploy.consul: {}`.
- Leave the shared Consul setup as it is: the deploy job copies `default/` (mq, redis) and `auth_svc.py` writes `service-client/*`.

**Why:** We're moving away from Consul. If service code still reads custom values from Consul, the developer rewrites it to read envs, and the deployment already provides them.

**How:** Grep the service code for `config.getProperty` / `getEndPoint`. Turn each service-specific read into an env in the lolly yaml, then tell the developer which env names to read.
