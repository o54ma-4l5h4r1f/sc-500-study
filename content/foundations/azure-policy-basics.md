+++
slug = "azure-policy-basics"
title = "Azure Policy basics"
summary = "Rules that Azure resources must follow: definitions, initiatives, assignments, scope, exclusions vs exemptions, every effect (Deny, Audit, Modify, Append, DeployIfNotExists, AuditIfNotExists, Disabled, DenyAction, Manual), remediation tasks and their managed identity, and when compliance is evaluated."
order = 11
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/governance/policy/overview",
  "https://learn.microsoft.com/en-us/azure/governance/policy/concepts/effect-basics",
  "https://learn.microsoft.com/en-us/azure/governance/policy/concepts/effect-deny-action",
  "https://learn.microsoft.com/en-us/azure/governance/policy/concepts/effect-append",
  "https://learn.microsoft.com/en-us/azure/governance/policy/concepts/effect-manual",
  "https://learn.microsoft.com/en-us/azure/governance/policy/concepts/compliance-states",
  "https://learn.microsoft.com/en-us/azure/governance/policy/concepts/assignment-structure",
  "https://learn.microsoft.com/en-us/azure/governance/policy/concepts/exemption-structure",
  "https://learn.microsoft.com/en-us/azure/governance/policy/concepts/scope",
  "https://learn.microsoft.com/en-us/azure/governance/policy/how-to/remediate-resources",
  "https://learn.microsoft.com/en-us/azure/governance/policy/how-to/get-compliance-data",
  "https://learn.microsoft.com/en-us/azure/governance/policy/assign-policy-portal",
]
captured = 2026-11-15
related_objectives = ["D1.3.1", "D1.3.2", "D1.3.3", "D1.3.4", "D1.3.9", "D3.2.9", "D3.3.2", "D4.1.2"]
+++

## In one sentence

**Azure Policy** is a rulebook for Azure resources: you pick or write a rule ("storage accounts must not allow public network access"), attach it to part of your estate, and Azure then reports the resources that break it, blocks new ones that would break it, or fixes them for you.

## Why it exists

Think of a building code. The inspector does not care *who* built the extension. He cares whether it has a fire exit. Azure Policy is that inspector for cloud resources.

A company can have thousands of resources and hundreds of people creating them. Nobody can click into each one to check that encryption, logging and network rules are set. Policy checks every resource against every rule, automatically, forever.

As an analyst you saw the *result* of missing controls: an alert about a storage account open to the internet. As an engineer you stop it happening. Policy is the main tool for that, and Defender for Cloud's recommendations and compliance standards are built on top of it (see `defender-for-cloud-basics`).

## Mental model

Four objects, in the order you meet them:

| Object | Everyday analogy | What it is |
|---|---|---|
| **Policy definition** | One rule in the building code | A JSON rule: an *if* condition plus **one** effect. Hundreds are built in; you can write custom ones. |
| **Initiative** (CLI/PowerShell: **policy set**) | A chapter of the code ("fire safety") | A bundle of definitions with one goal, assigned as one item. Regulatory standards are initiatives. |
| **Assignment** | "This code applies to *this* street" | A definition or initiative attached to a scope. Everything below inherits it. |
| **Remediation task** | The contractor sent to fix old buildings | A job that brings *existing* non-compliant resources into line. |

```svg
<svg viewBox="0 0 780 340" role="img" aria-label="Azure Policy flow: a definition or initiative is assigned to a scope, resources are evaluated, the effect decides what happens, and a remediation task using the assignment's managed identity fixes existing resources">
  <defs>
    <marker id="apbarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="line-accent"/>
    </marker>
  </defs>
  <rect class="box" x="20" y="30" width="180" height="70" rx="8"/>
  <text x="110" y="60" text-anchor="middle">Policy definition</text>
  <text x="110" y="82" text-anchor="middle" class="muted">if + one effect</text>
  <rect class="box" x="20" y="120" width="180" height="70" rx="8"/>
  <text x="110" y="150" text-anchor="middle">Initiative</text>
  <text x="110" y="172" text-anchor="middle" class="muted">bundle of definitions</text>
  <rect class="box-accent" x="255" y="75" width="210" height="70" rx="8"/>
  <text x="360" y="105" text-anchor="middle" class="label-accent">Assignment</text>
  <text x="360" y="127" text-anchor="middle" class="muted">MG / sub / RG / resource</text>
  <line class="line-accent" x1="200" y1="65" x2="253" y2="100" marker-end="url(#apbarrow)"/>
  <line class="line-accent" x1="200" y1="155" x2="253" y2="122" marker-end="url(#apbarrow)"/>
  <rect class="box" x="520" y="75" width="240" height="70" rx="8"/>
  <text x="640" y="105" text-anchor="middle">Evaluate resources</text>
  <text x="640" y="127" text-anchor="middle" class="muted">on change + every 24 h</text>
  <line class="line-accent" x1="465" y1="110" x2="518" y2="110" marker-end="url(#apbarrow)"/>
  <rect class="box-risk" x="440" y="215" width="150" height="80" rx="8"/>
  <text x="515" y="245" text-anchor="middle" class="label-risk">Deny</text>
  <text x="515" y="267" text-anchor="middle" class="muted">block new breakers</text>
  <rect class="box" x="610" y="215" width="150" height="80" rx="8"/>
  <text x="685" y="245" text-anchor="middle">Audit</text>
  <text x="685" y="267" text-anchor="middle" class="muted">just report</text>
  <rect class="box-ok" x="210" y="215" width="210" height="80" rx="8"/>
  <text x="315" y="243" text-anchor="middle">DeployIfNotExists</text>
  <text x="315" y="263" text-anchor="middle">or Modify</text>
  <text x="315" y="283" text-anchor="middle" class="muted">fix it with a managed ID</text>
  <line class="line" x1="640" y1="145" x2="515" y2="213" marker-end="url(#apbarrow)"/>
  <line class="line" x1="660" y1="145" x2="685" y2="213" marker-end="url(#apbarrow)"/>
  <line class="line" x1="620" y1="145" x2="340" y2="213" marker-end="url(#apbarrow)"/>
  <rect class="box-ok" x="20" y="230" width="170" height="50" rx="8"/>
  <text x="105" y="260" text-anchor="middle">Remediation task</text>
  <line class="line-accent" x1="210" y1="255" x2="192" y2="255" marker-end="url(#apbarrow)"/>
  <text x="105" y="310" text-anchor="middle" class="muted">fixes EXISTING resources</text>
</svg>
```

## What actually matters

**1. A definition has exactly one effect.** The effect is what happens when the rule's *if* matches. The full list is `addToNetworkGroup`, `append`, `audit`, `auditIfNotExists`, `deny`, `denyAction`, `deployIfNotExists`, `disabled`, `manual`, `modify`, `mutate`. Nine matter for this exam:

| Effect | What it does | Fixes existing resources? |
|---|---|---|
| `Deny` | Blocks the create or update request. Existing breakers are only *marked* non-compliant. | No |
| `Audit` | Lets the request through, logs a warning, marks the resource non-compliant. | No |
| `AuditIfNotExists` | Reports when a *related* resource is missing (for example, a storage account with no diagnostic setting). | No |
| `DeployIfNotExists` | Deploys the missing related resource (a diagnostic setting, an agent, Defender for SQL). Runs a template. | ==+Yes, via a remediation task== |
| `Modify` | Adds, changes or removes properties or tags on the resource itself. | ==+Yes, via a remediation task== |
| `Append` | Adds fields to the request on create or update (for example, an allowed IP rule). Meant for non-tag fields; use `Modify` for tags. If it would overwrite a value, it acts like `Deny`. | No — only marks non-compliant |
| `Disabled` | Switches the rule off without deleting it. Useful as a parameter value. | — |
| `DenyAction` | Blocks a *named action* on a resource. The only supported action is `delete`. Shows the resource as **Protected**. | — |
| `Manual` | You attest compliance yourself (an "attestation"). Default state is **Unknown**. Used in regulatory standards for controls a machine can't check. | — |

**2. Evaluation order.** On a create or update request, Policy checks `disabled` first, then `append` and `modify` (they may change the request), then `deny`, then `audit`, `manual`, `auditIfNotExists`, and `denyAction` last. After the resource provider succeeds, `auditIfNotExists` and `deployIfNotExists` run. The point: ==@Deny is checked before Audit==, so a denied resource is not double-logged.

**3. Scope, inheritance, exclusions and exemptions.** You assign at a **management group** (a folder of subscriptions), a **subscription**, a **resource group** or one **resource** (see `azure-resource-hierarchy`). Everything below inherits. Two ways to carve a hole:

| | **Exclusion** (`notScopes`) | **Exemption** |
|---|---|---|
| Set where | On the assignment | A separate object on the resource or scope |
| Effect | Sub-scope is not evaluated at all | Resource is evaluated but shows **Exempt** |
| Counts in compliance? | No | Yes |
| Reason recorded? | No | Category **Waiver** (risk accepted for now) or **Mitigated** (met another way) |
| Expiry? | No | Optional `expiresOn` |

An exemption needs `Microsoft.Authorization/policyExemptions/write` **and** the `exempt/Action` verb on the assignment. **Resource Policy Contributor** and **Security Admin** both hold exemption read/write.

**4. Layering is cumulative most restrictive.** Several assignments on one resource are each evaluated on their own. If any one denies, the request is denied. A looser policy lower down does ==!not== override a Deny above it.

**5. When evaluation happens.** A new or updated assignment applies after about **5 minutes**, then an evaluation cycle starts. A created or updated resource shows its result about **15 minutes** later. A standard compliance scan runs **once every 24 hours**. You can also trigger an **on-demand scan** for a subscription or resource group (`az policy state trigger-scan`).

**6. Remediation and the managed identity.** `Deny` and `Audit` only act on requests as they arrive. To fix resources that already exist you run a **remediation task**. It works only for `deployIfNotExists` and `modify`. The task needs an identity to sign in as: the assignment's **managed identity** (an Entra identity Azure creates for a service, with no password to store — see `entra-id-basics`). Each assignment has one, system-assigned or user-assigned. In the portal, Policy grants that identity the roles listed in the definition's `roleDefinitionIds` automatically. From CLI, PowerShell or a template ==!you must grant the roles yourself==, or remediation fails. One task remediates up to **50,000** resources.

**7. Enforcement mode.** `enforcementMode` = **Default** (portal: *Policy enforcement = Enabled*) enforces the effect. **DoNotEnforce** (portal: *Disabled*) evaluates and reports but does not block — the safe way to test a new Deny. Remediation tasks can still run under DoNotEnforce.

**8. Policy is not RBAC, and not a lock.**

| | Azure RBAC | Azure Policy | Resource lock |
|---|---|---|---|
| Question | ==@Who== may do this action? | ==@What state== must the resource be in? | May anyone delete or change *this* resource? |
| Cares who asked? | Yes | No | No |
| Blocks an Owner? | — | Yes (Deny) | Yes |

**Who can manage Policy:** **Resource Policy Contributor** covers most Policy work. **Owner** has full rights. **Contributor** and **Reader** can read Policy objects; Contributor can trigger remediation but ==!cannot create definitions or assignments==. Granting the managed identity its roles needs **Owner** or **User Access Administrator**.

## Where it lives

**Azure portal** only (search **Policy**). Left menu:

- **Definitions** — browse built-in definitions and initiatives; **+ Policy definition** / **+ Initiative definition** for custom ones.
- **Assignments > Assign policy** (or **Assign initiative**) — tabs **Basics** (Scope, Exclusions, definition, Policy enforcement) → **Parameters** → **Remediation** (managed identity, *Create a remediation task*) → **Non-compliance messages** → **Review + create**.
- **Exemptions** — create and review exemptions.
- **Compliance** — per assignment, compliant vs non-compliant resources. Drill in for per-resource detail.
- **Remediation** — tabs **Policies to remediate** and **Remediation tasks**.

```bash
az policy assignment create --name deny-public-storage --policy <definitionId> \
  --scope /subscriptions/<subId> --mi-system-assigned --location westeurope
az policy remediation create --name fix1 --policy-assignment <assignmentId>
az policy exemption create --name waiver1 --policy-assignment <assignmentId> \
  --exemption-category Waiver --expires-on 2026-12-31T00:00:00Z
az policy state trigger-scan --resource-group <rg>
```

![Searching for Policy in the Azure portal](https://learn.microsoft.com/en-us/azure/governance/policy/media/assign-policy-portal/search-policy.png){Step 1 — search "policy" in the portal search bar}

![The Assignments pane with Assign policy highlighted](https://learn.microsoft.com/en-us/azure/governance/policy/media/assign-policy-portal/select-assignments.png){Assignments — where every assignment is created}

![Remediation tab creating a system-assigned managed identity with Log Analytics Contributor](https://learn.microsoft.com/en-us/azure/governance/policy/media/remediate-resources/remediation-tab.png){Remediation tab — the portal creates the managed identity and grants it the role the definition lists (here Log Analytics Contributor)}

## Exam traps

- ==+DeployIfNotExists== and ==+Modify== are the only effects a remediation task can use. ==!Audit, Deny and Append never fix existing resources.==
- "Prevent creation" → `Deny`. "Prevent deletion at scale" → `DenyAction` (or a resource lock — see D1.3.4). "Report only" → `Audit` / `AuditIfNotExists`.
- "Tags must be added" → `Modify`, not `Append`.
- A new assignment covers **new** resources at once; **existing** ones stay non-compliant until a remediation task runs.
- Remediation from CLI/template fails → the managed identity is missing its role.
- "Exclude, but keep it visible and give a reason" → **exemption**, not exclusion.
- "Test a Deny without breaking anyone" → `enforcementMode` **DoNotEnforce**.
- Policy is evaluated even for an **Owner**; RBAC does not override a Deny.

## Skip

- **Skip** writing deep policy JSON: aliases, every condition operator, `count` expressions, functions. Know the shape (`if` → `then.effect`) and that aliases name resource properties.
- **Skip** `mutate` and `addToNetworkGroup`. They belong to AKS and Azure Virtual Network Manager scenarios and are taught on those pages (D3.3.2, D2.3.2) if needed.
- **Skip** definition **versions (preview)**, overrides and resource selectors. Not tested at foundation level.
- **Skip** the object-count limits table (500 definitions, 200 assignments per scope) and remediation tuning flags (`-ParallelDeploymentCount`, `-FailureThreshold`).
- **Skip** Resource Provider modes (`Microsoft.Kubernetes.Data`, `Microsoft.KeyVault.Data`) beyond knowing they let Policy look *inside* AKS and Key Vault.
- **Skip** policy-as-code pipelines here. Infrastructure as code is D1.3.9.

## Screenshots

![Choosing a policy definition from Available Definitions, with its version column](https://learn.microsoft.com/en-us/azure/governance/policy/media/assign-policy-portal/select-available-definition.png){Basics tab — pick the scope, then the definition. The Version column is preview; ignore it}

![Policy definition version options](https://learn.microsoft.com/en-us/azure/governance/policy/media/assign-policy-portal/select-version.png){Version options (preview) — enrol in minor or preview versions. Not exam material}

![Policy Compliance page with a non-compliant assignment](https://learn.microsoft.com/en-us/azure/governance/policy/media/assign-policy-portal/policy-compliance.png){Compliance — which assignments have non-compliant resources}

![Deleting a policy assignment from Compliance](https://learn.microsoft.com/en-us/azure/governance/policy/media/assign-policy-portal/delete-assignment.png){Clean-up — delete an assignment from the ellipsis menu}

![Compliance page with filtering options and details](https://learn.microsoft.com/en-us/azure/governance/policy/media/getting-compliance-data/compliance-page-small.png){Compliance page — filter by scope, type and state}

![Compliance details page with counts and resource details](https://learn.microsoft.com/en-us/azure/governance/policy/media/getting-compliance-data/compliance-details-small.png){One assignment's compliance details — which resources fail and why}

![Component Compliance tab for a Resource Provider mode assignment](https://learn.microsoft.com/en-us/azure/governance/policy/media/getting-compliance-data/compliance-components.png){Component compliance — Resource Provider mode policies (for example, inside a key vault) report per component}

![Activity log entries for Azure Policy activities and evaluations](https://learn.microsoft.com/en-us/azure/governance/policy/media/getting-compliance-data/compliance-activitylog-small.png){Policy writes audit and deny events to the Activity log — useful when you investigate "why was my deployment blocked?"}

![Resource compliance table showing a manual policy with compliance reason unknown](https://learn.microsoft.com/en-us/azure/governance/policy/media/effect-manual/manual-policy-portal.png){Manual effect — resources start as Unknown until someone attests them}

![Searching for Policy in All services](https://learn.microsoft.com/en-us/azure/governance/policy/media/remediate-resources/search-policy.png){Another route in: All services > Policy}

![The Remediation node on the Policy page](https://learn.microsoft.com/en-us/azure/governance/policy/media/remediate-resources/select-remediation.png){Remediation — lists every deployIfNotExists and modify assignment that can be remediated}

![Grid of resources to remediate](https://learn.microsoft.com/en-us/azure/governance/policy/media/remediate-resources/select-resources.png){New remediation task — narrow by scope and location, then Remediate}

![Remediation tasks tab showing progress](https://learn.microsoft.com/en-us/azure/governance/policy/media/remediate-resources/task-progress.png){Remediation tasks — progress of running and finished tasks}

![Context menu for a resource in a remediation task](https://learn.microsoft.com/en-us/azure/governance/policy/media/remediate-resources/resource-task-context-menu.png){Per-resource menu — open the deployment to read the error when a fix fails}

![Storage accounts exposed to public networks in a resource group](https://learn.microsoft.com/en-us/azure/governance/policy/media/getting-compliance-data/resource-group01.png){Compliance states example — some storage accounts expose public network access}

![Storage account compliance in a resource group](https://learn.microsoft.com/en-us/azure/governance/policy/media/getting-compliance-data/resource-group03.png){The same resource group after evaluation — compliant vs non-compliant}

![Policy compliance details from the Compliance page](https://learn.microsoft.com/en-us/azure/governance/policy/media/getting-compliance-data/simple-compliance.png){Compliance percentage = (Compliant + Exempt + Unknown) ÷ total resources — exempt resources count as compliant}
