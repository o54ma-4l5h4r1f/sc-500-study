+++
slug = "virtual-machines-and-disks"
title = "Virtual machines and disks"
summary = "What an Azure VM is made of — image, size, managed OS and data disks, temp disk, Generation 1 vs 2, the VM agent and extensions — the three security types (Standard, Trusted launch, Confidential) at a glance, and why a public RDP or SSH port is the first thing attackers find."
order = 17
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/virtual-machines/overview",
  "https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/overview",
  "https://learn.microsoft.com/en-us/azure/virtual-machines/managed-disks-overview",
  "https://learn.microsoft.com/en-us/azure/virtual-machines/disks-types",
  "https://learn.microsoft.com/en-us/azure/virtual-machines/generation-2",
  "https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/overview",
  "https://learn.microsoft.com/en-us/azure/virtual-machines/extensions/agent-windows",
  "https://learn.microsoft.com/en-us/azure/virtual-machines/trusted-launch",
  "https://learn.microsoft.com/en-us/azure/confidential-computing/confidential-vm-overview",
  "https://learn.microsoft.com/en-us/azure/confidential-computing/virtual-machine-options",
  "https://learn.microsoft.com/en-us/azure/virtual-machines/disk-encryption",
]
captured = 2026-11-15
related_objectives = ["D3.2.1", "D3.2.2", "D3.2.3", "D3.2.5", "D3.2.6", "D3.2.7", "D3.2.8", "D3.2.9", "D4.1.5"]
+++

## In one sentence

An **Azure virtual machine (VM)** is a computer you rent by the minute: Azure gives it a **size** (CPU and memory), boots it from an **image** onto a **managed OS disk**, plugs it into a subnet through a NIC, and lets you add software remotely through **extensions** run by the **VM agent**.

## Why it exists

As an analyst, VMs were hostnames in `SecurityEvent` and device pages in Defender. Now you configure what they're born with: how they boot, how their disks are encrypted, whether they're reachable from the internet, and which security agents get pushed onto them. A VM is **IaaS** (infrastructure as a service) — Microsoft runs the physical host; ==!you own the operating system==, its patches and its exposure.

Think of **renting a furnished flat**. The landlord owns the building and the wiring (Azure host). You choose the flat size (VM size) and furniture pack (image), and you get keys to the front door (RDP/SSH). Whether you leave that front door open to the street is your decision.

## Mental model

```svg
<svg viewBox="0 0 760 340" role="img" aria-label="An Azure VM resource is built from an image and a size. It has a managed OS disk, optional managed data disks, a non-persistent temp disk, a NIC in a subnet, a security type, and a VM agent that installs extensions">
  <defs>
    <marker id="vmdbarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="line-accent"/>
    </marker>
  </defs>
  <rect class="box" x="20" y="30" width="160" height="56" rx="8"/>
  <text x="100" y="54" text-anchor="middle">Image</text>
  <text x="100" y="74" text-anchor="middle" class="muted">Marketplace / gallery</text>
  <rect class="box" x="20" y="110" width="160" height="56" rx="8"/>
  <text x="100" y="134" text-anchor="middle">Size</text>
  <text x="100" y="154" text-anchor="middle" class="muted">e.g. D2s_v5</text>
  <rect class="box" x="20" y="190" width="160" height="56" rx="8"/>
  <text x="100" y="214" text-anchor="middle">Security type</text>
  <text x="100" y="234" text-anchor="middle" class="muted">Standard / TL / CVM</text>
  <rect class="box-accent" x="230" y="30" width="260" height="280" rx="10"/>
  <text x="360" y="56" text-anchor="middle" class="label-accent">VM resource</text>
  <rect class="box-ok" x="250" y="72" width="220" height="40" rx="6"/>
  <text x="360" y="97" text-anchor="middle">VM agent → extensions</text>
  <rect class="box" x="250" y="124" width="220" height="40" rx="6"/>
  <text x="360" y="149" text-anchor="middle">NIC → subnet</text>
  <rect class="box" x="250" y="176" width="220" height="40" rx="6"/>
  <text x="360" y="201" text-anchor="middle">OS disk (managed)</text>
  <rect class="box" x="250" y="228" width="220" height="32" rx="6"/>
  <text x="360" y="249" text-anchor="middle">Data disks (managed)</text>
  <rect class="box-risk" x="250" y="268" width="220" height="32" rx="6"/>
  <text x="360" y="289" text-anchor="middle" class="label-risk">Temp disk: wiped</text>
  <line class="line-accent" x1="180" y1="58" x2="228" y2="80" marker-end="url(#vmdbarrow)"/>
  <line class="line-accent" x1="180" y1="138" x2="228" y2="138" marker-end="url(#vmdbarrow)"/>
  <line class="line-accent" x1="180" y1="218" x2="228" y2="200" marker-end="url(#vmdbarrow)"/>
  <rect class="box" x="540" y="60" width="200" height="90" rx="8"/>
  <text x="640" y="88" text-anchor="middle">Extensions</text>
  <text x="640" y="110" text-anchor="middle" class="muted">AMA, MDE, Guest</text>
  <text x="640" y="130" text-anchor="middle" class="muted">Attestation, scripts</text>
  <rect class="box-risk" x="540" y="190" width="200" height="90" rx="8"/>
  <text x="640" y="218" text-anchor="middle" class="label-risk">Public IP + 3389/22</text>
  <text x="640" y="240" text-anchor="middle" class="muted">brute-forced within</text>
  <text x="640" y="260" text-anchor="middle" class="muted">minutes of exposure</text>
  <line class="line-accent" x1="470" y1="92" x2="538" y2="100" marker-end="url(#vmdbarrow)"/>
  <line class="line" x1="470" y1="144" x2="538" y2="225" marker-end="url(#vmdbarrow)"/>
</svg>
```

## What actually matters

**1. Image and size.** An **image** is the template disk: a Marketplace image (Windows Server, Ubuntu…) or your own in an **Azure Compute Gallery**. The **size** sets vCPUs, memory, disk and network limits. Size names encode features — for example a `D` family is general purpose, `E` is memory-optimized, an `s` means premium storage support, and confidential sizes start `DC`/`EC`. Not every size supports every security feature.

**2. Managed disks.** Disks are Azure resources of their own, stored as page blobs that Azure manages for you.

| Disk | What | Persists? |
|---|---|---|
| **OS disk** | Holds the operating system; created from the image | Yes |
| **Data disks** | Extra volumes you attach | Yes |
| **Temp disk** | Local scratch on the host (page file, cache) | ==!No — wiped on redeploy, resize or host move== |

Disk types, fastest to cheapest: **Ultra Disk**, **Premium SSD v2**, **Premium SSD**, **Standard SSD**, **Standard HDD**. All managed disks are encrypted at rest by default with **server-side encryption (SSE)** using platform-managed keys; you can switch to customer-managed keys, add **encryption at host** (covers temp disk and caches) or use **Azure Disk Encryption** inside the guest. Those choices are D3.2.1. ==!The temp disk isn't covered by SSE== unless you turn on encryption at host (v5-and-later sizes encrypt it automatically).

**3. Generation 1 vs Generation 2.**

| | Gen 1 | Gen 2 |
|---|---|---|
| Firmware | BIOS | **UEFI** |
| Secure Boot, vTPM | No | Yes (via Trusted launch) |
| Change later? | Can upgrade Gen1 → Gen2 Trusted launch | — |

==@Trusted launch requires Gen 2.==

**4. Security types (at a glance — detail in D3.2.8).** Chosen on the **Basics** tab at creation:

| Security type | What it adds | Protects against |
|---|---|---|
| **Standard** | Nothing extra | — |
| **Trusted launch** | **Secure Boot** (only signed boot loaders, kernels, drivers), **vTPM** (virtual TPM 2.0 for keys and boot measurements), **boot integrity monitoring** via the **Guest Attestation** extension + Defender for Cloud | Bootkits, rootkits, kernel malware |
| **Confidential** | Hardware-encrypted memory with **AMD SEV-SNP** or **Intel TDX**, optional **confidential OS disk encryption** bound to the vTPM, attestation before boot | The host, hypervisor and cloud operator reading memory |

Trusted launch is ==+the default== for new Gen 2 VMs from the portal, CLI and PowerShell, and costs nothing extra. Confidential VMs need confidential sizes (`DCasv5`, `ECasv5`, `DCesv6`…).

**5. The VM agent and extensions.** The **Azure VM Agent** (Windows: `WindowsAzureGuestAgent`; Linux: `waagent`) runs inside the VM, is installed by default in Marketplace images, and talks to the host via 168.63.129.16. It installs and runs **extensions** — small add-ons pushed from Azure: the **Azure Monitor Agent**, **Microsoft Defender for Endpoint (MDE)**, **Guest Attestation**, **Machine Configuration** (formerly Guest Configuration, D3.2.9), **Custom Script Extension**, **Run Command**. Extensions run as **SYSTEM/root**. ==!Anyone who can write extensions to a VM can run code as admin on it== — the `Microsoft.Compute/virtualMachines/extensions/write` permission (in Contributor and Virtual Machine Contributor) is effectively local admin.

**6. RDP and SSH exposure.** Management ports — **RDP 3389** (Windows) and **SSH 22** (Linux) — opened to `Any`/`Internet` on a public IP are scanned and brute-forced constantly. The portal even warns you at creation. Safer options, each its own objective:

| Option | Idea |
|---|---|
| **No public IP + Azure Bastion** | Browser-based RDP/SSH through a managed jump host (D3.2.2) |
| **Just-in-time (JIT) VM access** | NSG port opened only on request, for a limited time, from one IP (D3.2.3) |
| **NSG restricted to known IPs** | Better than Any, still exposed (D2.3.1) |

Also: prefer **SSH keys** over passwords on Linux, and Entra ID login for VMs where possible.

## Where it lives

**Azure portal > Virtual machines > Create > Azure virtual machine.** Basics tab: **Image**, **Size**, **Security type** (Standard / Trusted launch virtual machines / Confidential virtual machines) with **Configure security features** (Secure Boot, vTPM, integrity monitoring), **Authentication type** (SSH public key / Password), **Public inbound ports** (None / Allow selected ports). Disks tab: OS disk type, encryption type, encryption at host. Networking tab: VNet, subnet, public IP, NIC NSG.

On an existing VM: `Settings > Disks`, `Settings > Extensions + applications`, `Settings > Configuration` (security type, Secure Boot, vTPM), `Connect` (Bastion, native RDP/SSH, JIT request).

```bash
az vm create -g rg -n vm1 --image Ubuntu2204 --size Standard_D2s_v5 \
  --security-type TrustedLaunch --enable-secure-boot true --enable-vtpm true \
  --public-ip-address "" --nsg-rule NONE --generate-ssh-keys
az vm extension list -g rg --vm-name vm1 -o table
az vm extension set -g rg --vm-name vm1 --publisher Microsoft.Azure.Security.LinuxAttestation --name GuestAttestation
```

![Disk roles in action](https://learn.microsoft.com/en-us/azure/virtual-machines/media/virtual-machines-managed-disks-overview/disk-types.png){OS disk, data disks and the temporary disk — only the temp disk lives on the host and is lost}

## Exam traps

- ==!Temp disk data doesn't survive== a redeploy or resize, and isn't SSE-encrypted without encryption at host.
- Secure Boot / vTPM missing from the options → the VM is **Gen 1** or an unsupported size. Trusted launch needs **Gen 2**.
- Boot integrity alerts in Defender for Cloud need **Secure Boot + vTPM + Guest Attestation extension**.
- "Protect data in use from the cloud operator" → **Confidential VM**, not Trusted launch.
- Extension write rights = remote code execution as SYSTEM/root. Treat VM Contributor as server admin.
- "Remove internet exposure of RDP" → no public IP + **Bastion**, or **JIT**. Changing the RDP port number is not a control.

## Skip

- **Skip** VM size families and the naming grammar beyond the few letters above. Sizes are cost and performance.
- **Skip** disk performance numbers (IOPS, throughput, bursting) and the disk-selection decision tree.
- **Skip** availability sets, zones, scale sets and VM pricing models (spot, reservations).
- **Skip** virtualization-based security (HVCI, Credential Guard) detail — know only that Trusted launch makes them possible on Windows.
- **Skip** Intel SGX application enclaves and confidential GPU VMs.
- **Skip** extension authoring and the agent's internal plug-in architecture.

## Screenshots

![Managed disk decision tree](https://learn.microsoft.com/en-us/azure/virtual-machines/media/disks-types/managed-disk-decision-tree.png){Choosing a disk type by latency, IOPS and throughput (skip the numbers)}

![Breakdown of the DCadsv5 size series name](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/media/size-series-breakdown.png){Size series name decoded — DC = confidential general purpose, a = AMD, d = local temp disk, s = premium storage}

![Breakdown of the DC8ads_v5 size name](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/media/size-instance-breakdown.png){A full size name — the number is the vCPU count}

![Customer-managed keys workflow for managed disks](https://learn.microsoft.com/en-us/azure/virtual-machines/media/disk-encryption/customer-managed-keys-sse-managed-disks-workflow.svg){Customer-managed keys for SSE — Key Vault → disk encryption set → disks (D3.2.1)}

![YouTube video for selecting the right size for your VM.](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes/media/vm-series-video-play.png)
