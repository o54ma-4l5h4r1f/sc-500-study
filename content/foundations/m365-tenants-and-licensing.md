+++
slug = "m365-tenants-and-licensing"
title = "Tenants, subscriptions and licences: Entra ID Free/P1/P2, Entra Suite, Microsoft 365 E3/E5/E7, Copilot, Agent 365, Azure-billed Defender plans"
summary = "Two billing worlds share one Entra tenant: per-user licences (Microsoft 365, Entra ID P1/P2, Entra Suite, Copilot, Agent 365) and pay-as-you-go Azure subscriptions (Defender for Cloud plans, Key Vault, Sentinel). Knowing which licence unlocks which control answers many 'what do you need first?' questions."
order = 8
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/microsoft-365/enterprise/subscriptions-licenses-accounts-and-tenants-for-microsoft-cloud-offerings",
  "https://learn.microsoft.com/en-us/entra/fundamentals/licensing",
  "https://learn.microsoft.com/en-us/entra/fundamentals/what-is-entra",
  "https://learn.microsoft.com/en-us/copilot/microsoft-365/microsoft-365-copilot-licensing",
  "https://learn.microsoft.com/en-us/microsoft-agent-365/overview",
  "https://learn.microsoft.com/en-us/copilot/security/security-copilot-inclusion",
  "https://learn.microsoft.com/en-us/copilot/security/security-compute-units-capacity",
  "https://learn.microsoft.com/en-us/defender-xdr/prerequisites",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-cloud-introduction",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/connect-azure-subscription",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/concept-cloud-security-posture-management",
]
captured = 2026-11-15
related_objectives = ["D1.1.1", "D1.1.2", "D1.1.3", "D2.3.5", "D3.1.2", "D3.1.4", "D3.1.6", "D3.1.11", "D4.1.1", "D4.1.3", "D4.3.1"]
+++

## In one sentence

One **Microsoft Entra tenant** sits behind two different billing worlds — **per-user licences** (Microsoft 365, Entra ID P1/P2, Entra Suite, Copilot, Agent 365) and **pay-as-you-go Azure subscriptions** (Defender for Cloud plans, Key Vault, Sentinel, Foundry) — and many security controls simply don't appear until the right one is in place.

## Why it exists

Microsoft sells features in layers. The exam uses that: "What should you do **first**?" is often "acquire Entra ID P2", and a distractor may be a control that needs a licence the scenario lacks.

Think of a gym. Your **membership card** (a per-user licence) lets *you* use the pool and classes. The **building's electricity bill** (the Azure subscription) is paid by the amount used, whoever uses it. Both belong to the same gym (the tenant). Buying more memberships does nothing for the electricity, and paying the electricity bill doesn't let anyone into a class.

For your own labs this matters too: you have an **Azure subscription only**. Anything per-user-licensed (P1/P2, E5, Copilot, Agent 365) is a portal tour on this site.

## Mental model

```svg
<svg viewBox="0 0 880 340" role="img" aria-label="One Entra tenant with two billing worlds: per-user licences assigned in the Microsoft 365 admin center (Entra ID P1 and P2, Entra Suite, Microsoft 365 E3, E5, E7, Copilot, Agent 365) and Azure subscriptions billed by consumption (Defender for Cloud plans, Key Vault, Sentinel, Foundry, Security Copilot SCUs)">
  <rect class="box-accent" x="200" y="15" width="480" height="50" rx="8"/>
  <text x="440" y="46" text-anchor="middle" class="label-accent">Microsoft Entra tenant (one directory)</text>
  <line class="line" x1="330" y1="65" x2="230" y2="100"/>
  <line class="line" x1="550" y1="65" x2="650" y2="100"/>
  <rect class="box" x="20" y="100" width="410" height="220" rx="8"/>
  <text x="225" y="128" text-anchor="middle" class="label-accent">Per-user licences</text>
  <text x="225" y="150" text-anchor="middle" class="muted">assigned to users (M365 admin center)</text>
  <text x="40" y="182">Entra ID Free / P1 / P2, ID Governance</text>
  <text x="40" y="206">Entra Suite (Private Access, ...)</text>
  <text x="40" y="230">Microsoft 365 E3 / E5 / E7</text>
  <text x="40" y="254">Microsoft Copilot add-on</text>
  <text x="40" y="278">Microsoft Agent 365</text>
  <text x="40" y="302" class="muted">Workload ID: per workload identity</text>
  <rect class="box" x="450" y="100" width="410" height="220" rx="8"/>
  <text x="655" y="128" text-anchor="middle" class="label-accent">Azure subscription</text>
  <text x="655" y="150" text-anchor="middle" class="muted">billed by consumption (Azure invoice)</text>
  <text x="470" y="182">Defender for Cloud plans (CSPM, CWP)</text>
  <text x="470" y="206">Key Vault, Firewall, Bastion, VMs</text>
  <text x="470" y="230">Sentinel / Log Analytics ingestion</text>
  <text x="470" y="254">Foundry models, API Management</text>
  <text x="470" y="278">Security Copilot SCUs (non-E5)</text>
  <text x="470" y="302" class="muted">enabled per subscription or resource</text>
</svg>
```

## What actually matters

**1. The vocabulary.** An **organisation** holds **subscriptions** (agreements to use a service). SaaS subscriptions such as Microsoft 365 carry **licences**, each assigned to one **user account**. Azure subscriptions are charged by **resource consumption**. All of them trust the same **tenant**.

**2. Microsoft Entra ID editions — the most-tested ladder.**

| Edition | Adds (security-relevant) | Included in |
|---|---|---|
| **Free** | Users, groups, built-in roles, **security defaults**, basic sign-in and audit logs | Every Azure / Microsoft 365 subscription |
| **P1** | ==@Conditional Access==, dynamic groups, Entra **custom roles**, administrative-unit admins, app proxy | Microsoft 365 E3, E5, E7, Business Premium, F1/F3, EMS E3 |
| **P2** | ==@PIM==, **ID Protection** (risk-based Conditional Access), access reviews | Microsoft 365 E5, E7, Microsoft Defender Suite, EMS E5 |
| **Entra ID Governance** | Lifecycle workflows, advanced access reviews and entitlement management (PIM also licensed here) | Entra Suite, Microsoft 365 E7 |
| **Entra Suite** | Private Access, Internet Access, ID Governance, ID Protection, Verified ID premium | Standalone (needs P1 underneath) or Microsoft 365 E7 |
| **Workload ID** | Conditional Access, risk and reviews for service principals | Per workload identity per month |

Managed identities need **no** licence. Agent ID itself is available to all Entra customers, but **extending Entra security features to agents** (Conditional Access, ID Protection, lifecycle) needs **Microsoft Agent 365**.

**3. Microsoft 365 suites.** **E3** = the productivity suite plus **Entra ID P1**. **E5** = E3 + **Entra ID P2** + **Defender XDR** (Defender for Endpoint, Office 365, Identity, Cloud Apps) and Microsoft's premium compliance tooling. **E7** = E5 + **Microsoft Copilot** + **Agent 365** + **Entra Suite**. Defender XDR needs an E5-level licence (E5/A5, E3 + EMS E5, or equivalent security add-ons).

**4. AI licences.**

- **Microsoft Copilot** (the Microsoft 365 AI assistant): a per-user **add-on** for eligible plans such as E3 and E5; **included** in E7. **Copilot Chat** (web-grounded) comes with Microsoft 365 suites.
- **Microsoft Agent 365**: per-user; the control plane for agents (agent registry in the Microsoft 365 admin center). Included in E7, add-on to E5.
- **Security Copilot**: priced in **Security Compute Units (SCUs)** — **provisioned** (billed hourly, minimum 1) plus optional **overage**, billed through Azure. For Microsoft 365 **E5 and E7** tenants it is **included** and auto-provisioned: 400 SCUs per month per 1,000 user licences (rollout began 18 November 2025).

**5. Defender for Cloud is Azure-billed, not per-user.** Enabling Defender for Cloud gives **Foundational CSPM** free (secure score, recommendations, the Microsoft cloud security benchmark). **Defender CSPM** and the workload protection plans (**Defender for Servers, Storage, Databases, Containers, Key Vault, App Service, AI Services**…) are paid per protected resource on the **Azure subscription's** bill. Each plan gets a **30-day free trial** per subscription. To enable a plan you need **Security Admin** or **Owner** on the subscription. ==!An E5 licence does not turn on Defender for Servers.==

**6. When a licence lapses.** Conditional Access policies stay but can't be edited. PIM eligible assignments are **removed**. Plan your labs knowing that trials end.

## Where it lives

- **Microsoft 365 admin center > Billing > Licenses** — what you own; **Users > Active users > {user} > Licenses and apps** — assign.
- **Entra admin center > Entra ID > Overview** — shows the tenant's Entra edition; **Billing > Licenses** in Entra mirrors assignments; group-based licensing assigns by group.
- **Azure portal > Microsoft Defender for Cloud > Environment settings > {subscription} > Defender plans** — turn plans on/off and see trial status.
- **Azure portal > Cost Management + Billing** — the Azure side of the bill.

![An example organization with multiple subscriptions for Microsoft's cloud offerings](https://learn.microsoft.com/en-us/microsoft-365/media/subscriptions/subscriptions-fig1.png){One organisation, several subscriptions: Microsoft 365, Dynamics 365, Azure}

![An example organization with multiple subscriptions all using the same Microsoft Entra tenant](https://learn.microsoft.com/en-us/microsoft-365/media/subscriptions/subscriptions-fig3.png){All of them trust the same Entra tenant}

## Exam traps

- ==!Conditional Access is P1; PIM and risk-based policies are P2.== "Require MFA when sign-in risk is high" → P2.
- ==!Entra custom roles need P1 per assigned user==; Azure custom roles need no licence.
- ==+Defender for Cloud plans are enabled per subscription== (Azure billing), not by assigning user licences.
- Security defaults (free) and Conditional Access (P1) are alternatives, not layers.
- Entra Private Access needs **Entra Suite** (or the standalone Private Access licence), not just P1.
- Microsoft renames things: Azure AD → Entra ID; Microsoft 365 Copilot → Microsoft Copilot; Microsoft 365 E5 Security → Microsoft Defender Suite. Old names can still appear.

## Skip

- **Skip** prices, commitment tiers and the Entra governance feature-by-licence matrix row by row — those belong in reference pages, not exam prep.
- **Skip** Government, Education (A-series) and frontline (F-series) plan lists.
- **Skip** Tenant Governance limits (monitors, snapshots), External ID MAU billing and Domain Services SKUs.
- **Skip** Copilot eligibility lists for Exchange/SharePoint/Visio standalone plans.

## Screenshots

![An example of multiple licenses within subscriptions for Microsoft's SaaS-based cloud offerings](https://learn.microsoft.com/en-us/microsoft-365/media/subscriptions/subscriptions-fig2.png){Licences live inside SaaS subscriptions and are assigned to users}

![Identity as a Service (IDaaS) for your organization](https://learn.microsoft.com/en-us/microsoft-365/media/subscriptions/subscriptions-fig4.png){Entra ID as the shared identity service for every Microsoft cloud subscription}

![Diagram showing the core functionality of Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/defender-for-cloud-introduction/cloud-security-pillars.png){Defender for Cloud pillars: DevSecOps, CSPM, workload protection}

![Conceptual image of CNAPP and how the Defender for Cloud plans protect all of your resources](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/defender-for-cloud-introduction/defender-plans.png){The Defender plans — each one billed on the Azure subscription}

![Screenshot of the Azure portal with Microsoft Defender for Cloud selected](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/get-started/defender-for-cloud-search.png){Finding Defender for Cloud in the Azure portal}

![Screenshot of the Defender for Cloud overview dashboard](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/overview-page/overview.png){Defender for Cloud overview — available with the free Foundational CSPM}

![Screenshot that shows where to navigate to, to select environmental settings from](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/get-started/environmental-settings.png){Environment settings — where plans are enabled per subscription}

![Screenshot that shows where the enable button is located on the plans page](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/get-started/enable-all-plans.png){Enabling paid plans starts each plan's 30-day trial}

![Image showing security agents in your daily workflow](https://learn.microsoft.com/en-us/copilot/security/media/agents-in-workflow.png){Security Copilot agents included with Microsoft 365 E5 and E7}

![Image of the message center post messaging](https://learn.microsoft.com/en-us/copilot/security/media/message-center-post.png){The Microsoft 365 message center post announcing Security Copilot inclusion}

![Four pillars of Microsoft Agent 365: management, governance, security, and business value](https://learn.microsoft.com/en-us/microsoft-agent-365/media/overview/agent-365-four-pillars.png){Agent 365 — licensed per user; included in Microsoft 365 E7}
