+++
slug = "ai-agents-and-mcp-basics"
title = "AI agents and MCP basics (LLMs, tools, Copilot Studio, Entra Agent ID, AI-specific risks)"
summary = "What an LLM, a prompt, grounding and a hallucination are; what turns a model into an agent (instructions, tools, memory, identity); how MCP connects agents to tools; what Copilot Studio agents and Microsoft Entra Agent ID are; and the three AI risks this exam keeps returning to — prompt injection, jailbreaks and data oversharing — with the product that answers each."
order = 25
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/agent-framework/journey/llm-fundamentals",
  "https://learn.microsoft.com/en-us/agent-framework/journey/adding-tools",
  "https://learn.microsoft.com/en-us/dotnet/ai/get-started-mcp",
  "https://learn.microsoft.com/en-us/microsoft-copilot-studio/fundamentals-what-is-copilot-studio",
  "https://learn.microsoft.com/en-us/microsoft-copilot-studio/external-security-provider",
  "https://learn.microsoft.com/en-us/entra/agent-id/what-is-microsoft-entra-agent-id",
  "https://learn.microsoft.com/en-us/entra/agent-id/what-are-agent-identities",
  "https://learn.microsoft.com/en-us/entra/identity/conditional-access/agent-id",
  "https://learn.microsoft.com/en-us/copilot/plan/copilot-prompt-defense-in-depth",
  "https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/ai-agent-real-time-protection",
]
captured = 2026-11-15
related_objectives = ["D3.1.2", "D3.1.3", "D3.1.4", "D3.1.5", "D3.1.6", "D3.1.9", "D3.1.11"]
+++

## In one sentence

A **large language model (LLM)** only writes text; an **AI agent** wraps an LLM with instructions, **tools** and an **identity** so it can act; **MCP** is the standard plug that connects agents to tools — and every security control in D3.1 exists because an agent can be tricked, over-permissioned, or shown data it should not see.

## Why it exists

As an analyst you met AI as a helper — Security Copilot summarising an incident. As an engineer you now secure AI that *other people* build: a helpdesk agent in Copilot Studio, a Foundry agent that reads SharePoint, a coding agent with access to a repo.

Think of a very fast, very well-read **intern with a master key**. They write a neat answer in seconds. They believe whatever is written on any note handed to them — including a note an attacker slipped into a document. And if their badge opens every door the manager can open, they can carry out anything the manager can reach. The controls on this exam answer three questions about that intern: *what can trick them*, *who are they*, and *what can they reach*.

## Mental model

```svg
<svg viewBox="0 0 900 400" role="img" aria-label="Agent anatomy and risks: a user prompt and retrieved data enter the agent; the agent's model decides on tool calls through MCP servers; the agent has an Entra agent identity; risks are prompt injection on the way in, oversharing through the data it can read, and excessive actions through its tools">
  <defs>
    <marker id="aiaarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="line-accent"/>
    </marker>
  </defs>
  <rect class="box" x="15" y="40" width="170" height="70" rx="8"/>
  <text x="100" y="70" text-anchor="middle">User prompt</text>
  <text x="100" y="92" text-anchor="middle" class="muted">direct input</text>
  <rect class="box" x="15" y="150" width="170" height="70" rx="8"/>
  <text x="100" y="180" text-anchor="middle">Grounding data</text>
  <text x="100" y="202" text-anchor="middle" class="muted">files, mail, web</text>
  <rect class="box-accent" x="230" y="20" width="300" height="230" rx="10"/>
  <text x="380" y="46" text-anchor="middle" class="label-accent">Agent</text>
  <rect class="box" x="250" y="60" width="260" height="36" rx="6"/>
  <text x="380" y="83" text-anchor="middle">Instructions (system prompt)</text>
  <rect class="box" x="250" y="104" width="260" height="36" rx="6"/>
  <text x="380" y="127" text-anchor="middle">LLM = reasoning engine</text>
  <rect class="box" x="250" y="148" width="260" height="36" rx="6"/>
  <text x="380" y="171" text-anchor="middle">Memory (conversation)</text>
  <rect class="box-ok" x="250" y="192" width="260" height="40" rx="6"/>
  <text x="380" y="217" text-anchor="middle">Entra agent identity</text>
  <rect class="box" x="590" y="20" width="295" height="140" rx="10"/>
  <text x="737" y="46" text-anchor="middle" class="label-accent">Tools via MCP servers</text>
  <text x="737" y="75" text-anchor="middle">search, send mail, call API</text>
  <text x="737" y="100" text-anchor="middle">read SharePoint, run code</text>
  <text x="737" y="130" text-anchor="middle" class="muted">model asks; the host runs it</text>
  <line class="line-accent" x1="185" y1="75" x2="228" y2="75" marker-end="url(#aiaarrow)"/>
  <line class="line-accent" x1="185" y1="185" x2="228" y2="170" marker-end="url(#aiaarrow)"/>
  <line class="line-accent" x1="530" y1="110" x2="588" y2="90" marker-end="url(#aiaarrow)"/>
  <rect class="box-risk" x="15" y="290" width="270" height="95" rx="8"/>
  <text x="150" y="318" text-anchor="middle" class="label-risk">Prompt injection</text>
  <text x="150" y="342" text-anchor="middle">jailbreak (direct)</text>
  <text x="150" y="364" text-anchor="middle">hidden in a document (indirect)</text>
  <rect class="box-risk" x="315" y="290" width="270" height="95" rx="8"/>
  <text x="450" y="318" text-anchor="middle" class="label-risk">Oversharing</text>
  <text x="450" y="342" text-anchor="middle">agent reads what the</text>
  <text x="450" y="364" text-anchor="middle">user or agent can reach</text>
  <rect class="box-risk" x="615" y="290" width="270" height="95" rx="8"/>
  <text x="750" y="318" text-anchor="middle" class="label-risk">Excessive agency</text>
  <text x="750" y="342" text-anchor="middle">tool does harm: sends,</text>
  <text x="750" y="364" text-anchor="middle">deletes, exfiltrates</text>
</svg>
```

## What actually matters

**1. Generative AI words.**

| Term | Meaning |
|---|---|
| **LLM** | a model trained to predict the next **token** (a word-piece) |
| **Prompt** | the text sent to the model; the **system prompt** is the hidden instructions the builder sets |
| **Grounding** (retrieval, RAG) | adding real data — files, mail, an index — to the prompt so the answer is about your world |
| **Hallucination** | a confident, plausible, wrong answer |
| **Non-deterministic** | the same prompt can give a different answer next time |

An LLM on its own ==!cannot act== — it only produces text. Risk appears when you connect it to data and tools.

**2. What makes an agent.** An agent is a program that uses a model to read input, reason, and decide on actions. Its parts: **model**, **instructions**, **knowledge** (grounding sources), **tools** (functions or APIs it may call), **memory** and — the part engineers own — an **identity** with permissions. Agents range from read-only assistants to autonomous agents that run on a trigger with no human present. The more autonomy, the more you need **human-in-the-loop** approval before sensitive actions.

![Diagram showing the tool-calling loop: the LLM interacts with external tools and memory in a loop before returning a final response.](https://learn.microsoft.com/en-us/agent-framework/workflows/resources/images/ai-agent.png){An agent loop: the model decides, a tool acts, the result goes back to the model}

**3. MCP — USB for AI tools.** The **Model Context Protocol (MCP)** is an open standard for how an AI app reaches tools and data. The **host** is the AI app (VS Code, Copilot Studio, an agent). Inside it, one **client** per server holds the connection. An **MCP server** offers **tools** (functions the model may ask to run), **resources** (read-only data) and **prompts** (templates). An MCP server is an API like any other: it needs authentication, and API Management can front it.

![A diagram showing the architecture pattern of MCP, including hosts, clients, and servers.](https://learn.microsoft.com/en-us/dotnet/ai/media/mcp/model-context-protocol-architecture-diagram.png){MCP: one host, one client per server, many servers}

**4. Copilot Studio agents.** **Microsoft Copilot Studio** is Microsoft's low-code tool for building agents, used by business "makers" rather than developers. Agents live in a Power Platform environment, connect to SharePoint, Dataverse and connectors, and publish to Teams, Microsoft 365 Copilot or websites. Each new agent gets an **agent identity** in Entra, and its creator is recorded as **sponsor**. Custom agents block **user prompt injection attacks (UPIA)** and **cross-domain prompt injection attacks (XPIA)** by default. On top of that, you can connect an **external threat detection** system — such as **Microsoft Defender** — that checks every tool call at runtime and blocks risky ones. That is the **real-time protection** of D3.1.3.

![Screenshot of the Copilot Studio Home page.](https://learn.microsoft.com/en-us/microsoft-copilot-studio/media/fundamentals-what-is-copilot-studio/home-page.png){Copilot Studio home page — where makers build agents}

**5. Microsoft Entra Agent ID.** Agents need identities just as apps need service principals. **Microsoft Entra Agent ID** adds:

- **Agent identity** — a directory identity for one agent, logged as an AI agent in sign-in and audit logs.
- **Agent identity blueprint** — a template for a class of agents. Policies and permissions set on the blueprint apply to every agent created from it, including future ones.
- **Sponsor** — a human accountable for the agent's lifecycle (different from an *owner*, who handles technical operations). Every agent identity needs one.
- **Access patterns** — an agent can act **on behalf of** a signed-in user (delegated), with its **own agent identity** (app-only), or through an **agent user** account. ==!A Conditional Access policy that targets one of these does not cover the others.==

Conditional Access for agents needs **Microsoft Agent 365** plus Entra ID P1 or Microsoft 365 E3 — or **Microsoft 365 E7** on its own. Agent-risk policies also need P2 or E5. Microsoft labels parts of Agent ID as preview.

![Diagram showing the data access patterns for agent identities.](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/agent-id/data-access-patterns-diagram.png){Three access patterns: on behalf of a user, own agent identity, agent user account}

**6. The three AI risks and who answers each.**

| Risk | What it is | Main controls |
|---|---|---|
| **Jailbreak** (direct prompt injection, UPIA) | the user types a prompt that tries to override the system prompt | Foundry guardrails (Prompt Shields), Copilot Studio built-in protection |
| **Indirect prompt injection** (XPIA) | hidden instructions in a document, web page, email or tool result | Foundry tool-response guardrails, Defender real-time protection, Defender for Office 365 |
| **Data oversharing** | the agent faithfully shows data the user can technically open but never should — overshared SharePoint sites | SharePoint permissions, sensitivity labels, Purview DSPM for AI, Restricted Content Discovery |

Detections land in the **Defender portal** (alerts, the `BehaviorInfo` table) and in **Purview Audit** (a `JailbreakDetected` flag on Copilot interaction records).

## Where it lives

AI security is spread across portals. This table is the map.

| Task | Portal |
|---|---|
| Build Copilot Studio agents; see **Protection status** | Copilot Studio (`copilotstudio.microsoft.com`) |
| Agent identities, blueprints, sponsors; Conditional Access for agents | Microsoft Entra admin center (`entra.microsoft.com`) |
| Real-time protection rules; agent blast radius | Microsoft Defender portal (`security.microsoft.com`) |
| DSPM for AI, oversharing assessments, audit | Microsoft Purview portal (`purview.microsoft.com`) |
| Manage, block or publish agents for Microsoft 365 users | Microsoft 365 admin center (`admin.microsoft.com`) |
| Foundry agents and guardrails | Foundry portal (`ai.azure.com`) |

## Exam traps

- ==!"The model itself enforces who can see what."== It does not. An agent sees what its identity — or the signed-in user — can reach. ==+Fix permissions and labels at the data source.==
- **Direct** injection comes from the user; **indirect** comes from content the agent reads. Different controls, different intervention points.
- A sponsor is accountable for the agent's existence; an owner runs it technically.
- MCP roles: ==+host = the app, client = the connector inside it, server = the tool provider.==
- Conditional Access for an agent identity does not apply to the same agent's agent-user account.

## Skip

- **Skip:** how LLMs are trained (pretraining, RLHF), temperature, tokenizers, context-window maths.
- **Skip:** MCP wire messages (JSON-RPC, `CallToolRequest`, transports).
- **Skip:** Copilot Studio topics, generative orchestration internals and building the external threat-detection web service yourself.
- **Skip:** Agent 365 licensing permutations and pricing. Know only "Agent 365 is needed for Entra security features on agents".
- **Skip:** Microsoft Defender for Endpoint local-agent runtime protection (Claude Code, Codex CLI). Endpoint scenario, not this exam's focus.

## Screenshots

![Diagram showing the identity management, access protection, governance, and compliance capabilities of Microsoft Entra Agent ID.](https://learn.microsoft.com/en-us/entra/agent-id/media/what-is-microsoft-entra-agent-id/microsoft-entra-agent-identity-capabilities.png){Entra Agent ID capabilities: identity, access, governance, compliance}

![Diagram showing a Conditional Access policy applied to agent identities from one blueprint.](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/agent-id/conditional-access-agent-identity-blueprint-diagram.png){A Conditional Access policy targeting one blueprint covers every agent created from it}

![Diagram showing the Conditional Access flow for agent identities.](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/agent-id/conditional-access-agent-diagram.png){Conditional Access evaluating an agent's token request, with custom security attributes}

![Screenshot of the Real-time protection page in the Microsoft Defender portal.](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/media/ai-agent-real-time-protection/real-time-protection-rules-list.png){Defender portal: real-time protection rules for AI agents. The default rule audits; custom rules block.}

![Screenshot of the Create rule wizard Rule details step.](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/media/ai-agent-real-time-protection/create-rule-details.png){Create rule: name, status, action}

![Screenshot of the Choose agents to exclude pane.](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/media/ai-agent-real-time-protection/create-rule-exclude-agents.png){Scope: exclude agents by platform}

![Screenshot of the detection types dropdown.](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/media/ai-agent-real-time-protection/create-rule-detection-types.png){Detection types include secret exfiltration and malicious content propagation}

![Screenshot of the Review and create step.](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/media/ai-agent-real-time-protection/create-rule-review.png){Review and create}

![Screenshot of the Real-time protection page with a rule selected.](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/media/ai-agent-real-time-protection/real-time-protection-rule-details.png){Rule details}

![Screenshot of the Prompt evidence collection page.](https://learn.microsoft.com/en-us/defender-xdr/security-for-ai/media/ai-agent-real-time-protection/prompt-evidence-collection.png){Prompt evidence collection: keep the prompt content for investigation}
