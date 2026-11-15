+++
slug = "azure-monitor-agent-and-dcr"
title = "Azure Monitor Agent and data collection rules refresher (engineer view)"
summary = "A short refresher on how logs leave a machine and reach a workspace: the Azure Monitor Agent ships, a data collection rule says what, how and where, an association links the two, transformations trim at ingestion, Windows Event Forwarding lands in a different table, and custom tables end in _CL."
order = 29
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/azure-monitor/agents/azure-monitor-agent-overview",
  "https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/data-collection-rule-overview",
  "https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/data-collection-transformations",
  "https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/data-collection-endpoint-overview",
  "https://learn.microsoft.com/en-us/azure/azure-monitor/logs/create-custom-table",
  "https://learn.microsoft.com/en-us/azure/sentinel/connect-services-windows-based",
]
captured = 2026-11-15
related_objectives = ["D4.2.5", "D4.2.6", "D4.2.7", "D4.2.4", "D3.2.4"]
+++

## In one sentence

The **Azure Monitor Agent (AMA)** is the program on each machine that ships its logs; a **data collection rule (DCR)** is a separate Azure resource saying *what* to collect, *how* to trim it and *where* to send it; a **DCR association** links one machine to one rule.

## Why it exists

As an analyst you saw `SecurityEvent` rows appear. As an engineer you build the pipe that produces them — and you decide which events are worth paying for.

## Mental model

A **mailroom**. Each desk (machine) has a collector (AMA). Head office writes one **instruction sheet** (DCR): "collect logons and process starts, shred the rest, file in cabinet `SecurityEvent`." Each desk gets a copy by being put on the distribution list (**association**). The sheet is configuration only — ==!log data does not flow through the DCR resource==.

![Diagram that shows data collection rule associations connecting multiple virtual machines to data collection rules.](https://learn.microsoft.com/en-us/azure/azure-monitor/agents/media/azure-monitor-agent-overview/data-collection-rule-associations.png){Many-to-many: one DCR can serve many machines; one machine can have many DCRs — one association per pair}

## What's new for an engineer

- **AMA is the only supported agent.** The legacy Log Analytics agent (MMA/OMS) is retired. New collection = AMA + DCR.
- **Non-Azure machines need Azure Arc first** — AMA-based Sentinel connectors install only on Azure VMs or Arc-enabled servers.
- **Who can build it:** **Monitoring Contributor** creates DCRs and associations; installing the agent extension needs VM write rights. Create the DCR in the **same region** as the destination workspace.
- **Sentinel connectors are DCR wizards.** *Windows Security Events via AMA*, *Syslog via AMA*, *Common Event Format (CEF) via AMA* and *Windows Forwarded Events* each create a DCR when you select **+Add data collection rule**.

## What actually matters

**1. DCR parts.** `dataSources` (Windows event logs, Syslog, performance counters, text/JSON files) → `streams` (named data shapes such as `Microsoft-SecurityEvent`; custom ones start `Custom-`) → `destinations` (a workspace) → `dataFlows` (which stream to which destination, with optional `transformKql` and `outputStream`).

**2. Two places to filter.**

| Filter | Where it runs | Use |
|---|---|---|
| **XPath query** (`Security!*[System[(EventID=4624)]]`) | on the machine, before sending | cheapest — dropped events are never uploaded |
| **Transformation** (`transformKql`, starts from `source`) | in Azure, at ingestion, before storage | filter rows, drop or mask columns, route to a table |

A **workspace transformation DCR** applies a transformation to a table even for data that does not come through an agent DCR.

![Diagram that shows ingestion-time transformation for incoming data.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/data-collection-transformations/transformation-overview.png){Transformations run after Azure Monitor receives data and before it is stored}

**3. Same event, different table.**

| Route | Table |
|---|---|
| Sentinel **Windows Security Events via AMA** | ==@SecurityEvent== |
| **Windows Forwarded Events** connector | ==@WindowsEvent== |
| Plain Azure Monitor Windows event DCR | `Event` |

**Windows Event Forwarding (WEF)**: source machines forward events to a **Windows Event Collector (WEC)** server; AMA runs on the WEC and ships the forwarded log. ==!Many built-in Sentinel rules query `SecurityEvent` only== — WEF data in `WindowsEvent` will not trigger them unless you adapt the rule or use ASIM parsers.

**4. Custom tables end in `_CL`.** Data with no built-in table goes to a **custom table** (`MyApp_CL`). The portal adds `_CL` automatically. Choose its plan (Analytics, Basic, Auxiliary) at creation. Every table needs a `TimeGenerated` column. Text/JSON file collection and the Logs Ingestion API both write to custom tables through a DCR.

**5. Data collection endpoint (DCE) — usually not needed.** AMA uses public endpoints by default. You need a **DCE** for **Private Link** (Azure Monitor Private Link Scope) and for certain AMA data sources. The Logs Ingestion API can use the DCR's own endpoint instead of a DCE, unless the workspace uses Private Link.

## Where it lives

| Task | Where |
|---|---|
| Create/view DCRs | `Azure portal > Monitor > Settings > Data Collection Rules` |
| From Sentinel | `Microsoft Sentinel > Configuration > Data connectors > (connector) > Open connector page > +Add data collection rule` |
| DCE | `Azure portal > Monitor > Settings > Data collection endpoints` |
| Custom table | `Log Analytics workspace > Tables > Create > New custom log (DCR-based)` |
| Agent on a VM | `Virtual machines > (VM) > Extensions + applications` → `AzureMonitorWindowsAgent` / `AzureMonitorLinuxAgent` |

```bash
az vm extension set --name AzureMonitorWindowsAgent --publisher Microsoft.Azure.Monitor \
  --ids <vm-resource-id> --enable-auto-upgrade true
```

## Exam traps

- ==!Installing MMA/OMS for new collection== — wrong; ==+AMA + DCR==.
- ==!"A DCE is always required."== Only for Private Link and certain AMA data sources.
- WEF → `WindowsEvent`, not `SecurityEvent`.
- Filter at the source with **XPath** to save cost; use **transformations** to reshape or mask after arrival.
- One DCR applied to 50 machines = 50 associations.

## Skip

- **Skip:** multi-stage/pipeline transformations, Event Hubs and platform-metrics DCR scenarios.
- **Skip:** DCE regional layouts (config vs ingestion endpoints per region).
- **Skip:** the legacy-agent migration feature tables.
- **Skip:** Logs Ingestion API code samples.

## Screenshots

![Two-panel diagram emphasizing centralized DCR management.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/data-collection-rule-overview/data-collection-rule-centralized-control.svg){One DCR managed centrally, applied to many sources}

![Diagram that shows clients sending data along a separate data path from the DCR.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/data-collection-rule-overview/azure-monitor-pipeline-simple.png){Data path is separate from the DCR}

![Diagram that shows basic operation for Azure Monitor agent using DCR.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/data-collection-rule-overview/data-collection-virtual-machine.png){AMA on a VM using a DCR}

![Diagram that shows basic operation for event hub data sent to Azure Monitor.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/data-collection-rule-overview/data-collection-event-hubs.png){Event Hubs scenario (skip-list item)}

![Diagram that shows basic operation for DCR collecting platform metrics.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/data-collection-rule-overview/data-collection-platform-metrics.png){Platform metrics scenario (skip-list item)}

![Diagram that shows basic operation for DCR using Logs ingestion API.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/data-collection-rule-overview/data-collection-direct-ingestion.png){Logs Ingestion API into a custom table}

![Diagram that shows basic operation of workspace transformation DCR.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/data-collection-rule-overview/data-collection-workspace-transformation.png){Workspace transformation DCR}

![Diagram that shows operation of the workspace transformation DCR.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/data-collection-transformations/workspace-transformation-dcr.png){Workspace transformation applied to a table}

![Diagram that compares standard DCR transformations with workspace DCR transformations.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/data-collection-transformations/transformation-comparison.png){Standard vs workspace transformations}

![Diagram showing the flow of data from pipeline transformation to Azure Monitor transformation.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/pipeline-transformations/workflow.png){Pipeline transformations (skip-list item)}

![Screenshot of resources for a DCE in the Azure portal.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/data-collection-endpoint-overview/data-collection-endpoint-resources.png){DCE resources}

![Screenshot of the DCR overview page with DCE.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/data-collection-endpoint-overview/data-collection-rule-dce.png){DCR linked to a DCE}

![Screenshot of a diagram that shows resources in one region.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/data-collection-endpoint-overview/data-collection-endpoint-one-region.png){DCE, DCR and workspace in one region}

![Screenshot of a diagram that shows resources in multiple regions.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/data-collection-endpoint-overview/data-collection-endpoint-regionality.png){Multi-region DCE layout (skip-list item)}

![Screenshot of a diagram that shows resources in multiple regions with multiple workspaces.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/data-collection-endpoint-overview/data-collection-endpoint-regionality-multiple-workspaces.png){Multiple regions and workspaces (skip-list item)}

![Screenshot that shows data collection endpoints.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/data-collection-endpoint-overview/data-collection-endpoint-overview.png){Data collection endpoints list}

![Screenshot that shows data collection endpoint basics.](https://learn.microsoft.com/en-us/azure/azure-monitor/data-collection/media/data-collection-endpoint-overview/data-collection-endpoint-basics.png){Create a DCE: Basics}

![Screenshot showing new data collection rule.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/tutorial-logs-ingestion-portal/create-custom-table-portal.png){Create a DCR-based custom table}

![Screenshot showing custom log browse for files.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/tutorial-logs-ingestion-portal/custom-log-browse-files.png){Upload a sample file}

![Screenshot showing custom log data preview.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/tutorial-logs-ingestion-portal/custom-log-data-preview.png){Preview the sample data}

![Screenshot showing initial custom log data query.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/tutorial-logs-ingestion-portal/custom-log-query-01.png){Write the transformation}

![Screenshot showing custom log final schema.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/tutorial-logs-ingestion-portal/custom-log-final-schema.png){Final schema with TimeGenerated}

![Screenshot showing custom log create.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/tutorial-logs-ingestion-portal/custom-log-create.png){Create the table}

![Screenshot that shows the Tables screen for a Log Analytics workspace.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/search-job/search-results-on-log-analytics-tables-screen.png){Tables list}

![Screenshot that shows the Delete Table screen for a table in a Log Analytics workspace.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/search-job/delete-table.png){Delete a custom table}

![Screenshot showing the Schema Editor screen with the Add a column and Delete a column buttons.](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/media/create-custom-table/add-or-delete-column-azure-monitor-logs.png){Schema editor: add or delete columns}
