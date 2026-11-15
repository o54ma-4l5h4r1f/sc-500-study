+++
slug = "defender-portal-tour"
title = "Microsoft Defender portal refresher (engineer view)"
summary = "A short refresher on security.microsoft.com for someone who triaged incidents there: the four core experiences, Microsoft Defender unified RBAC (permission groups, workload activation), and where Defender for Cloud alerts, Microsoft Sentinel and Security Exposure Management show up for this exam."
order = 27
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/defender-xdr/microsoft-365-defender-portal",
  "https://learn.microsoft.com/en-us/defender-xdr/manage-rbac",
  "https://learn.microsoft.com/en-us/defender-xdr/activate-defender-rbac",
  "https://learn.microsoft.com/en-us/defender-xdr/compare-rbac-roles",
  "https://learn.microsoft.com/en-us/azure/sentinel/microsoft-sentinel-defender-portal",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/concept-integration-365",
  "https://learn.microsoft.com/en-us/security-exposure-management/microsoft-security-exposure-management",
]
captured = 2026-11-15
related_objectives = ["D4.2.1", "D4.2.2", "D3.1.5", "D4.1.1", "D4.1.3"]
+++

## In one sentence

The **Microsoft Defender portal** (`https://security.microsoft.com`) is the one console where alerts from every Defender product, Defender for Cloud and Microsoft Sentinel become incidents — and, for an engineer, where **unified RBAC** decides who can see and change what.

## Why it exists

From SOC work you know the portal as the incident queue. Its purpose is correlation: one attack touches email, a laptop, an identity and a cloud resource, and the portal stitches those alerts into one incident. The engineer view adds two jobs: *feeding* it (enabling the plans and connectors that produce the alerts) and *governing* it (roles and scopes).

## Mental model

Think of a **control room with one wall of screens**. Each Defender product, Defender for Cloud and Sentinel is a camera feed. The analyst watches the wall. The engineer installs the cameras (plans, connectors) and hands out the keys to the room (unified RBAC roles, scoped by data source).

## What's new for an engineer

- **The alert you triaged is produced by a plan you now enable.** A Defender for Storage alert exists only because someone turned on that plan in Defender for Cloud. Defender for Cloud alerts and incidents flow into the Defender portal automatically and correlate with endpoint, identity and email alerts.
- **Sentinel lives here now.** Microsoft Sentinel is generally available in the Defender portal, even without Defender XDR or E5. ==@After 31 March 2027 Sentinel is no longer supported in the Azure portal== — Defender portal only. New tenants since July 2025 whose onboarding user is Owner or User Access Administrator are onboarded to the Defender portal automatically.
- **Permissions moved too.** In the Azure portal Sentinel uses Azure RBAC. In the Defender portal it can use **Microsoft Defender unified RBAC**, activated per workspace.

## What actually matters

**1. Four core experiences.** **Incidents & alerts**, **Hunting**, **Actions & submissions**, **Threat analytics**. You see only what your licences include — a missing menu is usually a licence, not a fault.

**2. Unified RBAC.** One permission model for the Defender workloads, replacing each product's own roles. Custom roles combine permissions from three **permission groups**:

| Permission group | Covers |
|---|---|
| **Security operations** | incidents, alerts, hunting, response actions, cloud inventory |
| **Security posture** | vulnerability management, Secure Score, exposure management, recommendations |
| **Authorization and settings** | managing roles, core security settings (environment connections, plan enablement) |

Each assignment picks users/groups, **data sources** (for example Defender for Endpoint, Defender for Cloud, Exposure Management) and, for cloud, **scopes**. Workloads must be **activated** before unified RBAC is enforced for them — until then the old model applies. ==!Sentinel is activated per workspace.== Defender for Cloud and Exposure Management are active by default. Entra roles still work: **Security Administrator**, **Security Operator**, **Security Reader**, Global Administrator. You need Security Administrator or an **Authorization** permission to manage roles.

![Screenshot of the activate workloads page](https://learn.microsoft.com/en-us/defender-xdr/media/activate-defender-rbac/m365-defender-rbac-activate-workloads1.png){System > Permissions > Microsoft Defender XDR > Roles: Activate workloads}

**3. Where this exam's features surface.**

| Feature | In the Defender portal |
|---|---|
| Defender for Cloud alerts | **Incidents & alerts**, correlated with other products |
| Cloud assets and posture | the cloud asset inventory under **Assets**, and Exposure management |
| Microsoft Sentinel | **Microsoft Sentinel** section (Configuration, content hub, data connectors) |
| Security Exposure Management | **Exposure management** — attack surface map, attack paths, initiatives, Secure Score |
| AI agent blast radius, real-time protection | Exposure graph / AI agent pages (D3.1.5) |

**Security Exposure Management** builds an **exposure graph** of devices, identities, cloud assets (Azure, AWS, GCP via Defender for Cloud) and the external attack surface, then shows **attack paths** to critical assets. It is limited or unavailable if you run Sentinel in the Defender portal without Defender services.

![Screenshot of the Defender portal left navigation with the Microsoft Sentinel section.](https://learn.microsoft.com/en-us/azure/sentinel/media/microsoft-sentinel-defender-portal/navigation-defender-portal.png){Sentinel's menus inside the Defender portal}

## Where it lives

- Portal: `https://security.microsoft.com`.
- Roles: `System > Permissions > Microsoft Defender XDR > Roles` → **Create custom role**, **Workload settings** / **Activate workloads**.
- Settings: `System > Settings > Microsoft Defender XDR`.
- Sentinel workspaces: `System > Settings > Microsoft Sentinel`.

## Exam traps

- ==!Creating a unified RBAC role does nothing for a workload that is not activated.== Activate the workload (per workspace for Sentinel).
- ==!"Sentinel in the Azure portal is supported indefinitely."== Support ends **31 March 2027**.
- Defender for Cloud alerts appear in the Defender portal only for **enabled plans** — no plan, no alert.
- Unified RBAC in the Defender portal and Azure RBAC in the Azure portal are separate systems; Defender for Cloud plan enablement in Azure still needs Azure roles such as **Security Admin** on the subscription.

## Skip

- **Skip:** incident-page tabs, the alert queue filters and Go hunt — analyst ground, not engineering.
- **Skip:** multitenant management and importing legacy roles.
- **Skip:** the full unified RBAC permission catalogue; know the three groups and activation.

## Screenshots

![The Incidents page in the Microsoft Defender portal.](https://learn.microsoft.com/en-us/defender-xdr/media/incidents-ss-incidents.png){Incidents queue}

![Screenshot that shows the attack story page for an incident in the Microsoft Defender portal.](https://learn.microsoft.com/en-us/defender-xdr/media/microsoft-365-defender-portal/incidents-ss-incident-summary.png){Attack story of an incident}

![Screenshot of the search bar in the Microsoft Defender portal.](https://learn.microsoft.com/en-us/defender-xdr/media/microsoft-365-defender-portal/search-panel.png){Unified search}

![Screenshot of the notifications icon in the Microsoft Defender portal.](https://learn.microsoft.com/en-us/defender-xdr/media/microsoft-365-defender-portal/notifications-panel.png){Notifications}

![Microsoft Defender XDR training icon.](https://learn.microsoft.com/en-us/defender-xdr/media/microsoft-365-defender-portal/m365-defender-secure-organization.png){Learning hub entry}

![Screenshot of the page where you can choose workloads to activate.](https://learn.microsoft.com/en-us/defender-xdr/media/activate-defender-rbac/defender-activate-workloads.png){Workload toggles; View Workspaces activates Sentinel per workspace}
