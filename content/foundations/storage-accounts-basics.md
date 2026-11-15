+++
slug = "storage-accounts-basics"
title = "Storage accounts basics"
summary = "What an Azure storage account is: account types, the four data services (blob, file, queue, table), the three ways in (access keys, SAS, Microsoft Entra ID with data-plane RBAC roles), public network access, redundancy in one line, and encryption at rest."
order = 13
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/storage/common/storage-account-overview",
  "https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction",
  "https://learn.microsoft.com/en-us/azure/storage/common/authorize-data-access",
  "https://learn.microsoft.com/en-us/azure/storage/common/storage-account-keys-manage",
  "https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview",
  "https://learn.microsoft.com/en-us/azure/storage/blobs/authorize-data-operations-portal",
  "https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles/storage",
  "https://learn.microsoft.com/en-us/azure/storage/blobs/anonymous-read-access-prevent",
  "https://learn.microsoft.com/en-us/azure/storage/common/storage-network-security-set-default-access",
  "https://learn.microsoft.com/en-us/azure/storage/common/storage-redundancy",
  "https://learn.microsoft.com/en-us/azure/storage/common/storage-service-encryption",
]
captured = 2026-11-15
related_objectives = ["D2.1.1", "D2.1.2", "D2.1.3", "D2.1.4", "D2.3.6", "D1.3.8"]
+++

## In one sentence

An **Azure storage account** is a named, internet-addressable box in Azure that holds four kinds of data — **blobs** (files/objects), **file shares**, **queues** and **tables** — and every request to it must prove who it is with a key, a signed link, or a Microsoft Entra identity.

## Why it exists

Every cloud app needs somewhere to put files, logs, backups and messages. A storage account is that place. It is also the most common thing attackers find left open: an anonymous container, a leaked access key, a SAS link that never expires.

Picture a **warehouse with a front office**. The warehouse is the storage account. The front office checks every visitor. Visitors can show a **master key** (the access key — opens everything), a **signed delivery note** (a SAS — opens one shelf, for a while), or an **employee badge** (an Entra identity, checked against a role list). Separately, a **gate** decides which roads can even reach the front office (network access). Securing storage means controlling all four.

## Mental model

```svg
<svg viewBox="0 0 780 350" role="img" aria-label="Storage account contains blob, file, queue and table services. Requests pass a network gate first, then one of three authorization methods: access key, shared access signature, or Microsoft Entra ID with an RBAC data role">
  <defs>
    <marker id="sabarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="line-accent"/>
    </marker>
  </defs>
  <rect class="box" x="20" y="140" width="130" height="60" rx="8"/>
  <text x="85" y="166" text-anchor="middle">Client</text>
  <text x="85" y="186" text-anchor="middle" class="muted">app, user, VM</text>
  <rect class="box-risk" x="190" y="125" width="150" height="90" rx="8"/>
  <text x="265" y="152" text-anchor="middle" class="label-risk">Network gate</text>
  <text x="265" y="176" text-anchor="middle" class="muted">public access on /</text>
  <text x="265" y="196" text-anchor="middle" class="muted">selected / disabled</text>
  <line class="line-accent" x1="150" y1="170" x2="188" y2="170" marker-end="url(#sabarrow)"/>
  <rect class="box" x="385" y="30" width="170" height="60" rx="8"/>
  <text x="470" y="56" text-anchor="middle">Access key</text>
  <text x="470" y="76" text-anchor="middle" class="muted">full control</text>
  <rect class="box" x="385" y="140" width="170" height="60" rx="8"/>
  <text x="470" y="166" text-anchor="middle">SAS token</text>
  <text x="470" y="186" text-anchor="middle" class="muted">limited, time-boxed</text>
  <rect class="box-ok" x="385" y="250" width="170" height="60" rx="8"/>
  <text x="470" y="276" text-anchor="middle">Entra ID + role</text>
  <text x="470" y="296" text-anchor="middle" class="muted">recommended</text>
  <line class="line" x1="340" y1="150" x2="383" y2="65" marker-end="url(#sabarrow)"/>
  <line class="line" x1="340" y1="170" x2="383" y2="170" marker-end="url(#sabarrow)"/>
  <line class="line" x1="340" y1="190" x2="383" y2="275" marker-end="url(#sabarrow)"/>
  <rect class="box-accent" x="600" y="30" width="160" height="280" rx="8"/>
  <text x="680" y="58" text-anchor="middle" class="label-accent">Storage account</text>
  <text x="680" y="105" text-anchor="middle">Blob</text>
  <text x="680" y="155" text-anchor="middle">File</text>
  <text x="680" y="205" text-anchor="middle">Queue</text>
  <text x="680" y="255" text-anchor="middle">Table</text>
  <line class="line-accent" x1="555" y1="60" x2="598" y2="100" marker-end="url(#sabarrow)"/>
  <line class="line-accent" x1="555" y1="170" x2="598" y2="170" marker-end="url(#sabarrow)"/>
  <line class="line-accent" x1="555" y1="280" x2="598" y2="240" marker-end="url(#sabarrow)"/>
</svg>
```

![Relationship between a storage account, containers and blobs](https://learn.microsoft.com/en-us/azure/storage/blobs/media/storage-blobs-introduction/blob1.png){Inside the blob service: account → container → blob. A container is like a folder}

## What actually matters

**1. Account types and services.** Pick the type at creation; ==!you can't change it later==.

| Type | Services | Use |
|---|---|---|
| **Standard general-purpose v2** (`StorageV2`) | Blob (incl. Data Lake), Queue, Table, Files | Default for almost everything |
| **Premium block blobs** | Blob only | Low latency, many small objects |
| **Premium file shares** | Files only (SMB and NFS) | Enterprise file shares |
| **Premium page blobs** | Page blobs only | Disk-like random I/O |

Each service has its own endpoint: `https://<account>.blob.core.windows.net`, `.file.`, `.queue.`, `.table.`. Account names are 3–24 lowercase letters and numbers, unique across all of Azure. That public name is why private endpoints need DNS work later (see `private-dns-and-name-resolution`).

**2. Access keys (Shared Key).** Every account has **two** 512-bit keys, **key1** and **key2**. ==!Either key gives full access to all data== and can sign SAS tokens. Two keys exist so you can rotate one while apps use the other. Store them in Key Vault, rotate them, set a **key expiration policy** (a reminder interval), and ideally set **Allow storage account key access = Disabled**. Microsoft Entra Conditional Access only protects an account once Shared Key is disallowed.

Who can read the keys? Any role with `Microsoft.Storage/storageAccounts/listkeys/action`: **Owner**, **Contributor**, **Storage Account Contributor**, **Reader and Data Access**. ==!Plain Reader cannot.==

**3. Shared access signatures (SAS).** A SAS is a token appended to a URL that grants **specific permissions, on specific resources, for a specific time**. Three types:

| SAS type | Signed with | Scope | Stored access policy? |
|---|---|---|---|
| **User delegation SAS** | A user delegation key from **Microsoft Entra ID** | Blob (see contradiction below) | No |
| **Service SAS** | Account key | One service | ==+Yes== |
| **Account SAS** | Account key | One or more services, plus service-level operations | No |

- A **stored access policy** sits on a container, share, queue or table. A service SAS that references it inherits its start, expiry and permissions. Delete or change the policy and every SAS tied to it is **revoked** — without rotating keys. Limit: **five** per container.
- Revoking an *ad hoc* key-signed SAS means **rotating the key** that signed it.
- Azure Storage doesn't track SAS tokens and can't audit their creation.
- Microsoft recommends **user delegation SAS** whenever a SAS is needed. Creating one needs the `generateUserDelegationKey` action — the **Storage Blob Delegator** role, or any Storage Blob Data role.

> **Two Learn articles disagree.** The SAS overview says a user delegation SAS works for Blob, Queue, Table and Azure Files. The "Authorize access to data" article says it ==!applies to blobs only== and lists it as unsupported for Files, Queue and Table. Until they agree, answer "user delegation SAS = Blob Storage (and Data Lake)", which both pages support.

**4. Microsoft Entra ID with data-plane roles.** This is Microsoft's recommended way in — especially with **managed identities**. Azure roles split into two planes:

| Plane | Example roles | Lets you |
|---|---|---|
| **Control (management) plane** | Owner, Contributor, Reader, Storage Account Contributor | Create, configure, delete the *account* |
| **Data plane** | **Storage Blob Data Owner / Contributor / Reader**, **Storage Queue Data Contributor / Reader**, **Storage Table Data Contributor / Reader**, **Storage File Data SMB Share Reader / Contributor / Elevated Contributor** | Read and write the *data* |

==!Owner does not give data access through Entra ID.== It can still reach data by reading the keys. To browse blobs in the portal with your Entra account you need **Reader** (to see the account) **plus** a Storage Blob Data role.

**5. Public network access.** By default an account accepts traffic from any network. Portal choices: **Enabled from all networks**, **Enabled from selected networks** (virtual networks, IP ranges, resource instances, trusted Azure services — firewall default action **Deny**), **Disabled** (only private endpoints), or **Secured by perimeter** (network security perimeter). Network rules ==!only take effect when the default action is Deny==. Separately, **Allow Blob anonymous access** must stay off: anonymous read lets anyone read a public container with no credentials at all.

**6. Redundancy, in one line.** **LRS** = 3 copies in one datacenter; **ZRS** = 3 copies across availability zones; **GRS/GZRS** add a copy in a paired region; **RA-** versions let you read that secondary copy.

**7. Encryption at rest.** All data is encrypted with **256-bit AES**, always. ==!You can't turn it off.== Keys are **Microsoft-managed** by default; you can switch to **customer-managed keys** in Key Vault for control and revocation (D2.1.1). Secure transfer (HTTPS only) and a minimum TLS version protect data in transit.

## Where it lives

**Azure portal > Storage accounts > *account*:**

- `Security + networking > Networking` — public network access, virtual networks, IP rules, exceptions, private endpoint connections.
- `Security + networking > Access keys` — view and rotate key1/key2; *Set rotation reminder*.
- `Security + networking > Shared access signature` — create an account SAS.
- `Settings > Configuration` — *Allow storage account key access*, *Allow Blob anonymous access*, *Secure transfer required*, *Minimum TLS version*.
- `Security + networking > Encryption` — Microsoft-managed vs customer-managed keys.
- `Access control (IAM)` — assign data-plane roles.

```bash
az storage account update -n mystg -g rg --allow-shared-key-access false --allow-blob-public-access false
az storage account update -n mystg -g rg --public-network-access Disabled
az storage account keys renew -n mystg -g rg --key key1
az role assignment create --assignee <objectId> --role "Storage Blob Data Reader" --scope <accountId>
```

![Viewing storage account access keys in the Azure portal](https://learn.microsoft.com/en-us/azure/storage/common/media/storage-account-keys-manage/portal-connection-string.png){Access keys — two keys, each with full access. Rotate one at a time}

![Components of a resource URI with a SAS token](https://learn.microsoft.com/en-us/azure/storage/common/media/storage-sas-overview/sas-storage-uri.svg){A SAS is just query parameters on a URL — permissions, start, expiry, signature}

## Exam traps

- "Least privilege for an app to read blobs" → managed identity + ==+Storage Blob Data Reader==, not an access key and not Reader.
- "Revoke a SAS without rotating keys" → it must be a **service SAS tied to a stored access policy**. Account SAS and user delegation SAS can't use one.
- "Most secure SAS" → **user delegation SAS** (Entra-backed, no account key).
- Contributor and Storage Account Contributor can read data **through the keys**. Disable Shared Key to close that path.
- Network rules do nothing while the default action is **Allow**.
- Storage encryption at rest is **always on**; the choice is only *who manages the key*.
- Account type is fixed at creation.

## Skip

- **Skip** access tiers (hot, cool, cold, archive) and lifecycle management. Cost features, not security.
- **Skip** scalability targets and request-rate limits.
- **Skip** failover mechanics and the paired-region list. Know the redundancy acronyms only.
- **Skip** legacy account types (general-purpose v1, legacy blob).
- **Skip** client-side encryption and customer-provided keys per request. CMK at account level is the tested option.
- **Skip** Azure Files identity setup (AD DS, Entra Kerberos) and SFTP local users here; storage objectives cover what's tested.

## Screenshots

![Creating a key expiration policy in the Azure portal](https://learn.microsoft.com/en-us/azure/storage/common/media/storage-account-keys-manage/portal-key-expiration-policy.png){Key expiration policy — a rotation reminder, not an automatic rotation}

![Selecting the built-in policy that monitors key rotation](https://learn.microsoft.com/en-us/azure/storage/common/media/storage-account-keys-manage/policy-definition-select-portal.png){Azure Policy can report accounts whose keys are past the rotation interval}

![Creating the policy assignment for key expiration](https://learn.microsoft.com/en-us/azure/storage/common/media/storage-account-keys-manage/policy-assignment-create.png){Assign it to a scope (see azure-policy-basics)}

![Compliance report for the key expiration policy](https://learn.microsoft.com/en-us/azure/storage/common/media/storage-account-keys-manage/policy-compliance-report-portal.png){Compliance report — which accounts haven't rotated keys}

![Front-end proxy service pattern](https://learn.microsoft.com/en-us/azure/storage/common/media/storage-sas-overview/sas-storage-fe-proxy-service.png){Pattern 1 — all data goes through your own front-end service}

![SAS provider service pattern](https://learn.microsoft.com/en-us/azure/storage/common/media/storage-sas-overview/sas-storage-provider-service.png){Pattern 2 — a small service hands out SAS tokens; clients go straight to storage}

![Locally redundant storage](https://learn.microsoft.com/en-us/azure/storage/common/media/storage-redundancy/locally-redundant-storage.png){LRS — three copies in one datacenter}

![Zone-redundant storage](https://learn.microsoft.com/en-us/azure/storage/common/media/storage-redundancy/zone-redundant-storage.png){ZRS — three copies across availability zones}

![Geo-redundant storage](https://learn.microsoft.com/en-us/azure/storage/common/media/storage-redundancy/geo-redundant-storage.png){GRS / RA-GRS — plus copies in a secondary region}

![Geo-zone-redundant storage](https://learn.microsoft.com/en-us/azure/storage/common/media/storage-redundancy/geo-zone-redundant-storage.png){GZRS / RA-GZRS — zones in the primary, plus a secondary region}

![Navigating to blob data in the Azure portal](https://learn.microsoft.com/en-us/azure/storage/blobs/media/authorize-data-operations-portal/blob-access-portal.png){Browsing containers in the portal}

![Accessing containers with the account key](https://learn.microsoft.com/en-us/azure/storage/blobs/media/authorize-data-operations-portal/auth-method-access-key.png){Portal authentication method = Access key — works for anyone who can list keys}

![Accessing containers with a Microsoft Entra account](https://learn.microsoft.com/en-us/azure/storage/blobs/media/authorize-data-operations-portal/auth-method-azure-ad.png){Portal authentication method = Microsoft Entra user account — needs a Storage Blob Data role}

![Default to Microsoft Entra authorization when creating an account](https://learn.microsoft.com/en-us/azure/storage/blobs/media/authorize-data-operations-portal/default-auth-account-create-portal.png){"Default to Microsoft Entra authorization in the Azure portal" at creation}

![Default to Microsoft Entra authorization on an existing account](https://learn.microsoft.com/en-us/azure/storage/blobs/media/authorize-data-operations-portal/default-auth-account-update-portal.png){The same setting on an existing account}

![Configuring a metric to sum blob transactions](https://learn.microsoft.com/en-us/azure/storage/blobs/media/anonymous-read-access-prevent/configure-metric-blob-transactions.png){Before disallowing anonymous access, measure anonymous requests with a metric}

![Aggregated anonymous requests against Blob storage](https://learn.microsoft.com/en-us/azure/storage/blobs/media/anonymous-read-access-prevent/metric-anonymous-blob-requests.png){Anonymous requests split out by authentication type}

![Disallowing anonymous access for an account](https://learn.microsoft.com/en-us/azure/storage/blobs/media/anonymous-read-access-prevent/blob-public-access-portal.png){Configuration > Allow Blob anonymous access = Disabled}

![Query results for the anonymous access setting across accounts](https://learn.microsoft.com/en-us/azure/storage/blobs/media/anonymous-read-access-prevent/check-public-access-setting-accounts.png){Resource Graph query — which accounts still allow anonymous access}

![Compliance report for the anonymous access audit policy](https://learn.microsoft.com/en-us/azure/storage/blobs/media/anonymous-read-access-prevent/compliance-report-policy-portal.png){Audit policy — report accounts that allow anonymous access}

![Error when creating a storage account that violates a Deny policy](https://learn.microsoft.com/en-us/azure/storage/blobs/media/anonymous-read-access-prevent/deny-policy-error.png){Deny policy — creation blocked (see azure-policy-basics)}
