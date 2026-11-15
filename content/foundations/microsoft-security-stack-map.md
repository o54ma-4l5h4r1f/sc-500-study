+++
slug = "microsoft-security-stack-map"
title = "Microsoft security stack map for the engineer: which product owns which control, in which portal"
summary = "A one-page map of the products this exam touches — Entra (incl. Agent ID), Key Vault, Azure Policy, Defender for Cloud (CSPM + CWP), Defender XDR, Sentinel, Purview, Security Copilot, Foundry and Agent 365 — what each protects, who configures it, and which portal it lives in."
order = 9
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/entra/fundamentals/whatis",
  "https://learn.microsoft.com/en-us/azure/key-vault/general/overview",
  "https://learn.microsoft.com/en-us/azure/governance/policy/overview",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-cloud-introduction",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/security-policy-concept",
  "https://learn.microsoft.com/en-us/defender-xdr/microsoft-365-defender",
  "https://learn.microsoft.com/en-us/azure/sentinel/overview",
  "https://learn.microsoft.com/en-us/purview/purview",
  "https://learn.microsoft.com/en-us/copilot/security/microsoft-security-copilot",
  "https://learn.microsoft.com/en-us/azure/foundry/what-is-foundry",
  "https://learn.microsoft.com/en-us/entra/agent-id/what-is-microsoft-entra-agent-id",
  "https://learn.microsoft.com/en-us/microsoft-agent-365/overview",
]
captured = 2026-11-15
related_objectives = ["D1.1.2", "D1.2.1", "D1.3.1", "D1.3.3", "D3.1.2", "D3.1.6", "D3.1.8", "D3.1.9", "D3.1.10", "D3.1.11", "D4.1.1", "D4.1.3", "D4.2.1", "D4.3.1"]
+++

## In one sentence

This exam spans about ten Microsoft products, and each one owns a **different layer** — identity, secrets, rules, cloud posture, threat detection, SIEM, data, AI — so the first step in any question is "which product, in which portal?"

## Why it exists

As an analyst you saw the **output** of this stack: alerts and incidents. An engineer configures the **inputs**: the policy that blocks a public storage account, the plan that scans a VM, the guardrail that filters a prompt. Microsoft's product names overlap ("Defender for Cloud" vs "Defender XDR"; "Copilot" vs "Security Copilot"), and distractors exploit that.

Think of a city. **Entra** is the passport office. **Key Vault** is the bank vault. **Azure Policy** is the building code. **Defender for Cloud** is the building inspector plus the alarm system on each building. **Defender XDR** guards people's devices, mailboxes and accounts. **Sentinel** is the city-wide CCTV control room. **Purview** classifies documents and stops them leaving. **Foundry** is the AI workshop, and **Agent 365** is the register of every robot working in town.

## Mental model

```svg
<svg viewBox="0 0 880 520" role="img" aria-label="Security stack in layers: identity (Entra ID, Agent ID), protect and govern (Key Vault, Azure Policy, Defender for Cloud), detect and respond (Defender XDR, Sentinel, Security Copilot), data (Purview), AI platform and agents (Foundry, Agent 365); each layer labelled with its portal">
  <rect class="box-accent" x="20" y="20" width="840" height="70" rx="8"/>
  <text x="440" y="46" text-anchor="middle" class="label-accent">Identity — Microsoft Entra ID, Entra Agent ID</text>
  <text x="440" y="72" text-anchor="middle" class="muted">Entra admin center: users, apps, consent, Conditional Access, PIM, agent identities</text>
  <rect class="box" x="20" y="110" width="270" height="120" rx="8"/>
  <text x="155" y="138" text-anchor="middle" class="label-accent">Key Vault</text>
  <text x="155" y="162" text-anchor="middle">keys, secrets, certs</text>
  <text x="155" y="210" text-anchor="middle" class="muted">Azure portal</text>
  <rect class="box" x="305" y="110" width="270" height="120" rx="8"/>
  <text x="440" y="138" text-anchor="middle" class="label-accent">Azure Policy</text>
  <text x="440" y="162" text-anchor="middle">rules on resources</text>
  <text x="440" y="186" text-anchor="middle">Audit, Deny, Modify, DINE</text>
  <text x="440" y="210" text-anchor="middle" class="muted">Azure portal</text>
  <rect class="box" x="590" y="110" width="270" height="120" rx="8"/>
  <text x="725" y="138" text-anchor="middle" class="label-accent">Defender for Cloud</text>
  <text x="725" y="162" text-anchor="middle">CSPM = posture</text>
  <text x="725" y="186" text-anchor="middle">CWP plans = threat alerts</text>
  <text x="725" y="210" text-anchor="middle" class="muted">Azure portal (+ Defender)</text>
  <rect class="box" x="20" y="250" width="270" height="120" rx="8"/>
  <text x="155" y="278" text-anchor="middle" class="label-accent">Defender XDR</text>
  <text x="155" y="302" text-anchor="middle">endpoint, email,</text>
  <text x="155" y="326" text-anchor="middle">identity, SaaS apps</text>
  <text x="155" y="350" text-anchor="middle" class="muted">Defender portal</text>
  <rect class="box" x="305" y="250" width="270" height="120" rx="8"/>
  <text x="440" y="278" text-anchor="middle" class="label-accent">Microsoft Sentinel</text>
  <text x="440" y="302" text-anchor="middle">SIEM + SOAR</text>
  <text x="440" y="326" text-anchor="middle">any log source</text>
  <text x="440" y="350" text-anchor="middle" class="muted">Defender portal</text>
  <rect class="box" x="590" y="250" width="270" height="120" rx="8"/>
  <text x="725" y="278" text-anchor="middle" class="label-accent">Security Copilot</text>
  <text x="725" y="302" text-anchor="middle">gen-AI assistant, agents</text>
  <text x="725" y="326" text-anchor="middle">billed in SCUs</text>
  <text x="725" y="350" text-anchor="middle" class="muted">own portal + embedded</text>
  <rect class="box" x="20" y="390" width="270" height="110" rx="8"/>
  <text x="155" y="418" text-anchor="middle" class="label-accent">Microsoft Purview</text>
  <text x="155" y="442" text-anchor="middle">labels, DLP, audit,</text>
  <text x="155" y="466" text-anchor="middle">DSPM for AI</text>
  <text x="155" y="488" text-anchor="middle" class="muted">Purview portal</text>
  <rect class="box-ok" x="305" y="390" width="270" height="110" rx="8"/>
  <text x="440" y="418" text-anchor="middle" class="label-accent">Microsoft Foundry</text>
  <text x="440" y="442" text-anchor="middle">models, agents,</text>
  <text x="440" y="466" text-anchor="middle">guardrails, AI gateway</text>
  <text x="440" y="488" text-anchor="middle" class="muted">Foundry portal</text>
  <rect class="box-ok" x="590" y="390" width="270" height="110" rx="8"/>
  <text x="725" y="418" text-anchor="middle" class="label-accent">Microsoft Agent 365</text>
  <text x="725" y="442" text-anchor="middle">agent registry, governance</text>
  <text x="725" y="466" text-anchor="middle">across Entra/Defender/Purview</text>
  <text x="725" y="488" text-anchor="middle" class="muted">Microsoft 365 admin center</text>
</svg>
```

## What actually matters

**1. The products, one line each.**

| Product | What it does | Where you configure it | Billing |
|---|---|---|---|
| **Microsoft Entra ID** | Identities, sign-in, tokens, Conditional Access, PIM | Entra admin center | Per-user (Free/P1/P2) |
| **Entra Agent ID** | Identities built for AI agents (agent identities, blueprints, sponsors) | Entra admin center | Available to all; security features need Agent 365 |
| **Azure Key Vault** | Stores and controls keys, secrets and certificates (Standard and Premium tiers; Managed HSM for single-tenant HSMs) | Azure portal | Azure |
| **Azure Policy** | Evaluates resource properties against rules; effects such as **Audit**, **Deny**, **Modify**, **DeployIfNotExists** | Azure portal | Free |
| **Defender for Cloud** | A **CNAPP**: **CSPM** (posture — secure score, recommendations, compliance, attack paths) + **CWP** plans (threat detection for servers, storage, databases, containers, Key Vault, AI services…) + DevOps security | Azure portal; plans can also be managed from the Defender portal | Azure, per plan |
| **Defender XDR** | Correlates Defender for Endpoint, Office 365, Identity and Cloud Apps into incidents | Defender portal | Per-user (E5-level) |
| **Microsoft Sentinel** | Cloud SIEM and SOAR on a Log Analytics workspace | Defender portal (Azure portal support ends after ==@31 March 2027==) | Azure (ingestion/retention) |
| **Microsoft Purview** | Data security, governance and compliance: sensitivity labels, DLP, audit, insider risk, **DSPM for AI** | Purview portal | Per-user |
| **Security Copilot** | Generative-AI assistant and agents for security teams | Standalone portal and embedded in Defender, Entra, Purview, Intune | SCUs (included with E5/E7) |
| **Microsoft Foundry** | Platform to build and run AI models and agents with RBAC, networking, guardrails and policy under one Azure resource provider | Foundry portal (and Azure portal) | Azure |
| **Microsoft Agent 365** | Control plane for all agents: registry, governance, security signals from Defender, Entra and Purview | Microsoft 365 admin center | Per-user |

**2. Defender for Cloud ≠ Defender XDR.** Defender for **Cloud** protects **Azure, AWS and GCP resources** (VMs, storage, SQL, containers) and their configuration. Defender **XDR** protects **people's stuff** — devices, mailboxes, identities, SaaS apps. Both feed incidents into the Defender portal, which is why they blur. Defender for Servers uses Defender for **Endpoint** under the hood — the one bridge between them.

**3. CSPM vs CWP — the two halves of Defender for Cloud.** **CSPM** (cloud security posture management) asks "is it configured safely?" — preventive, produces **recommendations**. **CWP** (cloud workload protection) asks "is it being attacked right now?" — detective, produces **security alerts**. Foundational CSPM is free; Defender CSPM and every CWP plan are paid.

**4. Azure Policy vs Defender for Cloud.** In Azure, Defender for Cloud's security standards (such as the **Microsoft cloud security benchmark**, MCSB) are built from Azure Policy definitions and surface as **recommendations**. Policy **enforces** (Deny); Defender for Cloud **reports and prioritises**. Defender-portal posture policies are Audit-only — ==!Deny must still come from Azure Policy==.

**5. "Copilot" means three different things.** **Microsoft Copilot** = the Microsoft 365 assistant for knowledge workers. **Security Copilot** = the security-team assistant. **Copilot Studio** = a low-code tool for building agents. Purview DSPM for AI and Copilot Studio real-time protection secure the first and third; Security Copilot is a tool you configure for your SOC.

**6. The AI security chain.** A Foundry agent is built in **Foundry** (with **guardrails**), may be fronted by an **AI gateway** in API Management, gets an identity from **Entra Agent ID**, is inventoried in **Agent 365**, has its data use watched by **Purview DSPM for AI**, and its threats detected by **Defender for AI Services** (a Defender for Cloud CWP plan). The exam's AI section walks this chain.

## Where it lives

- Entra admin center `entra.microsoft.com` · Azure portal `portal.azure.com` (Key Vault, Policy, Defender for Cloud) · Defender portal `security.microsoft.com` (XDR, Sentinel, cloud security, exposure management) · Purview portal `purview.microsoft.com` · Security Copilot `securitycopilot.microsoft.com` · Foundry portal `ai.azure.com` · Microsoft 365 admin center `admin.cloud.microsoft` (Agents).
- The portal-by-task table is on the *Admin portals, Cloud Shell, Azure CLI and PowerShell* primer.

![Diagram showing the core functionality of Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/defender-for-cloud-introduction/cloud-security-pillars.png){Defender for Cloud: DevSecOps, CSPM and cloud workload protection}

![Diagram showing the identity management, access protection, governance, and compliance capabilities that Microsoft Entra Agent ID provides for AI agents](https://learn.microsoft.com/en-us/entra/agent-id/media/what-is-microsoft-entra-agent-id/microsoft-entra-agent-identity-capabilities.png){Entra Agent ID: identity, access protection, governance and compliance for agents}

## Exam traps

- ==!Defender for Cloud is not Defender XDR.== VM vulnerability scanning, storage malware scanning, SQL threat alerts → Defender for Cloud plans. Phishing email, endpoint EDR incidents → Defender XDR.
- ==!Defender for Cloud doesn't block deployments.== Azure Policy **Deny** does.
- ==+Sentinel's future is the Defender portal.== Azure portal support ends after 31 March 2027.
- **Purview** protects *data* (labels, DLP); **Key Vault** protects *keys and secrets*. "Classify documents" ≠ Key Vault.
- **Agent 365** inventories and governs agents across vendors; **Foundry** builds them; **Entra Agent ID** gives them identities. Don't swap them.

## Skip

- **Skip** each product's full feature tour — later objective pages teach the controls the exam asks about.
- **Skip** Purview's data-governance (Unified Catalog, data map) side; the exam's Purview items are DSPM for AI and audit.
- **Skip** Sentinel's analyst features (hunting, workbooks, UEBA) beyond what D4.2 quick reviews cover.
- **Skip** Security Copilot's prompt-writing guidance; the exam asks about workspaces, roles, plugins and agents.
- **Skip** Agent 365's cost-management and business-value pillar.

## Screenshots

![Conceptual image of CNAPP and how the Defender for Cloud plans protect all of your resources](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/defender-for-cloud-introduction/defender-plans.png){The Defender for Cloud plans across environments}

![Screenshot of the Microsoft Sentinel content hub in the Defender portal](https://learn.microsoft.com/en-us/azure/sentinel/media/overview/content-hub-defender-portal.png){Sentinel content hub in the Defender portal}

![Screenshot of the Microsoft Sentinel content hub in the Azure portal](https://learn.microsoft.com/en-us/azure/sentinel/media/overview/content-hub-azure-portal.png){The same content hub in the Azure portal}

![Screenshot of the Microsoft Sentinel data connectors page in the Defender portal](https://learn.microsoft.com/en-us/azure/sentinel/media/overview/data-connector-list-defender.png){Data connectors, Defender portal}

![Screenshot of the data connectors page in Microsoft Sentinel](https://learn.microsoft.com/en-us/azure/sentinel/media/overview/data-connectors.png){Data connectors, Azure portal}

![Screenshot of the MITRE coverage page in Microsoft Defender](https://learn.microsoft.com/en-us/azure/sentinel/media/overview/mitre-coverage-defender.png){MITRE ATT&CK coverage, Defender portal}

![Screenshot of the MITRE coverage page](https://learn.microsoft.com/en-us/azure/sentinel/media/overview/mitre-coverage.png){MITRE ATT&CK coverage, Azure portal}

![Screenshot of an incident investigation that shows an entity and connected entities in an interactive graph](https://learn.microsoft.com/en-us/azure/sentinel/media/overview/map-timeline.png){Incident investigation graph}

![Screenshot of example automated workflow in Azure Logic Apps](https://learn.microsoft.com/en-us/azure/sentinel/media/overview/logic-app.png){SOAR: a Logic Apps playbook triggered by an incident}

![Screenshot of a redirect link from the Azure portal to the Defender portal](https://learn.microsoft.com/en-us/azure/sentinel/media/overview/redirect-no-defender.png){Azure portal redirecting Sentinel users to the Defender portal}

![Solution areas in Microsoft Purview](https://learn.microsoft.com/en-us/purview/media/purview-areas.png){Purview solution areas: data security, data governance, risk and compliance}

![Image of how Security Copilot works](https://learn.microsoft.com/en-us/copilot/security/media/security-copilot-diagram.png){How Security Copilot works: prompts, orchestrator, plugins, responses}

![Four pillars of Microsoft Agent 365](https://learn.microsoft.com/en-us/microsoft-agent-365/media/overview/agent-365-four-pillars.png){Agent 365: management, governance, security, business value}

![Screenshot showing Core security settings permissions for posture policy management in Microsoft Defender unified RBAC](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/security-policy-concept/posture-policy-permissions.png){Defender portal: permissions for managing posture policies}

![Screenshot showing the MDC security baseline policy details in the Microsoft Defender portal](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/security-policy-concept/security-baseline-policy.png){The built-in security baseline posture policy}

![Screenshot of the cloud scope options when creating a posture policy in the Microsoft Defender portal](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/security-policy-concept/create-posture-policy-details-scope.png){Choosing the cloud scope for a posture policy}

![Screenshot of the regulatory compliance framework filter during posture policy rule selection](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/security-policy-concept/create-posture-policy-rules.png){Selecting rules by compliance framework}

![Screenshot showing the Rules management page for posture policies in the Microsoft Defender portal](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/security-policy-concept/posture-policy-rules-management.png){Posture policy rules management}

![Screenshot showing Azure security policies with an option to go to Microsoft Defender for additional policies](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/security-policy-concept/azure-security-policies-go-to-defender.png){Azure portal security policies linking to the Defender portal}
