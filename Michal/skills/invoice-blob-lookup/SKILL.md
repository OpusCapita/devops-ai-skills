---
name: invoice-blob-lookup
description: Locate an IPA/invoice tenant's blob files (attachments, documents) in Azure storage and check them. Use when debugging missing/broken attachments, "Failed to archive attachment", or when you need to download a tenant's blob.
---

# Find a tenant's blobs in Azure storage

- Storage account name: read it from Consul key `invoice/blob/account` for the environment.
- Container: `md5(tenantId without the c_ prefix)` + `-data`
  ```bash
  echo -n "MyTenant" | md5sum   # tenant c_MyTenant -> <md5>-data
  ```
- Blob name: the path after `/blob/api/<tenant>/data/`, without a leading slash.
- Don't call the blob service's `POST /api/<tenant>/data/containerName`. It returns 403 without a service token, so compute the md5 yourself.
- Access: `az storage blob list|download --auth-mode key --account-name <acct> --container-name <md5>-data --subscription "<sub>"`

Known cause: "Failed to archive attachment" was traced to **0-byte blobs**. Check `contentLength` first.
