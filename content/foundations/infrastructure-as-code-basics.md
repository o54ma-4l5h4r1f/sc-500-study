+++
slug = "infrastructure-as-code-basics"
title = "Infrastructure as code: ARM templates, Bicep, Terraform, what-if and pipelines"
summary = "Infrastructure as code describes Azure resources in text files — ARM JSON, Bicep or Terraform — that a pipeline deploys at a chosen scope. Security engineers scan those files for misconfigurations before deployment and use Azure Policy as the guardrail at deployment time."
order = 3
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/overview",
  "https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview",
  "https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/deploy-what-if",
  "https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/deployment-modes",
  "https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/deploy-to-management-group",
  "https://learn.microsoft.com/en-us/azure/developer/terraform/overview",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-devops-introduction",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/iac-vulnerabilities",
  "https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/deploy-github-actions",
  "https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/what-is-azure-pipelines",
]
captured = 2026-11-15
related_objectives = ["D1.3.9", "D1.3.1", "D1.2.6", "D4.1.1", "D1.3.3"]
+++

## In one sentence

**Infrastructure as code (IaC)** means you describe the Azure resources you want — a storage account, a key vault, its firewall rules — in a text file, keep that file in source control, and let a tool deploy it the same way every time.

## Why it exists

Clicking through the portal is fine once. It is not fine for 40 environments. Clicks are not repeatable, not reviewable and leave no record of *why* a setting changed.

Think of a building plan versus giving verbal instructions to a builder. A plan can be checked by an inspector **before** the first brick is laid. It can be reused for the next building. When the inspector finds that the plan has no fire exits, fixing the drawing is cheap; fixing the finished building is not.

That inspector is you. If a template says `allowBlobPublicAccess: true`, every environment built from it is exposed. Catching that in the file — **"shift left"** — is far cheaper than catching it later as a Defender for Cloud recommendation or, worse, as an incident you triage.

## Mental model

```svg
<svg viewBox="0 0 880 300" role="img" aria-label="IaC flow: developer writes Bicep or Terraform in a repository, pipeline scans the template, runs what-if, deploys through Azure Resource Manager where Azure Policy evaluates the request, and resources are created">
  <defs>
    <marker id="fiacarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="currentColor"/>
    </marker>
  </defs>
  <rect class="box" x="10" y="40" width="150" height="80" rx="8"/>
  <text x="85" y="72" text-anchor="middle">Template file</text>
  <text x="85" y="96" text-anchor="middle" class="muted">Bicep / ARM / TF</text>
  <rect class="box" x="185" y="40" width="150" height="80" rx="8"/>
  <text x="260" y="72" text-anchor="middle">Repository</text>
  <text x="260" y="96" text-anchor="middle" class="muted">GitHub / Azure Repos</text>
  <rect class="box-accent" x="360" y="40" width="150" height="80" rx="8"/>
  <text x="435" y="72" text-anchor="middle" class="label-accent">Pipeline scan</text>
  <text x="435" y="96" text-anchor="middle" class="muted">IaC rules, what-if</text>
  <rect class="box-accent" x="535" y="40" width="150" height="80" rx="8"/>
  <text x="610" y="66" text-anchor="middle" class="label-accent">Resource Manager</text>
  <text x="610" y="88" text-anchor="middle">RBAC check</text>
  <text x="610" y="108" text-anchor="middle">Azure Policy</text>
  <rect class="box-ok" x="710" y="40" width="160" height="80" rx="8"/>
  <text x="790" y="72" text-anchor="middle">Resources</text>
  <text x="790" y="96" text-anchor="middle" class="muted">at chosen scope</text>
  <line class="line-accent" x1="160" y1="80" x2="183" y2="80" marker-end="url(#fiacarrow)"/>
  <line class="line-accent" x1="335" y1="80" x2="358" y2="80" marker-end="url(#fiacarrow)"/>
  <line class="line-accent" x1="510" y1="80" x2="533" y2="80" marker-end="url(#fiacarrow)"/>
  <line class="line-accent" x1="685" y1="80" x2="708" y2="80" marker-end="url(#fiacarrow)"/>
  <rect class="box-risk" x="360" y="170" width="325" height="70" rx="8"/>
  <text x="522" y="198" text-anchor="middle" class="label-risk">Two places to stop a bad setting</text>
  <text x="522" y="222" text-anchor="middle">before deploy (scan) and at deploy (Deny)</text>
  <text x="440" y="280" text-anchor="middle" class="muted">Defender for Cloud DevOps security shows scan findings across repos</text>
</svg>
```

## What actually matters

**1. Three languages, one idea.**

| Tool | What it is | Key facts |
|---|---|---|
| **ARM template** | A JSON file read natively by Azure Resource Manager | Sections: `parameters`, `variables`, `resources`, `outputs`. Verbose. |
| **Bicep** | Microsoft's shorter language for the same thing | A Bicep file is ==@transpiled (converted) to ARM JSON== before deployment. Anything ARM can deploy, Bicep can. |
| **Terraform** | HashiCorp's multi-cloud IaC tool | Uses the **AzureRM** provider. Workflow: `terraform plan` (preview) then `terraform apply`. Keeps a **state file** that records what it deployed — the state file can hold secrets, so protect it. |

All three are **declarative**: you describe the end state, not the steps. Deploying the same file twice gives the same result (**idempotent**).

**2. A small Bicep example.** This is the kind of file you will read on the exam — and the kind of setting you will be asked to fix.

```bicep
param location string = resourceGroup().location

resource sa 'Microsoft.Storage/storageAccounts@2023-05-01' = {
  name: 'stsecuredemo001'
  location: location
  sku: { name: 'Standard_LRS' }
  kind: 'StorageV2'
  properties: {
    allowBlobPublicAccess: false     // block anonymous blob access
    minimumTlsVersion: 'TLS1_2'
    supportsHttpsTrafficOnly: true
    publicNetworkAccess: 'Disabled'  // reach it only via private endpoint
  }
}
```

```bash
az deployment group what-if  --resource-group rg-lab --template-file main.bicep
az deployment group create   --resource-group rg-lab --template-file main.bicep
```

**3. Deployment scopes.** A deployment targets one of four scopes from the resource hierarchy: **resource group** (`az deployment group`), **subscription** (`az deployment sub`), **management group** (`az deployment mg`) or **tenant** (`az deployment tenant`). Policy definitions, policy assignments and role assignments are often deployed at management group or subscription scope; storage accounts at resource group scope. In Bicep the file declares it with `targetScope = 'managementGroup'`.

**4. What-if previews before you deploy.** `what-if` asks Resource Manager to predict the changes without making them. Change types: ==@Create, Delete, Ignore, NoChange, NoEffect, Modify, Deploy==. **Delete** only appears in **complete** mode. Deployment modes: **Incremental** (default — leaves resources not in the file alone) and **Complete** (deletes resources in the resource group that the file doesn't list; Microsoft is phasing complete mode out in favour of **deployment stacks**).

**5. Why security engineers scan IaC.** Defender for Cloud's **DevOps security** connects **GitHub, Azure DevOps and GitLab**. The **Microsoft Security DevOps** GitHub action or Azure DevOps extension runs scanners in the pipeline. Setting `categories: 'IaC'` limits it to template scanning. The included IaC scanners are **Template Analyzer** (ARM and Bicep, with PSRule), **Checkov** and **Terrascan** (ARM, Terraform, Kubernetes, Helm, Docker, CloudFormation and more). Findings show in **Defender for Cloud > DevOps security**.

**6. Policy is the guardrail at deployment time.** A template still goes through Resource Manager, so an Azure Policy with the **Deny** effect blocks a non-compliant resource even if the scan was skipped. Scan = early warning. Policy = enforcement.

**7. Pipelines at a glance.** A **pipeline** is an automated script that runs when code changes. **GitHub Actions** workflows live in `.github/workflows/*.yml`. **Azure Pipelines** (Azure DevOps) use `azure-pipelines.yml`. To deploy to Azure, the pipeline signs in as a **service principal** or **workload identity** — ideally with **OpenID Connect federation** (no stored secret), covered on the Entra ID page.

## Where it lives

- **Azure portal > {resource group} > Settings > Deployments** — deployment history, including who deployed what.
- **Azure portal > {resource} > Automation > Export template** — see any resource as ARM JSON.
- **Azure portal > Microsoft Defender for Cloud > DevOps security** — connected repos and IaC findings. Connect repos under **Environment settings > Add environment > GitHub / Azure DevOps / GitLab**.
- CLI: `az deployment group|sub|mg|tenant what-if|create`; `az bicep build` converts Bicep to JSON.

![Diagram that shows the difference between imperative and declarative deployment](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/media/overview/template-processing.png){A template is turned into REST calls to Resource Manager — one request per resource}

![Bicep deployment what-if operation output](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/media/deploy-what-if/resource-manager-deployment-whatif-change-types.png){what-if output: colour-coded change types before anything is deployed}

## Exam traps

- ==!Bicep is not a separate deployment engine.== It compiles to ARM JSON; the same Resource Manager, RBAC and Policy apply.
- ==!Incremental mode does not delete== resources missing from the file. Only complete mode (or a deployment stack) does.
- ==+Scan in the pipeline, enforce with Policy.== A question asking how to *prevent* deployment of non-compliant resources wants Azure Policy **Deny**; one asking how to *find* misconfigurations in templates before deployment wants DevOps security / IaC scanning.
- `what-if` makes **no** changes. It is safe on production.
- A Terraform state file and a pipeline secret are both secrets. Protect them like keys.

## Skip

- **Skip** template syntax depth: functions, loops, conditions, modules, linked and nested templates, template specs. Read a template; don't author complex ones.
- **Skip** the Bicep IntelliSense demo and VS Code extension features.
- **Skip** Terraform backends, workspaces and HCL syntax beyond "provider, plan, apply, state".
- **Skip** Azure Pipelines agent pools, stages and YAML schema. Know only where a workflow file lives and that it signs in with a service principal or workload identity.
- **Skip** the three-tier and nested-template architecture diagrams in the templates overview as design guidance; they illustrate how one file can deploy many tiers, nothing more.

## Screenshots

![Screenshot of the Azure portal showing deployment history](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/media/overview/deployment-history.png){Every template deployment is recorded on the resource group}

![Diagram that shows a three-tier application deployment using a single template](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/media/overview/3-tier-template.png){One template can deploy a whole application tier set}

![Diagram that shows a three-tier application deployment using nested templates](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/media/overview/nested-tiers-template.png){Nested templates split one deployment into parts}

![Diagram that shows a three-tier application deployment with separate resource groups](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/media/overview/tier-templates.png){Separate templates per tier and resource group}

![Screenshot of requesting ARM template support](https://learn.microsoft.com/en-us/azure/includes/media/template-support/request-arm-template-support.png){Microsoft's page for requesting template support for a resource type}

![A screen capture of authoring a Bicep file in real time](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/media/overview/bicep-intellisense.gif){Writing Bicep with editor help}

![A diagram comparing deployment between a Bicep file and infrastructure as code not in a template](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/media/overview/bicep-processing.png){Bicep is converted to an ARM template, then deployed by Resource Manager}

![Bicep deployment what-if operation output deployment mode complete](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/media/deploy-what-if/resource-manager-deployment-whatif-output-confirm.png){what-if in complete mode — note the Delete change type}

![Screenshot of the top of the DevOps security page](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/defender-for-devops-introduction/devops-security-overview-2.png){Defender for Cloud DevOps security: all onboarded repos and their metrics}

![Screenshot of the scan finding metrics sections of the page](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/defender-for-devops-introduction/security-overview.png){Scan findings, including IaC misconfigurations}

![Screenshot of the DevOps environment posture management recommendation card](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/defender-for-devops-introduction/posture-management.png){Posture recommendations for the DevOps environments themselves}

![Screenshot of DevOps advanced security coverage per source code management system onboarded](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/defender-for-devops-introduction/advanced-security.png){Advanced security coverage per source code system}

![Screenshot that shows the DevOps inventory table](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/defender-for-devops-introduction/inventory-grid.png){DevOps inventory table}

![Screenshot that highlights the Edit this file icon for the msdevopssec.yml file](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/tutorial-iac-vulnerabilities/workflow-yaml.png){GitHub: the Microsoft Security DevOps workflow file}

![Screenshot that shows the information to add to the YAML file](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/tutorial-iac-vulnerabilities/add-to-yaml.png){Add categories: 'IaC' to scan templates only}

![Screenshot that shows where to select Commit changes on the GitHub page](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/tutorial-iac-vulnerabilities/commit-change.png){Commit the workflow change}

![Screenshot that shows where the azuredeploy.json file is located](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/tutorial-iac-vulnerabilities/deploy-json.png){A sample ARM template to scan}

![Screenshot that shows you how to create a new file](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/tutorial-iac-vulnerabilities/create-file.png){Add the template to the repo}

![Screenshot that shows that the new file you created is added to your repository](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/tutorial-iac-vulnerabilities/file-added.png){The template committed — the next run scans it}

![Screenshot that shows where to add the IaC categories line in the pipeline configuration YAML file](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/tutorial-iac-vulnerabilities/addition-to-yaml.png){Azure DevOps: the same IaC category on the MicrosoftSecurityDevOps@1 task}

![Screenshot of Azure Pipelines overview](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/media/pipelines-overview.png){Azure Pipelines: build and deploy automation in Azure DevOps}
