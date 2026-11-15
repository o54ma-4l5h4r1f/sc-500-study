+++
slug = "azure-arc-and-hybrid-machines"
title = "Azure Arc and hybrid machines"
summary = "How a server outside Azure — on-premises, AWS EC2 or GCP — becomes an Azure resource through the Connected Machine agent, so Azure Policy, Machine Configuration, extensions and Defender for Servers can govern it like an Azure VM; plus onboarding roles, status lifecycle and the local lock-down controls."
order = 18
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/azure-arc/servers/overview",
  "https://learn.microsoft.com/en-us/azure/azure-arc/servers/agent-overview",
  "https://learn.microsoft.com/en-us/azure/azure-arc/servers/learn/quick-enable-hybrid-vm",
  "https://learn.microsoft.com/en-us/azure/azure-arc/servers/security-identity-authorization",
  "https://learn.microsoft.com/en-us/azure/azure-arc/servers/security-extensions",
  "https://learn.microsoft.com/en-us/azure/azure-arc/servers/manage-vm-extensions",
  "https://learn.microsoft.com/en-us/azure/governance/machine-configuration/overview",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/quickstart-onboard-machines",
]
captured = 2026-11-15
related_objectives = ["D3.2.4", "D3.2.5", "D3.2.6", "D3.2.9", "D4.1.4", "D1.3.1"]
+++

## In one sentence

**Azure Arc-enabled servers** gives a machine that runs *outside* Azure — a physical server in your datacenter, a VMware VM, an AWS EC2 instance or a GCP VM — a resource record and a managed identity *inside* Azure, so Azure Policy, extensions and Defender for Cloud can manage and protect it as if it were an Azure VM.

## Why it exists

Azure's controls only work on Azure **resources** — things with a Resource ID in a resource group (see `azure-resource-hierarchy`). An Azure VM is born as one. Your on-premises domain controller is not. So Policy can't audit it, Defender for Servers can't onboard it, and you can't push the Azure Monitor Agent to it.

Most companies are **hybrid**. A security engineer must apply the *same* baseline to every server, wherever it runs. Arc closes that gap.

Think of a **visitor badge**. A contractor walks into your building; the access-control system doesn't know them. Reception prints a badge with their name and photo. Now the doors, cameras and sign-in log recognise them. Azure Arc is the reception desk. The **Azure Connected Machine agent** is the badge printer.

## Mental model

```svg
<svg viewBox="0 0 780 340" role="img" aria-label="An on-premises or other-cloud server runs the Connected Machine agent, which creates an Arc machine resource with a managed identity in Azure. Azure Policy, Machine Configuration, extensions such as the Azure Monitor Agent and Defender for Endpoint, and Defender for Servers then act on that resource">
  <defs>
    <marker id="arcbarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="line-accent"/>
    </marker>
  </defs>
  <line x1="260" y1="20" x2="260" y2="320" class="divider"/>
  <text x="130" y="36" text-anchor="middle" class="muted">Outside Azure</text>
  <text x="520" y="36" text-anchor="middle" class="muted">Inside Azure</text>
  <rect class="box" x="20" y="50" width="220" height="260" rx="8"/>
  <text x="130" y="74" text-anchor="middle">On-prem / EC2 / GCP VM</text>
  <rect class="box-accent" x="35" y="90" width="190" height="80" rx="6"/>
  <text x="130" y="115" text-anchor="middle" class="label-accent">Connected Machine</text>
  <text x="130" y="135" text-anchor="middle" class="label-accent">agent (azcmagent)</text>
  <text x="130" y="157" text-anchor="middle" class="muted">outbound 443 only</text>
  <rect class="box-ok" x="35" y="190" width="190" height="100" rx="6"/>
  <text x="130" y="214" text-anchor="middle">Extensions</text>
  <text x="130" y="236" text-anchor="middle" class="muted">AMA, MDE.Windows,</text>
  <text x="130" y="256" text-anchor="middle" class="muted">Custom Script, Key</text>
  <text x="130" y="276" text-anchor="middle" class="muted">Vault, Machine Config</text>
  <rect class="box-accent" x="290" y="60" width="220" height="100" rx="8"/>
  <text x="400" y="88" text-anchor="middle" class="label-accent">Arc machine resource</text>
  <text x="400" y="112" text-anchor="middle">Microsoft.HybridCompute</text>
  <text x="400" y="134" text-anchor="middle" class="muted">RG, tags, RBAC, managed ID</text>
  <rect class="box" x="550" y="50" width="210" height="70" rx="8"/>
  <text x="655" y="80" text-anchor="middle">Azure Policy</text>
  <text x="655" y="102" text-anchor="middle" class="muted">+ Machine Configuration</text>
  <rect class="box" x="550" y="140" width="210" height="70" rx="8"/>
  <text x="655" y="170" text-anchor="middle">Defender for Servers</text>
  <text x="655" y="192" text-anchor="middle" class="muted">Defender for Cloud</text>
  <rect class="box" x="550" y="230" width="210" height="70" rx="8"/>
  <text x="655" y="260" text-anchor="middle">Log Analytics / Sentinel</text>
  <text x="655" y="282" text-anchor="middle" class="muted">via AMA + DCR</text>
  <line x1="225" y1="125" x2="288" y2="110" class="line-accent" marker-end="url(#arcbarrow)"/>
  <line x1="510" y1="95" x2="548" y2="85" class="line-accent" marker-end="url(#arcbarrow)"/>
  <line x1="510" y1="125" x2="548" y2="170" class="line-accent" marker-end="url(#arcbarrow)"/>
  <line x1="225" y1="260" x2="548" y2="265" class="line-accent" marker-end="url(#arcbarrow)"/>
</svg>
```

![Diagram of Azure Connected Machine agent architectural overview](https://learn.microsoft.com/en-us/azure/azure-arc/servers/media/agent-overview/connected-machine-agent.png){Inside the agent: HIMDS (identity and heartbeat), the machine configuration agent (policy), the extension manager (installs add-ons) and the Arc proxy}

## What actually matters

**1. An Arc machine is a real Azure resource.** It has a Resource ID (type `Microsoft.HybridCompute/machines`), sits in a resource group, takes tags and RBAC, and appears next to Azure VMs. ==!Arc is not for Azure VMs== — they're already resources, and Microsoft doesn't support Arc on them. The agent only needs **outbound HTTPS (443)** — directly, through a proxy, or over Private Link. No inbound ports.

**2. Onboarding.** In the portal you generate a script (`Azure Arc > Machines > Onboard/Create > Onboard existing machines`) choosing subscription, resource group, region (where metadata is stored), OS and connectivity method. Run it with local **Administrator** (Windows) or **root** (Linux). For many machines, use a **service principal** with the **Azure Connected Machine Onboarding** role, and deploy through Group Policy, Configuration Manager, Ansible or similar. For **AWS and GCP**, Defender for Cloud's multicloud connector can **auto-provision** Arc onto EC2 instances and GCP VMs — details in `aws-and-gcp-for-azure-people`.

| Role | Allows |
|---|---|
| **Azure Connected Machine Onboarding** | Read and create Arc machines. Can't delete them or manage extensions. For the onboarding account. |
| **Azure Connected Machine Resource Administrator** | Read, modify, re-onboard, delete machines; manage extensions. |
| Contributor / Owner | Also enough. |

**3. Every Arc machine gets a system-assigned managed identity.** Its certificate is valid for up to **90 days** and renews when **45** or fewer days remain. Apps on the machine can use it to reach Azure resources (Key Vault, Storage) without stored secrets.

**4. Status lifecycle.** The agent sends a heartbeat every **5 minutes**. No heartbeat → **Disconnected** within **15–30 minutes**. Disconnected for **45 days** → may become **Expired**; an admin must disconnect and reconnect it. While disconnected, extension operations queue for up to 6 hours and Policy guest assignments are kept for 14 days.

**5. Extensions are how controls arrive — and how attackers could.** The extension manager installs add-ons: **Azure Monitor Agent**, **Defender for Endpoint (MDE.Windows / MDE.Linux)**, **Key Vault**, **Custom Script**, **Run Command**, **Hybrid Runbook Worker**. They run as **Local System / root**. ==!Anyone with extension-write rights on the Arc resource is effectively a local admin on the server.== For sensitive servers (domain controllers, payment systems), lock the agent down **on the machine itself**:

| Control (`azcmagent config set …`) | Effect |
|---|---|
| `extensions.allowlist "Publisher/Type,…"` | Only listed extensions can install. Preferred over a block list. |
| `extensions.blocklist "…"` | Listed extensions refused |
| `extensions.enabled false` | Extension manager off |
| `guestconfiguration.enabled false` | Machine Configuration agent off |
| `incomingconnections.enabled false` | Remote access features off |
| `config.mode monitor` | Microsoft-managed allow list of monitoring/security extensions only; guest config and remote connectivity off |

These local settings ==+can't be overridden from Azure==, not even by an Owner or Global Administrator. Azure Policy can also restrict extensions, but anyone who can edit policy assignments can undo that.

**6. Policy and Machine Configuration on Arc.** Azure Policy assignments apply to Arc machines like any resource: DeployIfNotExists policies can install AMA or MDE at scale (see `azure-policy-basics`). **Azure Machine Configuration** (formerly Guest Configuration) audits — and optionally fixes — settings *inside* the OS (password policy, installed software, TLS settings). It's built into the Arc agent; Azure VMs need the extension instead (D3.2.9).

**7. Defender for Cloud uses the same door.** Defender for Cloud onboards non-Azure servers **through Arc**. Once an Arc machine sits in a subscription with **Defender for Servers** enabled, it gets Defender for Endpoint, vulnerability assessment and recommendations like an Azure VM (D3.2.5). Foundational CSPM recommendations and secure score also cover Arc machines.

## Where it lives

**Azure portal** (not the Defender portal):

| Task | Path |
|---|---|
| List Arc machines and status | `Azure Arc > Machines` |
| Generate an onboarding script | `Azure Arc > Machines > Onboard/Create > Onboard existing machines` |
| Extensions on one machine | `<machine> > Settings > Extensions` |
| Who can manage it | `<machine> > Access control (IAM)` |
| See it in Defender for Cloud | `Microsoft Defender for Cloud > Inventory` |

On the machine:

```bash
azcmagent show                                   # status, resource ID, service health
azcmagent check                                  # test connectivity to Arc endpoints
azcmagent config set extensions.allowlist "Microsoft.Azure.Monitor/AzureMonitorWindowsAgent,Microsoft.Azure.AzureDefenderForServers/MDE.Windows"
azcmagent config set guestconfiguration.enabled false
azcmagent config list
```

![Successful machine connection in the Azure portal](https://learn.microsoft.com/en-us/azure/azure-arc/servers/learn/media/quick-enable-hybrid-vm/enabled-machine.png){After the script runs, the server appears under Azure Arc > Machines with status Connected}

![Azure Arc-enabled server access control](https://learn.microsoft.com/en-us/azure/azure-arc/servers/media/security-identity-authorization/access-control-page.png){Access control (IAM) on an Arc machine — the same RBAC page as any Azure resource}

## Exam traps

- Non-Azure server + Azure Policy / Machine Configuration / Defender for Servers → ==+onboard to Azure Arc first==.
- Azure VM → ==!no Arc==.
- Onboarding many servers with least privilege → service principal with **Azure Connected Machine Onboarding** (not Contributor, not Resource Administrator).
- "Stop Azure admins pushing scripts to the domain controller" → local **extension allow list** (or monitor mode) on the server. Azure RBAC and Policy alone can be changed by Azure admins.
- AWS/GCP servers in Defender for Servers → the multicloud connector auto-provisions Arc (see `aws-and-gcp-for-azure-people`).
- The Connected Machine agent is not the log shipper. AMA is a separate extension.
- Disconnected 45 days → Expired → manual reconnect.

## Skip

- **Skip** agent folder paths, log paths, service process names and resource limits.
- **Skip** the instance metadata list (BIOS UUID, SMBIOS, cloud fields).
- **Skip** Arc gateway, Private Link Scope and proxy configuration detail. Know "outbound 443, public/proxy/private".
- **Skip** Azure Update Manager, Automation, change tracking, VM insights and Extended Security Updates via Arc. Server management, not security engineering here.
- **Skip** Arc for Kubernetes, Arc SQL Server and Arc data services; they belong to other objectives.
- **Skip** cloning guidance and disaster recovery for Arc agents.

## Screenshots

Every remaining image from the cited Microsoft Learn articles, per the image rule.

![A successful machine connection in the Azure portal.](https://learn.microsoft.com/en-us/azure/azure-arc/servers/media/quick-enable-hybrid-vm/enabled-machine.png)
