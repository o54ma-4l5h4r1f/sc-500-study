+++
slug = "conditional-access-basics"
title = "Conditional Access basics: signals, decision, enforcement"
summary = "Conditional Access is Entra ID's if-then engine: IF this user, app, location, device and risk THEN block, or grant with MFA/compliant device, plus session limits. Learn assignments, grant and session controls, report-only mode, named locations and the Entra ID P1 requirement."
order = 7
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/entra/identity/conditional-access/overview",
  "https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-policies",
  "https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-report-only",
  "https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-assignment-network",
  "https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-grant",
  "https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-session",
  "https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-cloud-apps",
  "https://learn.microsoft.com/en-us/entra/fundamentals/security-defaults",
  "https://learn.microsoft.com/en-us/entra/fundamentals/licensing",
]
captured = 2026-11-15
related_objectives = ["D1.1.2", "D1.1.3", "D1.1.1", "D3.1.4", "D2.3.5", "D1.2.3"]
+++

## In one sentence

**Conditional Access** is Entra ID's policy engine: at every sign-in it reads **signals** (who, which app, from where, on what device, how risky), makes a **decision** (block, or allow if conditions are met), and **enforces** it — "IF a user opens the Azure portal from outside the office, THEN require phishing-resistant MFA."

## Why it exists

A password alone proves little. Blanket MFA for everything annoys users and still treats a risky sign-in from an unknown country like a routine one from the office.

Think of an airport. Everyone shows a boarding pass. Staff entering airside also show a badge. Anyone flagged as suspicious goes to a second check. Nobody boards a flight to a closed airport. The rules depend on **who** you are, **where** you are going and **what** looks wrong. Conditional Access is that checkpoint logic for every Microsoft cloud sign-in.

As an analyst you saw the results: sign-in logs with "Conditional Access: Success/Failure", or risky sign-ins. Now you write the rules.

## Mental model

```svg
<svg viewBox="0 0 880 330" role="img" aria-label="Conditional Access: signals (user or agent, target resource, network, device, client app, risk) feed an if-then policy; the decision is block or grant with controls; session controls then limit the session">
  <defs>
    <marker id="fcaarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" fill="currentColor"/>
    </marker>
  </defs>
  <text x="130" y="28" text-anchor="middle" class="label-accent">IF (assignments)</text>
  <rect class="box" x="20" y="40" width="220" height="250" rx="8"/>
  <text x="130" y="70" text-anchor="middle">users / groups / agents</text>
  <text x="130" y="100" text-anchor="middle">target resources (apps)</text>
  <text x="130" y="130" text-anchor="middle">network (named loc.)</text>
  <text x="130" y="160" text-anchor="middle">device platform / filter</text>
  <text x="130" y="190" text-anchor="middle">client apps</text>
  <text x="130" y="220" text-anchor="middle">sign-in / user risk</text>
  <text x="130" y="265" text-anchor="middle" class="muted">all combined with AND</text>
  <line class="line-accent" x1="240" y1="165" x2="318" y2="165" marker-end="url(#fcaarrow)"/>
  <text x="440" y="28" text-anchor="middle" class="label-accent">THEN (grant)</text>
  <rect class="box-risk" x="320" y="40" width="240" height="70" rx="8"/>
  <text x="440" y="70" text-anchor="middle" class="label-risk">Block access</text>
  <text x="440" y="94" text-anchor="middle" class="muted">stops here</text>
  <rect class="box-ok" x="320" y="130" width="240" height="160" rx="8"/>
  <text x="440" y="156" text-anchor="middle">Grant, requiring:</text>
  <text x="440" y="182" text-anchor="middle">MFA / auth strength</text>
  <text x="440" y="206" text-anchor="middle">compliant device</text>
  <text x="440" y="230" text-anchor="middle">hybrid joined device</text>
  <text x="440" y="254" text-anchor="middle">approved app, ToU...</text>
  <text x="440" y="278" text-anchor="middle" class="muted">all or one of selected</text>
  <line class="line-accent" x1="560" y1="210" x2="638" y2="210" marker-end="url(#fcaarrow)"/>
  <text x="750" y="28" text-anchor="middle" class="label-accent">AND (session)</text>
  <rect class="box" x="640" y="130" width="220" height="160" rx="8"/>
  <text x="750" y="160" text-anchor="middle">sign-in frequency</text>
  <text x="750" y="186" text-anchor="middle">persistent browser</text>
  <text x="750" y="212" text-anchor="middle">app enforced limits</text>
  <text x="750" y="238" text-anchor="middle">CA App Control</text>
  <text x="750" y="264" text-anchor="middle">CAE, token protection</text>
</svg>
```

## What actually matters

**1. Licence.** Conditional Access needs ==@Microsoft Entra ID P1== (included in Microsoft 365 E3/E5, Business Premium, EMS E3). **Risk-based** conditions (sign-in risk, user risk) need ==@Entra ID P2== because the risk comes from **Entra ID Protection**. Conditional Access for AI agents (Agent ID) also needs a **Microsoft Agent 365** licence. If a licence lapses, existing policies are **not** deleted; they can be viewed and deleted but no longer edited. Without P1, the free option is **security defaults** — a fixed, all-or-nothing baseline (MFA registration, MFA for admins, block legacy authentication) that you must turn off before using Conditional Access.

**2. Assignments — the IF.**

- **Users / groups / agents**: include and exclude users, groups, directory roles, guests; workload identities with a Workload ID licence. ==!Always exclude your break-glass (emergency access) accounts.==
- **Target resources**: cloud apps (for example *Windows Azure Service Management API* = the Azure portal, CLI and PowerShell), **user actions** (register security info, register or join devices), or **authentication contexts** (a tag an app or PIM can request for a sensitive step).
- **Network**: **named locations** — IPv4/IPv6 ranges or **countries/regions**, optionally marked **trusted**. Location comes from the public IP (or Authenticator GPS).
- **Conditions**: sign-in risk, user risk, device platforms, client apps (browser vs mobile/desktop), filter for devices.

All assignments in one policy are combined with **AND**.

**3. Access controls — the THEN.** **Grant** is either **Block access** or **Grant access** with one or more requirements: *require multifactor authentication*, *require authentication strength* (for example phishing-resistant MFA), *require device to be marked as compliant* (Intune), *require Microsoft Entra hybrid joined device*, *require approved client app*, *require app protection policy*, *require password change*, *require terms of use*. Choose **Require all the selected controls** (default) or **Require one of the selected controls**.

**Session** controls shape the session after sign-in: **app enforced restrictions** (Exchange Online and SharePoint Online only), **Conditional Access App Control** (Defender for Cloud Apps — block download, monitor session), **sign-in frequency**, **persistent browser session**, **continuous access evaluation**, **disable resilience defaults**.

**4. Several policies apply at once.** Every matching policy must be satisfied. A **Block** in any matching policy wins — evaluation stops. There is no priority order to configure.

**5. Report-only mode.** A policy state between Off and On. The policy is **evaluated and logged but not enforced** — results show as *Report-only: Success / Failure / User action required / Not applied* on the sign-in log's **Report-only** tab and in the **policy impact** view. ==+Deploy new policies in report-only first.== Report-only doesn't work for policies scoped to **user actions**.

**6. Policies evaluate at token issuance.** A user newly added to a targeted group is caught at their next token, not instantly — which is why PIM activation can require an authentication context.

## Where it lives

- **Microsoft Entra admin center > Entra ID > Conditional Access > Policies** — create, set state (**Report-only**, **On**, **Off**).
- **Conditional Access > Named locations** — IP ranges and countries. Needs at least **Conditional Access Administrator**.
- **Conditional Access > Overview / Coverage** and **Insights and reporting** — gaps and impact.
- **Monitoring > Sign-in logs > {sign-in} > Conditional Access / Report-only** tabs — what each policy did.
- **Role to manage policies:** ==@Conditional Access Administrator== (or Security Administrator).

![Diagram showing concept of Conditional Access signals plus decision to enforce organizational policy](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/common-conditional-access-media/conditional-access-signal-decision-enforcement.png){Signals → decision → enforcement}

![Blank Conditional Access policy](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/concept-conditional-access-policies/conditional-access-blank-policy.png){A new policy: assignments on the left, access controls below, state at the bottom}

## Exam traps

- ==!Conditional Access runs after first-factor authentication.== It does not stop a password spray from validating passwords; it decides what happens next.
- ==!Report-only is not "audit mode for everything"== — it can't evaluate user-action policies, and it never prompts the user.
- ==+"Require MFA only outside the office"== = include **Any location**, exclude the **trusted** named location.
- Device platform comes from the user agent — unverified. Don't build a security boundary on platform alone.
- Risk-based policies need **P2**; plain Conditional Access needs **P1**. Security defaults and Conditional Access can't run together.
- The Azure portal, CLI and PowerShell are covered by the **Windows Azure Service Management API** target, a common "protect admin access to Azure" answer.

## Skip

- **Skip** the Zero Trust policy-engine marketing diagram's partner signal list and the coverage tab's app-by-app breakdown.
- **Skip** the grant-control prompt order and resilience defaults internals.
- **Skip** MFA trusted IPs (legacy, IPv4 only) — named locations replace it.
- **Skip** Global Secure Access compliant network and token protection detail here; Private Access and authentication methods have their own objectives.
- **Skip** policy templates as a list; know that templates exist and start in report-only.

## Screenshots

![Diagram that shows Conditional Access as the Zero Trust policy engine](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/overview/conditional-access-central-policy-engine-zero-trust.png){Conditional Access as the central policy engine for Zero Trust}

![Screenshot of the Conditional Access overview page](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/overview/conditional-access-overview.png){Conditional Access overview}

![Screenshot of the Conditional Access coverage tab](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/overview/conditional-access-coverage-tab.png){Coverage: which apps have no policy}

![Screenshot of the Conditional Access policies list page](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/overview/conditional-access-policies-list.png){Policy list with states}

![Screenshot of Conditional Access policy impact example](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/concept-conditional-access-report-only/policy-impact-example-report-only.png){Policy impact view for a report-only policy}

![Screenshot showing the report-only tab in a sign-in log](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/concept-conditional-access-report-only/report-only-detail-in-sign-in-log.png){Sign-in log Report-only tab: what would have happened}

![Screenshot showing the network assignment condition in a Conditional Access policy](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/concept-assignment-network/network-assignment.png){Network assignment: any location, trusted, or selected named locations}

![Screenshot of named locations in the Microsoft Entra admin center](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/concept-assignment-network/named-locations.png){Named locations list}

![Screenshot of creating a new location using countries](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/concept-assignment-network/new-location-countries.png){A countries-based named location}

![Screenshot of a Conditional Access policy grant control requiring phishing-resistant multifactor authentication](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/concept-conditional-access-grant/conditional-access-grant.png){Grant controls — here an authentication strength requiring phishing-resistant MFA}

![Screenshot of a Conditional Access policy session controls](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/concept-conditional-access-session/conditional-access-session.png){Session controls pane}

![Screenshot of CAE settings in a new Conditional Access policy](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/concept-conditional-access-session/continuous-access-evaluation-session-controls.png){Customize continuous access evaluation}

![Screenshot of a Conditional Access policy and the target resources panel](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/concept-conditional-access-cloud-apps/conditional-access-cloud-apps-or-actions.png){Target resources: cloud apps, user actions or authentication context}

![Screenshot showing the management of authentication contexts](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/concept-conditional-access-cloud-apps/conditional-access-authentication-context-get-started.png){Authentication contexts — tags for sensitive actions}

![Screenshot showing how to add a Conditional Access authentication context to a policy](https://learn.microsoft.com/en-us/entra/identity/conditional-access/media/concept-conditional-access-cloud-apps/conditional-access-authentication-context-in-policy.png){Targeting an authentication context in a policy}

![Screenshot of the Microsoft Entra admin center with the toggle to enable security defaults](https://learn.microsoft.com/en-us/entra/fundamentals/media/security-defaults/security-defaults-entra-admin-center.png){Security defaults — the free, fixed alternative; turn it off to use Conditional Access}

![Screenshot showing an example of the Approve sign in request window with a number to enter](https://learn.microsoft.com/en-us/entra/fundamentals/media/security-defaults/approve-sign-in-request.png){Security defaults MFA prompt with number matching in Microsoft Authenticator}
