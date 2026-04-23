# Deployment Planning & Sizing

Before deploying CelestaCX, you need to make three key decisions:

1. **What deployment topology do you need?** — Single node, multi-node, or high availability
2. **How much hardware do you need?** — Based on your expected agent count and conversation load
3. **What Kubernetes distribution will you use?** — Based on your environment and infrastructure preferences

This page walks you through each of these decisions so you arrive at the [Kubernetes Setup →](#) and [CelestaCX Installation Guide →](#) steps with a clear plan.

---

### Step 1 — Choose Your Deployment Topology

CelestaCX supports three deployment topologies. Choose the one that matches your environment and reliability requirements.

| Topology | Description | When to Use |
| --- | --- | --- |
| Single Node | All CelestaCX services run on one machine | Development, testing, or proof-of-concept (POC) only. Not suitable for production. |
| Multi-Node | Services distributed across multiple Kubernetes nodes | Production deployments with moderate load and no strict uptime SLA |
| High Availability (HA) | Multiple control plane nodes, external load balancer, replicated storage | Production deployments requiring resilience, failover, and zero-downtime maintenance |

⚠️ **Single node deployments are not recommended for production use.** If a single node goes offline, the entire CelestaCX platform becomes unavailable. Use multi-node or HA for any live customer-facing deployment.

---

### Step 2 — Estimate Your Hardware Requirements

Hardware requirements depend primarily on three factors:

- **Number of concurrent agents** — how many agents are logged in and active at the same time
- **Conversation volume** — how many conversations per second the system needs to handle
- **Channel mix** — voice and video are significantly more resource-intensive than digital/chat channels

Use the sizing tiers below as a starting point. If your requirements fall between tiers, size up to the next tier.

---

#### Sizing Tiers

#### Tier 1 — Small Deployment

*Suitable for: Up to 100 concurrent agents, digital channels only (no voice)*

| Component | Specification |
| --- | --- |
| Nodes | 3 worker nodes |
| CPU per node | 8 vCPU |
| RAM per node | 32 GB |
| Storage per node | 200 GB SSD |
| Network | 1 Gbps |

---

#### Tier 2 — Medium Deployment

*Suitable for: 100–500 concurrent agents, mixed digital and voice channels*

| Component | Specification |
| --- | --- |
| Nodes | 5 worker nodes |
| CPU per node | 16 vCPU |
| RAM per node | 64 GB |
| Storage per node | 500 GB SSD |
| Network | 1 Gbps |
| Separate Media Server node | 8 vCPU / 16 GB RAM (for voice/video) |

---

#### Tier 3 — Large Deployment

*Suitable for: 500–1000+ concurrent agents, high-volume voice and digital*

| Component | Specification |
| --- | --- |
| Nodes | 8+ worker nodes |
| CPU per node | 32 vCPU |
| RAM per node | 128 GB |
| Storage per node | 1 TB SSD |
| Network | 10 Gbps |
| Separate Media Server node | 16 vCPU / 32 GB RAM (for voice/video) |
| Separate Rasa node (if using AI bots) | 4 vCPU / 16 GB RAM |

---

#### Additional Sizing Notes

**Voice and Video** If you are deploying CelestaCX with voice or video channels, the Media Server must run on a **dedicated node** separate from the main CelestaCX services. Voice processing is CPU-intensive and will compete with other services if co-located.

**AI Bots (Rasa)** If you are using Rasa for NLU and self-service automation, Rasa should also run on a **dedicated node**. Rasa's machine learning models are memory-intensive and benefit from isolation.

**Storage** Use **SSD storage** throughout — spinning disk is not suitable for CelestaCX's database workloads. If you are retaining voice recordings, plan additional storage accordingly. A rough estimate for voice recording storage is **100 MB per hour of recorded audio per agent**.

**Multi-Tenancy (CelestaCX 5.0+)** For multi-tenant deployments serving multiple customer organizations, size your cluster based on the **combined** agent count and conversation volume across all tenants, then add a 20–30% overhead buffer for tenant management services.

---

### Step 3 — High Availability Planning

If you are deploying with High Availability, you need additional infrastructure on top of the worker nodes.

| Component | Requirement |
| --- | --- |
| Control plane nodes | Minimum 3 nodes (odd number for quorum) |
| CPU per control plane node | 4 vCPU |
| RAM per control plane node | 8 GB |
| External Load Balancer | NGINX or HAProxy — separate from the cluster |
| Shared storage | Required for replicated block volumes across nodes |

> 👉 For full HA setup instructions, see [High Availability →](#)

---

### Step 4 — Choose Your Kubernetes Distribution

CelestaCX supports several Kubernetes distributions. The table below helps you choose based on your environment.

| Distribution | Footprint | Recommended For |
| --- | --- | --- |
| RKE2 | Medium | Production deployments — recommended for enterprise and partner use |
| K3s | Small | Lightweight production or resource-constrained environments |
| K0s | Small | Small deployments and edge environments |
| MicroK8s | Small | Development and testing |
| Rancher | Large | Enterprises already using Rancher for multi-cluster management |
| KIND | Very small | Local development and CI/CD pipelines only |

> 💡 **OctaveBytes recommendation:** Use **RKE2** for all production customer deployments. It is the most thoroughly tested distribution with CelestaCX and supports HA, air-gap installation, and enterprise security features out of the box.

---

### Step 5 — Pre-Deployment Checklist

Before moving to the Kubernetes setup and installation steps, confirm the following:

*(Add a blue Info Panel macro around this checklist)*

**Infrastructure**

- Hardware provisioned and meets the sizing requirements for your tier
- All nodes are running a supported operating system (Ubuntu 20.04+ or RHEL 8.4+)
- All nodes can communicate with each other on the required ports
- SSD storage is available on all nodes

**Networking & DNS**

- A Fully Qualified Domain Name (FQDN) has been registered for this CelestaCX deployment — e.g., `celestacx.yourcompany.com`
- DNS records are configured to point the FQDN to your load balancer or primary node IP
- SSL/TLS certificates are available for the FQDN (or you will use self-signed certificates)
- Required ports are open on your firewall (80, 443, and Kubernetes inter-node ports)

**Access & Credentials**

- SSH access is available to all nodes
- You have a Helm registry access token provided by OctaveBytes to pull the CelestaCX Helm charts
- You have your CelestaCX license file

**Planning**

- Deployment topology decided (single node / multi-node / HA)
- Kubernetes distribution chosen
- Voice/video requirements identified (dedicated Media Server node if needed)
- AI bot requirements identified (dedicated Rasa node if needed)

---

### Deployment Decision Summary

Once you have worked through the steps above, you should be able to fill in this summary before proceeding:

| Decision | Your Choice |
| --- | --- |
| Deployment topology | (Single Node / Multi-Node / HA) |
| Kubernetes distribution | (RKE2 / K3s / K0s / other) |
| Sizing tier | (Tier 1 / Tier 2 / Tier 3) |
| Number of worker nodes | (e.g., 3) |
| CPU per node | (e.g., 16 vCPU) |
| RAM per node | (e.g., 64 GB) |
| Dedicated Media Server node? | (Yes / No) |
| Dedicated Rasa node? | (Yes / No) |
| FQDN | (e.g., ) yourcompany.com |

> 👉 Once this table is filled in, you're ready to proceed to [Kubernetes Setup →](#)
