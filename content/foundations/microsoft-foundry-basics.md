+++
slug = "microsoft-foundry-basics"
title = "Microsoft Foundry basics (resource, projects, deployments, agents, guardrails, AI gateway)"
summary = "What Microsoft Foundry is (formerly Azure AI Studio / Azure AI Foundry), how the Foundry resource and its projects fit together, what a model deployment and an agent are, how guardrails (formerly content filters) and Azure AI Content Safety work, which Foundry roles exist after the 2026 rename, and where the AI gateway in API Management fits."
order = 24
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/foundry/what-is-foundry",
  "https://learn.microsoft.com/en-us/azure/foundry/concepts/architecture",
  "https://learn.microsoft.com/en-us/azure/foundry/concepts/rbac-foundry",
  "https://learn.microsoft.com/en-us/azure/foundry/agents/overview",
  "https://learn.microsoft.com/en-us/azure/foundry/guardrails/guardrails-overview",
  "https://learn.microsoft.com/en-us/azure/foundry/guardrails/intervention-points",
  "https://learn.microsoft.com/en-us/azure/foundry/guardrails/how-to-create-guardrails",
  "https://learn.microsoft.com/en-us/azure/ai-services/content-safety/overview",
  "https://learn.microsoft.com/en-us/azure/foundry-classic/concepts/ai-resources",
  "https://learn.microsoft.com/en-us/azure/api-management/genai-gateway-capabilities",
]
captured = 2026-11-15
related_objectives = ["D3.1.7", "D3.1.8", "D3.1.9", "D3.1.10", "D3.1.4"]
+++

## In one sentence

**Microsoft Foundry** is Azure's platform for building with AI models and AI agents: one Azure resource (the **Foundry resource**) holds your model deployments and security settings, and **projects** inside it are where teams build agents and apps.

## Why it exists

Before Foundry, building one AI app on Azure meant stitching together an Azure OpenAI resource, an Azure AI Services resource, an Azure Machine Learning "hub", a storage account and a key vault, each with its own permissions and network settings. Security teams had five things to lock down for one chatbot.

Foundry collapses that into one resource type under one Azure resource provider (`Microsoft.CognitiveServices`). Networking, role assignments, policies and model deployments are set once at the resource. Teams then create projects underneath, like folders, and inherit those settings.

**Naming — check what a question means.** Microsoft renamed this product twice:

| Previous | Current |
|---|---|
| ==@Azure AI Studio==, then ==@Azure AI Foundry== | **Microsoft Foundry** |
| Azure AI Services | **Foundry Tools** |
| Hub + Azure OpenAI + Azure AI Services | **Foundry resource** with projects |
| old portal | **Foundry (classic)** portal; the new portal is **Foundry** |

The training module behind D3.1.9 still uses the slug *azure-ai-foundry*. Treat all three names as the same product.

## Mental model

Think of a **office building with a security desk**.

- The **Foundry resource** is the building. Building-wide rules live here: network access (public, private endpoint), encryption keys, managed identity, connections to other services, and **model deployments**.
- **Projects** are the floors. Each team works on its own floor with its own files, agents and evaluations. Projects are child resources; they can have their own role assignments but share the building's rules by default.
- **Connected resources** (Storage, Key Vault, Azure AI Search) are neighbouring buildings. ==!They keep their own locks.== You secure them separately.
- **Guardrails** are the security desk scanning what goes in and out.
- The **AI gateway** in API Management is a gatehouse at the street, in front of the whole building.

![Diagram showing the Foundry resource hierarchy with a governance boundary containing model deployments, security settings, connections, and two projects.](https://learn.microsoft.com/en-us/azure/foundry/media/architecture/architecture.svg){Foundry resource = governance boundary (deployments, security, connections) with projects inside. Storage, Key Vault and AI Search are separate boundaries.}

```svg
<svg viewBox="0 0 900 330" role="img" aria-label="Request path: an app or user calls the AI gateway in API Management, which forwards to a Foundry model deployment or agent; guardrails scan user input, tool calls, tool responses and output">
  <defs>
    <marker id="fdyarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="line-accent"/>
    </marker>
  </defs>
  <rect class="box" x="15" y="110" width="130" height="80" rx="8"/>
  <text x="80" y="145" text-anchor="middle">App / user</text>
  <text x="80" y="167" text-anchor="middle" class="muted">prompt</text>
  <rect class="box-ok" x="180" y="95" width="170" height="110" rx="8"/>
  <text x="265" y="125" text-anchor="middle" class="label-accent">AI gateway</text>
  <text x="265" y="148" text-anchor="middle">API Management</text>
  <text x="265" y="170" text-anchor="middle" class="muted">token limits, auth</text>
  <text x="265" y="190" text-anchor="middle" class="muted">logging</text>
  <rect class="box-accent" x="385" y="20" width="500" height="290" rx="10"/>
  <text x="635" y="45" text-anchor="middle" class="label-accent">Foundry resource</text>
  <rect class="box-risk" x="405" y="65" width="460" height="45" rx="6"/>
  <text x="635" y="93" text-anchor="middle">Guardrail: user input, tool call, tool response, output</text>
  <rect class="box" x="405" y="125" width="220" height="80" rx="6"/>
  <text x="515" y="155" text-anchor="middle">Model deployment</text>
  <text x="515" y="178" text-anchor="middle" class="muted">e.g. gpt-4o-mini</text>
  <rect class="box" x="645" y="125" width="220" height="80" rx="6"/>
  <text x="755" y="155" text-anchor="middle">Project: agent</text>
  <text x="755" y="178" text-anchor="middle" class="muted">model + tools</text>
  <rect class="box" x="405" y="220" width="460" height="70" rx="6"/>
  <text x="635" y="248" text-anchor="middle">Resource settings: network, identity, CMK</text>
  <text x="635" y="272" text-anchor="middle" class="muted">connections to Storage, Key Vault, AI Search</text>
  <line class="line-accent" x1="145" y1="150" x2="178" y2="150" marker-end="url(#fdyarrow)"/>
  <line class="line-accent" x1="350" y1="150" x2="403" y2="110" marker-end="url(#fdyarrow)"/>
</svg>
```

## What actually matters

**1. A model deployment is how you get to use a model.** A model in the **model catalog** is just a listing. To call it you create a **deployment** on the Foundry resource. The deployment has a name, an endpoint and a **deployment type** that decides where prompts are processed: **Global** (any Azure region), **Data Zone** (stays in the US or EU zone) or **Standard/Regional** (one region). Data-residency questions turn on this choice.

**2. An agent is a model plus instructions plus tools.** In the **Foundry Agent Service** an agent combines a deployed model, instructions ("you are a helpdesk assistant") and **tools** — actions it may take, such as searching an index, calling an API or running code. A **prompt agent** is defined declaratively and run by Foundry. A **hosted agent** is your own code packaged and run by Foundry. Agents can be published to Microsoft Teams, Microsoft 365 Copilot and the **Entra Agent Registry**. Each agent's identity in Microsoft Entra is what Conditional Access for agents and Entra Agent ID governance act on (see the AI agents primer).

**3. Guardrails (formerly content filters).** A **guardrail** is a named set of **controls**. Each control says: which **risk** to detect, which **intervention point** to scan, and which **action** to take (annotate, or annotate and block). The four intervention points:

| Intervention point | What is scanned | Applies to |
|---|---|---|
| **User input** | the prompt | models and agents |
| **Tool call** (preview) | what the agent is about to send to a tool | agents only |
| **Tool response** (preview) | what came back from the tool | agents only |
| **Output** | the final answer | models and agents |

Risks include harmful content categories (hate, sexual, violence, self-harm) with severity levels, **user input attacks** (jailbreaks — detected by **Prompt Shields**), **indirect attacks** (prompt injection hidden in documents or tool results) and **protected material**. Every deployment gets a Microsoft default guardrail. ==!You cannot edit the default== — you create a custom guardrail and assign it to deployments or agents. Turning blocking off needs an approved Limited Access request.

**4. Azure AI Content Safety is the engine.** The detection models behind guardrails are **Azure AI Content Safety**. You can also call Content Safety directly (Content Safety Studio and APIs) for models outside Foundry.

**5. Roles were renamed in 2026.** Foundry has its own built-in Azure roles:

| New name | Old name | In short |
|---|---|---|
| **Foundry User** | Azure AI User | build and use: agents, evaluations, files (data plane) — least-privilege developer role |
| **Foundry Project Manager** | Azure AI Project Manager | manage projects and assign Foundry User |
| **Foundry Account Owner** | Azure AI Account Owner | manage the resource, deployments, guardrails |
| **Foundry Owner** | Azure AI Owner | full access, including data plane |

Microsoft's docs say role IDs and permissions are unchanged; ==!expect either name in questions==. The split mirrors **control plane** (create deployments, projects, settings) versus **data plane** (build agents, run evaluations, upload files). Give each project's managed identity **Foundry User** too.

**6. Where the AI gateway fits.** The **AI gateway** is not a Foundry feature. It is a set of **API Management** capabilities placed in front of Foundry and Azure OpenAI endpoints: token-based rate limits, Entra authentication to the back end with a managed identity, content safety checks, semantic caching, load balancing and token-usage metrics. Foundry guardrails protect inside the model call; the gateway controls who may call and how much.

![Diagram summarizing AI gateway capabilities of Azure API Management.](https://learn.microsoft.com/en-us/azure/api-management/media/genai-gateway-capabilities/capabilities-summary.png){AI gateway capabilities in API Management}

## Where it lives

Two portals — beginners get lost here first.

- **Foundry portal** (`ai.azure.com`): projects, model catalog, deployments, agents, **Build > Guardrails**. The **New Foundry** toggle switches between the new and classic portals.
- **Azure portal**: the Foundry resource itself — `Azure portal > (search "Foundry") > (resource)` for **Networking**, **Identity**, **Encryption**, **Access control (IAM)** and **Diagnostic settings**.
- **API Management** (Azure portal): `(APIM instance) > APIs > Create from Azure resource > Microsoft Foundry` to put the AI gateway in front.
- **Defender for Cloud** (Azure or Defender portal): the **AI services** plan and the Data and AI security dashboard.

```bash
az cognitiveservices account create -g rg-lab -n fdy-lab --kind AIServices --sku S0 -l eastus2 \
  --custom-domain fdy-lab --allow-project-management
```

## Exam traps

- ==!"Azure AI Foundry" and "Microsoft Foundry" are different products.== Same product, renamed.
- ==!Locking down the Foundry resource secures the connected storage account.== It does not — connected resources have their own network rules and RBAC.
- Guardrails scan **content**; they do not decide **who** may call the model. Identity and rate limits belong to RBAC and the AI gateway.
- Tool call and tool response intervention points are **agents only** (preview).
- The developer role is **Foundry User** (Azure AI User), not Contributor. Contributor on the resource is control plane, not data plane.

## Skip

- **Skip:** hub-based projects and the classic hub resource (`Microsoft.MachineLearningServices/workspace`). Legacy; new work is on Foundry projects. Know only that hubs still exist for some open-source model and ML training cases.
- **Skip:** the full deployment-type pricing table (provisioned throughput units, batch). Know Global vs Data Zone vs Regional for data residency.
- **Skip:** evaluations, fine-tuning, prompt flow and the SDK/Responses API changes.
- **Skip:** voice agents, Foundry Tools (Speech, Vision, Language) and model-by-model catalog details.
- **Skip:** third-party guardrail integrations.

## Screenshots

![Diagram shows access for Owner.](https://learn.microsoft.com/en-us/azure/foundry/media/rbac-foundry/owner.png){Owner: full control-plane access and role assignment}

![Diagram shows access for Foundry Owner.](https://learn.microsoft.com/en-us/azure/foundry/media/rbac-foundry/foundry-owner.png){Foundry Owner (formerly Azure AI Owner)}

![Diagram shows access for Foundry Account Owner.](https://learn.microsoft.com/en-us/azure/foundry/media/rbac-foundry/foundry-account-owner.png){Foundry Account Owner (formerly Azure AI Account Owner)}

![Diagram shows access for Foundry Project Manager.](https://learn.microsoft.com/en-us/azure/foundry/media/rbac-foundry/foundry-project-manager.png){Foundry Project Manager (formerly Azure AI Project Manager)}

![Diagram shows access for Foundry User.](https://learn.microsoft.com/en-us/azure/foundry/media/rbac-foundry/foundry-user.png){Foundry User (formerly Azure AI User): the least-privilege builder role}

![Screenshot of Microsoft Foundry model import in the Azure portal.](https://learn.microsoft.com/en-us/azure/api-management/media/genai-gateway-capabilities/ai-foundry-import.png){API Management: import a Microsoft Foundry endpoint as an API}

![Diagram of limiting Azure OpenAI Service tokens in API Management.](https://learn.microsoft.com/en-us/azure/api-management/media/genai-gateway-capabilities/token-rate-limiting.png){Token rate limiting at the gateway}

![Diagram of semantic caching in API Management.](https://learn.microsoft.com/en-us/azure/api-management/media/genai-gateway-capabilities/semantic-caching.png){Semantic caching: similar prompts answered from cache}

![Diagram of content safety policy in API Management.](https://learn.microsoft.com/en-us/azure/api-management/media/genai-gateway-capabilities/content-safety.png){Content safety policy at the gateway}

![Diagram of using backend load balancing in API Management.](https://learn.microsoft.com/en-us/azure/api-management/media/genai-gateway-capabilities/backend-load-balancing.png){Load balancing across model back ends}

![Diagram of using backend circuit breaker in API Management.](https://learn.microsoft.com/en-us/azure/api-management/media/genai-gateway-capabilities/backend-circuit-breaker.png){Circuit breaker for failing back ends}

![Diagram of emitting token metrics using API Management.](https://learn.microsoft.com/en-us/azure/api-management/media/genai-gateway-capabilities/emit-token-metrics.png){Emitting token-usage metrics}

![Screenshot of analytics for language model APIs in the portal.](https://learn.microsoft.com/en-us/azure/api-management/media/api-management-howto-llm-logs/analytics-workbook-small.png){Language model API analytics}

![Screenshot of MCP servers in API Center in the portal.](https://learn.microsoft.com/en-us/azure/api-management/media/genai-gateway-capabilities/mcp-registry-api-center.png){MCP servers registered in API Center}
