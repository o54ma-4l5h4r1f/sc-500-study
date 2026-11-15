+++
slug = "azure-rbac-and-entra-roles"
title = "Azure RBAC and Microsoft Entra roles: definitions, assignments, scope, deny assignments, elevate access"
summary = "Two separate permission systems: Azure RBAC controls Azure resources (assigned at management group, subscription, resource group or resource scope); Entra roles control the directory and Microsoft 365. Learn role definitions (Actions, NotActions, DataActions), assignments, inheritance, deny assignments and the elevate-access switch."
order = 6
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/role-based-access-control/overview",
  "https://learn.microsoft.com/en-us/azure/role-based-access-control/role-definitions",
  "https://learn.microsoft.com/en-us/azure/role-based-access-control/role-assignments",
  "https://learn.microsoft.com/en-us/azure/role-based-access-control/deny-assignments",
  "https://learn.microsoft.com/en-us/azure/role-based-access-control/rbac-and-directory-admin-roles",
  "https://learn.microsoft.com/en-us/azure/role-based-access-control/elevate-access-global-admin",
  "https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/concept-understand-roles",
  "https://learn.microsoft.com/en-us/entra/fundamentals/licensing",
]
captured = 2026-11-15
related_objectives = ["D1.3.5", "D1.3.6", "D1.3.7", "D1.1.1", "D1.2.3", "D2.1.4", "D1.1.6", "D4.2.2"]
+++

## In one sentence

Azure has **two separate permission systems**: **Azure RBAC** decides what someone may do to **Azure resources** (VMs, key vaults, storage), and **Microsoft Entra roles** decide what someone may do in the **directory and Microsoft 365** (users, apps, Conditional Access) — and neither grants the other by default.

## Why it exists

As an analyst you were *given* roles: Microsoft Sentinel Responder, Security Reader. Now you design and audit them. "Who has Owner on production?" and "why can this app read every blob?" are engineer questions.

Think of a building again. **Azure RBAC** is the key system for rooms and equipment. **Entra roles** are the powers at the badge office: issuing badges, changing door rules. The badge-office manager can't open the server room with their office key — unless they deliberately cut themselves a master key (that is **elevate access**, below).

## Mental model

```svg
<svg viewBox="0 0 860 360" role="img" aria-label="A role assignment joins three things: a security principal (who), a role definition (what), and a scope (where). Azure RBAC controls Azure resources; Entra roles control the directory and Microsoft 365; a Global Administrator can elevate to User Access Administrator at root scope">
  <rect class="box-accent" x="20" y="20" width="250" height="90" rx="8"/>
  <text x="145" y="48" text-anchor="middle" class="label-accent">Security principal</text>
  <text x="145" y="72" text-anchor="middle">WHO</text>
  <text x="145" y="94" text-anchor="middle" class="muted">user, group, SP, managed identity</text>
  <rect class="box-accent" x="305" y="20" width="250" height="90" rx="8"/>
  <text x="430" y="48" text-anchor="middle" class="label-accent">Role definition</text>
  <text x="430" y="72" text-anchor="middle">WHAT</text>
  <text x="430" y="94" text-anchor="middle" class="muted">Actions, DataActions...</text>
  <rect class="box-accent" x="590" y="20" width="250" height="90" rx="8"/>
  <text x="715" y="48" text-anchor="middle" class="label-accent">Scope</text>
  <text x="715" y="72" text-anchor="middle">WHERE</text>
  <text x="715" y="94" text-anchor="middle" class="muted">MG, sub, RG, resource</text>
  <rect class="box-ok" x="230" y="135" width="400" height="44" rx="8"/>
  <text x="430" y="163" text-anchor="middle">= role assignment (Access control (IAM))</text>
  <line class="divider" x1="20" y1="205" x2="840" y2="205"/>
  <rect class="box" x="20" y="225" width="380" height="110" rx="8"/>
  <text x="210" y="252" text-anchor="middle" class="label-accent">Azure RBAC</text>
  <text x="210" y="276" text-anchor="middle">Azure resources via Resource Manager</text>
  <text x="210" y="298" text-anchor="middle" class="muted">Owner, Contributor, Reader,</text>
  <text x="210" y="320" text-anchor="middle" class="muted">User Access Administrator</text>
  <rect class="box" x="460" y="225" width="380" height="110" rx="8"/>
  <text x="650" y="252" text-anchor="middle" class="label-accent">Microsoft Entra roles</text>
  <text x="650" y="276" text-anchor="middle">directory + Microsoft 365</text>
  <text x="650" y="298" text-anchor="middle" class="muted">Global Admin, Privileged Role Admin,</text>
  <text x="650" y="320" text-anchor="middle" class="muted">Security Admin, App Admin</text>
  <text x="430" y="280" text-anchor="middle" class="label-risk">separate</text>
</svg>
```

## What actually matters

**1. A role assignment = who + what + where.** A **security principal** (user, group, service principal or managed identity) gets a **role definition** at a **scope**. Assign to **groups** where you can, so you manage membership, not hundreds of assignments.

**2. Role definitions have four permission lists.**

| Property | Meaning |
|---|---|
| `Actions` | **Control-plane** operations allowed — managing the resource (create a vault, change firewall rules). Format `Microsoft.KeyVault/vaults/write`. `*` is a wildcard. |
| `NotActions` | Control-plane operations **subtracted** from `Actions` |
| `DataActions` | **Data-plane** operations allowed — touching the data inside (read a blob, read a secret) |
| `NotDataActions` | Data-plane operations subtracted from `DataActions` |
| `AssignableScopes` | Where a **custom** role may be assigned. Built-in roles use `/` (everywhere). |

Effective control-plane permissions = `Actions` minus `NotActions`. ==!NotActions is not a deny.== If a second role grants the action, the user has it.

The **Contributor** role is `Actions: *` with `NotActions` that remove `Microsoft.Authorization/*/Write` and `/Delete` — so Contributor manages everything **except access**. ==!Owner on a subscription does not automatically read blob data== when the storage account uses Entra authorization: reading blobs is a `DataAction`, granted by roles like **Storage Blob Data Reader**.

**3. Core built-in Azure roles.**

| Role | Can manage resources | Can assign roles |
|---|---|---|
| **Owner** | Yes | Yes |
| **Contributor** | Yes | ==!No== |
| **Reader** | View only | No |
| **User Access Administrator** | No | Yes (any role, including Owner) |
| **Role Based Access Control Administrator** | No | Yes — can be limited by **conditions** to certain roles/principals |

Service-specific roles split control and data plane further: *Key Vault Contributor* (manage the vault) vs *Key Vault Secrets User* (read secrets).

**4. RBAC is additive; scope inherits downward.** Your effective permissions are the **sum** of all your assignments at the resource's scope and every parent scope. Contributor on a subscription plus Reader on one resource group = Contributor on that resource group.

**5. Deny assignments.** A **deny assignment** blocks named actions **even if** a role grants them. Resource Manager checks deny assignments **first**. ==!You can't create your own deny assignment directly.== Azure creates them — today mainly through **deployment stacks** deny settings (`DenyDelete`, `DenyWriteAndDelete`) and managed applications. You can view them under **Access control (IAM) > Deny assignments**.

Evaluation order: deny assignment → role assignments (does any grant the action at this scope?) → conditions (if any) → allow.

**6. Custom roles.** Azure custom roles: JSON with the properties above; up to **5,000 per tenant**; `AssignableScopes` may name only **one** management group. Entra custom roles: built from Entra permissions; ==@each user with an Entra custom role assignment needs Entra ID P1==. Built-in Entra roles are free.

**7. Entra roles.** About 60 built-in roles. Key ones: **Global Administrator** (everything in Entra and Microsoft 365 — not Azure resources by default), **Privileged Role Administrator** (assigns Entra roles, grants any consent), **Security Administrator**, **Security Reader**, **Conditional Access Administrator**, **Application Administrator / Cloud Application Administrator**. Entra role scope is the **tenant**, an **administrative unit** (a subset of users/groups/devices) or a single **app registration**.

**8. Elevate access.** A **Global Administrator** can flip **Access management for Azure resources** to **Yes**. That assigns them **User Access Administrator at root scope `/`** — every management group and subscription in the tenant. It is per-user, not tenant-wide. Use it to recover lost access, then switch it back to **No**. It is logged in the directory **Audit logs** (service *Azure RBAC (Elevated Access)*) and in the Activity log as "Assigns the caller to User Access Administrator role" — a classic detection.

## Where it lives

- **Azure portal > {any scope} > Access control (IAM)** — **Role assignments**, **Roles**, **Deny assignments**, **Check access**.
- **Entra admin center > Identity > Roles & admins > Roles & admins** — Entra role assignments and custom Entra roles.
- **Entra admin center > Identity > Overview > Properties > Access management for Azure resources** (Azure portal: **Microsoft Entra ID > Manage > Properties**) — the elevate-access toggle.
- CLI: `az role assignment create|list|delete`, `az role definition list --custom-role-only true`, `az role definition create --role-definition role.json`.

![Diagram showing how security principal, role definition, and scope create a role assignment](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/overview/rbac-overview.png){Role assignment = security principal + role definition + scope}

![Evaluation logic flowchart for determining access to a resource](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/overview/evaluation-logic.png){How Resource Manager decides: deny assignments first, then role assignments, then conditions}

![Diagram that shows Azure RBAC versus Microsoft Entra roles](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/rbac-and-directory-admin-roles/azure-roles-entra-roles.png){Two systems: Azure roles for Azure resources, Entra roles for the directory}

## Exam traps

- ==!Global Administrator has no Azure resource access by default.== It must elevate (or be given an Azure role).
- ==!Contributor can't assign roles.== "Let the team deploy and also grant access to their resources" needs Owner, User Access Administrator or (least privilege, with conditions) Role Based Access Control Administrator.
- ==+Least privilege for data:== give a data role (Storage Blob Data Reader, Key Vault Secrets User), not Owner or Contributor.
- Entra roles are **not** assigned in Access control (IAM); Azure roles are **not** assigned in Roles & admins.
- Classic administrators (Account Admin, Service Admin, Co-Admin) are retired — any answer relying on them is wrong.
- `NotActions` ≠ deny. Deny assignments ≠ something you author by hand.

## Skip

- **Skip** the per-provider operations reference and long action-string tables. Know the format and the control/data split.
- **Skip** RBAC data storage across regions, tool-version tables for data actions, and REST API versions.
- **Skip** the elevate-access REST and PowerShell walkthroughs and the Sentinel analytics-rule setup in that article — know the toggle, the resulting role and scope, and that it is logged.
- **Skip** the list of service-specific Entra roles (Exchange, Teams, SharePoint admins). Know the security-relevant ones above.
- **Skip** classic administrator history beyond "retired".

## Screenshots

![Diagram showing the security principal types for a role assignment](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/shared/rbac-security-principal.png){Security principals: user, group, service principal, managed identity}

![Diagram showing role definition example for a role assignment](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/shared/rbac-role-definition.png){A role definition is a list of permissions}

![Diagram showing scope levels for a role assignment](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/shared/rbac-scope.png){Scope levels for an assignment}

![Diagram showing how role assignments are transitive for groups](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/overview/rbac-groups-transitive.png){Assignments to a group reach its members — including nested groups}

![Diagram showing how multiple role assignments overlap](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/overview/rbac-multiple-roles.png){RBAC is additive: overlapping roles sum}

![Diagram showing Azure RBAC data in multiple regions](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/overview/rbac-data-stored.png){Role data is stored globally}

![Diagram of role-based access control extended to support both control plane and data plane actions](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/role-definitions/rbac-data-plane.png){Owner at subscription (control plane) versus Storage Blob Data Contributor (data plane)}

![Screenshot of Access control (IAM) page and Deny assignments tab](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/deny-assignments/access-control-deny-assignments.png){Access control (IAM) > Deny assignments}

![Screenshot of deny assignments columns pane](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/deny-assignments/deny-assignments-columns.png){Extra columns: excluded principals, does-not-apply-to-children, system protected}

![Screenshot of Users page for a deny assignment](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/deny-assignments/deny-assignment-users.png){Who a deny assignment applies to — and who is excluded}

![Screenshot of Denied Permissions page for a deny assignment](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/deny-assignments/deny-assignment-denied-permissions.png){The denied Actions / DataActions}

![Screenshot of Properties page for a deny assignment](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/deny-assignments/deny-assignment-properties.png){Deny assignment properties — System protected is always Yes}

![Diagram of the different roles in Azure](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/rbac-and-directory-admin-roles/rbac-admin-roles.png){Azure roles, Entra roles and the retired classic administrators}

![Screenshot of Access control (IAM) page in the Azure portal](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/shared/sub-role-assignments.png){Role assignments on a subscription}

![Screenshot of built-in roles in the Azure portal](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/shared/roles-list.png){Built-in Azure roles}

![Screenshot of Microsoft Entra roles in the Azure portal](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/rbac-and-directory-admin-roles/directory-admin-roles.png){Entra roles — a different list in a different place}

![Azure RBAC versus Microsoft Entra roles](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/media/concept-understand-roles/azure-roles-azure-ad-roles.png){The Entra docs' version of the same split}

![The three categories of Microsoft Entra built-in roles](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/media/concept-understand-roles/role-overlap-diagram.png){Entra built-in role categories: Entra-specific, service-specific, cross-service}

![Elevate access](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/elevate-access.png){Elevate access: Global Administrator becomes User Access Administrator at root scope}

![Select Properties for Microsoft Entra properties](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/azure-active-directory-properties.png){Microsoft Entra ID > Properties}

![Access management for Azure resources](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/aad-properties-global-admin-setting.png){The Access management for Azure resources toggle}

![Subscription role assignments with root scope](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/iam-root.png){The resulting assignment at root scope}

![Remove role assignments with root scope](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/iam-root-remove.png){Root-scope assignments can't be removed in IAM — flip the toggle back}

![Screenshot of banner that indicates there are users with elevated access](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/elevated-access-users-banner.png){Banner: users with elevated access}

![Screenshot of banner on Access control (IAM) page that indicates there are users with elevated access](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/elevated-access-users-iam-banner.png){The same warning on a subscription's IAM page}

![Screenshot of Users with elevated access pane](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/elevated-access-users-pane.png){List of users with elevated access}

![Screenshot of directory audit logs with Service filter set to Azure RBAC (Elevated Access)](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/entra-id-audit-logs-filter.png){Entra audit logs filtered to Azure RBAC (Elevated Access)}

![Screenshot of directory audit logs that shows audit log details when access is elevated](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/entra-id-audit-logs-elevated-details.png){Audit record of an elevation}

![Screenshot of directory audit logs that shows the Download Audit Logs pane](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/entra-id-audit-logs-download.png){Downloading the audit logs}

![Screenshot that shows activity logs for the directory in Azure Monitor](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/monitor-directory-activity.png){Directory activity in the Azure Activity log}

![Screenshot of Microsoft Sentinel with a workspace](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/sentinel-enable.png){Sentinel workspace used to alert on elevation}

![Screenshot of Content hub page with Microsoft Entra ID content selected](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/sentinel-entra-id-solution.png){Content hub: the Microsoft Entra ID solution}

![Screenshot of Microsoft Entra ID connector with Audit Logs selected](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/sentinel-connectors-entra-id-audit-logs.png){Entra ID connector streaming Audit Logs}

![Screenshot of Analytics page with the Azure RBAC (Elevate Access) selected](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/sentinel-analytics-rule.png){The Azure RBAC (Elevate Access) analytics rule template}

![Screenshot of Analytics rule wizard for Azure RBAC (Elevate Access)](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/sentinel-analytics-rule-wizard.png){Creating the rule}

![Screenshot of Incidents page with examples of elevate access incidents](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/elevate-access-global-admin/sentinel-incidents.png){Elevation incidents — the analyst's view of this engineer control}
