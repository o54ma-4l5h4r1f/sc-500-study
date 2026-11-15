+++
slug = "containers-and-kubernetes-basics"
title = "Containers and Kubernetes basics (AKS, ACR, Container Apps, Container Instances)"
summary = "What a container, image and registry are, how a Kubernetes cluster is built (control plane, nodes, node pools, pods, namespaces), and what AKS, Azure Container Registry, Azure Container Apps and Azure Container Instances each do — so the container security lessons have something to stand on."
order = 20
ms_learn_urls = [
  "https://learn.microsoft.com/en-us/azure/aks/core-aks-concepts",
  "https://learn.microsoft.com/en-us/azure/aks/what-is-aks",
  "https://learn.microsoft.com/en-us/azure/container-registry/container-registry-intro",
  "https://learn.microsoft.com/en-us/azure/container-registry/container-registry-concepts",
  "https://learn.microsoft.com/en-us/azure/container-registry/container-registry-skus",
  "https://learn.microsoft.com/en-us/azure/container-registry/container-registry-private-endpoints",
  "https://learn.microsoft.com/en-us/azure/container-apps/overview",
  "https://learn.microsoft.com/en-us/azure/container-apps/compare-options",
  "https://learn.microsoft.com/en-us/azure/container-apps/environment",
  "https://learn.microsoft.com/en-us/azure/container-instances/container-instances-overview",
  "https://learn.microsoft.com/en-us/azure/container-instances/container-instances-container-groups",
]
captured = 2026-11-15
related_objectives = ["D3.3.1", "D3.3.2", "D3.3.3", "D3.3.4", "D4.1.3"]
+++

## In one sentence

A **container** is an app packed together with everything it needs to run, started from a read-only **image** that lives in a **registry**; **Kubernetes** is the software that runs many containers across many machines, and Azure offers four ways to run them: **AKS**, **Container Apps**, **Container Instances** and (for web apps) App Service.

## Why it exists

Think of shipping containers. Before them, every cargo was packed differently and every port handled it differently. After them, any crane, ship or truck could move any box. Software containers do the same for apps. The developer packs the app, its libraries and its settings into one box. The box runs the same on a laptop, a test server and in Azure.

A **virtual machine (VM)** carries a whole operating system: kernel, drivers, services. A container does ==not==. It shares the host machine's kernel and carries only the app and its libraries. That makes it small (megabytes, not gigabytes) and fast to start (seconds, not minutes).

Small and fast means lots of them. A real app may be 50 containers that must be started, restarted when they crash, spread across machines and scaled up at lunchtime. Doing that by hand is impossible. **Kubernetes** (often written **K8s**) is the open-source "orchestrator" that does it. As an analyst you saw the security side: Defender for Containers alerts about a pod running a crypto miner. This page is the plumbing behind those alerts.

## Mental model

A container platform is a **shipping port**:

- **Image** — the sealed, labelled box design. Read-only. Built in layers (base OS layer, then runtime, then your app).
- **Registry** — the warehouse that stores boxes. Inside it, a **repository** holds all versions of one image, and a **tag** (such as `v1.2` or `latest`) names one version.
- **Container** — a box that has been opened and is running.
- **Kubernetes cluster** — the port. The **control plane** is the harbour master's office that decides where boxes go. **Nodes** are the cranes and docks that do the work.

```svg
<svg viewBox="0 0 900 360" role="img" aria-label="Container flow: a developer pushes an image to Azure Container Registry; AKS, Container Apps and Container Instances pull the image and run it as containers">
  <defs>
    <marker id="ckbarrow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto-start-reverse">
      <path d="M0,0 L10,5 L0,10 z" class="line-accent"/>
    </marker>
  </defs>
  <rect class="box" x="15" y="140" width="150" height="80" rx="8"/>
  <text x="90" y="172" text-anchor="middle">Developer</text>
  <text x="90" y="195" text-anchor="middle" class="muted">builds image</text>
  <rect class="box-accent" x="215" y="120" width="200" height="120" rx="10"/>
  <text x="315" y="150" text-anchor="middle" class="label-accent">Container Registry</text>
  <text x="315" y="175" text-anchor="middle">repository: web</text>
  <text x="315" y="198" text-anchor="middle" class="muted">tags v1, v2, latest</text>
  <text x="315" y="221" text-anchor="middle" class="muted">push in / pull out</text>
  <rect class="box" x="480" y="20" width="400" height="95" rx="8"/>
  <text x="680" y="48" text-anchor="middle" class="label-accent">AKS (managed Kubernetes)</text>
  <text x="680" y="72" text-anchor="middle">you see nodes, pods, namespaces, kubectl</text>
  <text x="680" y="95" text-anchor="middle" class="muted">most control, most to secure</text>
  <rect class="box" x="480" y="133" width="400" height="95" rx="8"/>
  <text x="680" y="161" text-anchor="middle" class="label-accent">Container Apps</text>
  <text x="680" y="185" text-anchor="middle">Kubernetes underneath, no Kubernetes API</text>
  <text x="680" y="208" text-anchor="middle" class="muted">serverless, scale to zero</text>
  <rect class="box" x="480" y="246" width="400" height="95" rx="8"/>
  <text x="680" y="274" text-anchor="middle" class="label-accent">Container Instances</text>
  <text x="680" y="298" text-anchor="middle">one container group on demand</text>
  <text x="680" y="321" text-anchor="middle" class="muted">no orchestration, no autoscale</text>
  <line class="line-accent" x1="165" y1="180" x2="213" y2="180" marker-end="url(#ckbarrow)"/>
  <line class="line" x1="415" y1="160" x2="478" y2="67" marker-end="url(#ckbarrow)"/>
  <line class="line" x1="415" y1="180" x2="478" y2="180" marker-end="url(#ckbarrow)"/>
  <line class="line" x1="415" y1="200" x2="478" y2="293" marker-end="url(#ckbarrow)"/>
</svg>
```

![Registry, repositories, and artifacts](https://learn.microsoft.com/en-us/azure/container-registry/media/container-registry-concepts/registry-elements.png){A registry holds repositories; a repository holds the versions (tags) of one image}

![Layers of a container image](https://learn.microsoft.com/en-us/azure/container-registry/media/container-registry-concepts/container-image-layers.png){An image is a stack of read-only layers. A vulnerable base layer is inherited by every image built on it — that is why image scanning matters.}

## What actually matters

**1. Kubernetes has two halves.** The **control plane** runs the cluster's brain: `kube-apiserver` (the API every command goes through), `etcd` (the key-value store holding the cluster state and configuration), `kube-scheduler` (picks a node for each new pod) and controllers. **Nodes** are the VMs that run your apps. Each node runs `kubelet` (makes sure containers are running), a network proxy, and a **container runtime** (`containerd` on AKS).

![Screenshot that shows Kubernetes control plane and node components.](https://learn.microsoft.com/en-us/azure/aks/media/concepts-clusters-workloads/control-plane-and-nodes.png){In AKS the control plane (left) is run by Azure. The nodes (right) are VMs in your subscription.}

**2. AKS = Azure runs the control plane, you own the nodes.** **Azure Kubernetes Service (AKS)** is managed Kubernetes. Azure operates the control plane for you. The nodes are ordinary Azure VMs, grouped into **node pools** that by default are **Virtual Machine Scale Sets**. The **system node pool** hosts critical system pods; **user node pools** host your apps. AKS also creates a second resource group, the **node resource group**, holding the VMs, scale sets and storage. AKS has two cluster modes: **AKS Automatic** (preset production defaults, including security guardrails) and **AKS Standard** (you configure everything). It also has three pricing tiers — Free, Standard, Premium. ==!The Standard pricing tier is not the same thing as Standard cluster mode.==

![Screenshot that shows Azure virtual machine and supporting resources for a Kubernetes node.](https://learn.microsoft.com/en-us/azure/aks/media/concepts-clusters-workloads/aks-node-resource-interactions.png){An AKS node is an Azure VM with disks and a network card — so VM security still applies to it}

![AKS overview graphic](https://learn.microsoft.com/en-us/azure/aks/media/what-is-aks/what-is-aks.png){What AKS manages for you}

**3. Pods and namespaces.** A **pod** is the smallest thing Kubernetes runs: one or more containers that share a network address and storage. Usually one container per pod. A **namespace** is a folder inside the cluster that groups pods and other objects, so you can split teams and scope access. AKS creates four by default: `default`, `kube-system` (Kubernetes' own components — do not deploy apps there), `kube-public` and `kube-node-lease`.

![Screenshot that shows Kubernetes namespaces to logically divide resources and applications.](https://learn.microsoft.com/en-us/azure/aks/media/concepts-clusters-workloads/namespaces.png){Namespaces divide one cluster into logical areas}

**4. ACR is your private registry.** **Azure Container Registry (ACR)** stores and serves images. It comes in three SKUs: **Basic**, **Standard**, **Premium**. All three support Microsoft Entra sign-in. ==@Only Premium== adds **private endpoints** (up to 200), public IP network rules and **geo-replication**. That single fact drives a lot of ACR network-security answers.

**5. Container Apps = Kubernetes without the Kubernetes.** **Azure Container Apps** runs containers serverlessly. It is built on Kubernetes plus open-source add-ons (KEDA for event-driven scaling, Dapr, Envoy), but ==!it does not give you the Kubernetes API==. Apps live inside a **Container Apps environment** — a security boundary backed by a virtual network. Apps in the same environment share that network and log destination. Apps can **scale to zero**.

![Azure Container Apps environments.](https://learn.microsoft.com/en-us/azure/container-apps/media/environments/azure-container-apps-environments.png){An environment is the boundary: apps inside share a VNet and a Log Analytics destination}

![Example scenarios for Azure Container Apps.](https://learn.microsoft.com/en-us/azure/container-apps/media/overview/azure-container-apps-example-scenarios.png){Typical Container Apps workloads: APIs, event processing, background jobs, microservices}

**6. Container Instances = one box, right now.** **Azure Container Instances (ACI)** starts a **container group** — a set of containers that share a host, network, storage and lifecycle — with no cluster at all. Containers are Hyper-V isolated. There is no autoscale or load balancing: five copies means five container groups. ACI can be placed in a VNet subnet (outbound then needs a NAT gateway) and supports managed identity, including pulling images from ACR with a managed identity instead of a password.

![Container groups diagram](https://learn.microsoft.com/en-us/azure/container-instances/media/container-instances-container-groups/container-groups-example.png){A container group: containers share one IP, ports and mounted volumes}

| | AKS | Container Apps | Container Instances |
|---|---|---|---|
| Kubernetes API access | ==@yes== | no | no |
| Who runs the control plane | Azure | Azure (hidden) | n/a |
| Nodes visible to you | yes (VMs, node pools) | no | no |
| Scale | you configure (autoscaler) | automatic, to zero | none — one group |
| Security boundary | cluster, namespaces | environment | container group |
| Typical use | full platform, many teams | microservices, APIs, jobs | quick task, sidecar, burst |

## Where it lives

All four are managed in the **Azure portal**; Defender for Containers findings appear in **Defender for Cloud** and in the Defender portal.

- **AKS:** `Azure portal > Kubernetes services > (cluster)` (Node pools, Networking, Security configuration). Inside the cluster you use `kubectl`.
- **ACR:** `Azure portal > Container registries > (registry) > Settings > Networking` (public access, private endpoints — Premium) and `Properties` (pricing plan).
- **Container Apps:** `Azure portal > Container Apps` and `Container Apps environments`.
- **Container Instances:** `Azure portal > Container instances`.

```bash
az aks create -g rg-lab -n aks-lab --node-count 1      # small lab cluster (bills per node VM)
az acr create -g rg-lab -n acrlab12345 --sku Premium   # Premium needed for private endpoint
az aks update -g rg-lab -n aks-lab --attach-acr acrlab12345   # let AKS pull from ACR
```

## Exam traps

- ==!"Containers include their own operating system kernel."== False — they share the host kernel. That is why a container escape is serious.
- ==!ACR private endpoint on Standard SKU== — not possible. ==+Premium only.==
- ==!Container Apps exposes kubectl / the Kubernetes API== — it does not. If a question needs Kubernetes RBAC, admission control or node pools, the answer is **AKS**.
- AKS nodes are VMs in **your** subscription (node resource group). Securing them is your job; the control plane is Azure's.
- "Standard" appears three times: AKS Standard mode, AKS Standard pricing tier, ACR Standard SKU. Read which one the question means.

## Skip

- **Skip:** Kubernetes object types beyond pod and namespace (Deployments, ReplicaSets, StatefulSets, DaemonSets). The security lessons name them when needed.
- **Skip:** AKS VM sizing, OS disk sizing, resource reservations and the Azure Linux 2.0 retirement notice. Operations, not security.
- **Skip:** ACR throughput limits (reads per minute, webhook counts, storage GiB per SKU). Know only "Premium = private endpoints + geo-replication".
- **Skip:** Dapr, KEDA scale rules and Container Apps workload profiles/billing. Know they exist; the exam asks about ingress, identity and networking.
- **Skip:** ACI volume types (git repo, empty directory) and zonal deployment.

## Screenshots

Every remaining image from the cited Microsoft Learn articles, per the image rule.

![The options to create a private endpoint for a new Azure container registry.](https://learn.microsoft.com/en-us/azure/container-registry/media/private-endpoints/private-link-create-portal.png)
