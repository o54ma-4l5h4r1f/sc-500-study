+++
slug = "diagnostic-settings-and-activity-log"
title = "Diagnostic settings, the Activity log and resource audit logs"
summary = "The three kinds of Azure platform telemetry (Activity log, resource logs, platform metrics), what a diagnostic setting is and its four destinations and limits, and the engineer's job of turning on the audit logs for Key Vault, Azure SQL and Storage that the security lessons depend on."
order = 30
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/activity-log",
  "https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/diagnostic-settings",
  "https://learn.microsoft.com/en-us/azure/azure-monitor/logs/resource-logs",
  "https://learn.microsoft.com/en-us/azure/key-vault/general/howto-logging",
  "https://learn.microsoft.com/en-us/azure/azure-monitor/reference/supported-logs/microsoft-keyvault-vaults-logs",
  "https://learn.microsoft.com/en-us/azure/azure-monitor/reference/tables/azkvauditlogs",
  "https://learn.microsoft.com/en-us/azure/azure-sql/database/auditing-setup",
  "https://learn.microsoft.com/en-us/azure/storage/blobs/monitor-blob-storage",
]
captured = 2026-11-15
related_objectives = ["D4.2.4", "D1.2.2", "D2.2.2", "D2.1.1", "D1.3.1"]
+++

## In one sentence

Azure records changes made *to* resources automatically (the **Activity log**), but the record of work done *inside* a resource — a secret read, a blob deleted, a SQL login — exists only if you create a **diagnostic setting** that sends those **resource logs** somewhere.

## Why it exists

Think of an office building. The front desk keeps a **visitor book** automatically: who came in and which room they rearranged. Each room also has its own **logbook** — the safe records every opening — but nobody collects those unless someone is told to. A diagnostic setting is the instruction "photocopy this room's logbook every day and send it to security".

As an analyst you queried `AzureActivity` and `AZKVAuditLogs`. Someone had to switch them on. An investigation that needs "who read this secret last Tuesday?" fails if Key Vault logging was never enabled — and you cannot turn it on retroactively.

## Mental model

Two planes, two logs:

- **Control plane** — managing a resource from outside (create, update, delete, change a firewall rule, assign a role). Goes through Azure Resource Manager. Recorded in the **Activity log**.
- **Data plane** — using the resource from inside (get a secret, read a blob, run a query). Recorded in **resource logs**.

```svg
<svg viewBox="0 0 900 340" role="img" aria-label="Activity log, resource logs and metrics flow through diagnostic settings to four destinations: Log Analytics workspace, storage account, Event Hubs and partner solutions; Sentinel reads only the workspace">
  <defs>
    <marker id="dsaarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="line-accent"/>
    </marker>
  </defs>
  <rect class="box-ok" x="15" y="20" width="260" height="80" rx="8"/>
  <text x="145" y="48" text-anchor="middle" class="label-accent">Activity log</text>
  <text x="145" y="70" text-anchor="middle">control plane, automatic</text>
  <text x="145" y="90" text-anchor="middle" class="muted">kept 90 days, free</text>
  <rect class="box-risk" x="15" y="125" width="260" height="80" rx="8"/>
  <text x="145" y="153" text-anchor="middle" class="label-risk">Resource logs</text>
  <text x="145" y="175" text-anchor="middle">data plane, OFF by default</text>
  <text x="145" y="195" text-anchor="middle" class="muted">Key Vault, SQL, Storage...</text>
  <rect class="box" x="15" y="230" width="260" height="80" rx="8"/>
  <text x="145" y="258" text-anchor="middle">Platform metrics</text>
  <text x="145" y="280" text-anchor="middle">numbers, automatic</text>
  <text x="145" y="300" text-anchor="middle" class="muted">CPU, requests, latency</text>
  <rect class="box-accent" x="330" y="110" width="200" height="110" rx="10"/>
  <text x="430" y="145" text-anchor="middle" class="label-accent">Diagnostic setting</text>
  <text x="430" y="170" text-anchor="middle">which categories</text>
  <text x="430" y="192" text-anchor="middle">to which destinations</text>
  <rect class="box-ok" x="590" y="20" width="295" height="60" rx="8"/>
  <text x="737" y="47" text-anchor="middle">Log Analytics workspace</text>
  <text x="737" y="68" text-anchor="middle" class="muted">query, Sentinel, alerts</text>
  <rect class="box" x="590" y="95" width="295" height="60" rx="8"/>
  <text x="737" y="122" text-anchor="middle">Storage account</text>
  <text x="737" y="143" text-anchor="middle" class="muted">cheap archive</text>
  <rect class="box" x="590" y="170" width="295" height="60" rx="8"/>
  <text x="737" y="197" text-anchor="middle">Event Hubs</text>
  <text x="737" y="218" text-anchor="middle" class="muted">stream to another SIEM</text>
  <rect class="box" x="590" y="245" width="295" height="60" rx="8"/>
  <text x="737" y="272" text-anchor="middle">Partner solution</text>
  <text x="737" y="293" text-anchor="middle" class="muted">Azure Native ISV services</text>
  <line class="line-accent" x1="275" y1="60" x2="328" y2="140" marker-end="url(#dsaarrow)"/>
  <line class="line-accent" x1="275" y1="165" x2="328" y2="165" marker-end="url(#dsaarrow)"/>
  <line class="line" x1="275" y1="270" x2="328" y2="195" marker-end="url(#dsaarrow)"/>
  <line class="line-accent" x1="530" y1="140" x2="588" y2="50" marker-end="url(#dsaarrow)"/>
  <line class="line" x1="530" y1="155" x2="588" y2="125" marker-end="url(#dsaarrow)"/>
  <line class="line" x1="530" y1="180" x2="588" y2="200" marker-end="url(#dsaarrow)"/>
  <line class="line" x1="530" y1="195" x2="588" y2="275" marker-end="url(#dsaarrow)"/>
</svg>
```

![Diagram that shows collection of activity logs, resource logs, and platform metrics.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/diagnostic-settings/platform-logs-metrics.png){Microsoft's version: platform telemetry and where diagnostic settings send it}

## What actually matters

**1. The Activity log is automatic, free and short-lived.** Azure keeps entries for ==@90 days== at no charge, then deletes them. Events usually appear within 3–20 minutes. To keep them longer or query them with other data, export the Activity log with a **subscription-level** diagnostic setting to a workspace — it lands in the `AzureActivity` table. It records control-plane writes, deletes and actions. ==!It is not a sign-in log== — failed authentication to Azure Resource Manager appears in Entra sign-in logs, not here.

**2. Resource logs are off until you create a diagnostic setting on each resource.** In a workspace they land in one of two shapes, the **collection mode**:

| Collection mode | Where | Notes |
|---|---|---|
| Azure diagnostics (legacy) | one shared `AzureDiagnostics` table | wide, harder to query |
| ==+Resource-specific== | one table per category (e.g. `AZKVAuditLogs`, `StorageBlobLogs`) | Microsoft recommends for new settings |

**3. Diagnostic setting rules.**

| Rule | Value |
|---|---|
| Destinations | Log Analytics workspace, storage account, Event Hubs, partner solution |
| Settings per resource | up to ==@5== |
| Destinations of the same type per setting | one (two workspaces = two settings) |
| Category groups | `allLogs` or `audit` (security-relevant subset) |
| Storage / Event Hubs region | same region as the resource |
| Filtering inside a category | not in the setting — use a DCR transformation |

Diagnostic settings can be deployed at scale with **Azure Policy** (`DeployIfNotExists`), which is how many Sentinel connectors work.

**4. Engineer angle: the three audit logs this exam leans on.**

| Service | How to turn on | Category | Workspace table |
|---|---|---|---|
| **Key Vault** | diagnostic setting on the vault | `AuditEvent` — the only Key Vault audit category | `AZKVAuditLogs` (resource-specific) or `AzureDiagnostics` |
| **Azure SQL Database** | **Auditing** blade on the server or database (Storage, Log Analytics, Event Hubs) | `SQLSecurityAuditEvents` | `AzureDiagnostics` |
| **Azure Storage** | diagnostic setting on **each service** (blob, file, queue, table), not the account | `StorageRead`, `StorageWrite`, `StorageDelete` | `StorageBlobLogs`, `StorageFileLogs`, … |

> **Two Microsoft pages disagree on the Key Vault table.** The *Supported logs for Microsoft.KeyVault/vaults* reference lists `AuditEvent` → `AzureDiagnostics`. The `AZKVAuditLogs` table reference describes a dedicated Key Vault audit table. The difference is the collection mode: Azure diagnostics mode writes to `AzureDiagnostics`; resource-specific mode writes to `AZKVAuditLogs`.

Two engineer gotchas. ==!SQL auditing to Log Analytics or Event Hubs silently creates a diagnostic setting named `SQLSecurityAuditEvents_...`== — delete it and auditing fails silently. Server-level auditing applies to every database on the server. For Storage, a setting on the account itself only gives account-level metrics; the read/write/delete logs come from the per-service settings.

**5. Choosing a destination.** Pick the **workspace** when people or Sentinel must query and alert on the logs. Pick a **storage account** when the requirement is cheap, long-term archive for auditors and nobody needs to query daily. Pick **Event Hubs** when another SIEM or tool must receive the stream. One setting can send to one of each type at the same time, so "archive to storage *and* analyse in Sentinel" is a single diagnostic setting with two destinations. The destination must already exist before you save the setting, and you need write rights on it.

## Where it lives

| Task | Where |
|---|---|
| View Activity log | `Azure portal > Monitor > Activity log` (or any resource > Activity log) |
| Export Activity log | `Monitor > Activity log > Export Activity Logs` (subscription setting) |
| Resource logs | `(resource) > Monitoring > Diagnostic settings > Add diagnostic setting` |
| Storage service logs | `(storage account) > Monitoring > Diagnostic settings` → choose **blob**, **file**, **queue** or **table** |
| SQL auditing | `(SQL server or database) > Security > Auditing` |

```bash
az monitor diagnostic-settings create --name kv-audit \
  --resource <key-vault-resource-id> --workspace <workspace-resource-id> \
  --logs '[{"categoryGroup":"audit","enabled":true}]' --export-to-resource-specific true
az monitor activity-log list --offset 14d
```

## Exam traps

- ==!"Resource logs are collected automatically."== Only the Activity log and metrics are. ==+Create a diagnostic setting.==
- Activity log = **who changed the resource**; resource log = **who used the data inside it**. "Who read the secret?" needs Key Vault `AuditEvent`, not the Activity log.
- Two workspaces need two diagnostic settings; maximum five per resource.
- Storage logs are per service (blob/file/queue/table).
- SQL auditing is configured on the **Auditing** blade, which creates the diagnostic setting for you.

## Skip

- **Skip:** Activity Log Insights workbooks and change history (screenshots below).
- **Skip:** partner-solution destinations and Event Hubs partitioning.
- **Skip:** the PowerShell and ARM variants of diagnostic settings; the portal and one CLI line are enough.
- **Skip:** SQL audit action groups (`BATCH_COMPLETED_GROUP` etc.) beyond knowing the default policy audits queries and logins — D2.2.2 covers it.

## Screenshots

![Screenshot that shows the activity log.](https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/media/activity-log/view-activity-log.png){Monitor > Activity log}

![Screenshot showing the Directory Activity option selected in the Activity log.](https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/media/activity-log/directory-activity-logs.png){Directory Activity: tenant-level events}

![Screenshot showing the Activity log blade for a management group.](https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/media/activity-log/management-group-scope.png){Activity log at management group scope}

![Screenshot that shows the Change history list for an event.](https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/media/activity-log/change-history-event.png){Change history for an event}

![Screenshot that shows the Change history page showing differences.](https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/media/activity-log/change-history-event-details.png){Before/after properties}

![Screenshot that shows activity log insights dashboards.](https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/media/activity-log/activity-logs-insights-main-screen.png){Activity log insights (skip-list item)}

![Screenshot that shows how to locate and open the Activity Log Insights workbook.](https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/media/activity-log/open-activity-log-insights-workbook.png){Open the insights workbook}

![Screenshot that shows how to locate and open the Activity Logs Insights at resource level.](https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/media/activity-log/activity-log-resource-level.png){Insights at resource level}

![Screenshot that shows the Azure Monitor menu with Activity log selected.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/diagnostic-settings/menu-activity-log.png){Monitor menu > Activity log}

![Screenshot that shows the Download as CSV button in the Azure portal activity log.](https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/media/activity-log/export-csv.png){Download as CSV}

![Screenshot that shows adding a diagnostic setting for existing settings.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/diagnostic-settings/edit-setting.png){Diagnostic settings list on a resource}

![Screenshot that shows Diagnostic setting details.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/diagnostic-settings/setting-new-blank.png){New diagnostic setting: categories on the left, destinations on the right}

![A screenshot showing the AzureDiagnostics table in a Log Analytics workspace.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/resource-logs/azure-diagnostics-table.png){The legacy AzureDiagnostics table}

![Screenshot that shows the Diagnostics settings mode selector.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/resource-logs/diagnostic-settings-mode-selector.png){Collection mode: Azure diagnostics vs Resource specific}

![Screenshot that shows how to select diagnostic settings.](https://learn.microsoft.com/en-us/azure/key-vault/media/diagnostics-portal-1.png){Key Vault > Diagnostic settings}

![Screenshot of diagnostic settings options.](https://learn.microsoft.com/en-us/azure/key-vault/media/diagnostics-portal-2.png){Key Vault: category groups audit and allLogs, destination workspace}

![Screenshot that shows how to save the options you selected.](https://learn.microsoft.com/en-us/azure/key-vault/media/diagnostics-portal-3.png){Save}

![Screenshot that shows the View server settings link highlighted on the database auditing page.](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/auditing-setup/auditing-get-started-server-inherit.png){SQL database auditing: View server settings}

![Screenshot that shows the storage options for Auditing.](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/auditing-setup/auditing-select-destination.png){SQL auditing destinations: Storage, Log Analytics, Event Hub}

![Screenshot that shows storage account authentication types for Auditing.](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/auditing-setup/auditing-select-storage.png){Storage destination authentication}

![Screenshot showing the Log Analytics selected workspace.](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/auditing-setup/auditing-select-log-analytics.png){Log Analytics destination}

![Screenshot showing the Event hub.](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/auditing-setup/auditing-select-event-hub.png){Event Hubs destination}

![Screenshot of accessing metrics in the Azure portal](https://learn.microsoft.com/en-us/azure/storage/blobs/media/monitor-blob-storage/access-metrics-portal.png){Blob storage metrics}

![Screenshot of accessing metrics with dimension in the Azure portal](https://learn.microsoft.com/en-us/azure/storage/blobs/media/monitor-blob-storage/access-metrics-portal-with-dimension.png){Metrics split by dimension}
