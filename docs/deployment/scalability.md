# Scalability

This page covers how to scale CelestaCX to handle growing contact center workloads  more agents, higher conversation volumes, additional channels, and increased data processing demands. Whether you're expanding an existing deployment or planning for future growth, this guide helps you scale efficiently.

---

### Understanding CelestaCX Scalability

CelestaCX is designed to scale in two dimensions:

**Vertical Scaling (Scale Up)** Adding more CPU, RAM, or storage to existing nodes. Useful for quick capacity increases but limited by hardware maximums.

**Horizontal Scaling (Scale Out)** Adding more nodes to your Kubernetes cluster. This is the recommended long-term scaling approach — Kubernetes distributes workloads automatically across available nodes.

> 💡 **CelestaCX is built for horizontal scaling.** The microservices architecture allows you to add capacity by simply adding nodes — no application reconfiguration required.

---

### When to Scale

Monitor these indicators to know when scaling is needed:

| Indicator | Warning Threshold | Action Required |
| --- | --- | --- |
| CPU utilization on worker nodes | >75% average over 1 hour | Add worker nodes or increase node CPU |
| Memory utilization on worker nodes | >80% average over 1 hour | Add worker nodes or increase node RAM |
| Agent count approaching license limit | >85% of licensed agents | Request license expansion from OctaveBytes |
| Database response times degrading | Query times >500ms | Scale database resources or add replicas |

### Scaling Strategy by Component

Different CelestaCX components scale differently. Here's how to scale each layer.

---

#### Layer 1 — Application Services

These are the core CelestaCX microservices: Agent Manager, Routing Engine, CIM, Channel Connectors.

**How they scale:** These services are **stateless**  they don't store data locally, so you can run multiple copies (replicas) across multiple nodes without conflict. Kubernetes automatically load-balances traffic between replicas.

**How to scale:**

**Check current replica count:**

bash

```
kubectl get deployment -n celestacx
```

**Scale a specific service:**

bash

```
# Example: Scale Agent Manager from 2 to 4 replicas
kubectl scale deployment agent-manager \
  --replicas=4 \
  --namespace celestacx
```

**Automate scaling with Horizontal Pod Autoscaler (HPA):**

bash

```
# Auto-scale Agent Manager based on CPU usage
kubectl autoscale deployment agent-manager \
  --namespace celestacx \
  --min=2 \
  --max=8 \
  --cpu-percent=70
```

This tells Kubernetes to automatically add replicas when CPU exceeds 70%, up to a maximum of 8 replicas.

#### Layer 2 — Databases

CelestaCX uses three databases: MongoDB (conversations), PostgreSQL (reporting), Redis (cache).

**MongoDB (Operational Data)**

MongoDB stores all active conversations, customer profiles, and channel sessions. This is the most critical database for performance.

**Scaling approach: Replica Set**

A MongoDB replica set runs multiple copies of the database across multiple nodes. One is the primary (handles writes), the others are secondaries (handle reads and provide failover).

**Current setup check:**

bash

```
kubectl exec -n celestacx \
  $(kubectl get pods -n celestacx -l app=mongodb \
  -o jsonpath='{.items[0].metadata.name}') \
  -- mongo --eval "rs.status()"
```

**Convert from single-node to replica set:**

If you installed MongoDB as a single instance and need to scale it, follow the MongoDB Replica Set Conversion Guide → in Platform Administration.

**Recommended replica set size:**

- **Small deployments (100–500 agents):** 3-member replica set (1 primary + 2 secondaries)
- **Large deployments (500+ agents):** 5-member replica set (1 primary + 4 secondaries)

**Increase MongoDB storage:**

If MongoDB storage is filling up (check with `kubectl get pvc -n celestacx`), expand the persistent volume:

bash

```
# Edit the PVC to increase size
kubectl edit pvc mongodb-data -n celestacx

# Change this line:
#   storage: 50Gi
# To:
#   storage: 100Gi
```

> ⚠️ **Storage expansion requires your storage class to support volume expansion.** Check with `kubectl get storageclass` — look for `ALLOWVOLUMEEXPANSION` set to `true`.

**PostgreSQL (Reporting Data)**

PostgreSQL stores historical reporting data — conversation summaries, agent performance metrics, and QM evaluations.

**Scaling approach: Read Replicas**

For reporting-heavy deployments, add read-only replicas of PostgreSQL. Reporting queries run against replicas, leaving the primary database free to handle writes from the data platform.

**Add a read replica:**

Update your `cx-values.yaml`:

yaml

```
postgresql:
  enabled: true
  replication:
    enabled: true
    numSynchronousReplicas: 1
    readReplicas: 2    # Add 2 read replicas
```

Apply the change:

bash

```
helm upgrade celestacx celestacx/celestacx \
  --namespace celestacx \
  --values cx-values.yaml
```

**When to add replicas:**

- You have 10+ supervisors running reports simultaneously
- Report generation is slowing down during peak hours
- PostgreSQL CPU usage exceeds 60% regularly

**Redis (Cache & Session Data)**

Redis stores agent state, active sessions, and temporary cache data. It is in-memory, so performance is excellent but capacity is limited by RAM.

**Scaling approach: Increase RAM or Add Redis Cluster**

**Option 1 — Increase RAM** (simplest for most deployments):

yaml

```
# In cx-values.yaml
redis:
  master:
    resources:
      requests:
        memory: 4Gi    # Increase from default 2Gi
      limits:
        memory: 8Gi
```

**Option 2 — Redis Cluster** (for very large deployments):

Contact OctaveBytes for guidance on deploying Redis in cluster mode for 1000+ concurrent agents.

#### Layer 3 — Voice & Media

If you are running voice or video channels, the **Media Server** is the most resource-intensive component.

**Scaling approach: Dedicated Nodes + Multiple Instances**

**Step 1 — Run Media Server on dedicated nodes**

Media processing is CPU and network-intensive. Isolate it from other workloads.

Label your media server nodes:

bash

```
kubectl label node cx-worker-04 workload=media-server
kubectl label node cx-worker-05 workload=media-server
```

Update `cx-values.yaml`:

yaml

```
mediaServer:
  enabled: true
  nodeSelector:
    workload: media-server
  replicas: 2    # Run 2 instances across labeled nodes
```

**Step 2 — Scale based on concurrent calls**

**Capacity per Media Server instance:**

- 1 instance handles ~100 concurrent voice calls
- For 500 concurrent calls, run 5 replicas

**Check current voice load:**

bash

```
kubectl top pod -n celestacx -l app=media-server
```

**Scale Media Server:**

bash

```
kubectl scale deployment media-server \
  --replicas=5 \
  --namespace celestacx
```

#### Layer 4 — Data Platform (ETL Pipelines)

The Data Platform extracts data from MongoDB and loads it into PostgreSQL for reporting.

**Scaling approach: Increase Pipeline Workers**

If your ETL pipelines are lagging (reports show data from hours ago instead of near real-time), increase pipeline parallelism:

yaml

```
# In cx-values.yaml
dataPlatform:
  workers: 4    # Increase from default 2
```

**Monitor ETL lag:**

bash

```
kubectl logs -n celestacx \
  $(kubectl get pods -n celestacx -l app=data-platform \
  -o jsonpath='{.items[0].metadata.name}') \
  | grep "lag"
```

---

### Scaling the Kubernetes Cluster Itself

When existing nodes are at capacity, add more nodes to the cluster.

#### Adding Worker Nodes

**Step 1 — Prepare the new node**

Follow [Kubernetes Setup → Part 1: Prepare All Nodes →](#) on the new server.

**Step 2 — Get the cluster token**

On an existing control plane node:

bash

```
sudo cat /var/lib/rancher/rke2/server/node-token
```

**Step 3 — Join the new worker**

On the new server:

bash

```
sudo mkdir -p /etc/rancher/rke2
sudo nano /etc/rancher/rke2/config.yaml
```

yaml

```
server: https://192.168.1.100:9345    # Load balancer IP (HA) or first control plane IP
token: <CLUSTER_TOKEN>
```

bash

```
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="agent" sudo sh -
sudo systemctl enable rke2-agent.service
sudo systemctl start rke2-agent.service
```

**Step 4 — Verify the node joined**

On a control plane node:

bash

```
kubectl get nodes
```

The new node should appear with status `Ready` within 2–3 minutes.

**Step 5 — Pods automatically distribute**

Kubernetes will immediately start scheduling new pods onto the new node. Existing pods will remain on their current nodes unless you manually rebalance.

---

#### Rebalancing Pods Across Nodes

If you've added new nodes and want to distribute existing pods more evenly:

**Check current pod distribution:**

bash

```
kubectl get pods -n celestacx -o wide
```

**Force rebalance by rolling restart:**

bash

```
kubectl rollout restart deployment -n celestacx
```

This restarts all pods one at a time. As they restart, Kubernetes spreads them across all available nodes based on current capacity.

### Performance Tuning Best Practices

Beyond scaling, these tuning steps improve performance without adding hardware.

---

#### 1. Enable Resource Requests and Limits

Tell Kubernetes exactly how much CPU and RAM each pod needs. This prevents resource contention and improves scheduling.

**Example for Agent Manager:**

yaml

```
# In cx-values.yaml
agentManager:
  resources:
    requests:
      cpu: 500m        # Minimum guaranteed
      memory: 1Gi
    limits:
      cpu: 2000m       # Maximum allowed
      memory: 4Gi
```

---

#### 2. Use Pod Anti-Affinity for Critical Services

Ensure replicas of the same service don't all run on the same node. If that node fails, you lose all replicas.

**Example:**

yaml

```
agentManager:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: app
            operator: In
            values:
            - agent-manager
        topologyKey: kubernetes.io/hostname
```

This forces each Agent Manager replica onto a different physical node.

---

#### 3. Optimize Database Connections

Each CelestaCX service maintains a connection pool to MongoDB and PostgreSQL. Tune pool sizes based on load.

**Default values:**

yaml

```
mongodb:
  connectionPool:
    minSize: 10
    maxSize: 50
```

**For high-load deployments (500+ agents):**

yaml

```
mongodb:
  connectionPool:
    minSize: 20
    maxSize: 100
```

---

#### 4. Enable Persistent Connections for Redis

Reusing Redis connections improves performance for agent state updates.

yaml

```
redis:
  persistentConnections: true
```

---

#### 5. Tune ActiveMQ for High Message Volumes

ActiveMQ is the message broker between CelestaCX services. For high conversation volumes, increase memory and consumer threads.

yaml

```
activemq:
  resources:
    memory: 4Gi    # Default is 2Gi
  consumers: 8     # Default is 4
```

---

### Capacity Planning Guidelines

Use this table to plan node requirements based on your target agent count.

| Target Agents | Worker Nodes | Control Plane Nodes | Total vCPU | Total RAM | MongoDB Storage |
| --- | --- | --- | --- | --- | --- |
| 50 agents | 2 | 1 (single-node) | 8 | 16 GB | 100 GB |
| 100 agents | 3 | 3 (HA recommended) | 16 | 24 GB | 250 GB |
| 250 agents | 5 | 3 | 16 | 32 GB | 250 GB |
| 500 agents | 8 | 3 | 24 | 42 GB | 500 GB |
| 1000 agents | 12 | 5 | 32 | 64 GB | 1 TB |

**Add 20% buffer** to all numbers to handle peak loads and future growth.

**Voice channel multiplier:** If running voice/video, add 1 dedicated media server node per 100 concurrent calls.

---

### Scaling Checklist

Before scaling, run through this checklist:

> *(Add a blue Info Panel macro around the following)*
> **Pre-Scaling Checklist:**
> - Monitored current resource usage (CPU, RAM, disk) for at least 1 week
> - Identified the bottleneck (compute, memory, database, network)
> - Reviewed current pod replica counts and distribution
> - Checked database performance and slow query logs
> - Verified license capacity for additional agents
> - Tested scaling in a non-production environment first
> - Scheduled scaling during a maintenance window
> - Backed up etcd and databases before making changes
> - Documented current configuration for rollback

---

### Troubleshooting Scaling Issues

| Problem | Likely Cause | Solution |
| --- | --- | --- |
| New pods stuck in Pending after scaling | Insufficient node resources | Check kubectl describe pod — add more worker nodes or increase node size |
| Pods evenly distributed but performance still poor | Database bottleneck | Check MongoDB/PostgreSQL CPU and slow queries — scale databases |
| Scaling HPA doesn't trigger | Metrics server not installed | Install metrics-server: kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml |
| New worker node joins but no pods schedule onto it | Node has taints or labels preventing scheduling | Check kubectl describe node for taints — remove with kubectl taint node <name> <taint>- |
| Agent Manager scaled but agents still experience delays | Session affinity breaking connections | Enable sticky sessions in NGINX ingress configuration |

---
