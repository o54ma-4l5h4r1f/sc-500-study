+++
slug = "azure-sql-basics"
title = "Azure SQL basics"
summary = "The Azure SQL family for a security engineer: logical server vs database, SQL Database vs SQL Managed Instance vs SQL Server on a VM, the server firewall and 'Allow Azure services', the Microsoft Entra admin, SQL authentication vs Entra authentication, and where TDE and auditing are set."
order = 14
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/azure-sql/azure-sql-iaas-vs-paas-what-is-overview",
  "https://learn.microsoft.com/en-us/azure/azure-sql/database/sql-database-paas-overview",
  "https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/sql-managed-instance-paas-overview",
  "https://learn.microsoft.com/en-us/azure/azure-sql/database/logical-servers",
  "https://learn.microsoft.com/en-us/azure/azure-sql/database/firewall-configure",
  "https://learn.microsoft.com/en-us/azure/azure-sql/database/network-access-controls-overview",
  "https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-aad-overview",
  "https://learn.microsoft.com/en-us/azure/azure-sql/database/authentication-azure-ad-only-authentication",
  "https://learn.microsoft.com/en-us/azure/azure-sql/database/transparent-data-encryption-tde-overview",
  "https://learn.microsoft.com/en-us/azure/azure-sql/database/auditing-overview",
  "https://learn.microsoft.com/en-us/azure/azure-sql/database/auditing-server-level-database-level",
]
captured = 2026-11-15
related_objectives = ["D2.2.1", "D2.2.2", "D2.2.3", "D2.3.6"]
+++

## In one sentence

**Azure SQL** is Microsoft's family of SQL Server databases in the cloud; for Azure SQL Database the security settings you care about — firewall, Entra admin, auditing, TDE key, Defender for SQL — sit mostly on a parent object called the **logical server**, not on the database itself.

## Why it exists

Databases hold the data attackers want most: customer records, payment details, credentials. Running SQL Server yourself means patching Windows, patching SQL, taking backups and configuring encryption by hand. Azure SQL takes over some or all of that work. How much it takes over decides **which security knobs you still own**.

Think of **three ways to run a restaurant kitchen**. Rent an empty building and fit it yourself (**SQL Server on a VM**). Rent a fitted kitchen with staff who maintain the ovens, but you still run your own brigade and recipes (**SQL Managed Instance**). Or order meals from a catering service: you choose the menu, they handle everything else (**SQL Database**).

## Mental model

| | **SQL Server on Azure VM** | **Azure SQL Managed Instance** | **Azure SQL Database** |
|---|---|---|---|
| Model | IaaS — you own the OS | PaaS — near-full SQL Server instance | PaaS — single database or elastic pool |
| Lives in your VNet? | Yes (it's a VM) | ==@Yes — always deployed into a subnet== | No — public endpoint by default; add a private endpoint |
| Parent object | The VM | The managed instance | The **logical server** |
| OS patching | You | Microsoft | Microsoft |
| Windows authentication | Yes | Yes (Windows auth for Entra principals) | ==!No== |
| SQL Agent, cross-database queries, CLR | Yes | Yes | No |
| Auditing | SQL Server Audit | SQL Server Audit to Storage / Log Analytics / Event Hubs | Azure SQL Auditing (server or database level) |
| TDE on by default | No | Yes for new databases | Yes for new databases |

```svg
<svg viewBox="0 0 760 330" role="img" aria-label="An Azure SQL logical server sits above its databases. Server-level settings such as firewall, Entra admin, auditing, TDE protector and Defender for SQL are inherited by every database on the server">
  <defs>
    <marker id="sqlbarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="line-accent"/>
    </marker>
  </defs>
  <rect class="box-accent" x="160" y="20" width="440" height="150" rx="8"/>
  <text x="380" y="46" text-anchor="middle" class="label-accent">Logical server: myserver.database.windows.net</text>
  <text x="180" y="78">Firewall rules + public network access</text>
  <text x="180" y="100">Microsoft Entra admin (one user or group)</text>
  <text x="180" y="122">Server auditing policy</text>
  <text x="180" y="144">TDE protector key + Defender for SQL</text>
  <rect class="box" x="60" y="230" width="180" height="70" rx="8"/>
  <text x="150" y="260" text-anchor="middle">Database A</text>
  <text x="150" y="282" text-anchor="middle" class="muted">inherits settings</text>
  <rect class="box" x="290" y="230" width="180" height="70" rx="8"/>
  <text x="380" y="260" text-anchor="middle">Database B</text>
  <text x="380" y="282" text-anchor="middle" class="muted">+ own DB firewall rule</text>
  <rect class="box" x="520" y="230" width="180" height="70" rx="8"/>
  <text x="610" y="260" text-anchor="middle">master</text>
  <text x="610" y="282" text-anchor="middle" class="muted">logins, server rules</text>
  <line class="line-accent" x1="300" y1="170" x2="160" y2="228" marker-end="url(#sqlbarrow)"/>
  <line class="line-accent" x1="380" y1="170" x2="380" y2="228" marker-end="url(#sqlbarrow)"/>
  <line class="line-accent" x1="460" y1="170" x2="600" y2="228" marker-end="url(#sqlbarrow)"/>
</svg>
```

## What actually matters

**1. The logical server is not a SQL Server instance.** In the portal it's called **SQL server**, which confuses everyone. It's a management container: it gives databases a connection name (`<server>.database.windows.net`), and it holds logins, firewall rules, auditing, threat detection and failover groups. Delete the server and its databases go with it. Databases inherit Azure RBAC from it. It exposes no instance-level features — that's what Managed Instance is for.

**2. The firewall blocks everything until you open it.** All connections to SQL Database are refused until a rule allows them. Clients connect on **TCP 1433**.

| | **Server-level IP rule** | **Database-level IP rule** |
|---|---|---|
| Applies to | Every database on the server | One database |
| Stored in | `master` | That database |
| Created with | Portal, PowerShell, CLI, or T-SQL `sp_set_firewall_rule` | ==!T-SQL only== (`sp_set_database_firewall_rule`) |
| Limit | 256 per server | — |

The database rule is checked first; if no match, the server rule. Changes can take up to **5 minutes** to apply.

**3. "Allow Azure services and resources to access this server" is wider than it sounds.** Turning it on adds a rule from `0.0.0.0` to `0.0.0.0` named **AllowAllWindowsAzureIps**. It lets in connections from ==!every Azure resource in every customer's subscription==, not just yours. It's **off** by default for new servers created in the portal. Prefer virtual network rules (service endpoints) or a private endpoint (see `virtual-networks-and-subnets`). **Public network access** can be set to **Disabled** so only private endpoints work.

**4. Two ways to authenticate.**

| | **SQL authentication** | **Microsoft Entra authentication** |
|---|---|---|
| Identity | Username + password stored in SQL | Entra user, group, service principal or managed identity |
| First admin | The **server admin** login you set at creation | The **Microsoft Entra admin** you set on the server |
| MFA, Conditional Access, central disable | No | ==+Yes== |
| Passwordless apps | No | Yes (managed identities) |

- Each server (or managed instance) has ==@exactly one Microsoft Entra admin==. Make it a **group** so several people can administer without sharing an account.
- **Microsoft Entra-only authentication** turns SQL authentication off for the whole server, including the original server admin login.
- Users inside a database can be **contained database users** mapped to Entra identities (`CREATE USER [name] FROM EXTERNAL PROVIDER`).

**5. Where TDE lives.** **Transparent data encryption** encrypts the database files, backups and logs at rest. It's ==+on by default== for every new SQL Database (and new Managed Instance databases). The database encryption key is protected by the **TDE protector**: a service-managed certificate by default, or a **customer-managed key** in Key Vault. For SQL Database the protector is set on the **logical server** and inherited by its databases (database-level CMK also exists). TDE can't encrypt the `master` system database.

**6. Where auditing lives.** Azure SQL Auditing writes events to an **Azure Storage account**, a **Log Analytics workspace** or **Event Hubs** — any combination. A **server-level** policy applies to all existing and new databases and ==!always applies==, whatever the database setting says. A database-level policy runs *alongside* it (double auditing), it doesn't override it. Sending to Log Analytics or Event Hubs creates a diagnostic setting with the `SQLSecurityAuditEvents` category — delete it and auditing stops silently.

**7. Managed Instance is a VNet citizen.** It's deployed into a dedicated **subnet** in your VNet, so it's private by default; an optional public endpoint uses port **3342**. Its security settings sit on the instance itself.

## Where it lives

**Azure portal** (the hub at `Azure SQL` groups all three products):

- `SQL servers > <server> > Security > Networking` — Public network access (Disabled / Selected networks), firewall rules, virtual network rules, **Allow Azure services…**, private endpoint connections.
- `SQL servers > <server> > Settings > Microsoft Entra ID` — **Set admin**, *Support only Microsoft Entra authentication for this server*.
- `SQL servers > <server> > Security > Auditing` (server policy) and `SQL databases > <db> > Security > Auditing` (database policy).
- `SQL servers > <server> > Security > Transparent data encryption` — service-managed vs customer-managed key.
- `SQL servers > <server> > Security > Microsoft Defender for Cloud` — Defender for SQL status.

```bash
az sql server firewall-rule create -g rg -s myserver -n office --start-ip-address 203.0.113.10 --end-ip-address 203.0.113.10
az sql server ad-admin create -g rg -s myserver --display-name "SQL Admins" --object-id <groupObjectId>
az sql server ad-only-auth enable -g rg -n myserver
az sql server update -g rg -n myserver --enable-public-network false
```

![Azure SQL Database firewall diagram](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/firewall-configure/sqldb-firewall-1.png?view=azuresql){Every connection — from the internet or from Azure — passes the server firewall first}

![Manage server firewall settings page](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/network-access-controls-overview/manage-server-firewall.png?view=azuresql){Networking — the "Allow Azure services and resources to access this server" checkbox lives here}

## Exam traps

- "Firewall rule for one database only" → **database-level rule via T-SQL**. The portal can't create it.
- "Allow only *our* Azure app" → ==!not== "Allow Azure services" (that admits every tenant's resources). Use a VNet rule or private endpoint.
- "Enforce MFA / disable SQL logins" → **Microsoft Entra-only authentication** with an Entra admin group.
- Only **one** Entra admin per server — use a group.
- TDE is already on for new databases; the exam question is usually *who holds the key* (service-managed vs CMK).
- Server-level auditing can't be switched off for one database by a database setting.
- Windows authentication → **Managed Instance** or **SQL on VM**, never SQL Database.

## Skip

- **Skip** DTU vs vCore purchasing models, service tiers, elastic pool sizing and serverless compute. Cost and performance, not security.
- **Skip** high-availability architecture, geo-replication and failover groups beyond knowing failover groups are a server-level setting.
- **Skip** the T-SQL differences list for Managed Instance and connection policy (proxy vs redirect).
- **Skip** Import/Export and Data Sync caveats around "Allow Azure services".
- **Skip** column-level features (Always Encrypted, dynamic data masking, row-level security) here; D2.2.1 covers what's tested.

## Screenshots

![Azure SQL portal hub](https://learn.microsoft.com/en-us/azure/azure-sql/media/azure-sql-iaas-vs-paas-what-is-overview/azure-sql-hub.png?view=azuresql){The Azure SQL hub groups SQL Database, Managed Instance and SQL VMs}

![Hybrid cloud SQL Server options](https://learn.microsoft.com/en-us/azure/azure-sql/media/azure-sql-iaas-vs-paas-what-is-overview/hybrid-cloud-sql-server.png?view=azuresql){IaaS vs PaaS — how much you manage}

![Find the right option wizard](https://learn.microsoft.com/en-us/azure/azure-sql/includes/media/sql-database-create-manage-portal/find-the-right-option.png?view=azuresql){Portal helper for choosing a deployment option}

![Azure SQL product comparison page](https://learn.microsoft.com/en-us/azure/azure-sql/includes/media/sql-database-create-manage-portal/product-comparison-details.png?view=azuresql){Product comparison details}

![Elastic pools in basic, standard and premium editions](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/sql-database-paas-overview/sqldb_elastic_pools.png?view=azuresql){Elastic pools — many databases sharing resources (skip the sizing)}

![Azure monitoring architecture for SQL Database](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/sql-database-paas-overview/architecture.png?view=azuresql){Monitoring architecture — metrics and logs flow to Azure Monitor}

![Key features for Azure SQL Managed Instance](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/media/sql-managed-instance-paas-overview/key-features.png?view=azuresql){Managed Instance key features}

![Managed Instance application deployment topologies](https://learn.microsoft.com/en-us/azure/azure-sql/managed-instance/media/sql-managed-instance-paas-overview/application-deployment-topologies.png?view=azuresql){Managed Instance sits in your VNet — apps reach it privately}

![First steps to create a new SQL Database](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/logical-servers/create-database-1.png?view=azuresql){Creating a database also creates (or picks) its logical server}

![Copy server name and Set server firewall on the toolbar](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/firewall-create-server-level-portal-quickstart/server-name.png?view=azuresql){Database overview — Set server firewall}

![Configuring a server-level IP firewall rule](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/firewall-create-server-level-portal-quickstart/server-firewall-rule.png?view=azuresql){Add a server-level IP rule}

![Set server firewall setting in the Azure portal](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/firewall-configure/sql-database-server-set-firewall-rule.png?view=azuresql){Set server firewall}

![Networking page for server-level IP firewall rules](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/firewall-configure/sql-database-server-firewall-settings.png?view=azuresql){Networking page — rules list}

![Microsoft Entra authentication diagram for Azure SQL](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/authentication-aad-overview/azure-ad-authentication-diagram.png?view=azuresql){Entra authentication — the database trusts tokens from your tenant}

![Relationship between subscriptions and the Entra tenant](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/authentication-aad-overview/2-subscription-relationship.png?view=azuresql){The server must trust the tenant its subscription belongs to}

![Administrator structure for Microsoft Entra ID with SQL](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/authentication-aad-overview/3-admin-structure.png?view=azuresql){Admin structure — one SQL server admin login plus one Entra admin (user or group)}

![Activity log entry for Microsoft Entra-only authentication](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/authentication-azure-ad-only-authentication/azure-ad-only-authentication-portal-sql-server-activity-log.png?view=azuresql){Turning Entra-only authentication on or off is recorded in the Activity log}

![Transparent data encryption menu for a server](https://learn.microsoft.com/en-us/azure/azure-sql/database/media/transparent-data-encryption-tde-overview/tde-byok-support.png?view=azuresql){Server > Transparent data encryption — choose service-managed or customer-managed key}
