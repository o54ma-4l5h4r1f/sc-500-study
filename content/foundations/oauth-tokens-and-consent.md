+++
slug = "oauth-tokens-and-consent"
title = "OAuth 2.0, OpenID Connect, tokens and consent — for non-developers"
summary = "Apps never get your password: they get short-lived tokens from Entra ID. Learn the four OAuth roles, access vs ID vs refresh tokens, scopes, delegated vs application permissions, user vs admin consent, and why an illicit consent grant survives a password reset."
order = 5
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/entra/identity-platform/v2-protocols",
  "https://learn.microsoft.com/en-us/entra/identity-platform/security-tokens",
  "https://learn.microsoft.com/en-us/entra/identity-platform/access-tokens",
  "https://learn.microsoft.com/en-us/entra/identity-platform/refresh-tokens",
  "https://learn.microsoft.com/en-us/entra/identity-platform/permissions-consent-overview",
  "https://learn.microsoft.com/en-us/entra/identity-platform/application-consent-experience",
  "https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/user-admin-consent-overview",
  "https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/configure-user-consent",
  "https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/grant-admin-consent",
  "https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/configure-admin-consent-workflow",
  "https://learn.microsoft.com/en-us/defender-office-365/detect-and-remediate-illicit-consent-grants",
]
captured = 2026-11-15
related_objectives = ["D1.1.5", "D1.1.4", "D1.1.6", "D3.1.4", "D3.1.6", "D3.3.5", "D3.3.9", "D3.1.7"]
+++

## In one sentence

**OAuth 2.0** is the protocol that lets an app get a limited, time-boxed **token** to call an API for you, without ever seeing your password; **OpenID Connect (OIDC)** sits on top and adds a token that proves *who* signed in.

## Why it exists

Think of a hotel. At reception you prove who you are once (passport = your password and MFA). Reception gives you a **key card**. The card opens your room and the gym, not other rooms, and it stops working at checkout. If you lose it, nobody learns your passport number.

Tokens are those key cards. The app (a mail client, a CRM, a script) shows the card to an API such as **Microsoft Graph** — the API behind Microsoft 365 and Entra data. **Consent** is the moment someone agrees to what the card may open.

As an analyst you may have triaged "suspicious OAuth app" alerts. Here you learn the machinery behind them — and the settings that stop them.

## Mental model

```svg
<svg viewBox="0 0 860 330" role="img" aria-label="OAuth roles: the resource owner (user) consents; the client app requests a token from the authorization server (Entra ID); the authorization server issues access, ID and refresh tokens; the client presents the access token to the resource server (an API such as Microsoft Graph)">
  <defs>
    <marker id="foauarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="currentColor"/>
    </marker>
  </defs>
  <rect class="box" x="20" y="30" width="200" height="70" rx="8"/>
  <text x="120" y="58" text-anchor="middle">Resource owner</text>
  <text x="120" y="82" text-anchor="middle" class="muted">the user</text>
  <rect class="box" x="20" y="200" width="200" height="70" rx="8"/>
  <text x="120" y="228" text-anchor="middle">Client</text>
  <text x="120" y="252" text-anchor="middle" class="muted">the app or script</text>
  <rect class="box-accent" x="330" y="30" width="230" height="70" rx="8"/>
  <text x="445" y="58" text-anchor="middle" class="label-accent">Authorization server</text>
  <text x="445" y="82" text-anchor="middle" class="muted">Microsoft Entra ID</text>
  <rect class="box" x="640" y="200" width="200" height="70" rx="8"/>
  <text x="740" y="228" text-anchor="middle">Resource server</text>
  <text x="740" y="252" text-anchor="middle" class="muted">API, e.g. Graph</text>
  <line class="line" x1="120" y1="100" x2="120" y2="198" marker-end="url(#foauarrow)"/>
  <text x="130" y="155" class="muted">1 uses app</text>
  <line class="line-accent" x1="220" y1="215" x2="400" y2="102" marker-end="url(#foauarrow)"/>
  <text x="235" y="150" class="muted">2 sign in + consent</text>
  <line class="line-accent" x1="470" y1="102" x2="290" y2="230" marker-end="url(#foauarrow)"/>
  <text x="400" y="185">3 tokens</text>
  <line class="line-accent" x1="220" y1="250" x2="638" y2="250" marker-end="url(#foauarrow)"/>
  <text x="430" y="290" text-anchor="middle">4 access token as bearer: "Authorization: Bearer ..."</text>
</svg>
```

## What actually matters

**1. The three tokens.**

| Token | Purpose | Who reads it | Default lifetime |
|---|---|---|---|
| **Access token** | Lets the client call one API (the *audience*) | The API | Random ==@60–90 minutes== (75 average) |
| **ID token** | Proves who signed in (OIDC) | The client app | — |
| **Refresh token** | Gets new access tokens without a new sign-in | Entra ID only (encrypted) | ==@90 days== (24 hours for single-page apps) |

Tokens are **JSON Web Tokens (JWTs)**: signed bundles of **claims** — who the subject is, which tenant, which app asked, which API it is for (`aud`), what permissions it carries (`scp` or `roles`). Anyone holding an access token can use it until it expires — so stolen tokens are a real attack path (token theft), even with MFA.

**2. Scopes are the permissions inside the token.** A **scope** is a named permission an API exposes, such as `User.Read` or `Mail.Send` on Microsoft Graph. The app asks for scopes; consent grants them; the access token carries them.

**3. Delegated vs application permissions — the core distinction.**

| | Delegated permission | Application permission |
|---|---|---|
| Other name | Scope | ==@App role== / app-only |
| User signed in? | Yes — app acts **on behalf of** the user | No — app acts **as itself** (daemon, backup, automation) |
| What it can reach | Only what **both** the permission **and** the user allow | **Everything** the permission covers, tenant-wide |
| Token claim | `scp` | `roles` |
| Who can consent | User (low-risk perms) or admin | ==!Admin only== |

Example: `Files.Read.All` **delegated** reads only files the signed-in user can open. `Files.Read.All` **application** reads every file in the tenant. Same name, very different blast radius.

**4. Consent: who says yes.**

- **User consent** — a user approves an app for **their own** data, through the consent prompt. Allowed only for permissions that don't need admin consent, and only if tenant settings allow it.
- **Admin consent** — an admin approves for the **whole organisation** (*tenant-wide admin consent*). Needed for all application permissions and for high-privilege delegated permissions such as `Directory.ReadWrite.All`.
- **Who can grant tenant-wide admin consent:** ==@Privileged Role Administrator== (any permission, any API); **Cloud Application Administrator**, **Application Administrator** or **AI Administrator** (anything *except* Microsoft Graph application permissions). One wrinkle: approving a *request* for Graph application permissions through the admin consent workflow queue is documented as Global Administrator only.
- **Admin consent workflow** — when users can't consent, they can **request** approval; designated reviewers approve or deny.

**5. User consent settings (tenant-wide).** Four choices:

1. **Do not allow user consent** — admins approve every app.
2. ==+Allow user consent for apps from verified publishers, for selected permissions== (Microsoft's recommendation) — only low-impact permissions you classify, only verified publishers or apps registered in your tenant.
3. **Let Microsoft manage your consent settings** — the **default for new tenants**: users may consent to user-consentable delegated permissions except a Microsoft-maintained list of sensitive ones (`Mail.Read`, `Files.Read.All` and more).
4. **Allow user consent for apps** — any user, any non-admin permission, any app. ==!Riskiest.==

Changing these settings needs **Privileged Role Administrator** via Graph/PowerShell; in the Entra admin center the page says **Global Administrator**.

**6. Illicit consent grant — the attack.** The attacker registers an app in *their* tenant, asking for scopes like `Mail.Read` or `Files.Read.All`. They phish a user into clicking **Accept**. Your tenant now holds a service principal for that app with a valid grant. The app reads data with tokens — no password involved. ==!Resetting the password or enforcing MFA does not stop it==, because the app is not the user. Fix: revoke the grant / remove the service principal under **Enterprise applications**, then tighten user consent settings. Detection: the `Consent to application` audit event (check `IsAdminConsent`).

## Where it lives

- **Entra admin center > Identity > Applications > Enterprise applications > Consent and permissions > User consent settings** — the three options above, plus **Permission classifications**.
- **Enterprise applications > {app} > Security > Permissions** — see granted permissions; **Grant admin consent for {tenant}**; review and revoke.
- **Enterprise applications > Consent and permissions > Admin consent settings** — turn on the admin consent workflow and choose reviewers.
- **App registrations > {app} > API permissions** — what the app *requests*.
- Defender portal **Audit** (Purview audit) or Entra **Audit logs** — `Consent to application` events.

![Building blocks of the consent prompt](https://learn.microsoft.com/en-us/entra/identity-platform/media/application-consent-experience/consent_prompt.png){The consent prompt: app name, publisher (verified or not), the permissions requested, Accept/Cancel — this screen is where an illicit consent grant happens}

![Screenshot of the User consent settings pane](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/media/configure-user-consent/setting-for-all-users.png){User consent settings: the three options}

## Exam traps

- ==!A refresh token is not sent to the API.== Only the access token is. The refresh token goes back to Entra ID.
- ==!Application permissions cannot be user-consented.== If a question says "users must not need admin involvement" and the app is a daemon, that is impossible.
- ==+Least privilege:== choose delegated over application when a user is present; choose the narrowest scope (`Mail.Read` not `Mail.ReadWrite`).
- Cloud Application Administrator can't grant **Microsoft Graph application** permissions — that needs Privileged Role Administrator.
- An OAuth app attack is remediated in **Enterprise applications** (the service principal), not App registrations — the attacker's app registration lives in their tenant.
- OIDC is about **sign-in (authentication)**; OAuth is about **access (authorization)**. SAML is an older sign-in protocol many enterprise SaaS apps still use.

## Skip

- **Skip** the individual OAuth flows (authorization code with PKCE, client credentials, on-behalf-of, device code) beyond: *client credentials = app-only, no user*.
- **Skip** JWT claim-by-claim reference tables and token signature validation. Know `aud`, `scp`, `roles`.
- **Skip** configurable token lifetime policies and continuous access evaluation internals; Conditional Access session controls cover the testable part.
- **Skip** the PowerShell script walkthrough in the illicit consent article; know the attack, the audit event and the remediation location.

## Screenshots

![Screenshot of a diagram showing the roles in OAuth 2.0](https://learn.microsoft.com/en-us/entra/identity-platform/media/v2-flows/protocols-roles.svg){The four OAuth roles: authorization server, client, resource owner, resource server}

![Image shows illustration of access scenarios](https://learn.microsoft.com/en-us/entra/identity-platform/media/permissions-consent-overview/access-scenarios.png){Delegated access (with a user) versus app-only access (no user)}

![Consent prompt for scenario 1a](https://learn.microsoft.com/en-us/entra/identity-platform/media/application-consent-experience/consent_prompt_1a.png){A user consenting to an app for their own data}

![Screenshot that shows the traditional consent prompt](https://learn.microsoft.com/en-us/entra/identity-platform/media/application-consent-experience/consent_prompt_1b.png){Traditional consent prompt}

![Screenshot of the consent prompt telling the user to ask an admin for access to the app](https://learn.microsoft.com/en-us/entra/identity-platform/media/application-consent-experience/consent_prompt_2b.png){When the user can't consent: "Need admin approval"}

![Consent prompt for scenario 3a](https://learn.microsoft.com/en-us/entra/identity-platform/media/application-consent-experience/consent_prompt_3a.png){An admin's prompt includes "Consent on behalf of your organization"}

![Screenshot of explicit admin consent through the Microsoft Entra admin center](https://learn.microsoft.com/en-us/entra/identity-platform/media/consent-framework/grant-consent.png){Granting admin consent from the admin center}

![Screenshot shows how to grant tenant-wide admin consent](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/media/grant-tenant-wide-admin-consent/grant-tenant-wide-admin-consent.png){Enterprise application > Permissions > Grant admin consent}

![Screenshot of configure admin consent workflow settings](https://learn.microsoft.com/en-us/entra/identity/enterprise-apps/media/configure-admin-consent-workflow/enable-admin-consent-workflow.png){Admin consent workflow: users request, reviewers approve}
