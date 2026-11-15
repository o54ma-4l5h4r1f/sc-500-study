+++
slug = "purview-basics"
title = "Microsoft Purview basics (portal, sensitivity labels, DLP, Audit, DSPM for AI, SharePoint oversharing)"
summary = "What Microsoft Purview is and how its portal is laid out; what sensitivity labels, data loss prevention and Audit (Standard vs Premium) do; what Data Security Posture Management (and its classic 'DSPM for AI') adds for Copilot and agents; and why SharePoint oversharing is the AI-era data problem, with the controls that fix it."
order = 26
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/purview/purview",
  "https://learn.microsoft.com/en-us/purview/purview-portal",
  "https://learn.microsoft.com/en-us/purview/sensitivity-labels",
  "https://learn.microsoft.com/en-us/purview/dlp-learn-about-dlp",
  "https://learn.microsoft.com/en-us/purview/audit-solutions-overview",
  "https://learn.microsoft.com/en-us/purview/data-security-posture-management-learn-about",
  "https://learn.microsoft.com/en-us/purview/data-security-posture-management-application-card",
  "https://learn.microsoft.com/en-us/purview/dspm-for-ai",
  "https://learn.microsoft.com/en-us/purview/ai-m365-copilot",
  "https://learn.microsoft.com/en-us/copilot/plan/architecture-data-protection-auditing",
  "https://learn.microsoft.com/en-us/sharepoint/restricted-content-discovery",
]
captured = 2026-11-15
related_objectives = ["D3.1.1", "D3.1.2", "D3.1.11", "D4.2.10"]
+++

## In one sentence

**Microsoft Purview** is Microsoft's family of data security and compliance tools: it labels sensitive content, stops it leaking, records who did what, and — new for AI — shows where Copilot and agents could expose data that is shared too widely.

## Why it exists

Defender asks "is an attacker inside?" Purview asks "is our *data* safe, and can we prove what happened to it?"

AI made that second question urgent. Microsoft 365 Copilot and agents answer questions by searching everything **the user is allowed to open**. Most companies have years of SharePoint sites shared with "Everyone except external users" by accident. Before AI, nobody found those files. Now a junior employee asks Copilot "what are the salaries in finance?" and Copilot faithfully finds the spreadsheet. ==!Copilot did not break any permission — the permission was already wrong.== That is **oversharing**, and it is what D3.1.1 and D3.1.2 are about.

Think of a library where many private reading rooms were accidentally left unlocked. Nobody noticed because nobody walked the corridors. Then you hire a tireless assistant who walks into every unlocked room on request. Purview is how you find the unlocked rooms, put "staff only" signs on sensitive books, and log every visit.

## Mental model

```svg
<svg viewBox="0 0 900 360" role="img" aria-label="Purview controls around AI: sensitivity labels classify and protect content; DLP stops sensitive content leaving, including into Copilot; DSPM finds oversharing and risky AI use; Audit records every interaction; SharePoint controls fix permissions at the source">
  <defs>
    <marker id="pvsarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="line-accent"/>
    </marker>
  </defs>
  <rect class="box" x="15" y="120" width="190" height="110" rx="8"/>
  <text x="110" y="150" text-anchor="middle">SharePoint, OneDrive</text>
  <text x="110" y="172" text-anchor="middle">Exchange, Teams</text>
  <text x="110" y="205" text-anchor="middle" class="muted">your data</text>
  <rect class="box-accent" x="355" y="120" width="190" height="110" rx="8"/>
  <text x="450" y="150" text-anchor="middle" class="label-accent">Copilot / agent</text>
  <text x="450" y="175" text-anchor="middle">grounds answers in</text>
  <text x="450" y="197" text-anchor="middle">what the user can open</text>
  <rect class="box" x="695" y="135" width="190" height="80" rx="8"/>
  <text x="790" y="170" text-anchor="middle">User sees answer</text>
  <text x="790" y="192" text-anchor="middle" class="muted">prompt + response</text>
  <line class="line-accent" x1="205" y1="175" x2="353" y2="175" marker-end="url(#pvsarrow)"/>
  <line class="line-accent" x1="545" y1="175" x2="693" y2="175" marker-end="url(#pvsarrow)"/>
  <rect class="box-ok" x="15" y="20" width="190" height="75" rx="8"/>
  <text x="110" y="48" text-anchor="middle" class="label-accent">Sensitivity labels</text>
  <text x="110" y="72" text-anchor="middle" class="muted">classify, encrypt</text>
  <rect class="box-ok" x="355" y="20" width="190" height="75" rx="8"/>
  <text x="450" y="48" text-anchor="middle" class="label-accent">DLP</text>
  <text x="450" y="72" text-anchor="middle" class="muted">block labelled items</text>
  <rect class="box-ok" x="695" y="20" width="190" height="75" rx="8"/>
  <text x="790" y="48" text-anchor="middle" class="label-accent">Audit</text>
  <text x="790" y="72" text-anchor="middle" class="muted">who asked what</text>
  <rect class="box-risk" x="15" y="260" width="360" height="85" rx="8"/>
  <text x="195" y="288" text-anchor="middle" class="label-risk">Oversharing at the source</text>
  <text x="195" y="312" text-anchor="middle">fix: site permissions,</text>
  <text x="195" y="332" text-anchor="middle">Restricted Content Discovery</text>
  <rect class="box-accent" x="415" y="260" width="470" height="85" rx="8"/>
  <text x="650" y="288" text-anchor="middle" class="label-accent">DSPM (for AI)</text>
  <text x="650" y="312" text-anchor="middle">finds overshared sites, risky AI use,</text>
  <text x="650" y="332" text-anchor="middle">one-click policies</text>
</svg>
```

![Solution areas in Microsoft Purview.](https://learn.microsoft.com/en-us/purview/media/purview-areas.png){Purview's solution areas: data security, data compliance, data governance}

## What actually matters

**1. Sensitivity labels.** A **sensitivity label** is a tag such as *Public*, *General*, *Confidential*, *Highly Confidential* that travels with a file or email. A label can **encrypt** the content (only named people can open it, wherever the file goes), add **content marking** (header, footer, watermark), and — for Teams, groups and sites — control privacy and external sharing. Labels have a **priority order**; higher means more sensitive. Labels reach users through **label policies**, and **auto-labeling** applies them by rule. Copilot respects labels: for encrypted content the user needs both the **VIEW** and **EXTRACT** usage rights before Copilot will summarise it. With VIEW but not EXTRACT, Copilot only returns a link.

![Sensitivity label on the Excel ribbon and status bar.](https://learn.microsoft.com/en-us/purview/media/sensitivity-label-in-excel.png){A label as the user sees it in Excel}

**2. Data loss prevention (DLP).** A **DLP policy** finds sensitive content — by **sensitive information type** (credit card number, passport number) or by label — in a **location** and takes an **action**: audit, block, or block with override, plus a **policy tip** to the user. Locations include Exchange, SharePoint, OneDrive, Teams, Windows and macOS devices, and a **Microsoft 365 Copilot** location that stops Copilot from processing items with chosen labels.

![Diagram that shows exfiltration channels that Microsoft Purview DLP monitors and protects across.](https://learn.microsoft.com/en-us/purview/media/dlp-learn-about-dlp/dlp-protected-channels.png){Channels DLP watches}

**3. Audit — Standard vs Premium.** The **unified audit log** records user and admin actions across Microsoft 365, including Copilot and agent interactions. It is on by default.

| | Audit (Standard) | Audit (Premium) |
|---|---|---|
| Default retention | ==@180 days== (90 days for records before 17 October 2023) | ==@1 year== for Entra ID, Exchange, OneDrive, SharePoint; 180 days for others |
| Custom retention policies | no | yes, up to 1 year; ==@10 years== with an add-on |
| Intelligent insights (e.g. `MailItemsAccessed`) | no | yes |
| Licence | E3-level | E5-level or Purview Suite |

As an analyst you searched Audit. The engineer angle: Audit must be **on** before DSPM for AI can show anything, and retention depends on licence. (D4.2.10 covers querying Purview Audit from Defender XDR.)

**4. DSPM — and the naming change.** **Data Security Posture Management (DSPM)** is Purview's dashboard for data risk. The older **DSPM for AI** is now labelled ==@DSPM for AI (classic)== in the portal and Microsoft says the new **DSPM** replaces it, adding AI apps and agents. Both versions give you:

- a **data risk assessment** that runs ==@weekly on the top 100 SharePoint sites by usage==, with no activation needed, plus custom assessments;
- **AI observability** — inventory of AI apps and agents with recent activity and risky interactions;
- **one-click policies** — preconfigured DLP, Insider Risk and labelling policies (for example *Detect risky interactions in AI apps*);
- **Activity explorer** showing prompts, responses and sensitive data detected.

Allow at least **24 hours** after enabling policies for data to appear. Viewing prompt and response text needs the **Content Explorer Content Viewer** role group.

![Data Security Posture Management with sample data on the initial page, Posture.](https://learn.microsoft.com/en-us/purview/media/dspm-posture.png){The new DSPM: posture page}

![Diagram that shows how Microsoft Copilot works with items protected by Microsoft Purview sensitivity labels.](https://learn.microsoft.com/en-us/copilot/media/copilot-architecture-data-protection-auditing/copilot-sensitivity-labels.png){Copilot and labels: permissions first, then VIEW + EXTRACT for encrypted items, highest-priority label inherited by new content}

**5. Fixing SharePoint oversharing.** Purview finds oversharing; SharePoint fixes it. Options: correct site membership and sharing links; apply sensitivity labels to sites; and **Restricted Content Discovery (RCD)**, a site setting that hides the site from organisation-wide search and Copilot ==!without changing anyone's permissions== — people who already have access can still open it directly.

![Screenshot showing the Restricted marker for a site covered by Restricted Content Discovery.](https://learn.microsoft.com/en-us/sharepoint/sharepointonline/media/restricted-content-discovery/restricted-marker.png){A site with Restricted Content Discovery on}

![Diagram that shows how SharePoint and OneDrive oversharing controls and Microsoft Purview work together.](https://learn.microsoft.com/en-us/copilot/media/copilot-architecture-data-protection-auditing/copilot-oversharing-controls.png){Oversharing controls across SharePoint, OneDrive and Purview}

## Where it lives

- **Microsoft Purview portal**: `https://purview.microsoft.com` — **Solutions >** Information Protection (labels), Data Loss Prevention, Audit, **DSPM** and **DSPM for AI (classic)**. Roles: **Settings > Roles and scopes > Role groups**. You see only solution cards you are licensed and permitted for.
- **SharePoint admin center**: site settings including Restricted Content Discovery.
- **Microsoft 365 admin center**: `Copilot > Overview > Security` — the Copilot security dashboard (Global Reader to view, **AI Administrator** to change).
- A typical permission to start with DSPM for AI is the Entra **Compliance Administrator** role.

## Exam traps

- ==!"Copilot exposed data it had no permission to."== Copilot only returns what the user can open. ==+Fix permissions, labels and RCD at the source.==
- ==!Restricted Content Discovery removes access.== It only hides the site from org-wide search and Copilot.
- DLP acts on an **item**; Insider Risk Management looks at a **person's pattern** over time.
- Audit (Standard) retention is **180 days**; Premium is **1 year** for core workloads; 10 years needs an add-on.
- DSPM for AI depends on Audit being on.

## Skip

- **Skip:** data governance (Data Map, Unified Catalog, scanning) and Compliance Manager scoring.
- **Skip:** eDiscovery, Communication Compliance, Records Management and Data Lifecycle Management detail.
- **Skip:** building labels step by step (sublabels, justification prompts, mandatory labelling). The screenshots below show them; the exam asks what labels *do*.
- **Skip:** Double Key Encryption and Customer Key.
- **Skip:** the Purview browser extension and device onboarding for third-party AI sites — know they exist for ChatGPT/Gemini visibility.

## Screenshots

![Screenshot of the Microsoft Purview portal welcome dialog.](https://learn.microsoft.com/en-us/purview/media/purview-portal-welcome.png){Purview portal: first-visit welcome}

![Screenshot of the Microsoft Purview portal home page.](https://learn.microsoft.com/en-us/purview/media/purview-portal.png){Purview home page}

![Screenshot of Microsoft Purview portal search results.](https://learn.microsoft.com/en-us/purview/media/purview-portal-search-results.png){Global search}

![Screenshot of Microsoft Purview portal solution cards.](https://learn.microsoft.com/en-us/purview/media/purview-portal-solution-cards.png){Solution cards}

![Screenshot of the Microsoft Purview portal solutions page.](https://learn.microsoft.com/en-us/purview/media/purview-portal-solutions-page.png){Solutions page}

![Screenshot of the Compliance posture status card.](https://learn.microsoft.com/en-us/purview/media/purview-portal-compliance-posture-status.png){Compliance posture card (skip-list item)}

![Screenshot of the Trials and recommendations card.](https://learn.microsoft.com/en-us/purview/media/purview-portal-trials-recommendations.png){Trials and recommendations}

![Screenshot of the Knowledge Center card.](https://learn.microsoft.com/en-us/purview/media/purview-portal-knowledge-center.png){Knowledge Center}

![Screenshot of the Microsoft Purview portal left navigation.](https://learn.microsoft.com/en-us/purview/media/purview-portal-left-navigation.png){Left navigation}

![Screenshot of the Microsoft Purview portal settings pane.](https://learn.microsoft.com/en-us/purview/media/purview-portal-settings.png){Settings pane}

![Screenshot of the Microsoft Purview portal solution settings page.](https://learn.microsoft.com/en-us/purview/media/purview-portal-solution-settings.png){Per-solution settings}

![Screenshot of the Microsoft Purview portal help and support pane.](https://learn.microsoft.com/en-us/purview/media/purview-portal-help-support.png){Help and support}

![Sensitivity label applied to an email.](https://learn.microsoft.com/en-us/purview/media/sensitivity-label-on-email.png){Label on an email}

![Watermark and header applied to document.](https://learn.microsoft.com/en-us/purview/media/sensitivity-label-watermark-header.png){Content marking: watermark and header}

![Default prompt for a user to assign a required sensitivity label in Excel.](https://learn.microsoft.com/en-us/purview/media/sensitivity-label-required-prompt.png){Mandatory labelling prompt}

![Screenshot that shows scope options for sensitivity labels.](https://learn.microsoft.com/en-us/purview/media/sensitivity-labels-scopes.png){Label scopes: files, emails, meetings, groups and sites}

![Screenshot that shows options to change the priority of sensitivity labels.](https://learn.microsoft.com/en-us/purview/media/sensitivity-label-reorder-options.png){Label priority order}

![Example of sublabels from a sensitivity label.](https://learn.microsoft.com/en-us/purview/media/sensitivity-sublables.png){Sublabels}

![Prompt where users enter a justification.](https://learn.microsoft.com/en-us/purview/media/sensitivity-label-justification-required.png){Justification when lowering a label}

![Prompt in Outlook asking user to apply required label.](https://learn.microsoft.com/en-us/purview/media/sensitivity-labels-mandatory-prompt-outlook.png){Outlook mandatory label prompt}

![Learn more link on Sensitivity button on Ribbon.](https://learn.microsoft.com/en-us/purview/media/sensitivity-label-learn-more.png){Custom help link}

![Move option on the page for sensitivity label policies.](https://learn.microsoft.com/en-us/purview/media/sensitivity-label-policy-priority.png){Label policy priority}

![Alert info.](https://learn.microsoft.com/en-us/purview/media/alert-info-1.png){DLP alert details}

![event info.](https://learn.microsoft.com/en-us/purview/media/event-info-1.png){DLP event details}

![Data Security Posture Management with sample data on the Objectives page.](https://learn.microsoft.com/en-us/purview/media/dspm-objectives.png){New DSPM: data security objectives with guided workflows}

![Example screenshot of Microsoft Purview Data Security Posture Management for AI.](https://learn.microsoft.com/en-us/purview/media/example-dspm.png){DSPM for AI (classic) overview}

![Selecting the Microsoft Purview Data Security Posture Management for AI solution from the Microsoft Purview portal.](https://learn.microsoft.com/en-us/purview/media/dspm-solution.png){Solutions > DSPM for AI (classic)}

![Screenshot of the settings tab where you can enable Restricted Content Discovery.](https://learn.microsoft.com/en-us/sharepoint/sharepointonline/media/restricted-content-discovery/settings-tab-active-site.png){SharePoint admin center: enable Restricted Content Discovery on a site}

![Screenshot showing the Restrict content from Microsoft Copilot setting configured by a site admin.](https://learn.microsoft.com/en-us/sharepoint/sharepointonline/media/restricted-content-discovery/configuration-by-site-admin.png){Site admin view of the setting}

![Screenshot showing the Justification required dialog box.](https://learn.microsoft.com/en-us/sharepoint/sharepointonline/media/restricted-content-discovery/justification-by-site-admin.png){Justification when a site admin changes it}

![Diagram that shows where Microsoft Copilot interaction data is stored and audited.](https://learn.microsoft.com/en-us/copilot/media/copilot-architecture-data-protection-auditing/copilot-auditing-tools.png){Where Copilot interaction data is stored and which Purview tools audit it}

![Updated cloud attachments option for auto-apply retention label to include interactions for Copilot.](https://learn.microsoft.com/en-us/purview/media/cloud-attachments-copilot.png){Retention label option covering files referenced in Copilot interactions (skip-list item)}
