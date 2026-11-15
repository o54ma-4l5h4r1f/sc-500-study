+++
slug = "log-analytics-workspace"
title = "Log Analytics workspace refresher (engineer view)"
summary = "A short refresher on the Log Analytics workspace for someone who queried it as an analyst, now from the engineer's side: one-versus-many workspace design, access control modes and access modes, the three table plans, analytics vs long-term retention, Sentinel's 90 free days, and the daily cap."
order = 28
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/azure-monitor/logs/log-analytics-workspace-overview",
  "https://learn.microsoft.com/en-us/azure/azure-monitor/logs/data-platform-logs",
  "https://learn.microsoft.com/en-us/azure/azure-monitor/logs/workspace-design",
  "https://learn.microsoft.com/en-us/azure/azure-monitor/logs/manage-access",
  "https://learn.microsoft.com/en-us/azure/azure-monitor/logs/data-retention-configure",
  "https://learn.microsoft.com/en-us/azure/azure-monitor/logs/daily-cap",
  "https://learn.microsoft.com/en-us/azure/sentinel/billing",
  "https://learn.microsoft.com/en-us/azure/sentinel/manage-data-overview",
]
captured = 2026-11-15
related_objectives = ["D4.2.1", "D4.2.2", "D4.2.7", "D4.2.9"]
+++

## In one sentence

A **Log Analytics workspace** is a regional Azure resource that stores logs in **tables**; Microsoft Sentinel is switched on *on top of* one, so every design choice about the workspace — region, access, plans, retention, cap — is a Sentinel engineering decision.

## Why it exists

As an analyst you ran KQL against the workspace and never asked who built it. Somebody chose its region, decided who can read which tables, picked which tables are cheap and which are fast, and set how long evidence is kept. On this exam that somebody is you.

## Mental model

A **filing room** in one building (region). Each **cabinet** is a table with a fixed form layout (schema). Some cabinets sit by the desk for instant access (**Analytics** plan); some are cheaper cabinets in the basement (**Basic**, **Auxiliary**). Old files move to **long-term** storage before they are shredded. A **daily cap** is a rule that the post room stops accepting deliveries after a set weight per day.

![Diagram that shows the Azure Monitor Logs structure.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/data-platform-logs/logs-structure.png){A workspace holds tables; each table has its own plan and retention}

## What's new for an engineer

- **Design: start with one workspace.** Split only for a real reason: data residency (regions), separate Entra tenants, split billing, or different retention/access needs. Once Sentinel is enabled, ==!all data in the workspace is billed at Sentinel rates==, including operational logs.
- **Access control mode (a workspace setting).** **Use resource or workspace permissions** (default for workspaces created after March 2019) lets someone with rights on a VM read that VM's logs only. **Require workspace permissions** means rights on the workspace or table are needed.
- **Access mode (how the user opened Logs).** **Workspace-context** (from the workspace: every table you're allowed) versus **resource-context** (from a resource: only its records). Resource-context is how you give an app team its own logs without opening the whole workspace.
- **Roles.** **Log Analytics Reader** runs queries; **Log Analytics Contributor** changes settings such as retention and the daily cap. Sentinel's own roles sit on top (see D4.2.2).

## What actually matters

| Table plan | Use for | Query | Alerts |
|---|---|---|---|
| ==@Analytics== | detections, high-value security data | full KQL, included in price | all |
| ==@Basic== | medium-value, troubleshooting | KQL, charged per query, ==!30 days only== | simple log alerts |
| ==@Auxiliary== | verbose audit/compliance data | KQL (slower), charged per query | none |

**Retention.** **Analytics retention** = directly queryable period. Default **30 days** for most tables; on a Sentinel-enabled workspace ==@90 days of analytics retention are free==. Extend up to **2 years** for Analytics tables. **Long-term retention** follows, retrieved with search jobs. **Total retention** up to **12 years**. Shortening total retention waits 30 days before deleting, so mistakes can be reverted.

**Same thing, Defender portal names.** Once Sentinel is in the Defender portal, the **Analytics** plan shows as the ==@Analytics tier==, and long-term retention plus Auxiliary (lake-only) tables show as the ==@Data lake tier==. Basic has no separate Defender-portal tier. D4.2.9 has the full mapping.

**Daily cap.** A per-workspace GB/day limit. When hit, billable collection into Analytics and Basic tables **stops** until the reset hour; Auxiliary tables are not capped. ==!In production, hitting the cap blinds the SOC.== Use it in labs; in production alert on it.

## Where it lives

- Create: `Azure portal > Log Analytics workspaces > Create`.
- Access control mode: workspace **Overview** (view) and **Properties** (change).
- Default retention: workspace **> Usage and estimated costs > Data Retention**. CLI: `az monitor log-analytics workspace update -g rg-lab -n law-lab --retention-time 90`.
- Table plan and retention: workspace **> Tables >** (…) **> Manage table**.
- Daily cap: workspace **> Usage and estimated costs > Daily Cap**.

## Exam traps

- ==!"A Sentinel workspace is a separate database."== It is a Log Analytics workspace with Sentinel enabled.
- Basic plan = 30 days of querying, no analytics rules. Detections need **Analytics**.
- Daily cap stops collection — it does not throttle or queue.
- A user with rights only on a VM (not the workspace) can read that VM's logs only when the control mode is **Use resource or workspace permissions**.

## Skip

- **Skip:** Workspace Insights and Container Insights dashboards (screenshots below). Operations tooling.
- **Skip:** commitment-tier discounts and pricing tables — reference material, not exam content.
- **Skip:** granular row-level RBAC conditions; know the two modes.
- **Skip:** the Application Insights data cap screen (different product).

## Screenshots

![A diagram that shows how Azure Monitor and Azure Monitor Logs provide an end-to-end solution.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/data-platform-logs/azure-monitor-logs-overview.png){Azure Monitor Logs overview}

![A diagram that shows data being collected from data sources and transformed.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/data-platform-logs/azure-monitor-logs-collection-ingestion.png){Collection and ingestion into the workspace}

![A screenshot of a Log Analytics workspace in the Azure portal.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/data-platform-logs/log-analytics-workspace-for-all-log-data.png){A workspace in the Azure portal}

![Diagram that presents an overview of the capabilities provided by the Analytics, Basic and Auxiliary table plans.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/data-platform-logs/azure-monitor-logs-data-plans.png){The three table plans}

![Screenshot that shows Log Analytics Simple mode.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/log-analytics-explorer/log-analytics-simple-mode-user-interface.png){Log Analytics Simple mode}

![A screenshot that shows the built-in Container Insights monitoring experience.](https://learn.microsoft.com/en-us/azure/azure-monitor/containers/media/kubernetes-monitoring-overview/containers-insights-experience.png){Container Insights (skip-list item)}

![Screenshot that shows the Log Analytics Workspace insights overview tab.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/log-analytics-workspace-insights-overview/at-resource.png){Workspace Insights (skip-list item)}

![Screenshot that shows the workspace access control mode.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/manage-access/view-access-control-mode.png){Access control mode on the Overview page}

![Screenshot that shows changing workspace access mode.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/manage-access/change-access-control-mode.png){Change it on Properties}

![Diagram that shows analytics and long-term retention.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/data-retention-configure/interactive-auxiliary-retention-log-analytics-workspace.png){Analytics retention + long-term retention = total retention}

![Screenshot that shows changing the workspace data retention setting.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/manage-cost-storage/manage-cost-change-retention-01.png){Workspace default retention slider}

![Screenshot that shows the Manage table button for one of the tables in a workspace.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/logs-table-plans/log-analytics-table-configuration.png){Tables > Manage table}

![Screenshot that shows the data retention settings on the table configuration screen.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/data-retention-configure/log-analytics-configure-table-retention-basic.png){Per-table retention (Basic table)}

![Screenshot that shows the retention of an Auxiliary table.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/data-retention-configure/log-analytics-view-table-retention-auxiliary.png){Auxiliary table retention}

![Log Analytics configure data limit](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/manage-cost-storage/set-daily-volume-cap-01.png){Workspace daily cap}

![Application Insights configure data limit](https://learn.microsoft.com/en-us/azure/azure-monitor/app/media/pricing/pricing-003.png){Application Insights data cap — a different product's screen, shown because it is in the cited article}

![Screenshot showing the Microsoft Sentinel section of a sample bill.](https://learn.microsoft.com/en-us/azure/sentinel/media/billing/sample-bill-classic.png){Sentinel charges on a sample bill (reference only)}

![Screenshot of the connector page for Defender for Cloud Apps, with the free data types highlighted.](https://learn.microsoft.com/en-us/azure/sentinel/media/billing/data-types.png){Some connector data types are free to ingest}

![The connector selector with Sentinel under Classic connectors and My connectors under Unified connectors.](https://learn.microsoft.com/en-us/azure/sentinel/media/manage-data-overview/connector-type-selector.png)

![The Data and connectors page showing the Overview, Connectors, and Tables tabs.](https://learn.microsoft.com/en-us/azure/sentinel/media/manage-data-overview/data-and-connectors-tabs.png)

![Overview health insights showing ingestion volume, a connector that needs attention, and table ingestion fluctuations.](https://learn.microsoft.com/en-us/azure/sentinel/media/manage-data-overview/overview-data-health-insights.png)

![The Ingestion latency by table chart showing P95 latency trends for all tables.](https://learn.microsoft.com/en-us/azure/sentinel/media/manage-data-overview/overview-ingestion-latency-by-table.png)

![The Data ingestion volume by table chart showing daily volume trends for the top 10 tables.](https://learn.microsoft.com/en-us/azure/sentinel/media/manage-data-overview/overview-ingestion-volume-by-table.png)

![The Tables grid showing filters and table insights columns for tier, last data received, average daily ingestion, estimated daily cost, volume anomaly, and retention.](https://learn.microsoft.com/en-us/azure/sentinel/media/manage-data-overview/table-insights-grid.png)

![Diagram that depicts the analytics and data lake tiers in the Microsoft Defender portal.](https://learn.microsoft.com/en-us/azure/sentinel/media/manage-data-overview/tiers-retention-defender-portal.png)

![The Table Management screen in the Defender portal.](https://learn.microsoft.com/en-us/azure/sentinel/media/manage-data-overview/view-table-properties-microsoft-defender-portal.png)
