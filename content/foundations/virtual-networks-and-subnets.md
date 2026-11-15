+++
slug = "virtual-networks-and-subnets"
title = "Virtual networks and subnets"
summary = "Azure networking from zero: address space, subnets and the five reserved IPs, NICs, public vs private IPs, system routes and user-defined routes, peering, NSGs at a glance, service endpoints vs private endpoints, and the hub-and-spoke layout every network objective assumes."
order = 15
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview",
  "https://learn.microsoft.com/en-us/azure/virtual-network/concepts-and-best-practices",
  "https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-faq",
  "https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-network-interface",
  "https://learn.microsoft.com/en-us/azure/virtual-network/ip-services/private-ip-addresses",
  "https://learn.microsoft.com/en-us/azure/virtual-network/ip-services/public-ip-addresses",
  "https://learn.microsoft.com/en-us/azure/virtual-network/ip-services/default-outbound-access",
  "https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview",
  "https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview",
  "https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview",
  "https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview",
  "https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-overview",
  "https://learn.microsoft.com/en-us/azure/architecture/networking/architecture/hub-spoke",
]
captured = 2026-11-15
related_objectives = ["D2.3.1", "D2.3.2", "D2.3.3", "D2.3.4", "D2.3.5", "D2.3.6", "D2.3.7", "D2.3.8", "D2.3.9", "D2.1.2", "D3.2.2", "D3.3.8"]
+++

## In one sentence

A **virtual network (VNet)** is your own private network inside Azure — a range of private IP addresses you carve into **subnets**, where VMs and other resources get addresses, talk to each other, and reach the internet or other networks only along routes and rules you control.

## Why it exists

As an analyst you saw source and destination IPs in every alert. Now you decide which of those packets are even possible. Almost half of this exam's network objectives — NSGs, Azure Firewall, private endpoints, Bastion, VPN, Virtual WAN — are things you **put into** a VNet or **attach to** a subnet. Without the VNet picture, none of them make sense.

Picture an **office building**. The building is the VNet: one address range, walls around it. Each **floor** is a subnet. Each desk's **network socket** is a NIC. The building's **mail room sorting rules** are routes. The **door guards** on each floor are NSGs. A **sky-bridge** to the building next door is peering. Nothing from the street reaches a desk unless a door and a route allow it.

## Mental model

```svg
<svg viewBox="0 0 780 360" role="img" aria-label="A virtual network with address space 10.0.0.0/16 contains two subnets, each with its own NSG and route table. VMs attach through NICs with private IPs; one VM also has a public IP. The VNet peers to a second VNet and reaches the internet through system routes">
  <defs>
    <marker id="vnbarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="line-accent"/>
    </marker>
  </defs>
  <rect class="divider" x="20" y="20" width="520" height="320" rx="10"/>
  <text x="40" y="45" class="label-accent">VNet "app-vnet" 10.0.0.0/16</text>
  <rect class="box" x="40" y="65" width="230" height="250" rx="8"/>
  <text x="155" y="90" text-anchor="middle">Subnet web 10.0.1.0/24</text>
  <rect class="box-risk" x="55" y="105" width="200" height="34" rx="6"/>
  <text x="155" y="127" text-anchor="middle">NSG + route table</text>
  <rect class="box-accent" x="55" y="160" width="200" height="60" rx="6"/>
  <text x="155" y="185" text-anchor="middle">VM web1 (NIC)</text>
  <text x="155" y="207" text-anchor="middle" class="muted">10.0.1.4 + public IP</text>
  <text x="155" y="250" text-anchor="middle" class="muted">.0 .1 .2 .3 .255</text>
  <text x="155" y="272" text-anchor="middle" class="muted">reserved by Azure</text>
  <rect class="box" x="290" y="65" width="230" height="250" rx="8"/>
  <text x="405" y="90" text-anchor="middle">Subnet data 10.0.2.0/24</text>
  <rect class="box-risk" x="305" y="105" width="200" height="34" rx="6"/>
  <text x="405" y="127" text-anchor="middle">NSG + route table</text>
  <rect class="box-accent" x="305" y="160" width="200" height="60" rx="6"/>
  <text x="405" y="185" text-anchor="middle">VM db1 (NIC)</text>
  <text x="405" y="207" text-anchor="middle" class="muted">10.0.2.4, private only</text>
  <rect class="box-ok" x="305" y="240" width="200" height="60" rx="6"/>
  <text x="405" y="265" text-anchor="middle">Private endpoint</text>
  <text x="405" y="287" text-anchor="middle" class="muted">10.0.2.5 → storage</text>
  <rect class="box" x="590" y="60" width="170" height="70" rx="8"/>
  <text x="675" y="90" text-anchor="middle">Internet</text>
  <text x="675" y="112" text-anchor="middle" class="muted">via system route</text>
  <rect class="box" x="590" y="230" width="170" height="70" rx="8"/>
  <text x="675" y="260" text-anchor="middle">Peered VNet</text>
  <text x="675" y="282" text-anchor="middle" class="muted">10.1.0.0/16</text>
  <line class="line-accent" x1="255" y1="180" x2="588" y2="100" marker-end="url(#vnbarrow)"/>
  <line class="line-accent" x1="540" y1="265" x2="588" y2="265" marker-end="url(#vnbarrow)"/>
</svg>
```

## What actually matters

**1. Address space and subnets.** When you create a VNet you give it one or more **address spaces** in CIDR notation (for example `10.0.0.0/16` = 65,536 addresses). Use private ranges (RFC 1918: `10.x`, `172.16–31.x`, `192.168.x`) and ==!never overlap== with on-premises or other VNets you plan to connect — overlapping ranges can't peer or route. A VNet lives in **one region** and one subscription.

You split the space into **subnets**. Rules to remember:

- Azure reserves ==@five addresses in every subnet==: the first four (`.0` network, `.1` default gateway, `.2` and `.3` Azure DNS) and the last (broadcast). A `/24` gives 251 usable.
- Smallest IPv4 subnet: **/29** (3 usable). IPv6 subnets are exactly /64.
- Some services demand a **dedicated, specially named subnet**: `GatewaySubnet` (VPN/ExpressRoute gateways), `AzureFirewallSubnet` (min /26), `AzureBastionSubnet`. Others need a **delegated** subnet (Managed Instance, DNS Private Resolver).
- NSGs and route tables attach **per subnet**.

**2. NICs and IP addresses.** A VM connects through a **network interface (NIC)**. The NIC sits in one subnet and holds IP configurations.

| | Private IP | Public IP |
|---|---|---|
| From | The subnet's range | A separate **public IP address resource** |
| Reachable from | Inside the VNet, peered VNets, connected on-prem | ==!The whole internet== |
| Allocation | Dynamic (default) or static | Standard SKU is static |
| Security default | — | Standard SKU is **closed inbound** until an NSG allows traffic |

A public IP on a VM is the classic exposure: RDP (3389) or SSH (22) open to the internet. Avoid it — use Bastion or JIT (D3.2.2, D3.2.3).

**Default outbound access is being retired.** A VM with no explicit outbound method used to get a hidden Microsoft-owned public IP for outbound traffic. For API versions released ==@after March 31, 2026==, new VNets use **private subnets** by default: VMs need an explicit outbound method (NAT Gateway, load balancer outbound rules, a public IP, or a firewall via UDR). Existing VNets aren't changed.

**3. Routing: system routes and UDRs.** Every subnet gets **system routes** automatically: traffic inside the VNet goes direct, `0.0.0.0/0` goes to the **Internet**, and some private ranges go to **None** (dropped). Peering and gateways add routes. To force traffic somewhere else — usually through a firewall — you create a **route table** with **user-defined routes (UDRs)** and attach it to a subnet. Next hop types you can set:

| Next hop | Meaning |
|---|---|
| **Virtual appliance** | Send to an IP — a firewall or NVA (Azure Firewall's private IP) |
| **Virtual network gateway** | Send to the VPN gateway (on-premises) |
| **Virtual network** | Keep inside the VNet |
| **Internet** | Out to the internet |
| **None** | Drop the traffic |

The longest matching prefix wins; for equal prefixes, a UDR beats BGP routes, which beat system routes. Classic pattern: UDR `0.0.0.0/0 → Virtual appliance 10.0.0.4` on every workload subnet = all egress goes through Azure Firewall (D2.3.8).

**4. Peering.** **Virtual network peering** joins two VNets so they talk over Microsoft's backbone with private IPs, as if one network. **Global peering** crosses regions. ==!Peering is not transitive==: if A peers with B and B with C, A can't reach C unless you add A–C peering or route through an NVA/firewall in B. Peered VNets can share one VPN gateway (**gateway transit**).

**5. NSGs at a glance.** A **network security group** is a list of allow/deny rules on 5-tuple (source, source port, destination, destination port, protocol). Attach to a subnet, a NIC, or both. Priority **100–4096**, lowest number first, first match wins. Default rules sit at 65000–65500: **AllowVNetInBound**, **AllowAzureLoadBalancerInBound**, **DenyAllInBound**; outbound **AllowVNetOutBound**, **AllowInternetOutBound**, **DenyAllOutBound**. NSGs are stateful. Full detail: D2.3.1.

**6. Service endpoints vs private endpoints (preview — taught fully in D2.3.6).** Both keep PaaS traffic off the open internet. They are ==!not the same thing==:

| | **Service endpoint** | **Private endpoint** |
|---|---|---|
| What it is | A setting on a subnet | A NIC with a private IP in your subnet |
| Service is reached at | Its **public** IP, over the Azure backbone | A **private** IP from your VNet |
| Service sees | Your subnet's identity → storage/SQL "VNet rule" | A private connection to one specific resource |
| Works from on-premises / peered VNets | No | ==+Yes== |
| Needs DNS changes | No | ==@Yes — private DNS zone== (see `private-dns-and-name-resolution`) |
| Data exfiltration to other accounts | Possible without endpoint policies | Scoped to one resource |

**7. Hub and spoke.** The standard enterprise layout. A **hub** VNet holds shared things: Azure Firewall, VPN/ExpressRoute gateway, Bastion, DNS resolver. Each workload gets a **spoke** VNet peered to the hub. Spokes don't peer with each other; spoke-to-spoke traffic goes through the hub firewall via UDRs (or Azure Virtual Network Manager can build direct links). Virtual WAN is Microsoft's managed version of the hub (D2.3.3).

## Where it lives

**Azure portal > Virtual networks > *vnet***: `Settings > Address space`, `Subnets`, `Peerings`, `DNS servers`, `Service endpoints`, `Private endpoints`. Route tables (`Route tables > Routes / Subnets`) and NSGs (`Network security groups`) are their own resources you then associate. A NIC's `Effective routes` and `Effective security rules` show what's really applied — the first stop when traffic doesn't flow.

```bash
az network vnet create -g rg -n app-vnet --address-prefixes 10.0.0.0/16 \
  --subnet-name web --subnet-prefixes 10.0.1.0/24
az network route-table route create -g rg --route-table-name rt-spoke -n default \
  --address-prefix 0.0.0.0/0 --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.0.4
az network vnet peering create -g rg -n spoke-to-hub --vnet-name spoke --remote-vnet hub --allow-vnet-access
az network nic show-effective-route-table -g rg -n web1-nic -o table
```

![Hub-spoke network topology](https://learn.microsoft.com/en-us/azure/architecture/networking/architecture/_images/hub-spoke-network-topology-architecture.svg){Hub and spoke — shared firewall, gateway and Bastion in the hub; workloads in peered spokes}

![Virtual network service endpoints](https://learn.microsoft.com/en-us/azure/virtual-network/media/virtual-network-service-endpoints-overview/vnet_service_endpoints_overview.png){Service endpoint — the subnet reaches the service's public endpoint over the backbone, and the service firewall allows that subnet}

## Exam traps

- ==!Peering is not transitive.== Spoke-to-spoke needs a hub firewall + UDRs, direct peering, or AVNM connectivity.
- Five reserved IPs per subnet — "how many VMs fit in a /28?" → 16 − 5 = **11**.
- Force egress through a firewall → **UDR 0.0.0.0/0, next hop Virtual appliance**, attached to the *workload* subnet, not the firewall subnet.
- Service endpoint ≠ private endpoint. On-premises access to a PaaS resource privately → **private endpoint**.
- A VNet can't span regions; connect regions with **global peering**.
- Overlapping address spaces can't be peered.
- Standard public IPs are closed inbound by default; an NSG must allow traffic.

## Skip

- **Skip** IPv6 dual-stack detail, public IP prefixes, and Basic vs Standard public IP SKU history.
- **Skip** NIC features such as accelerated networking and IP forwarding (except: an NVA needs IP forwarding on).
- **Skip** BGP and route propagation internals beyond "gateway routes appear automatically".
- **Skip** subnet delegation lists, VNet encryption, and VNet limits tables.
- **Skip** NAT Gateway SNAT port maths. Know NAT Gateway = explicit outbound method.

## Screenshots

![Routing example network implementation](https://learn.microsoft.com/en-us/azure/virtual-network/media/virtual-networks-udr-overview/routing-example.png){Routing example — UDRs send subnet traffic through a virtual appliance}

![Virtual network peering transit](https://learn.microsoft.com/en-us/azure/virtual-network/media/virtual-networks-peering-overview/local-or-remote-gateway-in-peered-virual-network.png){Gateway transit — peered VNets share the hub's VPN gateway}

![Default peering routes vs route suppression](https://learn.microsoft.com/en-us/azure/virtual-network/media/virtual-networks-peering-overview/disable-peering-route-propagation.png){Suppressing peering routes so on-premises traffic is forced through the hub firewall}

![Private endpoint workflow approval](https://learn.microsoft.com/en-us/azure/private-link/media/private-endpoint-overview/private-link-paas-workflow.png){Private endpoint connections need approval from the resource owner (automatic if you own both)}

![Routing between spokes through Azure Firewall](https://learn.microsoft.com/en-us/azure/architecture/networking/architecture/_images/spoke-spoke-routing.svg){Spoke-to-spoke via the hub firewall}

![Virtual Network Manager direct spoke connectivity](https://learn.microsoft.com/en-us/azure/architecture/networking/architecture/_images/spoke-spoke-azure-virtual-network-manager.svg){Azure Virtual Network Manager can create direct spoke links (D2.3.2)}

![Spoke global mesh across regions](https://learn.microsoft.com/en-us/azure/architecture/networking/architecture/_images/hub-spoke.svg){Hubs in several regions with global mesh}

![Explicit outbound options](https://learn.microsoft.com/en-us/azure/virtual-network/ip-services/media/default-outbound-access/explicit-outbound-options.png){Explicit outbound methods — NAT Gateway, load balancer, public IP}

![Decision tree for default outbound access](https://learn.microsoft.com/en-us/azure/virtual-network/ip-services/media/default-outbound-access/decision-tree-load-balancer.png){Does this VM still use default outbound access?}

![Network interface overview](https://learn.microsoft.com/en-us/azure/virtual-network/media/virtual-network-network-interface/nic-overview.png){A NIC is its own resource}

![Network interface IP configurations](https://learn.microsoft.com/en-us/azure/virtual-network/media/virtual-network-network-interface/ip-configurations.png){IP configurations — private IP, optional public IP}

![NIC DNS server configuration](https://learn.microsoft.com/en-us/azure/virtual-network/media/virtual-network-network-interface/dns-servers.png){DNS servers can be set per NIC (overrides the VNet setting)}

![NIC network security group configuration](https://learn.microsoft.com/en-us/azure/virtual-network/media/virtual-network-network-interface/network-security-group.png){NSG associated at NIC level}

![Network interface properties](https://learn.microsoft.com/en-us/azure/virtual-network/media/virtual-network-network-interface/nic-properties.png){NIC properties}

![Effective security rules](https://learn.microsoft.com/en-us/azure/virtual-network/media/virtual-network-network-interface/effective-security-rules.png){Effective security rules — subnet NSG + NIC NSG combined}

![Effective routes](https://learn.microsoft.com/en-us/azure/virtual-network/media/virtual-network-network-interface/effective-routes.png){Effective routes — system routes plus UDRs actually in force}

![NIC application security group configuration](https://learn.microsoft.com/en-us/azure/virtual-network/media/virtual-network-network-interface/application-security-group.png){Application security group membership is set on the NIC (D2.3.1)}
