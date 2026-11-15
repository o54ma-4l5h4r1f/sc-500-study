+++
slug = "azure-portal-cli-basics"
title = "Admin portals, Cloud Shell, Azure CLI and PowerShell — which tool, which portal"
summary = "Six admin portals split the Microsoft security estate — Azure portal, Entra admin center, Defender portal, Purview portal, Microsoft 365 admin center, Foundry portal. Know which one owns which control, and how to do the same job from Cloud Shell with az or Az PowerShell."
order = 2
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/azure-portal/azure-portal-overview",
  "https://learn.microsoft.com/en-us/azure/cloud-shell/overview",
  "https://learn.microsoft.com/en-us/azure/cloud-shell/get-started/ephemeral",
  "https://learn.microsoft.com/en-us/azure/cloud-shell/get-started/classic",
  "https://learn.microsoft.com/en-us/cli/azure/manage-azure-subscriptions-azure-cli",
  "https://learn.microsoft.com/en-us/powershell/azure/get-started-azureps",
  "https://learn.microsoft.com/en-us/entra/fundamentals/entra-admin-center",
  "https://learn.microsoft.com/en-us/defender-xdr/microsoft-365-defender-portal",
  "https://learn.microsoft.com/en-us/purview/purview-portal",
  "https://learn.microsoft.com/en-us/microsoft-365/admin/admin-overview/admin-center-overview",
  "https://learn.microsoft.com/en-us/azure/foundry/what-is-foundry",
]
captured = 2026-11-15
related_objectives = ["D1.1.1", "D1.1.2", "D1.2.1", "D1.3.5", "D3.1.2", "D3.1.9", "D3.1.11", "D4.1.3", "D4.2.1", "D4.3.1"]
+++

## In one sentence

Microsoft splits its security controls across **six admin portals**, and almost anything you can click in the Azure portal you can also type in **Cloud Shell** with the **Azure CLI (`az`)** or **Azure PowerShell (`Az` cmdlets)**.

## Why it exists

As an analyst you lived in one or two portals: the Defender portal for incidents and maybe the Azure portal for Sentinel. An engineer touches all of them. The first question in many exam items is silently "where would you go to do this?" — and the distractors are the other portals.

Think of a large hospital. Building maintenance (Azure portal) owns the rooms and the pipes. The badge office (Entra admin center) decides who gets in. Security control (Defender portal) watches the cameras. Records management (Purview) decides which files are confidential. HR (Microsoft 365 admin center) hands out licences and staff accounts. The research lab (Foundry) builds the new AI tools. Each office has its own front desk.

## Mental model

```svg
<svg viewBox="0 0 860 400" role="img" aria-label="Six admin portals and what each owns: Azure portal for Azure resources, Entra admin center for identities, Defender portal for security operations, Purview portal for data, Microsoft 365 admin center for licences and agents, Foundry portal for AI apps and agents">
  <rect class="box-accent" x="20" y="20" width="260" height="110" rx="8"/>
  <text x="150" y="48" text-anchor="middle" class="label-accent">Azure portal</text>
  <text x="150" y="72" text-anchor="middle" class="muted">portal.azure.com</text>
  <text x="150" y="96" text-anchor="middle">VMs, storage, Key Vault,</text>
  <text x="150" y="116" text-anchor="middle">Policy, RBAC, Defender for Cloud</text>
  <rect class="box" x="300" y="20" width="260" height="110" rx="8"/>
  <text x="430" y="48" text-anchor="middle" class="label-accent">Entra admin center</text>
  <text x="430" y="72" text-anchor="middle" class="muted">entra.microsoft.com</text>
  <text x="430" y="96" text-anchor="middle">users, apps, consent, PIM,</text>
  <text x="430" y="116" text-anchor="middle">Conditional Access, Agent ID</text>
  <rect class="box" x="580" y="20" width="260" height="110" rx="8"/>
  <text x="710" y="48" text-anchor="middle" class="label-accent">Defender portal</text>
  <text x="710" y="72" text-anchor="middle" class="muted">security.microsoft.com</text>
  <text x="710" y="96" text-anchor="middle">incidents, hunting, XDR,</text>
  <text x="710" y="116" text-anchor="middle">Sentinel, exposure mgmt</text>
  <rect class="box" x="20" y="160" width="260" height="110" rx="8"/>
  <text x="150" y="188" text-anchor="middle" class="label-accent">Purview portal</text>
  <text x="150" y="212" text-anchor="middle" class="muted">purview.microsoft.com</text>
  <text x="150" y="236" text-anchor="middle">labels, DLP, audit,</text>
  <text x="150" y="256" text-anchor="middle">DSPM for AI</text>
  <rect class="box" x="300" y="160" width="260" height="110" rx="8"/>
  <text x="430" y="188" text-anchor="middle" class="label-accent">Microsoft 365 admin center</text>
  <text x="430" y="212" text-anchor="middle" class="muted">admin.cloud.microsoft</text>
  <text x="430" y="236" text-anchor="middle">licences, users, billing,</text>
  <text x="430" y="256" text-anchor="middle">Agents (Agent 365 registry)</text>
  <rect class="box" x="580" y="160" width="260" height="110" rx="8"/>
  <text x="710" y="188" text-anchor="middle" class="label-accent">Foundry portal</text>
  <text x="710" y="212" text-anchor="middle" class="muted">ai.azure.com</text>
  <text x="710" y="236" text-anchor="middle">models, agents, guardrails,</text>
  <text x="710" y="256" text-anchor="middle">AI gateway, control plane</text>
  <line class="divider" x1="20" y1="300" x2="840" y2="300"/>
  <rect class="box-ok" x="160" y="320" width="540" height="60" rx="8"/>
  <text x="430" y="345" text-anchor="middle">Cloud Shell: az (Bash) or Az PowerShell</text>
  <text x="430" y="367" text-anchor="middle" class="muted">same Azure Resource Manager calls as the portal</text>
</svg>
```

## What actually matters

**1. Which portal owns what.** This is the single most testable fact on this page.

| You want to… | Go to |
|---|---|
| Create a key vault, storage account, VNet, VM; assign an **Azure** role; assign Azure Policy; add a resource lock | ==@Azure portal== |
| Enable Defender for Cloud plans, view recommendations and regulatory compliance | Azure portal (Defender for Cloud) — plan management can also move to the Defender portal **Cloud security** area |
| Create users/groups, app registrations, enterprise apps, consent settings, Conditional Access, PIM, authentication methods, **Entra** roles, Agent ID | ==@Entra admin center== |
| Investigate incidents, hunt, see Defender XDR, Sentinel in the unified view, attack paths and blast radius | ==@Defender portal== |
| Sensitivity labels, DLP, audit search, DSPM for AI | ==@Purview portal== |
| Assign licences, manage agents in the agent registry (Agent 365), tenant billing | ==@Microsoft 365 admin center== |
| Build and govern AI apps and agents, guardrails (content filters), connect an AI gateway | ==@Foundry portal== (Foundry resources also appear in the Azure portal) |

Many Entra blades are mirrored inside the Azure portal under **Microsoft Entra ID**. Both write to the same tenant. Microsoft's docs now point to the Entra admin center first.

**2. The Azure portal layout.** Global search bar at the top, a portal menu (flyout or docked), **Favorites**, and on each resource a **service menu** down the left with groups such as **Settings**, **Security**, **Monitoring**. Security settings for a resource usually sit under **Settings > Networking**, **Settings > Access control (IAM)** or the resource's **Security** group. Portal paths on this site are written as `Azure portal > Key vaults > {vault} > Settings > Access configuration`.

**3. Cloud Shell.** A browser terminal at `shell.azure.com` or the `>_` icon in the portal. Facts:

- Choose **Bash** (with `az`) or **PowerShell** (with `Az` cmdlets); both tools are preinstalled in either.
- You are **already signed in** as yourself — no `az login` needed.
- A session times out after ==@20 minutes== with no interactive activity.
- **Ephemeral session** = "No storage account required": files vanish when the session ends. **Mounted storage** = a 5-GB Azure Files share in a storage account you pay for; `$HOME` persists.
- The subscription needs the **Microsoft.CloudShell** resource provider registered.

**4. Azure CLI vs Azure PowerShell.** Same Azure Resource Manager underneath, different grammar.

| Job | Azure CLI | Azure PowerShell |
|---|---|---|
| Sign in (outside Cloud Shell) | `az login` | `Connect-AzAccount` |
| List subscriptions | `az account list` | `Get-AzSubscription` |
| Pick the subscription to work in | `az account set --subscription "{name or id}"` | `Set-AzContext -Subscription "{id}"` |
| Show current context | `az account show` | `Get-AzContext` |
| Create a resource group | `az group create -n rg-lab -l westeurope` | `New-AzResourceGroup -Name rg-lab -Location westeurope` |
| Assign an Azure role | `az role assignment create --assignee {id} --role "Reader" --scope {id}` | `New-AzRoleAssignment -ObjectId {id} -RoleDefinitionName Reader -Scope {id}` |

The **context** (CLI: *active subscription*) matters. A command runs against whichever subscription is current. Running a lab against the wrong subscription is the classic beginner mistake.

**5. Entra objects use Microsoft Graph, not Az.** `Az` cmdlets and most `az` groups manage **Azure resources**. Users, app registrations, consent grants and Conditional Access are **Entra** objects, managed through the **Microsoft Graph** API — in PowerShell via the Microsoft Graph PowerShell SDK (`Connect-MgGraph`, `Get-MgServicePrincipal`). The CLI has a small `az ad` group for apps and service principals.

## Where it lives

- Azure portal: `https://portal.azure.com` · Entra admin center: `https://entra.microsoft.com` · Defender portal: `https://security.microsoft.com` · Purview portal: `https://purview.microsoft.com` · Microsoft 365 admin center: `https://admin.cloud.microsoft` (older docs: `admin.microsoft.com`) · Foundry portal: `https://ai.azure.com` · Cloud Shell: `https://shell.azure.com`.
- **Azure portal > Subscriptions > {sub} > Settings > Resource providers** — register `Microsoft.CloudShell` (and any provider a lab needs).

![Screenshot showing the full screen portal view and a key to UI elements](https://learn.microsoft.com/en-us/azure/azure-portal/media/azure-portal-overview/portal-callouts.png){Azure portal layout: global search, portal menu, Cloud Shell icon, directory switcher}

![Screenshot of the Microsoft Entra admin center overview home page](https://learn.microsoft.com/en-us/entra/fundamentals/media/entra-admin-center/entra-admin-center-home.png){Entra admin center home — identity work starts here, not in the Azure portal}

## Exam traps

- ==!Entra roles are not assigned in Access control (IAM).== IAM is for **Azure** roles. Entra roles live in **Entra admin center > Roles & admins**.
- ==!Conditional Access and PIM are not Defender portal settings.== They are Entra.
- ==+Licences are assigned in the Microsoft 365 admin center== (or by group in Entra), not in the Azure portal.
- A question that says "by using the Azure CLI" still needs the right **scope** string and an active **subscription context**.
- Cloud Shell runs **as you** — it has exactly your permissions, nothing more.

## Skip

- **Skip** portal customisation: dashboards, themes, tile sizes, favourites management. Never tested.
- **Skip** installing the CLI or PowerShell locally, PowerShell context-persistence files and the `Azure:` drive. Cloud Shell has everything.
- **Skip** the Defender portal's notification and search panels, and the Purview portal's knowledge-center and trials cards. Know which portal; not every card on its home page.
- **Skip** the Foundry SDK, VS Code and `azd` developer surfaces. The exam asks about Foundry **controls**, not code.

## Screenshots

![Screenshot showing the Azure portal menu](https://learn.microsoft.com/en-us/azure/azure-portal/media/azure-portal-overview/azure-portal-menu.png){The portal menu}

![Screenshot of the Azure portal menu in flyout mode](https://learn.microsoft.com/en-us/azure/azure-portal/media/azure-portal-overview/azure-portal-overview-portal-menu-flyout.png){Portal menu in flyout mode}

![Screenshot of the Azure portal menu in docked mode](https://learn.microsoft.com/en-us/azure/azure-portal/media/azure-portal-overview/azure-portal-overview-portal-menu-expandcollapse.png){Portal menu in docked mode}

![Screenshot showing the star icon to add a service menu command to favorites](https://learn.microsoft.com/en-us/azure/azure-portal/media/azure-portal-overview/service-menu-add-favorite.png){Pinning a service-menu item to favourites}

![Screenshot of the Favorites folder in a service menu](https://learn.microsoft.com/en-us/azure/azure-portal/media/azure-portal-overview/service-menu-favorites.png){The Favorites folder in a resource's service menu}

![Screenshot of selecting resource providers in the Azure portal](https://learn.microsoft.com/en-us/azure/cloud-shell/get-started/media/ephemeral/resource-provider.png){Register the Microsoft.CloudShell resource provider once per subscription}

![Screenshot showing the prompt to select the shell](https://learn.microsoft.com/en-us/azure/cloud-shell/get-started/media/ephemeral/choose-shell.png){First launch: choose Bash or PowerShell}

![Screenshot showing the select subscription and optional storage prompt](https://learn.microsoft.com/en-us/azure/cloud-shell/get-started/media/ephemeral/getting-started.png){"No storage account required" = an ephemeral session}

![Screenshot showing the prompt to select the shell in the persistent-storage flow](https://learn.microsoft.com/en-us/azure/cloud-shell/get-started/media/new-storage/choose-shell.png){Persistent-storage flow: the same shell choice}

![Screenshot showing the select subscription and optional storage prompt in the persistent-storage flow](https://learn.microsoft.com/en-us/azure/cloud-shell/get-started/media/new-storage/getting-started.png){Persistent-storage flow: choose to mount a storage account}

![Screenshot showing the create storage account prompt](https://learn.microsoft.com/en-us/azure/cloud-shell/get-started/media/new-storage/create-storage.png){Creating the storage account that holds the 5-GB file share}

![Screenshot of the Microsoft Entra admin center Identity menu](https://learn.microsoft.com/en-us/entra/fundamentals/media/entra-admin-center/entra-admin-identity.png){Entra admin center: Identity menu (users, groups, applications, roles)}

![Screenshot of the Microsoft Entra admin center Protection menu](https://learn.microsoft.com/en-us/entra/fundamentals/media/entra-admin-center/entra-admin-protection.png){Protection menu: Conditional Access, ID Protection, authentication methods}

![Screenshot of the Microsoft Entra admin center Identity governance menu](https://learn.microsoft.com/en-us/entra/fundamentals/media/entra-admin-center/entra-admin-identity-governance.png){Identity governance menu: PIM, access reviews, entitlement management}

![Screenshot of the Microsoft Entra admin center Verified ID menu](https://learn.microsoft.com/en-us/entra/fundamentals/media/entra-admin-center/entra-admin-verified-id.png){Verified ID menu}

![Screenshot of the Microsoft Entra admin center Global Secure Access menu](https://learn.microsoft.com/en-us/entra/fundamentals/media/entra-admin-center/entra-admin-global-secure-access.png){Global Secure Access menu — where Entra Private Access is configured}

![Screenshot of the Microsoft Entra admin center Diagnose and solve menu](https://learn.microsoft.com/en-us/entra/fundamentals/media/entra-admin-center/entra-admin-diagnose-and-solve.png){Diagnose & solve menu}

![Screenshot of the Microsoft Entra admin center Learn and support menu](https://learn.microsoft.com/en-us/entra/fundamentals/media/entra-admin-center/entra-admin-learn-and-support.png){Learn & support menu}

![The Incidents page in the Microsoft Defender portal](https://learn.microsoft.com/en-us/defender-xdr/media/incidents-ss-incidents.png){Defender portal: the incident queue you know from SOC work}

![Screenshot that shows the attack story page for an incident in the Microsoft Defender portal](https://learn.microsoft.com/en-us/defender-xdr/media/microsoft-365-defender-portal/incidents-ss-incident-summary.png){Defender portal: an incident's attack story}

![Screenshot of the search bar in the Microsoft Defender portal](https://learn.microsoft.com/en-us/defender-xdr/media/microsoft-365-defender-portal/search-panel.png){Defender portal search bar}

![Screenshot of the notifications icon in the Microsoft Defender portal](https://learn.microsoft.com/en-us/defender-xdr/media/microsoft-365-defender-portal/notifications-panel.png){Defender portal notifications}

![Screenshot of the Microsoft Purview portal welcome dialog](https://learn.microsoft.com/en-us/purview/media/purview-portal-welcome.png){Purview portal welcome dialog}

![Screenshot of the Microsoft Purview portal home page](https://learn.microsoft.com/en-us/purview/media/purview-portal.png){Purview portal home: solution cards and compliance posture}

![Screenshot of Microsoft Purview portal search results](https://learn.microsoft.com/en-us/purview/media/purview-portal-search-results.png){Purview portal search}

![Screenshot of Microsoft Purview portal solution cards](https://learn.microsoft.com/en-us/purview/media/purview-portal-solution-cards.png){Purview solution cards}

![Screenshot of the Microsoft Purview portal solutions page](https://learn.microsoft.com/en-us/purview/media/purview-portal-solutions-page.png){Purview solutions page: core, risk and compliance, data governance, data security}

![Screenshot of the Compliance posture status card in the Microsoft Purview portal](https://learn.microsoft.com/en-us/purview/media/purview-portal-compliance-posture-status.png){Compliance posture status card}

![Screenshot of the Trials and recommendations card in the Microsoft Purview portal](https://learn.microsoft.com/en-us/purview/media/purview-portal-trials-recommendations.png){Trials and recommendations card}

![Screenshot of the Knowledge Center card in the Microsoft Purview portal](https://learn.microsoft.com/en-us/purview/media/purview-portal-knowledge-center.png){Knowledge Center card}

![Screenshot of the Microsoft Purview portal left navigation](https://learn.microsoft.com/en-us/purview/media/purview-portal-left-navigation.png){Purview left navigation}

![Screenshot of the Microsoft Purview portal settings pane](https://learn.microsoft.com/en-us/purview/media/purview-portal-settings.png){Purview personal settings pane}

![Screenshot of the Microsoft Purview portal solution settings page](https://learn.microsoft.com/en-us/purview/media/purview-portal-solution-settings.png){Purview solution settings}

![Screenshot of the Microsoft Purview portal help and support pane](https://learn.microsoft.com/en-us/purview/media/purview-portal-help-support.png){Purview help and support pane}

![Microsoft Defender XDR training icon.](https://learn.microsoft.com/en-us/defender-xdr/media/microsoft-365-defender-portal/m365-defender-secure-organization.png)
