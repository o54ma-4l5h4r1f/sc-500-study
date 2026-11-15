+++
slug = "private-dns-and-name-resolution"
title = "Private DNS and name resolution"
summary = "DNS for engineers who only ever read DNS logs: Azure-provided DNS at 168.63.129.16, private DNS zones, privatelink.* zones and their CNAME trick, virtual network links (registration vs resolution), why private endpoints break without DNS, and how on-premises resolves Azure names through conditional forwarders or Azure DNS Private Resolver."
order = 16
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances",
  "https://learn.microsoft.com/en-us/azure/virtual-network/what-is-ip-address-168-63-129-16",
  "https://learn.microsoft.com/en-us/azure/dns/private-dns-overview",
  "https://learn.microsoft.com/en-us/azure/dns/private-dns-virtual-network-links",
  "https://learn.microsoft.com/en-us/azure/dns/private-dns-autoregistration",
  "https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-dns",
  "https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-dns-integration",
  "https://learn.microsoft.com/en-us/azure/storage/common/storage-private-endpoints",
  "https://learn.microsoft.com/en-us/azure/dns/dns-private-resolver-overview",
]
captured = 2026-11-15
related_objectives = ["D2.3.6", "D2.3.7", "D2.3.5", "D2.3.8", "D2.1.2", "D1.2.4"]
+++

## In one sentence

**DNS** turns names like `mystg.blob.core.windows.net` into IP addresses; in Azure, **private DNS zones** linked to your VNets make those same names return **private** IPs inside your network, which is the piece that makes private endpoints actually get used.

## Why it exists

You've hunted through DNS logs for beaconing domains. Now you're the one who decides what a name resolves to.

Here is the problem. You create a **private endpoint** for a storage account: a NIC in your subnet with IP `10.0.2.5` (see `virtual-networks-and-subnets`). Your app still connects to `mystg.blob.core.windows.net` — the connection string doesn't change. If DNS still answers with the storage account's **public** IP, the app goes out over the internet, the private endpoint sits unused, and if you've disabled public access the app simply gets **403** or times out. ==!A private endpoint without the right DNS does nothing.==

Think of a **company phone directory**. The public phone book lists the head-office switchboard number for "Accounts". Inside the building, the internal directory lists Accounts' extension. Same name, different number depending on which directory you look in. Private DNS zones are the internal directory.

## Mental model

```svg
<svg viewBox="0 0 780 360" role="img" aria-label="Name resolution for a storage account with a private endpoint: inside a linked VNet, Azure DNS follows the CNAME to privatelink.blob.core.windows.net and the private DNS zone returns 10.0.2.5. From the internet, the same CNAME resolves to the public IP">
  <defs>
    <marker id="pdnbarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="line-accent"/>
    </marker>
  </defs>
  <rect class="box" x="20" y="20" width="740" height="56" rx="8"/>
  <text x="390" y="44" text-anchor="middle">Public DNS: mystg.blob.core.windows.net</text>
  <text x="390" y="66" text-anchor="middle" class="muted">CNAME → mystg.privatelink.blob.core.windows.net</text>
  <rect class="divider" x="20" y="100" width="420" height="240" rx="10"/>
  <text x="40" y="124" class="label-accent">Your VNet (linked to the zone)</text>
  <rect class="box" x="40" y="140" width="170" height="56" rx="8"/>
  <text x="125" y="164" text-anchor="middle">VM / app</text>
  <text x="125" y="184" text-anchor="middle" class="muted">asks for mystg…</text>
  <rect class="box" x="240" y="140" width="180" height="56" rx="8"/>
  <text x="330" y="164" text-anchor="middle">Azure DNS</text>
  <text x="330" y="184" text-anchor="middle" class="muted">168.63.129.16</text>
  <rect class="box-ok" x="40" y="230" width="380" height="90" rx="8"/>
  <text x="230" y="256" text-anchor="middle">Private DNS zone</text>
  <text x="230" y="278" text-anchor="middle">privatelink.blob.core.windows.net</text>
  <text x="230" y="300" text-anchor="middle" class="label-accent">A  mystg → 10.0.2.5</text>
  <line class="line-accent" x1="210" y1="168" x2="238" y2="168" marker-end="url(#pdnbarrow)"/>
  <line class="line-accent" x1="330" y1="196" x2="330" y2="228" marker-end="url(#pdnbarrow)"/>
  <rect class="box-risk" x="480" y="140" width="280" height="56" rx="8"/>
  <text x="620" y="164" text-anchor="middle" class="label-risk">Internet client</text>
  <text x="620" y="184" text-anchor="middle" class="muted">no access to private zone</text>
  <rect class="box" x="480" y="230" width="280" height="90" rx="8"/>
  <text x="620" y="256" text-anchor="middle">Public answer</text>
  <text x="620" y="278" text-anchor="middle">public IP of storage</text>
  <text x="620" y="300" text-anchor="middle" class="muted">blocked if public access off</text>
  <line class="line-accent" x1="620" y1="196" x2="620" y2="228" marker-end="url(#pdnbarrow)"/>
</svg>
```

![Overview of private endpoints for Azure Storage](https://learn.microsoft.com/en-us/azure/storage/common/media/storage-private-endpoints/storage-private-endpoints-overview.jpg){Storage private endpoint — clients in the VNet reach the account on a private IP; the same name resolves publicly outside}

## What actually matters

**1. DNS in three sentences.** A **record** maps a name to something: an **A record** maps a name to an IPv4 address; a **CNAME** says "this name is an alias — go look up that other name". A **zone** is a container of records for one domain (for example `contoso.com`). A **resolver** is the server your machine asks; it walks the chain until it gets an IP.

**2. Azure-provided DNS = 168.63.129.16.** Every VNet, by default, sends DNS queries to the virtual IP ==@168.63.129.16==. It resolves internet names and any **private DNS zone linked to that VNet**. It's reachable ==!only from inside an Azure VNet== — an on-premises server can't query it directly. You can override a VNet's (or NIC's) DNS servers with your own (custom DNS, for example a domain controller); then *your* server must forward Azure names to 168.63.129.16.

**3. Private DNS zones and virtual network links.** A **private DNS zone** is a zone only VNets you link can see. A zone does nothing until you add a **virtual network link**.

| Link type | Created when | Effect |
|---|---|---|
| **Resolution** link | Link without autoregistration | VMs in the VNet can **look up** records in the zone |
| **Registration** link | Link with **autoregistration** on | VMs in the VNet also get **A records created automatically** |

Limits that get tested: a VNet can have ==@only one registration zone==, but many resolution zones. A zone can have many linked VNets. Only one link between a given zone and a given VNet.

**4. `privatelink.*` zones and the CNAME trick.** When a private endpoint exists, Azure changes the service's public DNS so `mystg.blob.core.windows.net` becomes a **CNAME** to `mystg.privatelink.blob.core.windows.net`. Then:

- Inside a VNet linked to a private zone called **`privatelink.blob.core.windows.net`**, that name hits an **A record → 10.0.2.5** (the private endpoint).
- Outside, the `privatelink` name keeps resolving publicly to the public IP.

So the connection string never changes. Each service has its own recommended zone name:

| Service | Private DNS zone |
|---|---|
| Blob storage | `privatelink.blob.core.windows.net` |
| Azure Files | `privatelink.file.core.windows.net` |
| Azure SQL Database | `privatelink.database.windows.net` |
| Key Vault | `privatelink.vaultcore.azure.net` |

==!Use exactly the recommended name.== Records are created automatically only when you use it, and you need **one zone per service type** — don't mix services in one zone. When you tick "Integrate with private DNS zone" while creating the endpoint, Azure creates a **private DNS zone group** that adds and removes the A records for you.

**5. Hub-and-spoke: one zone, many links.** Create each `privatelink` zone **once** (usually in the hub) and link it to the hub and every spoke VNet whose clients need it. Creating duplicate zones of the same name in different VNets means records end up in one and not the other.

**6. On-premises clients need a forwarder.** On-premises DNS can't reach 168.63.129.16, so it needs a hop **inside** Azure:

| Option | How it works |
|---|---|
| **DNS forwarder VM** (or Azure Firewall DNS proxy) in a linked VNet | On-prem DNS conditionally forwards to it; it forwards to 168.63.129.16 |
| **Azure DNS Private Resolver** (managed, no VMs) | **Inbound endpoint** = a private IP in a dedicated subnet that on-prem DNS forwards to. **Outbound endpoint** + **DNS forwarding ruleset** = send Azure VMs' queries for on-prem domains *to* on-prem DNS |

The on-premises **conditional forwarder** must target the **public** service zone — `blob.core.windows.net` or `database.windows.net` — ==!not the `privatelink.` name==. Private Resolver endpoint subnets must be delegated to `Microsoft.Network/dnsResolvers`, minimum **/28**, and used for nothing else. A ruleset holds up to **1,000** rules.

## Where it lives

**Azure portal:**

- `Private DNS zones > + Create` — zone name such as `privatelink.blob.core.windows.net` (the zone is global, not regional).
- `Private DNS zones > <zone> > DNS Management > Virtual Network Links > + Add` — pick the VNet; tick **Enable auto registration** only for a VM-name zone, never for a `privatelink` zone.
- `Private endpoints > <pe> > Settings > DNS configuration` — shows the FQDN, private IP and zone group.
- `Virtual networks > <vnet> > Settings > DNS servers` — Default (Azure-provided) or Custom.
- `DNS private resolvers > + Create` — inbound/outbound endpoints and rulesets.

```bash
az network private-dns zone create -g rg -n privatelink.blob.core.windows.net
az network private-dns link vnet create -g rg -z privatelink.blob.core.windows.net \
  -n link-hub -v hub-vnet --registration-enabled false
az network private-endpoint dns-zone-group create -g rg --endpoint-name pe-stg \
  -n default --private-dns-zone privatelink.blob.core.windows.net --zone-name blob
nslookup mystg.blob.core.windows.net    # from a VM in the VNet: expect 10.x, via privatelink CNAME
```

![Single virtual network with Azure-provided DNS](https://learn.microsoft.com/en-us/azure/private-link/media/private-endpoint-dns/single-vnet-azure-dns.png){Simplest case — one VNet, zone linked, Azure DNS answers with the private IP}

![Azure DNS Private Resolver architecture](https://learn.microsoft.com/en-us/azure/dns/media/dns-resolver-overview/resolver-architecture.png){Private Resolver — inbound endpoint for on-prem → Azure, outbound endpoint + ruleset for Azure → on-prem}

## Exam traps

- Private endpoint works from one VNet but not a peered spoke → the spoke VNet ==+isn't linked== to the `privatelink` zone.
- On-premises resolves the public IP → no conditional forwarder to an Azure-side forwarder or **Private Resolver inbound endpoint**. Forward `blob.core.windows.net`, not `privatelink.blob.core.windows.net`.
- "On-prem DNS should forward to 168.63.129.16" → ==!wrong==; that IP only answers inside Azure.
- Custom DNS on a VNet (a domain controller) must forward Azure zones to 168.63.129.16, or private endpoints break.
- One **registration** zone per VNet; many **resolution** zones.
- Wrong zone name (`blob.core.windows.net` instead of `privatelink.blob.core.windows.net`) → you've hijacked all public blob resolution for that VNet.
- Public DNS resolving the `privatelink` CNAME from the internet is normal — DNS doesn't grant access.

## Skip

- **Skip** public Azure DNS zones (hosting `contoso.com` for the internet), DNSSEC and record types beyond A and CNAME.
- **Skip** the full table of every service's `privatelink` zone name. Know the four above and the pattern.
- **Skip** Private Resolver's architecture variants (centralized vs distributed rulesets) beyond inbound vs outbound.
- **Skip** fallback-to-internet on VNet links and DNS resolver policies.
- **Skip** reverse DNS and DNS suffix behaviour on VMs.

## Screenshots

![Private DNS scenario across virtual networks](https://learn.microsoft.com/en-us/azure/dns/media/private-dns-overview/scenario.png){Private DNS zone shared by several linked VNets}

![Enable auto registration on the add virtual network link page](https://learn.microsoft.com/en-us/azure/dns/media/privatedns-concepts/enable-autoregistration.png){Adding a VNet link — the auto registration tick box makes it a registration link}

![Hub and spoke with Azure-provided DNS](https://learn.microsoft.com/en-us/azure/private-link/media/private-endpoint-dns/hub-and-spoke-azure-dns.png){Hub and spoke — one zone, linked to every VNet that needs it}

![On-premises forwarding to Azure DNS without Private Resolver](https://learn.microsoft.com/en-us/azure/private-link/media/private-endpoint-dns/on-premises-forwarding-to-azure-no-resolver.png){On-prem → DNS forwarder VM in Azure → 168.63.129.16}

![On-premises using Azure private DNS zone through Private Resolver](https://learn.microsoft.com/en-us/azure/private-link/media/private-endpoint-dns/on-premises-using-azure-dns.png){On-prem clients pointed at the Private Resolver inbound endpoint}

![On-premises forwarding to Azure DNS via Private Resolver](https://learn.microsoft.com/en-us/azure/private-link/media/private-endpoint-dns/on-premises-forwarding-to-azure.png){Existing on-prem DNS conditionally forwards to the inbound endpoint}

![Hybrid scenario with private DNS zone](https://learn.microsoft.com/en-us/azure/private-link/media/private-endpoint-dns/hybrid-scenario.png){Hybrid — on-prem and peered VNets all resolve through one resolver and one zone}
