# CelestaCX Installation Guide
This guide walks you through installing CelestaCX on a prepared Kubernetes cluster. By the end of this page you will have a fully running CelestaCX deployment accessible at your configured domain.
 **Before You Start** This guide assumes you have completed both:
 
- [Deployment Planning & Sizing →](#) — hardware provisioned, topology decided
- [Kubernetes Setup →](#) — Kubernetes cluster running, all nodes Ready, Helm installed
 
Do not proceed until your cluster is fully verified. Installing CelestaCX on an unstable or misconfigured cluster will result in unpredictable failures. 
---
 
### What You'll Need
 | Item | Detail |
| --- | --- |
| Running Kubernetes cluster | All nodes in Ready state — see Kubernetes Setup → |
| Helm 3.x | Installed on your server node |
| Helm registry credentials | Provided by OctaveBytes — username and password to pull CelestaCX Helm charts |
| CelestaCX license file | Provided by OctaveBytes — a .lic file tied to your deployment FQDN |
| FQDN | Your configured domain — e.g. celestacx.yourcompany.com |
| SSL certificate | Either a valid certificate for your FQDN or use the self-signed option covered in Security & Certificates → |
| SMTP credentials (optional) | If you want email notifications and password reset emails enabled from day one | 
---
 
### Installation Overview
 
The installation consists of six stages:
 ```
Stage 1 → Add the CelestaCX Helm repository
Stage 2 → Create Kubernetes namespaces
Stage 3 → Create secrets (license, credentials)
Stage 4 → Configure your values file
Stage 5 → Run the Helm install
Stage 6 → Verify the installation
``` 
---
 
### Stage 1 — Add the CelestaCX Helm Repository
 
Add the CelestaCX Helm chart repository using the credentials provided by OctaveBytes:
 
bash
 ```
helm repo add celestacx https://charts.celestacx.com \
 --username <YOUR_USERNAME> \
 --password <YOUR_PASSWORD>

helm repo update
``` 
**Verify the repository was added:**
 
bash
 ```
helm repo list
``` 
You should see `celestacx` listed. Search for available charts:
 
bash
 ```
helm search repo celestacx
``` 
Note the latest chart version — you will use it in Stage 5.
 
---
 
### Stage 2 — Create Kubernetes Namespaces
 
CelestaCX uses dedicated namespaces to organize its services. Create them before installation:
 
bash
 ```
# Primary CelestaCX namespace
kubectl create namespace celestacx

# External services namespace (ingress, API gateway)
kubectl create namespace cx-external

# Monitoring namespace
kubectl create namespace monitoring
``` 
**Verify namespaces were created:**
 
bash
 ```
kubectl get namespaces
``` 
---
 
### Stage 3 — Create Secrets
 
Secrets store sensitive values — credentials, certificates, and license data — securely in Kubernetes rather than in plain text configuration files.
 
#### 3a — Create the License Secret
 
bash
 ```
kubectl create secret generic cx-license \
 --from-file=license.lic=/path/to/your/license.lic \
 --namespace celestacx
``` 
#### 3b — Create the Image Pull Secret
 
This allows Kubernetes to pull CelestaCX container images from the private registry:
 
bash
 ```
kubectl create secret docker-registry cx-registry \
 --docker-server=registry.celestacx.com \
 --docker-username=<YOUR_USERNAME> \
 --docker-password=<YOUR_PASSWORD> \
 --namespace celestacx
``` 
#### 3c — Create the SSL Certificate Secret
 
If you have a valid SSL certificate for your FQDN:
 
bash
 ```
kubectl create secret tls cx-tls \
 --cert=/path/to/your/certificate.crt \
 --key=/path/to/your/private.key \
 --namespace celestacx
``` 
If you are using a self-signed certificate for now, see [Security & Certificates →](#) for how to generate one and create this secret.
 
#### 3d — Create the Database Password Secret
 
bash
 ```
kubectl create secret generic cx-db-credentials \
 --from-literal=mongodb-password=<CHOOSE_A_STRONG_PASSWORD> \
 --from-literal=postgresql-password=<CHOOSE_A_STRONG_PASSWORD> \
 --from-literal=redis-password=<CHOOSE_A_STRONG_PASSWORD> \
 --namespace celestacx
``` 
> *(Add a yellow Note Panel macro around the following)*
> ⚠️ **Password Requirements** Use strong, randomly generated passwords for all database credentials. A minimum of 20 characters including uppercase, lowercase, numbers, and special characters is recommended. Store these passwords securely — you will need them for upgrades and recovery operations.
 
---
 
### Stage 4 — Configure Your Values File
 
The values file is where you customize CelestaCX for your specific deployment. Create a file called `cx-values.yaml` on your server node:
 
bash
 ```
nano cx-values.yaml
``` 
Add the following content, replacing all placeholder values with your own:
 
yaml
 ```
# ─────────────────────────────────────────
# CelestaCX Deployment Configuration
# ─────────────────────────────────────────

global:
 # Your deployment domain
 fqdn: celestacx.yourcompany.com

 # Deployment environment
 environment: production # Options: production, staging, development

 # Image pull secret name (created in Stage 3b)
 imagePullSecret: cx-registry

 # TLS secret name (created in Stage 3c)
 tlsSecret: cx-tls

# ─────────────────────────────────────────
# Identity & Access Management
# ─────────────────────────────────────────
iam:
 enabled: true
 adminUsername: admin
 adminEmail: admin@yourcompany.com
 # Admin password will be auto-generated on first install
 # and displayed in the install logs — change it immediately after login

# ─────────────────────────────────────────
# Database Configuration
# ─────────────────────────────────────────
mongodb:
 enabled: true
 credentialsSecret: cx-db-credentials
 storage: 50Gi # Adjust based on your sizing tier

postgresql:
 enabled: true
 credentialsSecret: cx-db-credentials
 storage: 20Gi

redis:
 enabled: true
 credentialsSecret: cx-db-credentials

# ─────────────────────────────────────────
# Voice & Media (set enabled: false if
# not using voice channels)
# ─────────────────────────────────────────
mediaServer:
 enabled: false # Set to true if deploying voice/video channels
 # nodeSelector: # Uncomment to pin to dedicated media server node
 # kubernetes.io/hostname: cx-media-node

# ─────────────────────────────────────────
# AI Bots / Rasa (set enabled: false if
# not using AI self-service)
# ─────────────────────────────────────────
rasa:
 enabled: false # Set to true if deploying Rasa NLU bots
 # nodeSelector: # Uncomment to pin to dedicated Rasa node
 # kubernetes.io/hostname: cx-rasa-node

# ─────────────────────────────────────────
# Email Notifications (optional)
# ─────────────────────────────────────────
smtp:
 enabled: false # Set to true to enable email notifications
 host: smtp.yourcompany.com
 port: 587
 username: noreply@yourcompany.com
 password: <SMTP_PASSWORD>
 fromAddress: noreply@yourcompany.com

# ─────────────────────────────────────────
# Licensing
# ─────────────────────────────────────────
license:
 secretName: cx-license

# ─────────────────────────────────────────
# Ingress Configuration
# ─────────────────────────────────────────
ingress:
 enabled: true
 className: nginx
 tlsEnabled: true
 tlsSecretName: cx-tls
``` 
> 💡 **Save this file.** You will need it again for upgrades and configuration changes. Store it in a secure location alongside your other deployment credentials.
 
---
 
### Stage 5 — Run the Helm Install
 
With your values file ready, install CelestaCX:
 
bash
 ```
helm install celestacx celestacx/celestacx \
 --namespace celestacx \
 --values cx-values.yaml \
 --version <CHART_VERSION> \
 --timeout 15m \
 --wait
``` 
Replace `<CHART_VERSION>` with the version noted in Stage 1.
 
The `--wait` flag tells Helm to wait until all pods are running before returning. This typically takes **8–15 minutes** on the first install as container images are pulled and databases are initialized.
 
**Watch the installation progress in a separate terminal:**
 
bash
 ```
kubectl get pods -n celestacx --watch
``` 
You should see pods progressively move from `Pending` → `Init` → `Running` .
 
---
 
### Stage 6 — Verify the Installation
 
Once the Helm install completes, run the following checks:
 
#### Check All Pods Are Running
 
bash
 ```
kubectl get pods -n celestacx
``` 
All pods should show status `Running` or `Completed` . If any pods are in `Error` or `CrashLoopBackOff` state, see the [Troubleshooting →](#) section.
 
#### Check Services Are Accessible
 
bash
 ```
kubectl get services -n celestacx
kubectl get ingress -n celestacx
``` 
#### Retrieve the Initial Admin Password
 
bash
 ```
kubectl logs -n celestacx \
 $(kubectl get pods -n celestacx -l app=iam -o jsonpath='{.items[0].metadata.name}') \
 | grep "Initial admin password"
``` 
Note this password — you will use it for your first login.
 
#### Access the Platform
 
Open a browser and navigate to your FQDN:
 | Interface | URL |
| --- | --- |
| Unified Admin | https://celestacx.yourcompany.com/admin |
| Agent Desk | https://celestacx.yourcompany.com/agent |
| IAM Console | https://celestacx.yourcompany.com/auth | 
Log in to the **Unified Admin** with:
 
- **Username:** `admin`
- **Password:** *(the initial admin password retrieved above)*
 **Change the admin password immediately after your first login.** The initial password is auto-generated and stored in logs — it should not remain as the active admin password. 
---
 
### Post-Installation Steps
 
After a successful installation, complete the following before handing the system to users:
 
**Security**
 
- Change the initial admin password
- Configure SSL certificates if using self-signed *(see* [*Security & Certificates →*](#) *)*
- Review and configure IAM roles and permissions
 
**Configuration**
 
- Add your first users and assign roles *(see* [*Identity & Access Management →*](#) *)*
- Configure your channels *(see* [*Channels & Integrations →*](#) *)*
- Set up routing rules and queues *(see* [*Configuration & Administration →*](#) *)*
- Configure business calendars and wrap-up codes
 
**Verification**
 
- Send a test message through at least one channel end-to-end
- Verify agent can log in to Agent Desk and receive a test conversation
- Verify supervisor can access dashboards and monitoring views
- Verify reports are populating in the Reports section
 
---
 
### Uninstalling CelestaCX
 
If you need to remove CelestaCX from your cluster:
 
bash
 ```
# Remove the CelestaCX Helm release
helm uninstall celestacx --namespace celestacx

# Remove the namespace and all resources within it
kubectl delete namespace celestacx
```

> *(Add a yellow Note Panel macro around the following)*
>
> ⚠️ **Data Loss Warning**
> Uninstalling CelestaCX and deleting the namespace will permanently delete all conversation data, customer profiles, and configuration stored in the cluster. Ensure you have taken a full backup before proceeding. See [Platform Administration →](#) for backup procedures.

---

## Troubleshooting Installation Issues

| Symptom | Likely Cause | Resolution |
|---|---|---|
| Pods stuck in `Pending` | Insufficient node resources | Check `kubectl describe pod <pod-name> -n celestacx` for resource errors — scale up nodes or reduce resource requests |
| Pods in `ImagePullBackOff` | Invalid registry credentials | Verify the `cx-registry` secret was created correctly with valid credentials |
| Helm install times out | Slow image pulls or slow node | Increase `--timeout` to `30m` and retry — large images can take time on slow connections |
| IAM pod in `CrashLoopBackOff` | Database not yet ready | Wait 2–3 minutes and check again — IAM depends on PostgreSQL which may still be initializing |
| Cannot access FQDN in browser | DNS not propagated or ingress misconfigured | Verify DNS points to correct IP with `nslookup celestacx.yourcompany.com` — check ingress with `kubectl get ingress -n celestacx` |
| SSL certificate warning in browser | Self-signed certificate in use | Expected if using self-signed — see [Security & Certificates →](#) to configure a valid certificate |

---

```
