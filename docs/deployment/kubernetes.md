# Kubernetes Setup

This page walks you through installing and configuring Kubernetes in preparation for CelestaCX deployment. We'll start with a complete single-node setup that you can use for development, testing, or small production deployments. Then, if you need multi-node or high availability, we'll show you what additional steps are required.

By the end of this page you will have a working Kubernetes cluster ready for CelestaCX installation.

> *(Add a yellow Note Panel macro around the following)*
> ⚠️ **Prerequisites** Before starting, confirm you have completed [Deployment Planning & Sizing →](#) and have your node(s) ready with:
> - **Ubuntu 20.04+** or **RHEL 8.4+** installed
> - **Root or sudo access**
> - **At least 8 vCPU and 32 GB RAM** per node
> - **SSH access** available

---

### Single-Node Kubernetes Setup

If you are deploying for development, POC, or a small production environment (up to 100 agents, digital channels only), a single-node setup is the right starting point. All Kubernetes services and CelestaCX components run on one machine.

This section takes you from a fresh Linux server to a fully working Kubernetes cluster step by step.

---

#### Step 1 — Prepare Your Node

Run all of the following commands on your server.

**Update the system:**

bash

```
sudo apt update && sudo apt upgrade -y
```

For RHEL/CentOS:

bash

```
sudo dnf update -y
```

**Disable swap***(required by Kubernetes):*

bash

```
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

**Verify swap is disabled:**

bash

```
free -h
# The "Swap" line should show 0B
```

**Set a hostname:**

bash

```
sudo hostnamectl set-hostname celestacx-node
```

**Configure your firewall** to allow the following ports:

| Port | Protocol | Purpose |
| --- | --- | --- |
| 6443 | TCP | Kubernetes API server |
| 10250 | TCP | Kubelet metrics |
| 80, 443 | TCP | HTTP/HTTPS for CelestaCX web interfaces |

For **Ubuntu with UFW:**

bash

```
sudo ufw allow 6443/tcp
sudo ufw allow 10250/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```

For **RHEL with firewalld:**

bash

```
sudo firewall-cmd --permanent --add-port=6443/tcp
sudo firewall-cmd --permanent --add-port=10250/tcp
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload
```

---

#### Step 2 — Install RKE2

We recommend **RKE2** for production deployments. It is security-focused, enterprise-ready, and the most thoroughly tested Kubernetes distribution with CelestaCX.

**Download and install RKE2:**

bash

```
curl -sfL https://get.rke2.io | sudo sh -
```

This downloads RKE2 and installs it on your system. It will take about 1–2 minutes.

---

#### Step 3 — Configure RKE2

Create the RKE2 configuration directory:

bash

```
sudo mkdir -p /etc/rancher/rke2
```

Create the configuration file:

bash

```
sudo nano /etc/rancher/rke2/config.yaml
```

Paste the following content, replacing `celestacx.yourcompany.com` with your actual FQDN and `192.168.1.10` with your server's IP address:

yaml

```
tls-san:
  - celestacx.yourcompany.com    # Your FQDN
  - 192.168.1.10                 # Your server's IP address
  - 127.0.0.1
write-kubeconfig-mode: "0644"
```

**What this does:**

- `tls-san` — Tells Kubernetes which domain names and IPs are valid for accessing the API server
- `write-kubeconfig-mode` — Makes the kubeconfig file readable so you can use kubectl without sudo

Save and exit (Ctrl+X, then Y, then Enter).

---

#### Step 4 — Start RKE2

**Enable RKE2 to start on boot:**

bash

```
sudo systemctl enable rke2-server.service
```

**Start RKE2:**

bash

```
sudo systemctl start rke2-server.service
```

This will take **2–5 minutes** as Kubernetes initializes for the first time. You can watch the progress:

bash

```
sudo journalctl -u rke2-server -f
```

Press Ctrl+C to stop watching the logs once you see "Node is ready" messages.

**Verify RKE2 is running:**

bash

```
sudo systemctl status rke2-server.service
```

You should see **Active: active (running)** in green.

---

#### Step 5 — Set Up kubectl

`kubectl` is the command-line tool you use to interact with Kubernetes. RKE2 installs its own version — you just need to configure your environment to use it.

**Add RKE2 binaries to your PATH:**

bash

```
export PATH=$PATH:/var/lib/rancher/rke2/bin
echo 'export PATH=$PATH:/var/lib/rancher/rke2/bin' >> ~/.bashrc
```

**Set the kubeconfig location:**

bash

```
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
echo 'export KUBECONFIG=/etc/rancher/rke2/rke2.yaml' >> ~/.bashrc
```

**Apply the changes:**

bash

```
source ~/.bashrc
```

**Test kubectl:**

bash

```
kubectl get nodes
```

You should see output like this:
```
NAME               STATUS   ROLES                       AGE   VERSION
celestacx-node     Ready    control-plane,etcd,master   2m    v1.28.x
```

The **STATUS** column should say **Ready**. If it says **NotReady**, wait 1–2 minutes and try again — it takes a moment for the network plugin to initialize.

---

#### Step 6 — Install Helm

Helm is the package manager for Kubernetes. CelestaCX is installed using Helm, so you need it on your system.

**Install Helm:**

bash

```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

**Verify Helm is installed:**

bash

```
helm version
```

You should see version information displayed.

---

#### Step 7 — Install the NGINX Ingress Controller

The ingress controller routes external traffic (HTTP/HTTPS) into your Kubernetes cluster. Without it, users can't access CelestaCX from their browsers.

**Add the NGINX ingress Helm repository:**

bash

```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

**Install NGINX ingress:**

bash

```
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=NodePort \
  --set controller.service.nodePorts.http=80 \
  --set controller.service.nodePorts.https=443
```

> 💡 **Why NodePort?** In a single-node setup, we use NodePort to expose services directly on ports 80 and 443. This means your server's IP on ports 80 and 443 will route traffic into Kubernetes.

**Verify the ingress controller is running:**

bash

```
kubectl get pods -n ingress-nginx
```

Wait until you see the ingress-nginx-controller pod showing status **Running**. This takes about 1–2 minutes.

---

#### Step 8 — Verify Your Cluster is Ready

Before proceeding to install CelestaCX, run these final checks:

**All nodes are Ready:**

bash

```
kubectl get nodes
```

Expected: Your node shows `Ready`

**All system pods are Running:**

bash

```
kubectl get pods -n kube-system
```

Expected: All pods show `Running` or `Completed`

**Ingress controller is Running:**

bash

```
kubectl get pods -n ingress-nginx
```

Expected: ingress-nginx-controller pod shows `Running`

**Helm is working:**

bash

```
helm list -A
```

Expected: Shows an empty list (or shows ingress-nginx if you ran the above command)

> *(Add a blue Info Panel macro around the following)*
> ✅ **Your single-node Kubernetes cluster is ready!** All checks passed. You can now proceed to [CelestaCX Installation Guide →](#)
