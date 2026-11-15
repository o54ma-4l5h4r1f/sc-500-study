+++
slug = "azure-resource-hierarchy"
title = "Azure resource hierarchy: tenant, management groups, subscriptions, resource groups, resources"
summary = "Azure is a set of nested boxes — Entra tenant, management groups, subscriptions, resource groups, resources — and access, policy and billing set on a box flow down to everything inside it. Every control you build on this exam is assigned at one of these scopes."
order = 1
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/overview",
  "https://learn.microsoft.com/en-us/azure/governance/management-groups/overview",
  "https://learn.microsoft.com/en-us/azure/role-based-access-control/scope-overview",
  "https://learn.microsoft.com/en-us/entra/fundamentals/how-subscriptions-associated-directory",
  "https://learn.microsoft.com/en-us/azure/reliability/regions-overview",
  "https://learn.microsoft.com/en-us/training/modules/describe-core-architectural-components-of-azure/6-describe-azure-management-infrastructure",
]
captured = 2026-11-15
related_objectives = ["D1.3.1", "D1.3.4", "D1.3.5", "D1.3.6", "D1.3.7", "D1.3.9", "D4.1.3", "D4.1.4", "D4.2.1"]
+++

## In one sentence

Everything in Azure lives inside nested boxes — **tenant > management groups > subscriptions > resource groups > resources** — and anything you set on a box (who may touch it, which rules apply, who pays) flows down to everything inside it.

## Why it exists

As an analyst you met Azure from the inside of one box: the Sentinel workspace. As a security engineer you choose **where** a control is attached. That choice decides how far it reaches.

Think of an office building. The **company** owns the building. Each **floor** belongs to a department. Each floor has **rooms**, and each room holds **equipment**. A master key for a floor opens every room on that floor. A building rule ("no visitors after 8 pm") covers every floor at once.

Azure works the same way, for three reasons:

- **Access.** Give a team rights once, high up, instead of on 500 separate resources.
- **Rules.** Assign an Azure Policy once ("only these regions", "storage must deny public access") and it covers everything below.
- **Money.** Each subscription gets its own bill, so costs split cleanly.

Almost every SC-500 control — role assignments, Azure Policy, resource locks, Defender for Cloud plans — is attached at one of these levels. If you can't picture the tree, you can't answer "at which scope should you assign this?"

## Mental model

```svg
<svg viewBox="0 0 820 500" role="img" aria-label="Azure hierarchy tree from Entra tenant to root management group, management groups, subscriptions, resource groups and resources, with RBAC, Azure Policy and locks inheriting downward but tags not inheriting">
  <defs>
    <marker id="fahharrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="currentColor"/>
    </marker>
  </defs>
  <rect class="box-accent" x="20" y="15" width="620" height="44" rx="8"/>
  <text x="330" y="43" text-anchor="middle" class="label-accent">Microsoft Entra tenant (the identity directory)</text>
  <rect class="box" x="150" y="80" width="360" height="40" rx="8"/>
  <text x="330" y="105" text-anchor="middle">Tenant Root Group (root management group)</text>
  <rect class="box" x="40" y="145" width="260" height="40" rx="8"/>
  <text x="170" y="170" text-anchor="middle">Management group: Production</text>
  <rect class="box" x="370" y="145" width="240" height="40" rx="8"/>
  <text x="490" y="170" text-anchor="middle">Management group: Dev</text>
  <rect class="box" x="40" y="215" width="260" height="44" rx="8"/>
  <text x="170" y="236" text-anchor="middle">Subscription: Prod-Apps</text>
  <text x="170" y="253" text-anchor="middle" class="muted">billing + access boundary</text>
  <rect class="box" x="370" y="215" width="240" height="44" rx="8"/>
  <text x="490" y="242" text-anchor="middle">Subscription: Dev-Sandbox</text>
  <rect class="box" x="20" y="290" width="300" height="44" rx="8"/>
  <text x="170" y="311" text-anchor="middle">Resource group: rg-payments</text>
  <text x="170" y="328" text-anchor="middle" class="muted">resources with one lifecycle</text>
  <rect class="box" x="370" y="290" width="240" height="44" rx="8"/>
  <text x="490" y="317" text-anchor="middle">Resource group: rg-test</text>
  <rect class="box-ok" x="20" y="365" width="140" height="50" rx="8"/>
  <text x="90" y="395" text-anchor="middle">Key vault</text>
  <rect class="box-ok" x="180" y="365" width="140" height="50" rx="8"/>
  <text x="250" y="395" text-anchor="middle">Storage</text>
  <rect class="box" x="370" y="365" width="110" height="50" rx="8"/>
  <text x="425" y="395" text-anchor="middle">VM</text>
  <rect class="box" x="500" y="365" width="110" height="50" rx="8"/>
  <text x="555" y="395" text-anchor="middle">SQL</text>
  <line class="line" x1="330" y1="59" x2="330" y2="80"/>
  <line class="line" x1="250" y1="120" x2="170" y2="145"/>
  <line class="line" x1="410" y1="120" x2="490" y2="145"/>
  <line class="line" x1="170" y1="185" x2="170" y2="215"/>
  <line class="line" x1="490" y1="185" x2="490" y2="215"/>
  <line class="line" x1="170" y1="259" x2="170" y2="290"/>
  <line class="line" x1="490" y1="259" x2="490" y2="290"/>
  <line class="line" x1="120" y1="334" x2="90" y2="365"/>
  <line class="line" x1="220" y1="334" x2="250" y2="365"/>
  <line class="line" x1="460" y1="334" x2="425" y2="365"/>
  <line class="line" x1="520" y1="334" x2="555" y2="365"/>
  <line class="line-accent" x1="670" y1="90" x2="670" y2="410" marker-end="url(#fahharrow)"/>
  <text x="685" y="140" class="label-accent">flows down:</text>
  <text x="685" y="165">RBAC roles</text>
  <text x="685" y="185">Azure Policy</text>
  <text x="685" y="205">resource locks</text>
  <text x="685" y="270" class="label-risk">does NOT:</text>
  <text x="685" y="292">tags</text>
  <text x="330" y="455" text-anchor="middle" class="muted">A role on Prod-Apps reaches the key vault. A role on rg-test does not.</text>
  <text x="330" y="480" text-anchor="middle" class="muted">Each subscription trusts exactly one tenant; a tenant can hold many subscriptions.</text>
</svg>
```

Read top to bottom. A setting made on a box applies to every box beneath it — and to nothing beside it.

## What actually matters

**1. The levels, defined.**

| Level | What it is | Key facts |
|---|---|---|
| **Microsoft Entra tenant** | Your organisation's identity directory: users, groups, apps. Also called the *directory*. | Every subscription trusts **exactly one** tenant for sign-in. One tenant can be trusted by many subscriptions. |
| **Management group** | A folder that holds subscriptions and other management groups | Tree up to ==@six levels== deep (not counting the root or subscription level). Up to **10,000** per directory. One parent each. |
| **Tenant Root Group** | The single top management group every directory gets | Its ID is the tenant ID. Can't be moved or deleted. New subscriptions land under it by default. |
| **Subscription** | The unit of **billing** and an **access boundary** | Each gets its own invoice. Typical split: production vs development. |
| **Resource group** | A container for resources that share a lifecycle | Every resource sits in **exactly one** group. Groups ==!can't be nested== or renamed. Deleting a group deletes everything in it. |
| **Resource** | One thing you create: a VM, a key vault, a storage account | Management groups, subscriptions and resource groups are also resources in Resource Manager's eyes. |

**2. Azure Resource Manager is the one front door.** The portal, the `az` CLI, PowerShell, ARM/Bicep templates and REST calls all go through **Azure Resource Manager (ARM)**. ARM authenticates the caller, checks Azure RBAC, applies Azure Policy and resource locks, then hands the request to the service. That is why one role or one policy works no matter which tool someone uses.

**3. Inheritance flows down, never up or sideways.** "Lower levels inherit settings from higher levels." A role assigned on a subscription applies to every resource group and resource in it. A policy assigned on a management group applies to every subscription under it, and a subscription owner can't remove it. A lock on a resource group protects every resource inside. ==!Tags are the exception==: a tag on a resource group is **not** copied to its resources (you use a policy to do that).

**4. Scope is written as a resource ID.** Every resource has a unique path, its **resource ID**, which spells out the tree. When a command asks for `--scope`, it wants one of these strings:

```
/providers/Microsoft.Management/managementGroups/{mgName}
/subscriptions/{subscriptionId}
/subscriptions/{subscriptionId}/resourceGroups/{rgName}
/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.KeyVault/vaults/{vaultName}
```

`{subscriptionId}` is a GUID (a long unique number). `Microsoft.KeyVault` is the **resource provider** — the Azure service that makes that kind of resource. Root scope, written `/`, sits above everything; you meet it again with "elevate access" on the RBAC page.

**5. Regions and geographies.** A **region** is one or more datacenters in one metro area — for example *West Europe*. Most resources are created in a region. Each region belongs to a **geography**, a data-residency boundary. A resource group also has a region: it stores the group's *metadata* there, while its resources may live elsewhere.

**6. A subscription can change tenant — and that is a security event.** Moving a subscription to a different directory breaks every role assignment in it, because the identities belong to the old tenant. Only do it on purpose.

## Where it lives

- **Azure portal > Management groups** — the tree. **Azure portal > Subscriptions** and **Resource groups** — the lists.
- **Any resource > Properties** (or **JSON View**) shows the **Resource ID**.
- **Any scope > Access control (IAM)** — where you assign roles at that level.
- **Switching tenant:** the portal's **Settings > Directories + subscriptions** switcher.
- CLI: `az account show` (current subscription), `az group list`, `az account management-group list`.

![Diagram that illustrates the four levels of scope in Azure: management groups, subscriptions, resource groups, and resources](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/media/overview/scope-levels.png){The four scope levels — settings applied at one level reach every level below}

![Diagram that shows the role of Resource Manager during Azure requests](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/media/overview/consistent-management-layer.png){Portal, CLI, PowerShell, SDKs and REST all pass through one front door: Azure Resource Manager}

## Exam traps

- ==!Permissions do not flow up.== Owner of a resource group is not Owner of the subscription.
- ==+Assign once, high up, for a broad requirement.== "All subscriptions must…" points to a **management group** scope; "only this app's resources" points to a **resource group** scope.
- ==!A subscription is not a tenant.== The tenant is the directory of identities. A subscription is a billing and access container that trusts one tenant.
- "A resource group inside a resource group" is always wrong.
- A tag on a resource group does not reach its resources — inheritance needs Azure Policy (`Inherit a tag from the resource group`-style policies).

## Skip

- **Skip** management-group move permissions and the role-versus-action chart in the management groups article. Know only that a management group has one parent and policies/roles inherit.
- **Skip** service groups, region pairs, sovereign clouds and logical-versus-physical availability-zone mapping. Know "region" and "geography".
- **Skip** resource-group limits (800 instances of a resource type) and billing account types (EA, MCA, CSP).
- **Skip** the training unit's history of subscriptions and accounts. The exam assumes the hierarchy; it tests what you attach to it.

## Screenshots

![Diagram showing a management group hierarchy from Tenant Root Group down to resources](https://learn.microsoft.com/en-us/training/wwl-azure/describe-core-architectural-components-of-azure/media/management-group-hierarchy.png){The full tree, from Tenant Root Group to individual resources, with policy and access inheriting downward}

![Diagram showing three resource group rules](https://learn.microsoft.com/en-us/training/wwl-azure/describe-core-architectural-components-of-azure/media/resource-group-rules.png){Resource group rules: one group per resource, no nesting or renaming, delete the group and everything goes}

![Diagram showing two subscription boundary types: billing boundary and access control boundary](https://learn.microsoft.com/en-us/training/wwl-azure/describe-core-architectural-components-of-azure/media/subscription-boundaries.png){A subscription is both a billing boundary and an access boundary}

![Diagram of a sample management group hierarchy](https://learn.microsoft.com/en-us/azure/governance/media/mg-org.png){A realistic management group tree, several levels deep}

![Diagram of a subset of the sample management group hierarchy](https://learn.microsoft.com/en-us/azure/governance/media/mg-org-sub.png){A slice of that tree — moving a subscription between branches changes what it inherits}

![Screenshot of activity logs and operations related to a selected management group](https://learn.microsoft.com/en-us/azure/governance/management-groups/media/al-mg.png){Management group changes land in the Activity log — something a SOC can alert on}

![Diagram of scope for a role assignment](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/scope-overview/rbac-scope-no-label.png){RBAC scope: a role assigned at one level covers everything beneath}

![Diagram of management group and subscription hierarchy](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/scope-overview/rbac-scope-management-groups.png){Management groups add scope levels above subscriptions}

![Screenshot of resource IDs for a storage account in Azure portal](https://learn.microsoft.com/en-us/azure/role-based-access-control/media/scope-overview/scope-resource-id.png){Where to find a resource ID in the portal}

![Screenshot that shows the trust relationship between Azure subscriptions and Microsoft Entra directories](https://learn.microsoft.com/en-us/entra/fundamentals/media/how-subscriptions-associated-directory/trust-relationship.png){Each subscription trusts one Entra tenant for sign-in}

![Screenshot that shows the Subscriptions page with the Change directory option highlighted](https://learn.microsoft.com/en-us/entra/fundamentals/media/how-subscriptions-associated-directory/change-directory-in-azure-subscriptions.png){A subscription can be moved to another directory — its role assignments do not survive the move}

![Screenshot that shows the Change the directory page](https://learn.microsoft.com/en-us/entra/fundamentals/media/how-subscriptions-associated-directory/edit-directory-ui.png){Changing a subscription's directory}

![Screenshot that shows the Directory switcher page](https://learn.microsoft.com/en-us/entra/fundamentals/media/how-subscriptions-associated-directory/directory-switcher.png){Switching between tenants your account belongs to}

![Diagram that shows a geography with two Azure regions, each with three availability zones](https://learn.microsoft.com/en-us/azure/reliability/media/regions-overview/regions.svg){Regions inside a geography; availability zones inside a region}
