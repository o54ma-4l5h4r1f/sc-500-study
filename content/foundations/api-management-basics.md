+++
slug = "api-management-basics"
title = "Azure API Management basics (gateway, products, keys, policies, tiers, networking)"
summary = "What Azure API Management is; its three parts (gateway, management plane, developer portal); APIs, operations and products; subscription keys; the four policy sections and five policy scopes; the classic and v2 tiers; and the virtual-network options per tier — the base for back-end API protection and the AI gateway."
order = 23
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/api-management/api-management-key-concepts",
  "https://learn.microsoft.com/en-us/azure/api-management/api-management-features",
  "https://learn.microsoft.com/en-us/azure/api-management/v2-service-tiers-overview",
  "https://learn.microsoft.com/en-us/azure/api-management/api-management-subscriptions",
  "https://learn.microsoft.com/en-us/azure/api-management/api-management-howto-policies",
  "https://learn.microsoft.com/en-us/azure/api-management/virtual-network-concepts",
]
captured = 2026-11-15
related_objectives = ["D3.3.9", "D3.1.7", "D3.3.8"]
+++

## In one sentence

**Azure API Management (APIM)** is a gateway that sits in front of your APIs: every call goes to APIM first, APIM checks it (key, token, rate limit, IP), rewrites it if needed, then forwards it to the real back end.

## Why it exists

An **API** (application programming interface) is a door that lets one program ask another for data — "give me order 42", "summarise this text". A company may have hundreds of them, written by different teams, running on App Service, Functions, AKS, on-premises servers or AI models. Securing each one separately means hundreds of different sign-in checks and rate limits.

Think of a hotel **front desk**. Guests do not wander to the kitchen or the laundry. They ask the front desk, which checks their key card, decides if the request is allowed and passes it on. APIM is that front desk for APIs. The back ends only need to trust APIM.

For this exam APIM appears twice: protecting back-end APIs with **policies** (D3.3.9), and as the **AI gateway** in front of Microsoft Foundry models (D3.1.7). Both rest on the ideas below.

## Mental model

APIM has three parts:

- **Gateway** (data plane) — the endpoint clients call, `https://NAME.azure-api.net`. It accepts calls, applies policies and forwards them to back ends.
- **Management plane** — where you configure APIM: in the Azure portal, CLI, Bicep or REST.
- **Developer portal** — an auto-generated website where API consumers read docs and get subscription keys.

![Diagram showing key components of Azure API Management.](https://learn.microsoft.com/en-us/azure/api-management/media/api-management-key-concepts/api-management-components.png){The three parts: gateway, management plane, developer portal}

```svg
<svg viewBox="0 0 900 300" role="img" aria-label="An API call passes through the four policy sections: inbound policies run on the request, the backend section forwards it, outbound policies run on the response, and on-error runs if anything fails">
  <defs>
    <marker id="apimarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="line-accent"/>
    </marker>
  </defs>
  <rect class="box" x="15" y="90" width="130" height="80" rx="8"/>
  <text x="80" y="125" text-anchor="middle">Client</text>
  <text x="80" y="147" text-anchor="middle" class="muted">key or token</text>
  <rect class="box-accent" x="180" y="20" width="540" height="215" rx="10"/>
  <text x="450" y="45" text-anchor="middle" class="label-accent">APIM gateway — one policy document</text>
  <rect class="box-ok" x="200" y="65" width="160" height="80" rx="6"/>
  <text x="280" y="95" text-anchor="middle" class="label-accent">inbound</text>
  <text x="280" y="118" text-anchor="middle" class="muted">validate token</text>
  <text x="280" y="136" text-anchor="middle" class="muted">rate limit, IP filter</text>
  <rect class="box" x="370" y="65" width="160" height="80" rx="6"/>
  <text x="450" y="95" text-anchor="middle" class="label-accent">backend</text>
  <text x="450" y="118" text-anchor="middle" class="muted">forward-request</text>
  <rect class="box-ok" x="540" y="65" width="160" height="80" rx="6"/>
  <text x="620" y="95" text-anchor="middle" class="label-accent">outbound</text>
  <text x="620" y="118" text-anchor="middle" class="muted">strip headers</text>
  <text x="620" y="136" text-anchor="middle" class="muted">mask data</text>
  <rect class="box-risk" x="370" y="165" width="160" height="55" rx="6"/>
  <text x="450" y="190" text-anchor="middle" class="label-risk">on-error</text>
  <text x="450" y="210" text-anchor="middle" class="muted">any failure</text>
  <rect class="box" x="755" y="90" width="130" height="80" rx="8"/>
  <text x="820" y="125" text-anchor="middle">Back end</text>
  <text x="820" y="147" text-anchor="middle" class="muted">App, Function, AI</text>
  <line class="line-accent" x1="145" y1="115" x2="198" y2="105" marker-end="url(#apimarrow)"/>
  <line class="line-accent" x1="360" y1="105" x2="368" y2="105" marker-end="url(#apimarrow)"/>
  <line class="line-accent" x1="530" y1="95" x2="753" y2="115" marker-end="url(#apimarrow)"/>
  <line class="line" x1="753" y1="145" x2="702" y2="125" marker-end="url(#apimarrow)"/>
  <line class="line" x1="540" y1="135" x2="147" y2="150" marker-end="url(#apimarrow)"/>
  <text x="450" y="265" text-anchor="middle" class="muted">scopes, broad to narrow: global, workspace, product, API, operation</text>
</svg>
```

## What actually matters

**1. APIs, operations, products.** An **API** in APIM is one back end you publish (for example "Orders API"). Each API has **operations** — the individual calls, such as `GET /orders/{id}`. A **product** bundles one or more APIs for a group of consumers ("Partner tier") and decides whether a subscription is required. A product that does not require a subscription is **open** — ==!anonymous access== unless a policy enforces authentication.

**2. Subscription keys are not identity.** A **subscription** gives a consumer two keys (primary and secondary, so you can rotate one at a time). The client sends a key in the `Ocp-Apim-Subscription-Key` header or the `subscription-key` query parameter. A subscription can be scoped to a product, a single API, or all APIs. There is also a built-in **all-access subscription** for admins. A key identifies *which subscription* is calling, ==!not which user==. For real authentication, add a policy that validates a Microsoft Entra token (`validate-jwt` or `validate-azure-ad-token`).

![Diagram showing the traditional flow for a Product scope subscription.](https://learn.microsoft.com/en-us/azure/api-management/media/api-management-subscriptions/product-subscription.png){Product-scoped subscription: the developer subscribes to a product and receives keys}

**3. Policies are the security engine.** A **policy** is an XML statement that runs on every call. One policy document has four sections:

| Section | Runs | Typical security use |
|---|---|---|
| `inbound` | on the request, before the back end | validate JWT, check key, rate limit, IP filter, set managed-identity token for the back end |
| `backend` | when forwarding | `forward-request` (only one element allowed) |
| `outbound` | on the response | remove headers that reveal server details, mask data |
| `on-error` | if anything fails | return a safe error message |

Policies can be set at **five scopes**, broad to narrow: **global** (all APIs), **workspace**, **product**, **API**, **operation**. The `base` element pulls in the parent scope's policies. ==!Remove the `base` element and the broader policies (such as a global JWT check) stop applying.== There is a built-in Azure Policy definition that audits this.

![Diagram that illustrates the five policy scopes.](https://learn.microsoft.com/en-us/azure/api-management/media/api-management-howto-policies/policy-scopes.png){Five policy scopes; the base element decides inheritance and order}

**4. Tiers: classic and v2.** Classic tiers: **Consumption**, **Developer** (no SLA — not for production), **Basic**, **Standard**, **Premium**. The newer **v2** tiers: **Basic v2**, **Standard v2**, **Premium v2**. Networking features depend on the tier, which is why tier questions appear.

**5. Networking options per tier.** Default: the gateway has a public endpoint and calls public back ends.

| Option | Tiers | Direction |
|---|---|---|
| **VNet injection — external mode** | Developer, Premium | gateway public, can reach VNet back ends |
| **VNet injection — internal mode** | Developer, Premium | gateway reachable ==@only inside the VNet== (internal load balancer) |
| **VNet injection (v2)** | Premium v2 | gateway private, inbound and outbound |
| **VNet integration (v2)** | Standard v2, Premium v2 | ==@outbound only== — gateway stays public |
| **Inbound private endpoint** | Developer, Basic, Standard, Standard v2, Premium, Premium v2 | ==@inbound only== |

You can disable public network access only **after** a private endpoint exists. A common pattern puts APIM in internal mode behind **Application Gateway with WAF** for public traffic.

![Diagram showing a connection to internal virtual network.](https://learn.microsoft.com/en-us/azure/api-management/media/virtual-network-concepts/api-management-vnet-internal.png){Internal mode: endpoints reachable only from inside the VNet}

**6. AI gateway.** The same gateway, with AI-specific policies (token limits, content safety, semantic caching), fronts Microsoft Foundry and Azure OpenAI models. See the Microsoft Foundry primer.

## Where it lives

All in the **Azure portal**.

- **Create / tier:** `Azure portal > API Management services > Create`; change tier under `(instance) > Deployment + infrastructure > Pricing tier`.
- **APIs and policies:** `(instance) > APIs > APIs > (API) > Design` → **Inbound processing / Outbound processing / Backend** → **+ Add policy** or the code editor. Global scope: **All APIs**.
- **Products:** `(instance) > APIs > Products`.
- **Subscriptions:** `(instance) > APIs > Subscriptions`.
- **Networking:** `(instance) > Deployment + infrastructure > Network` (VNet, private endpoints, public network access).
- **Managed identity:** `(instance) > Security > Managed identities`.

```xml
<inbound>
  <base />
  <validate-azure-ad-token tenant-id="contoso.onmicrosoft.com">
    <client-application-ids><application-id>00000000-0000-0000-0000-000000000000</application-id></client-application-ids>
  </validate-azure-ad-token>
  <rate-limit calls="100" renewal-period="60" />
</inbound>
```

## Exam traps

- ==!"A subscription key authenticates the user."== It identifies a subscription. ==+Validate an Entra token in an inbound policy== for user or app identity.
- ==!VNet integration (Standard v2) makes the gateway private.== It is outbound only. Private inbound needs a private endpoint, internal mode or Premium v2 injection.
- Internal/external VNet injection = ==@Developer or Premium== classic tiers only.
- Developer tier has no SLA — never the answer for production.
- Forgetting the `base` element silently drops inherited policies.
- An open product (no subscription required) means anonymous calls unless a policy blocks them.

## Skip

- **Skip:** the developer portal's customisation and content management.
- **Skip:** self-hosted gateways, workspaces and workspace gateways. Know workspaces exist for team separation.
- **Skip:** GraphQL resolvers, SOAP-to-REST conversion and policy expressions in C#.
- **Skip:** tier pricing, units and throughput figures. Learn only which tiers support which network option.

## Screenshots

![Diagram showing role of APIs in connected experiences.](https://learn.microsoft.com/en-us/azure/api-management/media/api-management-key-concepts/apis-connected-experiences.png){APIs connect apps, partners and devices to back-end services}

![Screenshot of API Management developer portal - administrator mode.](https://learn.microsoft.com/en-us/azure/api-management/media/api-management-key-concepts/cover.png){The developer portal in administrator mode}

![Diagram showing a connection to external virtual network.](https://learn.microsoft.com/en-us/azure/api-management/media/virtual-network-concepts/api-management-vnet-external.png){External mode: public endpoints, gateway can reach VNet back ends}

![Diagram of injecting an API Management instance in a virtual network to isolate inbound and outbound traffic.](https://learn.microsoft.com/en-us/azure/api-management/media/inject-vnet-v2/vnet-injection.png){Premium v2 VNet injection: gateway private, inbound and outbound}

![Diagram of integrating API Management instance with a delegated subnet.](https://learn.microsoft.com/en-us/azure/api-management/media/integrate-vnet-outbound/vnet-integration.png){v2 VNet integration: outbound only, through a delegated subnet}

![Diagram showing a secure connection to API Management using private endpoint.](https://learn.microsoft.com/en-us/azure/api-management/media/virtual-network-concepts/api-management-private-endpoint.png){Inbound private endpoint}

![Diagram that shows a secure inbound connection to API Management Standard v2 using private endpoint.](https://learn.microsoft.com/en-us/azure/api-management/media/private-endpoint/private-endpoint.png){Standard v2: private endpoint inbound + VNet integration outbound = full network isolation}

![Diagram showing Application Gateway in front of API Management instance.](https://learn.microsoft.com/en-us/azure/api-management/media/virtual-network-concepts/api-management-application-gateway.png){Application Gateway (with WAF) in front of APIM in internal mode}
