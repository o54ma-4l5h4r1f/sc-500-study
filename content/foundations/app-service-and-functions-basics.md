+++
slug = "app-service-and-functions-basics"
title = "App Service and Azure Functions basics"
summary = "What an App Service plan and an app are, how deployment slots work, which Azure Functions hosting plans exist, what triggers and bindings are, why inbound and outbound networking are two separate settings, and what Easy Auth does — the ground under the App Service and Functions security lessons."
order = 21
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/app-service/overview",
  "https://learn.microsoft.com/en-us/azure/app-service/overview-hosting-plans",
  "https://learn.microsoft.com/en-us/azure/app-service/deploy-staging-slots",
  "https://learn.microsoft.com/en-us/azure/app-service/networking-features",
  "https://learn.microsoft.com/en-us/azure/app-service/overview-authentication-authorization",
  "https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview",
  "https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale",
  "https://learn.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings",
  "https://learn.microsoft.com/en-us/azure/azure-functions/functions-networking-options",
]
captured = 2026-11-15
related_objectives = ["D3.3.5", "D3.3.7", "D3.3.8", "D2.3.6"]
+++

## In one sentence

**Azure App Service** hosts your web apps and APIs on Microsoft-managed servers, and **Azure Functions** runs small pieces of code when an event happens; both sit on the same App Service platform, so they share the same networking and sign-in features.

## Why it exists

Running a website on a VM means you patch Windows, install IIS or nginx, renew certificates and handle scaling. App Service takes that job away. You upload code (or a container) and Azure runs it. This is **PaaS** — platform as a service: Microsoft owns the OS, you own the app and its settings.

Functions goes one step further. Many jobs are tiny: "when a file lands in storage, resize it", "every night at 2 a.m., clean a table". You do not want a server waiting all day for that. A function wakes up on the event, runs, and goes back to sleep. This is called **serverless** — there are still servers, you just never see them.

For a security engineer these services matter because they are ==the most common internet-facing workloads== in Azure. Every app gets a public `*.azurewebsites.net` address by default. Locking that down — who can reach it, what it can reach, who can sign in — is the job of the D3.3 lessons.

## Mental model

An **App Service plan** is a block of flats you rent. An **app** is one flat in it.

- The **plan** sets the building: region, operating system (Windows or Linux), VM size, number of VM instances, and the **pricing tier**. You pay for the plan.
- Each **app** (web app, API app or function app) lives in one plan. All apps in a plan ==share the same VMs== and scale together.
- A **deployment slot** is a spare flat next door: a full copy of the app with its own address, used for staging.

```svg
<svg viewBox="0 0 900 340" role="img" aria-label="App Service networking: inbound traffic controlled by access restrictions and private endpoints; outbound traffic controlled by virtual network integration; the app itself runs in an App Service plan">
  <defs>
    <marker id="asfarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="line-accent"/>
    </marker>
  </defs>
  <rect class="box" x="15" y="60" width="170" height="70" rx="8"/>
  <text x="100" y="90" text-anchor="middle">Users / internet</text>
  <text x="100" y="112" text-anchor="middle" class="muted">public address</text>
  <rect class="box" x="15" y="200" width="170" height="70" rx="8"/>
  <text x="100" y="230" text-anchor="middle">Your VNet clients</text>
  <text x="100" y="252" text-anchor="middle" class="muted">private IP</text>
  <rect class="box-ok" x="225" y="40" width="180" height="110" rx="8"/>
  <text x="315" y="68" text-anchor="middle" class="label-accent">INBOUND</text>
  <text x="315" y="93" text-anchor="middle">access restrictions</text>
  <text x="315" y="116" text-anchor="middle">private endpoint</text>
  <text x="315" y="139" text-anchor="middle" class="muted">who can reach the app</text>
  <rect class="box-accent" x="445" y="40" width="200" height="250" rx="10"/>
  <text x="545" y="68" text-anchor="middle" class="label-accent">App Service plan</text>
  <rect class="box" x="465" y="85" width="160" height="50" rx="6"/>
  <text x="545" y="115" text-anchor="middle">web app (prod)</text>
  <rect class="box" x="465" y="150" width="160" height="50" rx="6"/>
  <text x="545" y="180" text-anchor="middle">slot: staging</text>
  <rect class="box" x="465" y="215" width="160" height="50" rx="6"/>
  <text x="545" y="245" text-anchor="middle">function app</text>
  <rect class="box-ok" x="685" y="40" width="200" height="110" rx="8"/>
  <text x="785" y="68" text-anchor="middle" class="label-accent">OUTBOUND</text>
  <text x="785" y="93" text-anchor="middle">VNet integration</text>
  <text x="785" y="116" text-anchor="middle">NAT gateway</text>
  <text x="785" y="139" text-anchor="middle" class="muted">what the app can reach</text>
  <rect class="box" x="685" y="200" width="200" height="70" rx="8"/>
  <text x="785" y="230" text-anchor="middle">SQL, storage, Key Vault</text>
  <text x="785" y="252" text-anchor="middle" class="muted">behind private endpoints</text>
  <line class="line-accent" x1="185" y1="95" x2="223" y2="95" marker-end="url(#asfarrow)"/>
  <line class="line-accent" x1="185" y1="235" x2="223" y2="130" marker-end="url(#asfarrow)"/>
  <line class="line-accent" x1="405" y1="110" x2="463" y2="110" marker-end="url(#asfarrow)"/>
  <line class="line-accent" x1="625" y1="110" x2="683" y2="95" marker-end="url(#asfarrow)"/>
  <line class="line-accent" x1="785" y1="150" x2="785" y2="198" marker-end="url(#asfarrow)"/>
</svg>
```

![Diagram that shows App Service inbound and outbound traffic.](https://learn.microsoft.com/en-us/azure/app-service/media/networking-features/default-behavior.png){Default behaviour: inbound and outbound traffic use different addresses and are controlled by different features}

## What actually matters

**1. Pricing tiers decide the features.** Free and Shared run your app on VMs shared with other customers. Basic, Standard, Premium, PremiumV2, PremiumV3 and PremiumV4 give dedicated VMs. **IsolatedV2** runs inside an **App Service Environment (ASE)** — dedicated VMs in your own virtual network. Many security features depend on the tier, so a "cannot enable X" question often has the answer "scale up the plan".

**2. Deployment slots.** Slots exist on ==@Standard, Premium and Isolated== tiers. Each slot is a live app with its own host name. You deploy to `staging`, test, then **swap** it with production. A swap moves content and most settings; a setting marked **Deployment slot setting** stays with its slot ("sticky"). Standard allows five slots. Slots run on the same plan VMs as production.

![Screenshot that shows selections for configuring a new deployment slot called 'staging' in the portal.](https://learn.microsoft.com/en-us/azure/app-service/media/web-sites-staged-publishing/configure-new-slot.png){Add a slot named staging; optionally clone settings from production}

![Screenshot that shows the checkbox for configuring an app setting as a slot setting in the Azure portal.](https://learn.microsoft.com/en-us/azure/app-service/media/web-sites-staged-publishing/set-slot-app-setting.png){The "Deployment slot setting" box makes a setting sticky — it does not move during a swap}

**3. Functions: function app, triggers, bindings.** A **function app** is the Azure resource; it holds one or more **functions**. Each function has ==exactly one **trigger**== — the event that starts it (HTTP request, timer, queue message, blob upload, Event Grid event). **Bindings** are optional plug-ins that read input or write output (for example, "write the result to this Cosmos DB container") without connection code. An HTTP-triggered function is a public web endpoint, protected by **function keys** by default.

**4. Functions hosting plans.** You pick one when you create the function app:

| Plan | What it is | Private endpoint (inbound) | VNet integration (outbound) |
|---|---|---|---|
| **Flex Consumption** | serverless, pay per run, scales to 1,000 instances; the recommended serverless plan | yes | yes |
| **Premium** (Elastic Premium) | prewarmed instances, no cold start | yes | yes |
| **Dedicated** | runs in an App Service plan | yes | yes |
| **Container Apps** | function in a Container Apps environment | managed by the environment | managed by the environment |
| **Consumption** (==!legacy==) | old serverless plan; Linux retired | ==!no== | ==!no== |

**5. Inbound and outbound are separate switches.** This is the single most useful networking idea.

- **Inbound** = who can reach the app. **Access restrictions** are an allow/deny list (IP ranges, VNet subnets, service tags) on the public endpoint. A **private endpoint** gives the app a private IP in your VNet so it can be reached privately.
- **Outbound** = what the app can reach. **Virtual network integration** lets the app send traffic into your VNet, so it can call a database behind a private endpoint. It ==!does not make the app private for inbound traffic==.

![Diagram that illustrates access restrictions.](https://learn.microsoft.com/en-us/azure/app-service/media/networking-features/access-restrictions.png){Access restrictions filter inbound traffic on the public endpoint}

![Diagram that illustrates virtual network integration.](https://learn.microsoft.com/en-us/azure/app-service/media/networking-features/vnet-integration.png){VNet integration is outbound only: the app reaches into the VNet}

**6. Easy Auth.** App Service has built-in **authentication and authorization**, nicknamed **Easy Auth**. Turn it on and a module in front of your code signs users in before requests reach the app — no code needed. Providers: Microsoft Entra, Facebook, Google, X, GitHub, Apple (preview) and any OpenID Connect provider. Functions get the same feature.

![Architecture diagram that shows a process in the site sandbox interacting with identity providers before allowing traffic to the deployed site.](https://learn.microsoft.com/en-us/azure/app-service/media/app-service-authentication-overview/architecture.png){Easy Auth runs beside your code and blocks unauthenticated requests before they reach it}

Both services also support **managed identities** — an identity Azure gives the app so it can reach Key Vault or SQL without a stored password.

## Where it lives

Everything here is in the **Azure portal**.

- **Plan tier:** `Azure portal > App Service plans > (plan) > Scale up`.
- **Slots:** `App Services > (app) > Deployment > Deployment slots`.
- **Networking:** `App Services > (app) > Settings > Networking` — **Inbound traffic configuration** (public network access, access restrictions, private endpoints) and **Outbound traffic configuration** (virtual network integration). Function apps have the same page.
- **Easy Auth:** `(app) > Settings > Authentication > Add identity provider`.
- **Identity:** `(app) > Settings > Identity`.

```bash
az webapp deployment slot create -g rg-lab -n app-lab --slot staging
az webapp vnet-integration add -g rg-lab -n app-lab --vnet vnet-lab --subnet snet-apps   # outbound
az webapp config access-restriction add -g rg-lab -n app-lab --rule-name office \
  --action Allow --ip-address 203.0.113.0/24 --priority 100                              # inbound
```

## Exam traps

- ==!"VNet integration makes my app private."== No — it is outbound only. ==+Private endpoint (plus disabling public access) makes it private inbound.==
- ==!Consumption plan function apps cannot use private endpoints or VNet integration.== Move to Flex Consumption, Premium or Dedicated.
- Deployment slots need ==@Standard or higher==. Free/Shared/Basic have none.
- All apps in one plan share VMs. Isolating one app's compute means a separate plan.
- Easy Auth is authentication at the platform edge; it does not replace authorization inside the app.

## Skip

- **Skip:** auto swap, swap with preview, traffic routing percentages and slot warm-up settings (screenshots below). Know what a swap and a sticky setting are.
- **Skip:** Hybrid Connections and gateway-required VNet integration. Legacy or niche connectivity options.
- **Skip:** App Service Environment internals and Isolated pricing. Know "ASE/IsolatedV2 = app in your own VNet".
- **Skip:** Functions languages, Durable Functions and binding-by-binding configuration.
- **Skip:** per-provider Easy Auth registration steps for Facebook, Google, X, GitHub.

## Screenshots

![Screenshot that shows how to open a deployment slot's management page in the portal.](https://learn.microsoft.com/en-us/azure/app-service/media/web-sites-staged-publishing/open-deployment-slot.png){Open a slot's own management page}

![Screenshot that shows selections for initiating a swap operation in the portal.](https://learn.microsoft.com/en-us/azure/app-service/media/web-sites-staged-publishing/swap-initiate.png){Start a swap}

![Screenshot that shows selections for configuring and completing a swap in the portal.](https://learn.microsoft.com/en-us/azure/app-service/media/web-sites-staged-publishing/swap-configure-source-target-slots.png){Choose source and target slots}

![Screenshot that shows how to configure a swap with preview in the portal.](https://learn.microsoft.com/en-us/azure/app-service/media/web-sites-staged-publishing/swap-with-preview.png){Swap with preview (multi-phase swap)}

![Screenshot that shows selections for configuring auto swap into the production slot in the portal.](https://learn.microsoft.com/en-us/azure/app-service/media/web-sites-staged-publishing/auto-swap.png){Auto swap into production}

![Screenshot that shows portal selections for routing a percentage of request traffic to a deployment slot.](https://learn.microsoft.com/en-us/azure/app-service/media/web-sites-staged-publishing/route-traffic-to-slot.png){Route a percentage of traffic to a slot}

![Screenshot that shows selections for deleting a deployment slot in the portal.](https://learn.microsoft.com/en-us/azure/app-service/media/web-sites-staged-publishing/delete-slot.png){Delete a slot}

![Screenshot that shows app properties, including possible outbound addresses.](https://learn.microsoft.com/en-us/azure/app-service/media/networking-features/app-properties.png){Properties lists the app's possible outbound IP addresses — what a downstream firewall must allow}

![Diagram that illustrates app-assigned address.](https://learn.microsoft.com/en-us/azure/app-service/media/networking-features/app-assigned-address.png){App-assigned address: a dedicated inbound IP}

![Diagram that shows the Hybrid Connections network flow.](https://learn.microsoft.com/en-us/azure/app-service/media/networking-features/hybrid-connections.png){Hybrid Connections (skip-list item)}

![Diagram that illustrates gateway-required virtual network integration.](https://learn.microsoft.com/en-us/azure/app-service/media/networking-features/gw-vnet-integration.png){Gateway-required VNet integration (legacy)}

![Diagram that illustrates an ASE in a virtual network.](https://learn.microsoft.com/en-us/azure/app-service/media/networking-features/app-service-environment.png){App Service Environment: the whole hosting environment sits in your VNet}

![Diagram that illustrates the use of private endpoints in a two-tier app.](https://learn.microsoft.com/en-us/azure/app-service/media/networking-features/multi-tier-app-private-endpoint.png){Two-tier app: the back-end app is reachable only through a private endpoint}

![Diagram that illustrates the use of service endpoints to help secure an app.](https://learn.microsoft.com/en-us/azure/app-service/media/networking-features/multi-tier-app.png){The same design with service endpoints and access restrictions}

![Screenshot of the "Add IP Restriction" pane with the Virtual Network type selected.](https://learn.microsoft.com/en-us/azure/app-service/media/app-service-ip-restrictions/access-restrictions-vnet-add.png){Adding an access restriction rule of type Virtual Network}

![Screenshot of the virtual network integration page where you can enable virtual network integration in your app.](https://learn.microsoft.com/en-us/azure/azure-functions/media/functions-networking-options/vnet-int-function-app.png){Function app: Networking > Virtual Network Integration}

![Select the VNet](https://learn.microsoft.com/en-us/azure/azure-functions/media/functions-networking-options/vnet-int-add-vnet-function-app.png){Pick the VNet and an empty subnet}

![VNETToggle](https://learn.microsoft.com/en-us/azure/azure-functions/media/functions-networking-options/virtual-network-trigger-toggle.png){Premium plan: enable virtual network triggers so a function can be triggered by a source inside the VNet}
