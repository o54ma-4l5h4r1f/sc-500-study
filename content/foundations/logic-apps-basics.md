+++
slug = "logic-apps-basics"
title = "Azure Logic Apps basics (Consumption vs Standard, triggers, connectors, identity)"
summary = "What a logic app, workflow, trigger, action and connector are; how Consumption differs from Standard (hosting, networking, identities); how a workflow signs in with a managed identity; and which parts of a workflow a security engineer has to lock down."
order = 22
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-overview",
  "https://learn.microsoft.com/en-us/azure/logic-apps/single-tenant-overview-compare",
  "https://learn.microsoft.com/en-us/azure/logic-apps/logic-apps-limits-and-config",
  "https://learn.microsoft.com/en-us/azure/logic-apps/authenticate-with-managed-identity",
  "https://learn.microsoft.com/en-us/azure/logic-apps/set-up-security-permissions",
]
captured = 2026-11-15
related_objectives = ["D3.3.6", "D4.2.8", "D3.3.9"]
+++

## In one sentence

**Azure Logic Apps** runs "when this happens, do these steps" recipes called **workflows**, built in a visual designer from ready-made plug-ins called **connectors** — and from SOC work you already know one kind: every Microsoft Sentinel **playbook** is a Logic Apps workflow.

## Why it exists

Businesses glue systems together all day. "When an invoice email arrives, save the PDF to SharePoint and post in Teams." "When a Sentinel incident is created, open a ServiceNow ticket." Writing code for each job is slow. Logic Apps gives you a designer and **1,400+ prebuilt connectors**, so the glue takes minutes.

As an analyst you *used* playbooks. As an engineer you now *own* the thing they run on. That changes the questions. Who can call this workflow's URL? Which identity does it sign in as, and what can that identity touch? Can someone read secrets in its run history? Can it reach private resources? Those are the D3.3.6 security controls.

## Mental model

A workflow is a **recipe card**:

- **Trigger** — the first line, "Start when…". Every workflow has ==exactly one== trigger. Examples: an HTTP request arrives (**Request** trigger), a timer fires (**Recurrence**), a file lands in storage, a Sentinel incident is created.
- **Actions** — every step after the trigger: call an API, send an email, write to a database. Actions run in sequence, in parallel, or inside conditions and loops.
- **Connectors** — the plug-in for one product (Outlook, Key Vault, Microsoft Sentinel). **Built-in** connectors run inside the Logic Apps runtime. **Managed** connectors are Microsoft-hosted proxies that run in global, multitenant Azure.
- **Connection** — the saved sign-in a managed connector uses, stored as a separate Azure resource called an **API connection**.

```svg
<svg viewBox="0 0 900 330" role="img" aria-label="Logic Apps security surface: inbound callers hit the request trigger, the workflow runs actions through connectors signed in with a managed identity, and run history stores inputs and outputs">
  <defs>
    <marker id="labarrow3" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="line-accent"/>
    </marker>
  </defs>
  <rect class="box-risk" x="15" y="40" width="190" height="110" rx="8"/>
  <text x="110" y="70" text-anchor="middle" class="label-risk">Who can call it?</text>
  <text x="110" y="95" text-anchor="middle">SAS key in the URL</text>
  <text x="110" y="117" text-anchor="middle">Entra OAuth policy</text>
  <text x="110" y="139" text-anchor="middle" class="muted">IP restrictions</text>
  <rect class="box-accent" x="245" y="20" width="260" height="200" rx="10"/>
  <text x="375" y="46" text-anchor="middle" class="label-accent">Workflow</text>
  <rect class="box" x="265" y="60" width="220" height="40" rx="6"/>
  <text x="375" y="85" text-anchor="middle">1 trigger (Request)</text>
  <rect class="box" x="265" y="110" width="220" height="40" rx="6"/>
  <text x="375" y="135" text-anchor="middle">action: get secret</text>
  <rect class="box" x="265" y="160" width="220" height="40" rx="6"/>
  <text x="375" y="185" text-anchor="middle">action: call API</text>
  <rect class="box-ok" x="555" y="40" width="330" height="80" rx="8"/>
  <text x="720" y="68" text-anchor="middle" class="label-accent">What does it sign in as?</text>
  <text x="720" y="92" text-anchor="middle">managed identity + least-privilege role</text>
  <text x="720" y="112" text-anchor="middle" class="muted">Key Vault, Storage, Sentinel, ARM</text>
  <rect class="box-risk" x="555" y="150" width="330" height="80" rx="8"/>
  <text x="720" y="178" text-anchor="middle" class="label-risk">Who can read run history?</text>
  <text x="720" y="202" text-anchor="middle">Secure inputs / Secure outputs</text>
  <text x="720" y="222" text-anchor="middle" class="muted">IP range for contents</text>
  <rect class="box" x="245" y="250" width="260" height="65" rx="8"/>
  <text x="375" y="277" text-anchor="middle">Standard only</text>
  <text x="375" y="299" text-anchor="middle" class="muted">VNet integration, private endpoint</text>
  <line class="line-accent" x1="205" y1="85" x2="263" y2="80" marker-end="url(#labarrow3)"/>
  <line class="line-accent" x1="485" y1="130" x2="553" y2="80" marker-end="url(#labarrow3)"/>
  <line class="line" x1="485" y1="180" x2="553" y2="190" marker-end="url(#labarrow3)"/>
  <line class="divider" x1="375" y1="220" x2="375" y2="250"/>
</svg>
```

![Screenshot that shows the workflow designer with an example order review workflow that uses condition and switch actions.](https://learn.microsoft.com/en-us/azure/logic-apps/media/logic-apps-overview/example-enterprise-workflow.png){A workflow in the designer: one trigger, then actions, a condition and a switch}

## What actually matters

**1. Logic app resource vs workflow.** The **logic app** is the Azure resource. The **workflow** is the recipe inside it, stored as a JSON definition (so it deploys as an ARM or Bicep template).

**2. Consumption vs Standard.** You pick the hosting model when you create the logic app. Most security differences come from this choice.

| | Consumption | Standard |
|---|---|---|
| Runs in | **multitenant** Azure Logic Apps (shared) | **single-tenant** Azure Logic Apps, ASEv3, or hybrid |
| Workflows per resource | ==@one== | many (stateful and stateless) |
| Built on | Logic Apps platform | Azure Functions / App Service runtime |
| Billing | pay per execution | hosting plan (Workflow Service Plan) |
| Private networking | ==!none== | ==+VNet integration (outbound) and private endpoints (inbound)== |
| Inbound IP filter | **Workflow settings > Access control configuration** | **Networking > Access restrictions** (inherited from App Service) |
| Managed identities | see contradiction below | system-assigned ==@on by default== + any number of user-assigned |

A **stateful** workflow saves each step's inputs and outputs to storage, so it has run history. A **stateless** one keeps them in memory only — faster, but no history.

> **Two Microsoft pages disagree on Consumption identities.** The *Limits and configuration* reference says a Consumption logic app can have "either the system-assigned identity *or* only one user-assigned identity". The *Authenticate with managed identity* article says a Consumption resource "can enable and have both the system-assigned identity and multiple user-assigned identities defined" — but a workflow uses only one identity at a time. Both pages agree on the safe exam answer: ==+Standard supports system-assigned plus multiple user-assigned; Consumption does not enable the system identity automatically.==

**3. Managed identity is how a workflow should sign in.** A **managed identity** is an Entra identity Azure creates for the logic app, with no password to store or rotate. You enable it on the logic app, grant it an Azure role on the target (for example **Key Vault Secrets User** on a vault), then choose **Managed identity** as the authentication type on the action or connection. Only connectors that support OAuth with Microsoft Entra can use it (HTTP, Azure Resource Manager, Key Vault, Blob Storage, Microsoft Sentinel and others). The risk: anyone who can *edit* the workflow can make it do anything its identity can do. Editing rights equal the identity's rights.

**4. The request trigger is a public URL.** A **Request** trigger creates an HTTPS endpoint. By default its URL carries a **shared access signature (SAS)** key — anyone with the URL can start the workflow. Ways to lock it down: enable **OAuth with Microsoft Entra ID** (an authorization policy that checks token claims such as `iss` and `aud`), disable SAS, restrict inbound IP ranges, **regenerate access keys** if a URL leaks, or front the workflow with **API Management**.

**5. Run history can leak secrets.** Run history shows every step's inputs and outputs. If a step fetched a secret from Key Vault, the secret appears there. Turn on **Secure Inputs** / **Secure Outputs** on that step to hide them, and restrict who can read run-history content by IP range. Use `securestring` parameters for values passed in at deployment.

**6. Roles.** **Logic App Contributor** can edit and manage Consumption logic apps; **Logic App Operator** can read, enable and disable them but not edit. Standard has **Logic Apps Standard Operator / Developer / Contributor**. Edit rights matter because of point 3.

## Where it lives

All in the **Azure portal** (Sentinel playbooks are created from the Defender portal or Sentinel, but they open here).

- **Create:** `Azure portal > Logic apps > Add` → choose **Consumption** or **Standard**.
- **Identity:** `(logic app) > Settings > Identity` → System assigned / User assigned.
- **Consumption inbound IP filter and run-history IP filter:** `(logic app) > Settings > Workflow settings > Access control configuration`.
- **Standard networking:** `(logic app) > Settings > Networking` → inbound (access restrictions, private endpoints) and outbound (VNet integration).
- **Access keys (SAS):** `(logic app) > Settings > Access keys` → regenerate.
- **Secure inputs/outputs:** designer → select the step → **Settings > Security**.
- **API connections:** search *API connections* in the portal; they sit in the logic app's resource group.

```bash
az logic workflow identity assign -g rg-lab -n la-lab --system-assigned   # Consumption
az role assignment create --assignee <principal-id> \
  --role "Key Vault Secrets User" --scope <key-vault-resource-id>
```

## Exam traps

- ==!"Consumption logic apps can use private endpoints."== They cannot. ==+Private networking needs Standard.==
- ==!Anyone with the Request trigger URL can run the workflow== unless you add OAuth, disable SAS or restrict IPs. The SAS key *is* the credential.
- ==!Secure Inputs/Outputs are off by default.== A secret fetched in a step is visible in run history until you turn them on.
- A managed identity has no rights until you assign it a role on the target resource.
- A Sentinel playbook is a Logic Apps workflow — not an Azure Automation runbook, not a Function.

## Skip

- **Skip:** integration accounts and B2B/EDI messaging (AS2, X12, EDIFACT). Business plumbing, not security.
- **Skip:** inline code (JavaScript, C#, PowerShell), custom connectors and AI agent workflows. Know they exist.
- **Skip:** ASEv3 and hybrid hosting sizing, Workflow Service Plan pricing tiers, high-throughput mode.
- **Skip:** the ARM-template syntax for `accessControl` and `openAuthenticationPolicies`. Know what the controls do, not the JSON.
- **Skip:** the per-connector managed-identity support tables — know only that the connector must support Entra OAuth.

## Screenshots

![Screenshot that shows logic app menu in Azure portal with "Workflow settings" selected and "High throughput" set to "On".](https://learn.microsoft.com/en-us/azure/logic-apps/media/logic-apps-limits-and-config/run-high-throughput-mode.png){Consumption: Workflow settings page (high-throughput mode — skip-list item). The same page holds Access control configuration.}

### Securing triggers and run history

![Request trigger HTTP URL](https://learn.microsoft.com/en-us/azure/logic-apps/media/set-up-security-permissions/request-trigger-url.png){The Request trigger URL includes the SAS signature — whoever holds it can start the workflow}

![Authorization page](https://learn.microsoft.com/en-us/azure/logic-apps/media/set-up-security-permissions/add-azure-active-directory-authorization-policies.png){Consumption: Settings > Authorization > Add policy (OAuth with Microsoft Entra ID)}

![Authorization policy details](https://learn.microsoft.com/en-us/azure/logic-apps/media/set-up-security-permissions/set-up-authorization-policy.png){Policy claims: Issuer (iss) and Audience (aud) at minimum}

![Proof-of-possession policy](https://learn.microsoft.com/en-us/azure/logic-apps/media/set-up-security-permissions/pop-policy-example.png){Policy type can be AAD or AADPOP (proof-of-possession token)}

![Standard workflow designer and selected API](https://learn.microsoft.com/en-us/azure/logic-apps/media/set-up-security-permissions/select-api-standard.png){Fronting a workflow with API Management is another way to control inbound calls}

![Trigger or action settings](https://learn.microsoft.com/en-us/azure/logic-apps/media/set-up-security-permissions/open-action-trigger-settings-standard.png){Select the step > Settings > Security}

![Secure Inputs and Secure Outputs](https://learn.microsoft.com/en-us/azure/logic-apps/media/set-up-security-permissions/turn-on-secure-inputs-outputs-standard.png){Turn on Secure Inputs and/or Secure Outputs}

![Secured token with lock icon](https://learn.microsoft.com/en-us/azure/logic-apps/media/set-up-security-permissions/select-secured-token-standard.png){Downstream tokens from a secured output show a lock icon}

![Run history with hidden inputs and outputs](https://learn.microsoft.com/en-us/azure/logic-apps/media/set-up-security-permissions/hidden-data-run-history-standard.png){Run history now hides the secured values}

![Secured outputs flow](https://learn.microsoft.com/en-us/azure/logic-apps/media/set-up-security-permissions/secure-outputs-as-inputs-flow.png){Secured outputs stay hidden when used as inputs by later actions}

![Secured outputs special cases](https://learn.microsoft.com/en-us/azure/logic-apps/media/set-up-security-permissions/secure-outputs-as-inputs-flow-special.png){Special cases: Compose, Parse JSON and Response actions}

![Secured inputs flow](https://learn.microsoft.com/en-us/azure/logic-apps/media/set-up-security-permissions/secure-inputs-impact-on-downstream.png){How secured inputs affect downstream visibility}

![Secured inputs special cases](https://learn.microsoft.com/en-us/azure/logic-apps/media/set-up-security-permissions/secure-inputs-flow-special.png){Secured inputs and the Compose, Parse JSON and Response actions}

### Managed identity walkthrough

![Screenshot that shows the Azure portal and Consumption logic app Identity page with the system-assigned identity enabled.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/enable-system-assigned-identity-consumption.png){Consumption: Identity > System assigned > On}

![Screenshot that shows the Consumption logic app Identity page with the system-assigned identity object ID.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/object-id-system-assigned-identity.png){The identity's object (principal) ID — what you grant roles to}

![Screenshot that shows the Azure portal and the Standard logic app Identity page with the system-assigned identity enabled.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/enable-system-assigned-identity-standard.png){Standard: system-assigned identity already on by default}

![Screenshot that shows the Azure portal search results with Managed Identities selected.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/find-select-managed-identities.png){User-assigned identities are their own resource type: Managed Identities}

![Screenshot that shows the Create User Assigned Managed Identity page with identity details.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/create-user-assigned-identity.png){Create a user-assigned identity}

![Screenshot that shows the Consumption logic app Identity page with Add selected for a user-assigned identity.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/add-user-assigned-identity-logic-app-consumption.png){Consumption: User assigned > Add}

![Screenshot that shows a Consumption logic app with a user-assigned managed identity selected.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/select-user-assigned-identity.png){Select the user-assigned identity}

![Screenshot that shows a Consumption logic app associated with a user-assigned managed identity.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/added-user-assigned-identity-consumption.png){Consumption logic app with its user-assigned identity}

![Screenshot that shows the Standard logic app Identity page with Add selected for a user-assigned identity.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/add-user-assigned-identity-logic-app-standard.png){Standard: add a user-assigned identity}

![Screenshot that shows a Standard logic app associated with a user-assigned managed identity.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/added-user-assigned-identity-standard.png){Standard logic app with system-assigned and user-assigned identities}

![Screenshot that shows an Azure key vault with the Create an access policy pane open.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/create-access-policy.png){Granting the identity access to Key Vault (here with a legacy access policy; Azure RBAC is the recommended model)}

![Screenshot that shows the access policy Permissions tab with List permission selected.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/select-access-policy-permissions.png){Least privilege: only the permissions the workflow needs}

![Screenshot that shows a Consumption workflow and an HTTP action with Authentication selected under Advanced parameters.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/built-in-authentication.png){HTTP action: Advanced parameters > Authentication}

![Screenshot that shows the Authentication property and Authentication type list in Advanced parameters.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/authentication-parameter.png){Authentication type list}

![Screenshot that shows a workflow built-in action, open Authentication type list, and Managed identity selected as the authentication type.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/built-in-managed-identity.png){Choose Managed identity}

![Screenshot that shows the Authentication section with the system-assigned managed identity selected, and Audience parameter.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/select-specific-managed-identity.png){Audience = the resource the token is for (default Azure Resource Manager)}

![Screenshot that shows a Consumption workflow and Azure Resource Manager action with Managed identity selected.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/select-managed-identity-resource-manager.png){Managed connector (Azure Resource Manager): connection authenticates with managed identity}

![Screenshot that shows a Consumption workflow and Azure Blob Storage action connection with Logic Apps Managed Identity selected.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/multi-identity.png){Multi-authentication connector: pick Logic Apps Managed Identity}

![Screenshot that shows the Authentication section with managed identities to select.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/built-in-select-identity-standard.png){Standard: choose among enabled identities}

![Screenshot that shows a Standard workflow, action Parameters pane, and Managed identity list.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/select-multi-identity-standard.png){Standard: identity list on a managed connector}

![Screenshot that shows a Consumption workflow and HTTP action configured to access an Azure resource.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/http-action-example-consumption.png){Example HTTP action calling an Azure resource with the identity}

![Screenshot that shows a Consumption workflow and HTTP action with Authentication selected under Advanced parameters.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/add-authentication-property.png){Add the Authentication property}

![Screenshot that shows a Consumption workflow and HTTP action with Managed identity selected as the authentication type.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/select-managed-identity.png){Managed identity selected}

![Screenshot that shows a Consumption workflow and HTTP action with System-assigned managed identity selected.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/select-system-assigned-identity-example.png){System-assigned chosen}

![Screenshot that shows a Consumption workflow and HTTP action with a user-assigned managed identity selected.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/select-user-assigned-identity-example.png){User-assigned chosen}

![Screenshot that shows a Consumption workflow and HTTP action with Audience set to the target resource ID.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/set-audience-url-target-resource.png){Audience set to the target resource}

![Screenshot that shows a Standard workflow and HTTP action configured to access an Azure resource.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/http-action-example-standard.png){Standard: the same HTTP action}

![Screenshot that shows a Consumption workflow and Azure Resource Manager action with connection name entered and managed identity selected.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/connection-azure-resource-manager-consumption.png){Consumption: name the managed-identity connection}

![Screenshot that shows a Standard workflow and Azure Resource Manager action connection with all enabled managed identities.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/connection-azure-resource-manager-standard.png){Standard: all enabled identities offered}

![Diagram that shows authentication connections between a logic app, token store, and target resource.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/api-connection-authentication-flow.png){An API connection uses two hops: logic app → token store, and token store → target resource}

![Screenshot that shows a Standard logic app Connections page with JSON View selected.](https://learn.microsoft.com/en-us/azure/logic-apps/media/authenticate-with-managed-identity/connections-json-view.png){Standard: connections.json holds each connection's authentication settings}
