+++
slug = "defender-for-cloud-basics"
title = "Defender for Cloud basics"
summary = "Defender for Cloud as an engineer sees it: CSPM (posture) vs CWPP (workload protection plans), free Foundational CSPM vs paid Defender CSPM, the plan list, the Microsoft cloud security benchmark, secure score, recommendations, environment settings and multicloud connectors at a glance."
order = 12
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-cloud-introduction",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/concept-cloud-security-posture-management",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/tutorial-enable-cspm-plan",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/connect-azure-subscription",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/security-policy-concept",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/concept-regulatory-compliance",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/update-regulatory-compliance-packages",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/secure-score-security-controls",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/review-security-recommendations",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/permissions",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/plan-multicloud-security-get-started",
]
captured = 2026-11-15
related_objectives = ["D1.2.6", "D1.2.7", "D1.3.2", "D1.3.3", "D2.1.3", "D2.2.3", "D3.1.8", "D3.1.10", "D3.2.5", "D3.2.6", "D3.2.7", "D3.3.1", "D4.1.1", "D4.1.2", "D4.1.3", "D4.1.4", "D4.1.5", "D4.1.6"]
+++

## In one sentence

**Microsoft Defender for Cloud** is the Azure service that checks your cloud resources for weak settings (posture) and, when you switch on paid plans, watches those resources for attacks (workload protection) — across Azure, AWS, GCP and on-premises machines.

## Why it exists

As an analyst you triaged Defender for Cloud alerts: "suspicious access to a key vault", "malware uploaded to a storage account". Those alerts don't appear by magic. ==Someone switched on the plan that produces them.== That someone is now you.

Think of a building inspector and a security guard. The **inspector** walks round with a checklist: fire doors shut? windows locked? He writes a list of fixes. The **guard** watches the cameras at night and raises the alarm when someone breaks in. Defender for Cloud does both jobs. The inspector is **CSPM**. The guard is the **workload protection plans**.

## Mental model

Microsoft calls Defender for Cloud a **CNAPP** (cloud-native application protection platform) with three parts:

| Part | Job | Everyday anchor | Cost |
|---|---|---|---|
| **CSPM** — cloud security posture management | Find misconfigurations; give recommendations and a secure score | The inspector's checklist | **Foundational CSPM** free; **Defender CSPM** paid |
| **CWPP** — cloud workload protection platform | Detect threats against running workloads; raise security alerts | The night guard | One paid plan per workload type |
| **DevSecOps** | Scan code repos and pipelines (GitHub, Azure DevOps, GitLab) | Checking the blueprints before building | Basic free; advanced in Defender CSPM |

```svg
<svg viewBox="0 0 760 330" role="img" aria-label="Defender for Cloud splits into posture management, which produces recommendations and secure score, and workload protection plans, which produce security alerts; both cover Azure, AWS, GCP and Arc machines">
  <defs>
    <marker id="dfcbarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="line-accent"/>
    </marker>
  </defs>
  <rect class="box" x="20" y="20" width="720" height="60" rx="8"/>
  <text x="380" y="45" text-anchor="middle">Environments: Azure subscriptions, AWS accounts,</text>
  <text x="380" y="67" text-anchor="middle">GCP projects, Arc machines, code repos</text>
  <rect class="box-accent" x="20" y="120" width="340" height="90" rx="8"/>
  <text x="190" y="148" text-anchor="middle" class="label-accent">CSPM (posture)</text>
  <text x="190" y="172" text-anchor="middle">Foundational = free</text>
  <text x="190" y="194" text-anchor="middle">Defender CSPM = paid extras</text>
  <rect class="box-ok" x="400" y="120" width="340" height="90" rx="8"/>
  <text x="570" y="148" text-anchor="middle">CWPP (workload plans)</text>
  <text x="570" y="172" text-anchor="middle">Servers, Storage, Databases,</text>
  <text x="570" y="194" text-anchor="middle">Containers, Key Vault, AI…</text>
  <line class="line-accent" x1="190" y1="80" x2="190" y2="118" marker-end="url(#dfcbarrow)"/>
  <line class="line-accent" x1="570" y1="80" x2="570" y2="118" marker-end="url(#dfcbarrow)"/>
  <rect class="box" x="20" y="250" width="340" height="60" rx="8"/>
  <text x="190" y="275" text-anchor="middle">Recommendations + secure score</text>
  <text x="190" y="297" text-anchor="middle" class="muted">"fix this setting"</text>
  <rect class="box-risk" x="400" y="250" width="340" height="60" rx="8"/>
  <text x="570" y="275" text-anchor="middle" class="label-risk">Security alerts</text>
  <text x="570" y="297" text-anchor="middle" class="muted">"someone is attacking this"</text>
  <line class="line-accent" x1="190" y1="210" x2="190" y2="248" marker-end="url(#dfcbarrow)"/>
  <line class="line-accent" x1="570" y1="210" x2="570" y2="248" marker-end="url(#dfcbarrow)"/>
</svg>
```

![Diagram showing the core functionality of Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/defender-for-cloud-introduction/cloud-security-pillars.png){The three pillars — posture, DevOps security, workload protection}

## What actually matters

**1. Foundational CSPM vs Defender CSPM.** Foundational CSPM is on as soon as Defender for Cloud is enabled on a subscription. It gives asset inventory, the **Microsoft cloud security benchmark**, **secure score**, **security recommendations**, remediation tools, workflow automation, data export and workbooks. **Defender CSPM** adds:

| Only in Defender CSPM (paid) | One-liner |
|---|---|
| Attack path analysis, cloud security explorer | Graph of how an attacker could chain weaknesses |
| Agentless VM vulnerability and **secrets scanning** | Reads disk snapshots, no agent (D1.2.6) |
| Governance rules | Assign owners and due dates to recommendations |
| Custom recommendations (KQL) | Your own checks |
| Data security posture (sensitive data discovery), AI and API posture | Data-aware and AI-aware risk |
| Risk prioritization, internet exposure analysis, EASM | Rank findings by real exposure |

**Regulatory standards beyond MCSB** (PCI DSS, ISO 27001, NIST…; D4.1.2) are ==!not Defender-CSPM-only==. The prerequisite is ==+any paid Defender plan except Defender for Servers Plan 1 or Defender for APIs Plan 1==, plus **Owner** or Policy Contributor (the docs' name; the built-in role is **Resource Policy Contributor**) to add a standard. The Defender CSPM tutorial still lists "regulatory compliance" among the plan's extras; the standards-assignment article is the specific rule, so follow it.

==!Agentless scanning needs the Subscription Owner== to enable Defender CSPM. A lower role can switch the plan on, but the scanner stays off.

**2. The workload protection plans.** Each plan is a separate switch and bill:

| Plan | Protects |
|---|---|
| **Defender for Servers** (Plan 1, Plan 2) | Windows/Linux machines in Azure, AWS, GCP, on-premises (via Arc). Includes Defender for Endpoint. |
| **Defender for Containers** | AKS, EKS, GKE, Arc Kubernetes, registries |
| **Defender for Storage** | Storage accounts: malware, SAS misuse, data leakage |
| **Defender for Databases** | Azure SQL, SQL servers on machines, open-source relational DBs, Cosmos DB |
| **Defender for App Service** | Web apps |
| **Defender for Key Vault** | Unusual key vault access |
| **Defender for Resource Manager** | Suspicious management-plane operations |
| **Defender for APIs** | APIs published through API Management |
| **AI Services** | Threats to generative AI apps (D3.1.8) |

Defender for DNS is folded into Defender for Servers **Plan 2** for new subscriptions.

**3. The security policy is Azure Policy underneath.** A **standard** is a set of controls. The default is the **Microsoft cloud security benchmark (MCSB)**, applied automatically to every subscription where Defender for Cloud is on. In Azure, a standard is an Azure Policy initiative (see `azure-policy-basics`). When a resource fails a control, Defender for Cloud raises a **recommendation**.

**4. Secure score.** One percentage. Higher = less risk. Points are grouped into **security controls**; each control's score grows as more resources in it become healthy. Rules: ==@only built-in MCSB recommendations count==; **preview** recommendations don't count; risk prioritization doesn't change it. There are now two models: the classic **Secure Score** in the Azure portal and the risk-based **Cloud Secure Score** in the Defender portal. They give different numbers.

**5. Recommendations.** Each one has a description, remediation steps, affected resources, severity and risk factors. From a recommendation you can **Fix** (quick fix), **Enforce** (deploy a DeployIfNotExists policy) or **Deny** (create a Deny policy), **Exempt** a resource, assign an **owner and due date**, or trigger a **workflow automation** (a Logic App).

**6. Environment settings is the control room.** It lists management groups, subscriptions, AWS and GCP connectors and DevOps connectors. You turn plans on **per subscription** (or per connector), and some plans can be tuned lower down. Plans **inherit downward** from the subscription. Multicloud connectors (details in `aws-and-gcp-for-azure-people`):

| Cloud | What you connect | How trust is set up |
|---|---|---|
| AWS | One account, or a **management account** (auto-adds member accounts) | CloudFormation (or Terraform) creates an OIDC identity provider and IAM roles |
| GCP | One project, or an **organization** | A `gcloud` script creates a workload identity pool, provider and service accounts |

Foundational CSPM is free on connectors too; Defender plans on connectors bill per resource.

**7. Who can do what** (Azure RBAC, at subscription scope unless noted):

| Action | Security Reader / Reader | Security Admin | Contributor | Owner |
|---|---|---|---|---|
| View alerts and recommendations | ✔ | ✔ | ✔ | ✔ |
| Enable/disable Defender plans | — | ✔ | ✔ | ✔ |
| Assign standards, edit security policy | — | ✔ | — | ✔ |
| Exempt recommendations | — | ✔ | — | ✔ |
| Apply a fix to a resource | — | — | ✔ (also at RG) | ✔ |
| Dismiss alerts | — | ✔ | ✔ | ✔ |

==!Security Admin can change policy but cannot apply a fix to a resource.== Contributor can fix but cannot assign standards.

## Where it lives

**Two portals, mid-transition.** The **Azure portal** (`portal.azure.com > Microsoft Defender for Cloud`) still holds full plan management, standards and environment settings. The **Defender portal** (`security.microsoft.com > Cloud Infrastructure` and `Exposure Management`) now shows recommendations, Cloud Secure Score and new **posture policies** (preview). Azure portal paths:

- `Defender for Cloud > Environment settings > <subscription> > Defender plans` — switch plans; **Settings** column for plan components (agentless scanning, etc.).
- `Defender for Cloud > Environment settings > <subscription> > Security policies` — standards (MCSB, regulatory).
- `Defender for Cloud > Recommendations` and `> Regulatory compliance` and `> Security alerts`.
- `Defender for Cloud > Environment settings > Add environment > Amazon Web Services / Google Cloud Platform`.

```bash
az security pricing create --name StorageAccounts --tier Standard   # turn on Defender for Storage
az security pricing list -o table                                   # see every plan's tier
```

![Searching for Microsoft Defender for Cloud in the Azure portal](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/get-started/defender-for-cloud-search.png){Find Defender for Cloud in the Azure portal}

![Environment settings navigation](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/get-started/environmental-settings.png){Environment settings — the list of subscriptions and connectors}

![Defender plans page with the Enable all button](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/get-started/enable-all-plans.png){Defender plans for one subscription — each plan is its own switch}

## Exam traps

- ==!Free is not everything.== Foundational CSPM gives MCSB, secure score and recommendations. Attack paths, agentless scanning, governance and custom recommendations need **Defender CSPM**. Extra regulatory standards need **any paid plan except Defender for Servers Plan 1 / Defender for APIs Plan 1**.
- Alerts come from **workload plans** (CWPP), not from CSPM. "No alerts for storage malware" → Defender for Storage is off.
- The Defender portal **has no Foundational CSPM**. Environments *managed in Defender* need Defender CSPM to get recommendations. The Azure portal still offers the free tier. Both statements are in the same Microsoft article; know which portal a question means.
- Defender posture policies support **Audit only**. "Block non-compliant resources" still needs an **Azure Policy Deny**.
- Secure score ignores preview recommendations and non-MCSB standards.
- Enabling plans: Security Admin, Contributor or Owner. Full plan capability (agentless scanning, auto-provisioning Arc): **Owner**.

## Skip

- **Skip** the secure score maths (score per resource × healthy resources). Know "more healthy resources = higher score".
- **Skip** the billable-resource tables and pricing. They live on a reference page, not here.
- **Skip** DevOps connector detail (pull-request annotations, code-to-cloud mapping). Not on the blueprint.
- **Skip** Defender Experts for Servers (managed service) and ServiceNow integration.
- **Skip** the Power BI and mobile-app views of secure score.
- **Skip** each plan's internals here — every plan has its own objective page (D2.1.3, D2.2.3, D3.2.5–D3.2.7, D3.3.1, D3.1.8, D1.2.7).

## Screenshots

![CNAPP diagram showing how Defender for Cloud plans protect resources](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/defender-for-cloud-introduction/defender-plans.png){CNAPP — which plan covers which layer}

![Defender for Cloud overview dashboard](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/overview-page/overview.png){Overview dashboard — secure score, alerts, inventory at a glance}

![Defender CSPM Settings option on the plans page](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/tutorial-enable-cspm-plan/cspm-settings.png){Defender CSPM — Settings opens the plan's components}

![Edit configuration for Defender CSPM components](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/tutorial-enable-cspm-plan/cspm-configuration.png){Components such as agentless scanning are switched on here}

![Components of the Microsoft cloud security benchmark](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/concept-regulatory-compliance/microsoft-security-benchmark.png){MCSB — the default standard; controls map to Azure, AWS and GCP checks}

![Sample regulatory compliance page](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/concept-regulatory-compliance/compliance-dashboard.png){Regulatory compliance dashboard — extra standards need a paid plan (not Servers P1 / APIs P1)}

![Recommendations flat list of critical storage account findings](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/review-security-recommendations/review-by-findings.png){Recommendations — flat list by finding}

![Recommendations grouped by resource](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/review-security-recommendations/review-by-resource.png){Grouped by resource, with risk level and owner}

![Group by title toggle on the recommendations page](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/review-security-recommendations/group-by-title.png){Group by title toggle}

![Recommendations page in the Defender portal](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/defender-portal-recommendations.png){The same recommendations in the Defender portal}

![Recommendation side pane in the Defender portal](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/review-security-recommendations/defender-portal-recommendation-side-pane.png){Recommendation side pane}

![Take action tab with Remediate, Assign owner, Exempt and Workflow automation](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/review-security-recommendations/take-action.png){Take action — remediate, assign owner and due date, exempt, or run a workflow automation}

![Graph tab showing attack paths for a recommendation](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/review-security-recommendations/recommendation-graph.png){Graph tab — attack paths through this weakness (Defender CSPM)}

![Graph tab with a node selected](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/review-security-recommendations/select-node.png){Selecting a node in the graph}

![Insights tab for a node](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/review-security-recommendations/insights.png){Insights for a node}

![Azure Resource Graph Explorer results for a recommendation](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/review-security-recommendations/run-query.png){Open query — the same findings in Azure Resource Graph}

![Core security settings permissions in Defender unified RBAC](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/security-policy-concept/posture-policy-permissions.png){Defender portal posture policies use Defender unified RBAC (Core security settings), not Azure RBAC}

![MDC security baseline policy details](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/security-policy-concept/security-baseline-policy.png){MDC Baseline Policy — Microsoft-managed, assigned to all connected environments by default}

![Cloud scope options when creating a posture policy](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/security-policy-concept/create-posture-policy-details-scope.png){Creating a custom posture policy — choose scope}

![Regulatory framework filter during posture policy rule selection](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/security-policy-concept/create-posture-policy-rules.png){Pick rules, filtered by framework}

![Rules management page for posture policies](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/security-policy-concept/posture-policy-rules-management.png){Rules management — where each rule is applied}

![Azure security policies with an option to go to Microsoft Defender](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/security-policy-concept/azure-security-policies-go-to-defender.png){Azure security policies still live in the Azure portal; Defender-portal policies are separate and are not synced}

![Overall secure score on the portal dashboard](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/single-secure-score-via-ui.png){Secure score as a percentage}

![Secure score in the Azure mobile app](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/single-secure-score-via-mobile.png){Mobile app view}

![Security posture page](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/security-posture-page.png){Security posture — score per environment}

![Bottom half of the security posture page](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/bottom-half.png){Per-subscription, per-account, per-project breakdown}

![Security controls that affect a secure score](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/security-controls.png){Security controls — each has a max score}

![Recommendations for the Remediate vulnerabilities control](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/remediate-vulnerabilities-control.png){One control's recommendations and potential score increase}

![Equation for a security control score](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/secure-score-equation-single-control.png){Control score equation (skip the maths)}

![Tooltips showing values for a control's current score](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/security-control-scoring-tooltips.png){Tooltip values behind a control score}

![Equation for a subscription's secure score](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/secure-score-equation-single-sub.png){Subscription score equation}

![Single-subscription secure score with all controls](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/secure-score-example-single-sub.png){Example — one subscription}

![List of controls and potential score increase](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/secure-score-example-single-sub-recs.png){Potential increase per control}

![Equation for multiple subscriptions](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/secure-score-equation-multiple-subs.png){Multi-subscription score is weighted, not averaged}

![Secure score for multiple subscriptions](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/secure-score-example-multiple-subs.png){Example — several subscriptions}

![Secure score on the main dashboard](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/score-on-main-dashboard.png){Secure score tile on the overview}

![Secure score for subscriptions on the dedicated page](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/score-on-dedicated-dashboard.png){Secure score page by subscription}

![Secure score for management groups](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/secure-score-management-groups.png){Secure score by management group}

![Secure score on the recommendations page](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/score-on-recommendations-page.png){Score shown above the recommendations list}

![Retrieving a secure score via the API](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/single-secure-score-via-api.png){Secure score via the REST API}

![Launching Azure Resource Graph Explorer from a recommendation](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/multi-factor-authentication-enforcement/opening-resource-graph-explorer.png){Query scores and findings in Azure Resource Graph}

![Secure score over time workbook](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/custom-dashboards-azure-workbooks/secure-score-over-time-snip.png){Secure Score Over Time workbook}

![Secure Score Over Time Power BI dashboard](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/power-bi-secure-score-dashboard.png){Optional Power BI dashboard (skip)}

![Cloud secure score calculation flow](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/cloud-secure-score-flow.png){Defender portal Cloud Secure Score — risk-based, uses asset criticality and risk factors}

![Cloud secure score formula](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-security-controls/cloud-secure-score-formula.png){Cloud Secure Score formula — a different model from the Azure portal score}

![Secure score landing page in the Defender portal](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-defender-portal/secure-score-landing-page.png){Exposure Management > Secure score}

![Cloud initiative homepage](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/secure-score-defender-portal/cloud-initiative-homepage.png){Exposure Management > Initiatives > Cloud Security}

![Multicloud protection with Defender for Cloud](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/planning-multicloud-security/get-started.png){Multicloud — one Defender for Cloud across Azure, AWS and GCP}

![The regulatory compliance page that shows you where to select the manage compliance policy button.](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/update-regulatory-compliance-packages/manage-compliance.png)

![Regulatory compliance dashboard options.](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/update-regulatory-compliance-packages/turn-standard-on.png)
