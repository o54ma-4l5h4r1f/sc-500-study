+++
slug = "aws-and-gcp-for-azure-people"
title = "AWS and GCP for Azure people"
summary = "Just enough Amazon Web Services and Google Cloud to connect them to Defender for Cloud: AWS accounts, Organizations, IAM roles, EC2 and S3; GCP organization, folders, projects, service accounts and Compute Engine; and how the AWS and GCP connectors map onto each, with federated trust instead of stored keys."
order = 19
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/architecture/aws-professional/accounts",
  "https://learn.microsoft.com/en-us/azure/architecture/aws-professional/security-identity",
  "https://learn.microsoft.com/en-us/azure/architecture/aws-professional/storage",
  "https://learn.microsoft.com/en-us/azure/architecture/gcp-professional/",
  "https://learn.microsoft.com/en-us/azure/architecture/gcp-professional/services",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/quickstart-onboard-aws",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/concept-authentication-architecture-aws",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/quickstart-onboard-gcp",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/authentication-architecture-google-cloud",
  "https://learn.microsoft.com/en-us/azure/defender-for-cloud/permissions",
]
captured = 2026-11-15
related_objectives = ["D4.1.4", "D4.1.1", "D4.1.3", "D3.2.4", "D3.2.5", "D3.3.1"]
+++

## In one sentence

AWS and Google Cloud organise things differently from Azure — an **AWS account** or a **GCP project** plays roughly the role of an Azure subscription — and Defender for Cloud connects to them through a **connector** that sets up a trust relationship so it can read their resources without you ever handing over a password or access key.

## Why it exists

Few companies run only Azure. The exam expects you to connect AWS and GCP to Defender for Cloud (D4.1.4) and protect EC2 instances or GKE clusters with the same plans you use in Azure. You can't follow the connector wizard if words like "management account", "IAM role" or "workload identity pool" mean nothing.

Think of **three office buildings owned by one company**, each built by a different architect. The rooms are similar — offices, server rooms, filing cabinets — but the floor plans and the names on the doors differ. You need a translation card before you walk in. This page is that card.

## Mental model

| Concept | **Azure** | **AWS** | **Google Cloud (GCP)** |
|---|---|---|---|
| Identity directory | Microsoft Entra **tenant** | IAM (+ IAM Identity Center) | The Google Cloud *account* |
| Top of the hierarchy | Tenant root management group | **Organization** root (+ **management account**) | **Organization** |
| Grouping layer | Management groups | **Organizational units (OUs)** | **Folders** |
| Billing & quota boundary | **Subscription** | **Account** | **Project** |
| Resource container | Resource group | (no direct match — tags) | Project (functionally closer to a resource group) |
| Guardrails | Azure Policy + RBAC | **Service control policies (SCPs)** | — |
| Workload identity | **Managed identity** / service principal | **IAM role** (assumed, temporary credentials) | **Service account** |
| Virtual machine | Azure VM | **EC2** instance | **Compute Engine** instance |
| Object storage | Storage account / blob container | **S3** bucket | Cloud Storage bucket |
| Time-limited link | SAS | Presigned URL | — |
| Activity log | Activity log | **CloudTrail** | Cloud Logging (audit logs) |
| Managed Kubernetes | AKS | EKS | GKE |

```svg
<svg viewBox="0 0 780 330" role="img" aria-label="Defender for Cloud in an Azure subscription holds a security connector for an AWS account or management account and one for a GCP project or organization. Trust is federated: Entra-issued tokens are exchanged for AWS STS credentials through an OIDC identity provider and IAM roles, or for Google STS tokens through a workload identity pool and service account">
  <defs>
    <marker id="awsgcpbarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="line-accent"/>
    </marker>
  </defs>
  <rect class="box-accent" x="250" y="20" width="280" height="110" rx="8"/>
  <text x="390" y="46" text-anchor="middle" class="label-accent">Defender for Cloud</text>
  <text x="390" y="70" text-anchor="middle">Azure subscription + RG</text>
  <text x="390" y="92" text-anchor="middle" class="muted">holds a security connector</text>
  <text x="390" y="114" text-anchor="middle" class="muted">per AWS / GCP scope</text>
  <rect class="box" x="20" y="180" width="330" height="130" rx="8"/>
  <text x="185" y="206" text-anchor="middle">AWS account / management acct</text>
  <text x="185" y="230" text-anchor="middle" class="muted">CloudFormation or Terraform</text>
  <text x="185" y="252" text-anchor="middle" class="muted">creates OIDC identity provider</text>
  <text x="185" y="274" text-anchor="middle" class="muted">+ IAM roles (CspmMonitorAws…)</text>
  <text x="185" y="296" text-anchor="middle" class="muted">tokens → AWS STS credentials</text>
  <rect class="box" x="430" y="180" width="330" height="130" rx="8"/>
  <text x="595" y="206" text-anchor="middle">GCP project / organization</text>
  <text x="595" y="230" text-anchor="middle" class="muted">gcloud script in Cloud Shell</text>
  <text x="595" y="252" text-anchor="middle" class="muted">creates workload identity pool</text>
  <text x="595" y="274" text-anchor="middle" class="muted">+ provider + service accounts</text>
  <text x="595" y="296" text-anchor="middle" class="muted">tokens → Google STS, impersonate</text>
  <line class="line-accent" x1="320" y1="130" x2="200" y2="178" marker-end="url(#awsgcpbarrow)"/>
  <line class="line-accent" x1="460" y1="130" x2="580" y2="178" marker-end="url(#awsgcpbarrow)"/>
</svg>
```

## What actually matters

**1. AWS in five words: accounts, Organizations, IAM, EC2, S3.**

- An **AWS account** is the basic boundary for resources, permissions and billing — the closest thing to an Azure subscription.
- **AWS Organizations** groups many accounts under an **organization root**, with one **management account** (the payer and admin account) and **OUs** in between. Azure has no equivalent of a dedicated management account; administration sits at the tenant.
- **IAM** holds users, groups, policies and **roles**. An IAM role is an identity with no password that a trusted principal **assumes** to get **temporary credentials** from **AWS STS** (Security Token Service). Its **trust policy** says who may assume it. For Azure people: a role ≈ a managed identity plus its role assignment.
- **EC2** is AWS's VM service. The **SSM Agent** (Systems Manager) on EC2 is how AWS runs commands on instances — Defender for Cloud relies on it to install Arc.
- **S3** stores objects in **buckets**, controlled by IAM and **bucket policies**.

**2. GCP in five words: organization, folders, projects, service accounts, Compute Engine.**

- The **organization** is the root. **Folders** group projects (like management groups).
- A **project** holds resources and is the billing and quota unit — similar to a subscription in billing, but closer to a resource group in how you use it.
- A **service account** is a non-human identity that workloads use (≈ managed identity / service principal). Others can **impersonate** it if granted permission.
- **Compute Engine** runs VMs; **Cloud Storage** holds buckets; **GKE** runs Kubernetes.

**3. The AWS connector.** Created in Defender for Cloud → **Environment settings → Add environment → Amazon Web Services**. Choices:

| Setting | Options |
|---|---|
| Onboard | **Single account**, or **Management account** — auto-creates connectors for existing and new member accounts. ==!Delegated administrator accounts aren't supported.== |
| Regions | All by default |
| Scan interval | Every 4, 6, 12 or 24 hours |
| Access | **Default access** (current + future features) or **Least privilege access** |
| Deployment | **AWS CloudFormation** (a StackSet for management accounts) or **Terraform** |

You need **Contributor** on the Azure subscription that will hold the connector. The template creates an **OpenID Connect (OIDC) identity provider** bound to a Microsoft-managed Entra application, plus **IAM roles** per plan (for example `CspmMonitorAws` for posture, `DefenderForCloud-AgentlessScanner` for snapshot scanning). Defender for Cloud gets an Entra token, AWS validates it and issues **short-lived STS credentials**. ==+No long-lived access keys are stored.== An AWS account already connected to Microsoft Sentinel needs extra steps before it can be connected here.

**4. The GCP connector.** **Add environment → Google Cloud Platform**. Choose a **single project** (project number and ID) or an **organization** (organization ID, with optional excluded projects and folders). Same scan intervals and access types. Defender for Cloud generates a **`gcloud` script** you run in GCP (Cloud Shell). It creates a **workload identity pool**, a **workload identity provider** per plan, **service accounts** and project-level policy bindings. Entra tokens are exchanged with **Google STS**, and Defender for Cloud **impersonates** the service account. Again, no stored keys.

**5. Servers in AWS and GCP need Arc.** Defender for Servers (and Defender for SQL on machines) on EC2 and GCP VMs works through **Azure Arc** (see `azure-arc-and-hybrid-machines`). The connector can **auto-provision** Arc — this needs the **Owner** role on the Azure subscription, and on AWS the **SSM Agent** with the `AmazonSSMManagedInstanceCore` policy. Arc machines for instances that no longer exist, or that are Disconnected or Expired, are removed after **seven days**. Posture (CSPM) needs no agent; it reads cloud APIs.

**6. What you get, by plan.** Foundational CSPM (free) gives AWS and GCP recommendations, secure score and inventory. Defender CSPM adds attack paths, agentless scanning and CIEM (permissions analysis — needs **Security Admin** plus `Application.ReadWrite.All` in the tenant). Workload plans on connectors include Defender for Servers, Containers (EKS/GKE), Databases (RDS / Cloud SQL) — see `defender-for-cloud-basics`.

## Where it lives

**Azure portal > Microsoft Defender for Cloud > Environment settings > Add environment > Amazon Web Services / Google Cloud Platform.** After creation, the **Connectivity status** column on Environment settings shows connector health; select it for issues and remediation steps. The connector itself is an Azure resource in the subscription and resource group you picked. In the Defender portal, cloud plan management currently supports **Azure only** — AWS and GCP connectors are still managed in the Azure portal.

![Connecting an AWS account from Environment settings](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/quickstart-onboard-aws/add-aws-account-environment-settings.png){Environment settings > Add environment > Amazon Web Services}

![Defender for Cloud authenticating to AWS](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/concept-authentication-architecture-aws/architecture-authentication-across-clouds.png){AWS trust — an Entra token is exchanged for temporary AWS STS credentials}

![GCP connector authentication process](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/concept-authentication-architecture-gcp/authentication-process.png){GCP trust — workload identity federation and service account impersonation}

## Exam traps

- AWS account ≈ subscription; AWS Organizations/OUs ≈ management groups; GCP project ≈ subscription (billing); GCP folders ≈ management groups.
- "Onboard every current and future AWS account" → connector on the ==+management account==, not one connector per account.
- Connector auth uses **federation** (OIDC → STS, workload identity federation). ==!No access keys or service-account key files.==
- EC2/GCP VM threat protection → Defender for Servers **+ Arc auto-provisioning** (Owner role, SSM Agent on AWS).
- Creating the connector needs **Contributor** on the Azure subscription; CIEM needs **Security Admin** + `Application.ReadWrite.All`.
- Delegated administrator AWS accounts can't be the onboarding account.

## Skip

- **Skip** AWS and GCP service catalogues beyond the table above. The exam won't ask about Lambda, BigQuery or Route 53.
- **Skip** the per-plan IAM permission lists and GCP API names the script enables.
- **Skip** CloudFormation template update procedures and the error-resolution table.
- **Skip** CloudTrail and GCP Cloud Logging ingestion (preview) — know they exist for CIEM context.
- **Skip** AWS-to-Azure migration guidance and region/zone comparisons.

## Screenshots

![Typical AWS account organizational structure](https://learn.microsoft.com/en-us/azure/architecture/aws-professional/images/aws-accounts.jpg){AWS — organization root, optional management account, OUs, accounts}

![Typical Azure account management structure](https://learn.microsoft.com/en-us/azure/architecture/aws-professional/images/azure-accounts.jpg){Azure — tenant, management groups, subscriptions, resource groups, resources}

![Azure management group, subscription and resource group hierarchy](https://learn.microsoft.com/en-us/azure/architecture/gcp-professional/images/subscription-hierarchy.png){The Azure hierarchy that GCP folders and projects map onto}

![Zone-redundant VM deployment across availability zones](https://learn.microsoft.com/en-us/azure/architecture/gcp-professional/images/availability-zones.png){Availability zones — same idea in all three clouds (skip)}

![Geography, region pair, regions and datacenters](https://learn.microsoft.com/en-us/azure/architecture/gcp-professional/images/region-pairs.png){Region pairs (skip)}

![AWS accounts in the Defender for Cloud overview](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/quickstart-onboard-aws/aws-account-in-overview.png){Connected AWS accounts appear alongside Azure subscriptions}

![Account details for an AWS connector](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/quickstart-onboard-aws/add-aws-account-details.png){Account details — single vs management account, regions, scan interval}

![Plan selection for an AWS account](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/quickstart-onboard-aws/add-aws-account-plans-selection.png){Select plans for the connector}

![Defender CSPM plan features for AWS](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/quickstart-onboard-aws/posture-plan-features.png){Defender CSPM components on AWS}

![Defender for Servers plan settings for AWS](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/quickstart-onboard-aws/enable-servers-plan.png){Defender for Servers settings — includes Arc auto-provisioning}

![Defender for databases settings for AWS](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/quickstart-onboard-aws/databases-settings.png){Defender for Databases settings}

![Defender for Containers plan selection for AWS](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/quickstart-onboard-aws/containers-features.png){Defender for Containers settings for EKS}

![Deployment method configuration for AWS](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/quickstart-onboard-aws/add-aws-account-configure-access.png){Configure access — default vs least privilege, CloudFormation vs Terraform}

![Connectivity status for a connected AWS account](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/quickstart-onboard-aws/environment-details-connector-health.png){Connector health — issues and remediation steps}

![CloudFormation identity provider entry created during onboarding](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/concept-authentication-architecture-aws/configure-access-roles.png){The OIDC identity provider the template creates in AWS IAM}

![GCP connector option location](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/quickstart-onboard-gcp/connector.png){Add environment > Google Cloud Platform}

![Toggles for all plans on a GCP connector](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/quickstart-onboard-gcp/select-plans.png){Select plans for the GCP connector}

![Deployment options and instructions for GCP access](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/quickstart-onboard-gcp/add-gcp-project-configure-access.png){Configure access — the generated gcloud script}

![Review and generate screen for GCP](https://learn.microsoft.com/en-us/azure/defender-for-cloud/media/quickstart-onboard-gcp/review-and-generate.png){Review and create the GCP connector}
