+++
slug = "entra-id-basics"
title = "Microsoft Entra ID for engineers: users, guests, groups, app registrations, service principals, managed identities"
summary = "Entra ID is the directory behind every Azure and Microsoft 365 sign-in. An engineer must tell its object types apart: users and guests, groups, app registrations (application objects), enterprise applications (service principals), managed identities and workload identities."
order = 4
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/entra/fundamentals/whatis",
  "https://learn.microsoft.com/en-us/entra/identity-platform/app-objects-and-service-principals",
  "https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/what-is-application-management",
  "https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/overview",
  "https://learn.microsoft.com/en-us/entra/workload-id/workload-identities-overview",
  "https://learn.microsoft.com/en-us/entra/external-id/user-properties",
  "https://learn.microsoft.com/en-us/entra/fundamentals/concept-learn-about-groups",
  "https://learn.microsoft.com/en-us/entra/fundamentals/licensing",
]
captured = 2026-11-15
related_objectives = ["D1.1.4", "D1.1.5", "D1.1.6", "D1.1.1", "D1.1.2", "D1.1.3", "D1.2.3", "D3.1.6", "D3.3.5"]
+++

## In one sentence

**Microsoft Entra ID** is the cloud directory that stores every identity your organisation uses — people, guests, groups, applications and Azure resources — and issues the tokens that let them sign in.

## Why it exists

As an analyst you read Entra sign-in logs and saw "user X signed in from Y". You rarely cared what *kind* of object X was. An engineer must, because each kind is secured differently. A user gets MFA. An app gets a certificate or, better, no credential at all. A guest gets limited directory rights.

Think of a large office's badge system. Staff get permanent badges. Visitors get **guest** badges and are escorted. Contractor firms are registered once at head office (**app registration**) and then issued a site badge at each building they work in (**service principal**). The building's own machines — lifts, cameras — carry built-in badges that can't be lent or copied (**managed identities**).

## Mental model

```svg
<svg viewBox="0 0 860 380" role="img" aria-label="Entra ID object types: users and guests are human identities; app registration is the global application object in its home tenant; each tenant using the app has a service principal shown under Enterprise applications; managed identities are service principals tied to Azure resources">
  <defs>
    <marker id="fentarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="currentColor"/>
    </marker>
  </defs>
  <text x="20" y="28" class="label-accent">Human identities</text>
  <rect class="box" x="20" y="40" width="190" height="60" rx="8"/>
  <text x="115" y="66" text-anchor="middle">Member user</text>
  <text x="115" y="88" text-anchor="middle" class="muted">your staff</text>
  <rect class="box" x="20" y="115" width="190" height="60" rx="8"/>
  <text x="115" y="141" text-anchor="middle">Guest user (B2B)</text>
  <text x="115" y="163" text-anchor="middle" class="muted">UserType = Guest</text>
  <rect class="box" x="20" y="190" width="190" height="60" rx="8"/>
  <text x="115" y="216" text-anchor="middle">Group</text>
  <text x="115" y="238" text-anchor="middle" class="muted">assign access once</text>
  <line class="divider" x1="240" y1="20" x2="240" y2="360"/>
  <text x="265" y="28" class="label-accent">Workload identities</text>
  <rect class="box-accent" x="265" y="40" width="270" height="70" rx="8"/>
  <text x="400" y="66" text-anchor="middle" class="label-accent">App registration</text>
  <text x="400" y="90" text-anchor="middle" class="muted">application object, home tenant</text>
  <rect class="box" x="265" y="150" width="270" height="70" rx="8"/>
  <text x="400" y="176" text-anchor="middle">Service principal (tenant A)</text>
  <text x="400" y="200" text-anchor="middle" class="muted">"Enterprise applications"</text>
  <rect class="box" x="560" y="150" width="280" height="70" rx="8"/>
  <text x="700" y="176" text-anchor="middle">Service principal (tenant B)</text>
  <text x="700" y="200" text-anchor="middle" class="muted">same app, other tenant</text>
  <line class="line-accent" x1="400" y1="110" x2="400" y2="148" marker-end="url(#fentarrow)"/>
  <line class="line-accent" x1="470" y1="110" x2="680" y2="148" marker-end="url(#fentarrow)"/>
  <text x="560" y="128" class="muted">template creates</text>
  <rect class="box-ok" x="265" y="260" width="270" height="70" rx="8"/>
  <text x="400" y="286" text-anchor="middle">Managed identity</text>
  <text x="400" y="310" text-anchor="middle" class="muted">service principal, no secret</text>
  <rect class="box-ok" x="560" y="260" width="280" height="70" rx="8"/>
  <text x="700" y="286" text-anchor="middle">Agent identity (Agent ID)</text>
  <text x="700" y="310" text-anchor="middle" class="muted">special service principal</text>
</svg>
```

## What actually matters

**1. Tenant and directory.** A **tenant** is your organisation's dedicated instance of Entra ID. Every Azure subscription trusts exactly one tenant. Entra ID was called **Azure Active Directory (Azure AD)** until 2023 — old docs and exam distractors still use that name. It is **not** Windows Server Active Directory: no OUs, no Group Policy, no Kerberos domain controllers you manage.

**2. Users, guests and groups.**

- **Member** users are your staff. **Guest** users are people from outside, invited through **B2B collaboration**; they sign in with their own home account and appear with `UserType = Guest`. Guests have restricted directory permissions by default (set under **External Identities > External collaboration settings**). A guest who has not accepted the invitation shows an invitation state of *Pending acceptance*.
- **Groups**: **Security** groups grant access to resources; **Microsoft 365** groups add a shared mailbox, calendar and SharePoint site. Membership is **assigned** (added by hand) or **dynamic** (a rule such as `department -eq "Finance"`; needs Entra ID P1). Assign roles and app access to groups, not to individuals.

**3. App registration vs enterprise application — the exam's favourite confusion.**

| | App registration | Enterprise application |
|---|---|---|
| Underlying object | ==@Application object== | ==@Service principal== |
| How many | One, in the app's **home tenant** | One **per tenant** where the app is used (including the home tenant) |
| What it defines | The app's identity globally: client ID, redirect URIs, the permissions it *requests*, its credentials (secrets/certificates), roles it exposes | What the app can **actually do here**: who may use it, which permissions were **granted (consented)**, SSO, Conditional Access target |
| Where | **Entra admin center > App registrations** | **Entra admin center > Enterprise applications** |

The application object is the **template**; the service principal is the **instance**. A SaaS app from another company (say, a third-party CRM) has its app registration in *their* tenant and only a service principal in *yours*. When an illicit app gets consent in your tenant, you clean it up under **Enterprise applications**.

There are three service principal types: **application** (made from an app registration), **managed identity**, and **legacy** (old apps made before app registrations). The Graph object for all of them is `servicePrincipal`.

**4. Credentials for an app.** An app registration can hold a **client secret** (a password — weakest, expires, gets leaked into code), a **certificate** (stronger), or a **federated identity credential** (trust a token from GitHub Actions, Kubernetes or another cloud — no stored secret at all). This last option is **workload identity federation**.

**5. Managed identities — identities without secrets.** A managed identity is a special service principal that Azure creates and whose credentials Azure rotates. Your code asks the local Azure endpoint for a token; nobody ever sees a password. No licence is needed.

| | System-assigned | User-assigned |
|---|---|---|
| Created as | A setting on one resource (VM, App Service, Function…) | A standalone Azure resource (`Microsoft.ManagedIdentity/userAssignedIdentities`) |
| Lifecycle | ==@Deleted with the resource== | Lives until you delete it |
| Sharing | One resource only | Can be attached to **many** resources; a resource can have several |
| Good for | One workload, simple | Many instances needing the same access; pre-granting access before the resource exists |

You then grant the managed identity an **Azure role** (for example *Key Vault Secrets User* on a vault) just like a user.

**6. Workload identities.** Microsoft's umbrella term for **applications, service principals and managed identities** — any non-human identity used by software. **Microsoft Entra Workload ID** (a paid add-on, licensed per workload identity) adds Conditional Access, ID Protection risk and access reviews for them. **Agent identities** from **Microsoft Entra Agent ID** are a newer, special kind of service principal built for AI agents, created from an **agent identity blueprint** and tied to a human **sponsor**.

## Where it lives

- **Entra admin center > Identity > Users** (all users; filter *User type = Guest*), **> Groups**.
- **Entra admin center > Identity > Applications > App registrations** and **> Enterprise applications**.
- **Azure portal > {resource} > Settings > Identity** — turn on a **system-assigned** identity or attach a **user-assigned** one. **Azure portal > Managed Identities** — create user-assigned identities.
- CLI: `az ad app list`, `az ad sp list`, `az identity create -g rg-lab -n id-app`.

![Relationship between app object and service principal object](https://learn.microsoft.com/en-us/entra/identity-platform/media/app-objects-and-service-principals/application-objects-relationship.svg){One application object in the home tenant; one service principal in every tenant that uses the app}

![Diagram that shows different types of machine and human identities](https://learn.microsoft.com/en-us/entra/workload-id/media/workload-identities-overview/identity-types.svg){Human identities versus machine identities — workload identities are the software kind}

## Exam traps

- ==!Deleting an app registration's service principal in your tenant does not delete the app== in its home tenant. And deleting the app registration removes the service principal in the home tenant only.
- ==!Permissions are requested on the app registration but granted on the service principal.== Reviewing consent = Enterprise applications > Permissions.
- ==+Prefer managed identity over a client secret== whenever the code runs on Azure. Prefer **workload identity federation** for GitHub Actions or workloads outside Azure.
- ==!A system-assigned identity cannot be shared== and disappears with its resource. "Several VMs need the same identity" → user-assigned.
- A guest is still a full identity in your tenant. Roles and app access can be assigned to it — which is why guests need review.

## Skip

- **Skip** the Microsoft Graph application object property list (`signInAudience`, `requiredResourceAccess` JSON). Know the concepts, not the schema.
- **Skip** dynamic group rule syntax beyond "rules need P1".
- **Skip** B2B redemption internals (invitation URLs, redemption order) and External ID for customers (CIAM). The exam cares that guests exist and need governing.
- **Skip** the Entra product-family tour (Verified ID, Internet Access) in the "What is Microsoft Entra" article — Private Access gets its own objective.

## Screenshots

![App registrations blade](https://learn.microsoft.com/en-us/entra/identity-platform/media/app-objects-and-service-principals/app-registrations-blade.png){App registrations — application objects whose home is this tenant}

![Enterprise apps blade](https://learn.microsoft.com/en-us/entra/identity-platform/media/app-objects-and-service-principals/enterprise-apps-blade.png){Enterprise applications — service principals in this tenant, including other companies' apps}

![Screen shot that shows the Managed application in local directory option in the overview](https://learn.microsoft.com/en-us/entra/identity-platform/media/app-objects-and-service-principals/find-service-principal.png){From an app registration, "Managed application in local directory" jumps to its service principal}

![Diagram showing how your own developed apps, preintegrated apps, and on-premises apps can be used as enterprise apps](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/media/what-is-application-management/app-management-overview.png){Enterprise apps can be your own apps, gallery SaaS apps or on-premises apps}

![Diagram that shows pain points in securing workload identities](https://learn.microsoft.com/en-us/entra/workload-id/media/workload-identities-overview/pain-points.png){Why workload identities are hard to secure: many credentials, unclear owners, no lifecycle}

![Diagram showing B2B collaboration users](https://learn.microsoft.com/en-us/entra/external-id/media/user-properties/table-user-properties.png){B2B user types: external guest, external member, internal guest, internal member}

![Screenshot of user profile before redemption](https://learn.microsoft.com/en-us/entra/external-id/media/user-properties/before-redemption.png){A guest before accepting the invitation}

![Screenshot of user profile after redemption](https://learn.microsoft.com/en-us/entra/external-id/media/user-properties/after-redemption-state-1.png){The same guest after redemption}

![Screenshot showing how to add a User type filter for guests](https://learn.microsoft.com/en-us/entra/external-id/media/user-properties/add-guest-filter.png){Filtering the user list by user type}

![Screenshot showing the filter for guest users](https://learn.microsoft.com/en-us/entra/external-id/media/user-properties/filter-guest-users.png){All guests in the tenant}

![Screenshot showing the External users option in the user settings](https://learn.microsoft.com/en-us/entra/external-id/media/user-properties/remove-guest-limitations.png){User settings that control what external users can see in the directory}
