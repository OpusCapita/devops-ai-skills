---
name: kong-acl-403-cache
description: Diagnose Kong gateway 403 "not authorized to access the requested service" that persists after an ACL permission migration was deployed. Use when a user gets 403 through Kong on a new or newly-permitted service although ACL says they're authorized.
---

# Kong 403 after ACL permission migration

## Mechanism
Kong's bouncer (`kong/script/bouncer.lua`) calls ACL `GET /api/isAuthorized/<role>/<service>` for each of the user's roles. It caches each answer in Redis (`redis-kong-main`) as `kongBouncer:<role>:<service>` for **24h**.
If a user hit the service before the permission migration was deployed, the cached `false` stays, and Kong keeps returning 403 even after ACL answers `true`.

ACL has no special case for admin. A role needs a Permission row for `<service>/<resourceGroup>`, added through a data migration in the `acl` repo (`src/server/db/data/permission-NNN.json` + `migrations/NNN-permission.data.main.js`).

## Diagnose
Check the ACL pod logs for `isAuthorized/.../<service>` calls. If there are none since the permission was deployed, Kong is answering from its cache.

## Fix (needs approval, narrowest first)
1. Delete only the matching keys: `kongBouncer:*:<service>` in redis-kong-main.
2. Or `kubectl rollout restart deploy/redis-kong-main`. It has no persistence (`appendonly no`, `save ""`), so this clears the whole cache, including logged-in user data: users may need to log in again.
