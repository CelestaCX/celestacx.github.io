# High Availability

High Availability (HA) ensures CelestaCX continues operating even when infrastructure components fail. In an HA deployment, no single server failure can take your contact center offline — the system automatically recovers by shifting workloads to healthy nodes.

This page is a complete guide to deploying CelestaCX with HA. We'll walk through the entire process from preparing multiple Kubernetes nodes to configuring an external load balancer and verifying failover works correctly.

---

### Do You Need High Availability?

Not every deployment requires HA. Use this table to decide:

| Your Situation | Recommendation |
| --- | --- |
| Development, testing, or proof-of-concept | ❌ Skip HA — use the single-node setup from Kubernetes Setup → |
| Production with planned maintenance windows acceptable | ⚠️ Multi-node (this page) without full HA may be sufficient |
| Production with zero acceptable downtime | ✅ Full HA required — follow this entire page |
| CCaaS / multi-tenant serving multiple customers | ✅ Full HA strongly recommended |
| Regulated industries (banking, healthcare, government) | ✅ Full HA required |

**HA adds infrastructure complexity and cost.** You need at least **6 servers** — 3 control plane nodes, 3 worker nodes, plus 1 load balancer. Your team must be comfortable managing a multi-node Kubernetes cluster. If you're unsure, start with a single-node deployment and plan to migrate to HA later once you've gained operational experience.

---

### What You're Building

Here's the complete HA architecture you'll have by the end of this page:

```
┌─────────────────────┐
                    │   NGINX Load        │
                    │   Balancer          │
                    │   (192.168.1.100)   │
                    └──────────┬──────────┘
                               │
           ┌───────────────────┼───────────────────┐
           │                   │                   │
           ▼                   ▼                   ▼
    ┌──────────┐        ┌──────────┐        ┌──────────┐
    │ Control  │        │ Control  │        │ Control  │
    │ Plane 1  │        │ Plane 2  │        │ Plane 3  │
    │  (etcd)  │◄──────►│  (etcd)  │◄──────►│  (etcd)  │
    └──────────┘        └──────────┘        └──────────┘
           │                   │                   │
           └───────────────────┼───────────────────┘
                               │
           ┌───────────────────┼───────────────────┐
           │                   │                   │
           ▼                   ▼                   ▼
    ┌──────────┐        ┌──────────┐        ┌──────────┐
    │ Worker 1 │        │ Worker 2 │        │ Worker 3 │
    │ CelestaCX│        │ CelestaCX│        │ CelestaCX│
    │ Services │        │ Services │        │ Services │
    └──────────┘        └──────────┘        └──────────┘
```

**Key components:**

- **3 control plane nodes** — Run Kubernetes itself (API server, etcd database, scheduler). Must be an odd number for quorum voting.
- **3+ worker nodes** — Run CelestaCX services. Losing one worker causes pods to move to remaining workers automatically.
- **1 load balancer** — Routes all traffic to healthy control plane nodes. If one control plane node fails, traffic automatically goes to the others.

---

### Hardware Requirements

| Component | Quantity | CPU | RAM | Storage |
| --- | --- | --- | --- | --- |
| Control plane nodes | 3 | 4 vCPU each | 8 GB each | 100 GB SSD each |
| Worker nodes | 3–8 (based on your sizing tier) | Per your sizing plan → | Per your sizing plan | Per your sizing plan |
| Load balancer | 1 | 2 vCPU | 4 GB | 20 GB |

**Total minimum:** 7 servers (3 control plane + 3 worker + 1 load balancer)

> 💡 **Operating system:** All nodes must run **Ubuntu 20.04+** or **RHEL 8.4+**

---

### Before You Start

> *(Add a blue Info Panel macro around the following)*
> ✅ **Prerequisites checklist:**
> - You have **7 servers** provisioned with the hardware specs above
> - All servers are running Ubuntu 20.04+ or RHEL 8.4+
> - All servers have **static IP addresses** assigned
> - You have **root/sudo access** to all servers
> - All servers can communicate with each other on the network
> - You have your **FQDN** ready — e.g. `celestacx.yourcompany.com`
> - DNS is configured to point your FQDN to the **load balancer IP**

---

### Part 1 — Prepare All Nodes

Before building the HA cluster, all nodes need basic preparation. Most of these steps are identical to the single-node setup.

📖 **Follow the Kubernetes Setup page first**

Go to [Kubernetes Setup →](https://octavebytes-docs.atlassian.net/wiki/x/AQAdC) and complete **Steps 1.1 through 1.4** (Update System, Disable Swap, Configure Firewall) on **every server** in your HA cluster — all 3 control plane nodes and all worker nodes.

Come back to this page once those steps are complete on all nodes.

---

#### Additional Steps for HA

Once you've completed the basic preparation from the Kubernetes Setup page, complete these additional steps on each node:

---

#### Step 1.1 — Set Unique Hostnames

Give each node a unique, descriptive hostname so you can identify them easily.

**On control plane node 1:**

bash

```
sudo hostnamectl set-hostname cx-control-01
```

**On control plane node 2:**

bash

```
sudo hostnamectl set-hostname cx-control-02
```

**On control plane node 3:**

bash

```
sudo hostnamectl set-hostname cx-control-03
```

**On worker nodes:**

bash

```
sudo hostnamectl set-hostname cx-worker-01
sudo hostnamectl set-hostname cx-worker-02
sudo hostnamectl set-hostname cx-worker-03
# ... and so on for additional workers
```

---

#### Step 1.2 — Configure /etc/hosts on All Nodes

On **every node** (control plane and workers), edit the hosts file:

bash

```
sudo nano /etc/hosts
```

Add entries for **all servers in your cluster**:
```
# Control plane nodes
192.168.1.10  cx-control-01
192.168.1.11  cx-control-02
192.168.1.12  cx-control-03

# Worker nodes
192.168.1.20  cx-worker-01
192.168.1.21  cx-worker-02
192.168.1.22  cx-worker-03

# Load balancer
192.168.1.100  cx-loadbalancer
```

Replace the IP addresses with your actual server IPs. Save and exit.

> 💡 **Why this matters:** This allows nodes to communicate with each other using hostnames instead of IPs, which makes troubleshooting and log reading much easier.

---

#### Step 1.3 — Open Additional Ports for HA

HA requires additional firewall ports beyond the single-node setup.

For **Ubuntu with UFW**, run on all nodes:

bash

```
# RKE2 node registration (required for HA)
sudo ufw allow 9345/tcp

# etcd communication (control plane nodes only)
sudo ufw allow 2379:2380/tcp

# CNI networking between nodes
sudo ufw allow 8472/udp
```

For **RHEL with firewalld**, run on all nodes:

bash

```
sudo firewall-cmd --permanent --add-port=9345/tcp
sudo firewall-cmd --permanent --add-port=2379-2380/tcp
sudo firewall-cmd --permanent --add-port=8472/udp
sudo firewall-cmd --reload
```

---

#### Step 1.4 — Install RKE2

Run this on **all control plane nodes and all worker nodes** (not the load balancer):

bash

```
curl -sfL https://get.rke2.io | sudo sh -
```

This downloads and installs RKE2. It takes 1–2 minutes per node.

**Do not start RKE2 yet.** Installation and starting are separate steps. We'll start RKE2 in the correct order in Part 3.

---

**Once all nodes are prepared, proceed to Part 2 below.**

### Part 2 — Configure the Load Balancer

The load balancer sits in front of your control plane nodes and distributes traffic across all three. If one control plane node fails, the load balancer automatically stops sending traffic to it.

We're using **NGINX** as a TCP load balancer.

---

#### Step 2.1 — Install NGINX

On your **dedicated load balancer server**, run:

bash

```
sudo apt update
sudo apt install nginx -y
```

---

#### Step 2.2 — Configure NGINX

Edit the NGINX configuration file:

bash

```
sudo nano /etc/nginx/nginx.conf
```

**Replace the entire contents** with the following configuration, updating the IP addresses to match your control plane and worker nodes:

```
events {}

stream {
  # RKE2 node registration port
  upstream rke2_api {
    least_conn;
    server 192.168.1.10:9345;   # cx-control-01
    server 192.168.1.11:9345;   # cx-control-02
    server 192.168.1.12:9345;   # cx-control-03
  }

  # Kubernetes API server
  upstream k8s_api {
    least_conn;
    server 192.168.1.10:6443;   # cx-control-01
    server 192.168.1.11:6443;   # cx-control-02
    server 192.168.1.12:6443;   # cx-control-03
  }

  # HTTPS traffic to CelestaCX
  upstream https_ingress {
    least_conn;
    server 192.168.1.20:443;    # cx-worker-01
    server 192.168.1.21:443;    # cx-worker-02
    server 192.168.1.22:443;    # cx-worker-03
  }

  # HTTP traffic to CelestaCX
  upstream http_ingress {
    least_conn;
    server 192.168.1.20:80;     # cx-worker-01
    server 192.168.1.21:80;     # cx-worker-02
    server 192.168.1.22:80;     # cx-worker-03
  }

  server {
    listen 9345;
    proxy_pass rke2_api;
  }

  server {
    listen 6443;
    proxy_pass k8s_api;
  }

  server {
    listen 443;
    proxy_pass https_ingress;
  }

  server {
    listen 80;
    proxy_pass http_ingress;
  }
}
```

Save and exit.

---

#### Step 2.3 — Test and Start NGINX

**Test the configuration for syntax errors:**

bash

```
sudo nginx -t
```

You should see:
```
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

**Enable and start NGINX:**

bash

```
sudo systemctl enable nginx
sudo systemctl restart nginx
```

**Verify NGINX is running:**

bash

```
sudo systemctl status nginx
```

You should see **Active: active (running)** in green.

---

### Part 3 — Initialize the First Control Plane Node

Now that the load balancer is ready, we'll start building the Kubernetes cluster by initializing the first control plane node.

---

#### Step 3.1 — Configure the First Control Plane Node

On **cx-control-01** (your first control plane node), create the RKE2 configuration:

bash

```
sudo mkdir -p /etc/rancher/rke2
sudo nano /etc/rancher/rke2/config.yaml
```

Paste the following, replacing the FQDN and IPs with your own:

yaml

```
tls-san:
  - celestacx.yourcompany.com      # Your FQDN
  - 192.168.1.100                  # Load balancer IP
  - 192.168.1.10                   # This node's IP
  - 127.0.0.1
write-kubeconfig-mode: "0644"
cluster-init: true                 # Initializes the etcd cluster
```

**What this does:**

- `tls-san` — Allows access to Kubernetes via the FQDN and load balancer IP
- `cluster-init: true` — Tells this node to start a new etcd cluster (only use on the first node)

Save and exit.

---

#### Step 3.2 — Start RKE2 on the First Node

bash

```
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service
```

This takes **3–5 minutes** as Kubernetes initializes. Watch the progress:

bash

```
sudo journalctl -u rke2-server -f
```

Wait until you see messages like "Node is ready". Press Ctrl+C to stop watching.

---

#### Step 3.3 — Get the Cluster Token

This token is needed to join the other nodes to the cluster:

bash

```
sudo cat /var/lib/rancher/rke2/server/node-token
```

**Copy the entire token.** It will look something like:
```
K10abc123def456ghi789jkl012mno345pqr678stu901vwx234yz::server:abc123def456
```

Save this somewhere safe — you'll need it in the next steps.

---

#### Step 3.4 — Set Up kubectl on the First Node

bash

```
export PATH=$PATH:/var/lib/rancher/rke2/bin
echo 'export PATH=$PATH:/var/lib/rancher/rke2/bin' >> ~/.bashrc

export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
echo 'export KUBECONFIG=/etc/rancher/rke2/rke2.yaml' >> ~/.bashrc

source ~/.bashrc
```

**Verify kubectl works:**

bash

```
kubectl get nodes
```

You should see your first control plane node listed with status **Ready**.

---

### Part 4 — Join the Remaining Control Plane Nodes

Now we'll add the second and third control plane nodes to form a highly available etcd cluster.

---

#### Step 4.1 — Configure the Second Control Plane Node

On **cx-control-02**, create the RKE2 configuration:

bash

```
sudo mkdir -p /etc/rancher/rke2
sudo nano /etc/rancher/rke2/config.yaml
```

Paste the following, replacing IPs and the token with your actual values:

yaml

```
tls-san:
  - celestacx.yourcompany.com
  - 192.168.1.100                  # Load balancer IP
  - 192.168.1.11                   # This node's IP
  - 127.0.0.1
server: https://192.168.1.100:9345 # Load balancer address
token: <PASTE_YOUR_CLUSTER_TOKEN_HERE>
write-kubeconfig-mode: "0644"
```

Save and exit.

**Start RKE2:**

bash

```
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service
```

---

#### Step 4.2 — Configure the Third Control Plane Node

On **cx-control-03**, repeat the same process:

bash

```
sudo mkdir -p /etc/rancher/rke2
sudo nano /etc/rancher/rke2/config.yaml
```

yaml

```
tls-san:
  - celestacx.yourcompany.com
  - 192.168.1.100
  - 192.168.1.12                   # This node's IP
  - 127.0.0.1
server: https://192.168.1.100:9345
token: <PASTE_YOUR_CLUSTER_TOKEN_HERE>
write-kubeconfig-mode: "0644"
```

Save and exit.

**Start RKE2:**

bash

```
sudo systemctl enable rke2-server.service
sudo systemctl start rke2-server.service
```

---

#### Step 4.3 — Verify All Control Plane Nodes

From **cx-control-01**, verify all three control plane nodes joined:

bash

```
kubectl get nodes
```

You should see:
```
NAME           STATUS   ROLES                       AGE   VERSION
cx-control-01  Ready    control-plane,etcd,master   10m   v1.28.x
cx-control-02  Ready    control-plane,etcd,master   2m    v1.28.x
cx-control-03  Ready    control-plane,etcd,master   1m    v1.28.x
```

All three should show **Ready** status.

---

### Part 5 — Prevent CelestaCX from Running on Control Plane Nodes

Control plane nodes should ONLY run Kubernetes itself, not CelestaCX services. We use "taints" to prevent workload scheduling on these nodes.

From **cx-control-01**, run:

bash

```
kubectl taint node cx-control-01 \
  node-role.kubernetes.io/control-plane=:NoSchedule

kubectl taint node cx-control-02 \
  node-role.kubernetes.io/control-plane=:NoSchedule

kubectl taint node cx-control-03 \
  node-role.kubernetes.io/control-plane=:NoSchedule
```

---

### Part 6 — Join Worker Nodes

Worker nodes run the actual CelestaCX services. Add them now.

---

#### Step 6.1 — Configure Each Worker Node

On **each worker node** (cx-worker-01, cx-worker-02, cx-worker-03, etc.), create the configuration:

bash

```
sudo mkdir -p /etc/rancher/rke2
sudo nano /etc/rancher/rke2/config.yaml
```

Paste the following (same config for all workers, just update the IP comment):

yaml

```
server: https://192.168.1.100:9345    # Load balancer IP
token: <PASTE_YOUR_CLUSTER_TOKEN_HERE>
```

Save and exit.

---

#### Step 6.2 — Start RKE2 Agent on Each Worker

On **each worker node**, run:

bash

```
curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="agent" sudo sh -
sudo systemctl enable rke2-agent.service
sudo systemctl start rke2-agent.service
```

---

#### Step 6.3 — Verify All Workers Joined

From **cx-control-01**, check all nodes:

bash

```
kubectl get nodes
```

You should now see:
```
NAME           STATUS   ROLES                       AGE   VERSION
cx-control-01  Ready    control-plane,etcd,master   15m   v1.28.x
cx-control-02  Ready    control-plane,etcd,master   7m    v1.28.x
cx-control-03  Ready    control-plane,etcd,master   6m    v1.28.x
cx-worker-01   Ready    worker                      2m    v1.28.x
cx-worker-02   Ready    worker                      2m    v1.28.x
cx-worker-03   Ready    worker                      2m    v1.28.x
```

All nodes should show **Ready**.

---

### Part 7 — Install Helm and NGINX Ingress Controller

These are the final pieces before CelestaCX can be installed.

---

#### Step 7.1 — Install Helm

On **cx-control-01**, install Helm:

bash

```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

**Verify:**

bash

```
helm version
```

---

#### Step 7.2 — Install NGINX Ingress Controller

bash

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=NodePort \
  --set controller.service.nodePorts.http=80 \
  --set controller.service.nodePorts.https=443
```

**Verify:**

bash

```
kubectl get pods -n ingress-nginx
```

Wait until the ingress-nginx-controller pod shows **Running**.

---

### Part 8 — Verify Your HA Cluster is Ready

Before proceeding to CelestaCX installation, run all these checks:

bash

```
# All nodes are Ready
kubectl get nodes

# All system pods are Running
kubectl get pods -n kube-system

# Ingress controller is Running
kubectl get pods -n ingress-nginx

# Helm is working
helm list -A
```

> *(Add a blue Info Panel macro around the following)*
> ✅ **HA cluster is ready when:**
> - All 6+ nodes show status `Ready`
> - All kube-system pods show `Running` or `Completed`
> - ingress-nginx-controller shows `Running`
> - NGINX load balancer is running and healthy

---

### Understanding Failover Behaviour

Now that your HA cluster is running, here's what happens when components fail:

| Failure Scenario | What Happens | User Impact |
| --- | --- | --- |
| One worker node fails | Kubernetes reschedules pods to remaining workers within ~60 seconds | Active conversations may pause 1–2 minutes while pods restart |
| One control plane node fails (2 of 3 remain) | etcd maintains quorum — cluster continues normally | No impact — operations continue |
| Two control plane nodes fail (only 1 remains) | etcd loses quorum — API becomes read-only | New changes blocked but existing services keep running |
| Load balancer fails | Traffic cannot reach cluster | Full outage — consider load balancer redundancy |

> 💡 **Active conversations resume after failover.** CelestaCX stores conversation state in MongoDB. When pods restart on a new node, agents can reconnect and continue conversations. Messages sent during the ~60-second failover window are queued and delivered once service resumes.

---

### Testing Failover

Once CelestaCX is installed, test that failover works:

**Test 1 — Worker node failure:**

bash

```
# Gracefully drain a worker node
kubectl cordon cx-worker-02
kubectl drain cx-worker-02 --ignore-daemonsets --delete-emptydir-data

# Verify pods moved to other workers
kubectl get pods -n celestacx -o wide

# Restore the node
kubectl uncordon cx-worker-02
```

**Test 2 — Control plane node failure:**

bash

```
# On cx-control-02, stop RKE2
sudo systemctl stop rke2-server.service

# From cx-control-01, verify cluster still works
kubectl get nodes
kubectl get pods -n celestacx

# Restart the stopped node
# (On cx-control-02)
sudo systemctl start rke2-server.service
```

---

### ETCD Backup and Restore

In an HA deployment, etcd stores the entire cluster state. **Regular backups are mandatory.**

**Create a manual backup:**

bash

```
sudo rke2 etcd-snapshot save \
  --name celestacx-backup-$(date +%Y%m%d)
```

**List backups:**

bash

```
sudo rke2 etcd-snapshot list
```

**Restore from backup** (run on cx-control-01 only):

bash

```
sudo rke2 etcd-snapshot restore \
  --cluster-reset \
  --cluster-reset-restore-path=/var/lib/rancher/rke2/server/db/snapshots/<SNAPSHOT_NAME>
```

**Automate daily backups:**

bash

```
# Add to crontab on all control plane nodes (run: crontab -e)
0 2 * * * /usr/local/bin/rke2 etcd-snapshot save \
  --name celestacx-backup-$(date +\%Y\%m\%d) >> \
  /var/log/etcd-backup.log 2>&1
```

**Store backups externally.** Copy etcd snapshots to a location outside the cluster (NFS, S3, or another server) daily. If the entire cluster fails, backups on the cluster nodes themselves are inaccessible.

---

### What's Next

Your HA Kubernetes cluster is complete. Proceed to:

👉 [CelestaCX Installation Guide →](#)
