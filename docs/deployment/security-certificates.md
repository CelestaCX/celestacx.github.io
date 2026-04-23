# Security & Certificates
A freshly installed CelestaCX deployment needs several security configurations applied before it is ready for production use. This page covers the three main areas:
 
1. **SSL/TLS Certificates** — securing all traffic to and from CelestaCX with HTTPS
2. **Post-Installation Security Hardening** — essential steps to lock down your deployment
3. **Certificate Renewal** — keeping certificates valid over time
 
---
 
### Part 1 — SSL/TLS Certificates
 
All CelestaCX interfaces — Agent Desk, Unified Admin, Customer Widget, and APIs — must be served over HTTPS. This requires a valid SSL/TLS certificate for your FQDN.
 
You have three options depending on your environment:
 | Option | Best For | Cost |
| --- | --- | --- |
| Option A — Let's Encrypt | Internet-accessible deployments | Free |
| Option B — Commercial Certificate | Enterprise deployments, intranet, or compliance requirements | Paid |
| Option C — Self-Signed Certificate | Development, testing, or air-gapped environments only | Free | 
---
 
#### Option A — Let's Encrypt (Recommended for Internet-Accessible Deployments)
 
Let's Encrypt provides free, automatically renewable SSL certificates. Use **cert-manager** to automate certificate issuance and renewal inside your Kubernetes cluster.
 
**Install cert-manager:**
 
bash
 ```
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install cert-manager jetstack/cert-manager \
 --namespace cert-manager \
 --create-namespace \
 --set installCRDs=true
``` 
**Verify cert-manager is running:**
 
bash
 ```
kubectl get pods -n cert-manager
``` 
All three cert-manager pods should show status `Running` .
 
**Create a ClusterIssuer for Let's Encrypt:**
 
bash
 ```
nano letsencrypt-issuer.yaml
``` 
yaml
 ```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
 name: letsencrypt-prod
spec:
 acme:
 server: https://acme-v02.api.letsencrypt.org/directory
 email: admin@yourcompany.com # Replace with your email
 privateKeySecretRef:
 name: letsencrypt-prod
 solvers:
 - http01:
 ingress:
 class: nginx
``` 
bash
 ```
kubectl apply -f letsencrypt-issuer.yaml
``` 
**Create a Certificate resource for your FQDN:**
 
bash
 ```
nano celestacx-certificate.yaml
``` 
yaml
 ```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
 name: celestacx-tls
 namespace: celestacx
spec:
 secretName: cx-tls
 issuerRef:
 name: letsencrypt-prod
 kind: ClusterIssuer
 dnsNames:
 - celestacx.yourcompany.com # Replace with your FQDN
``` 
bash
 ```
kubectl apply -f celestacx-certificate.yaml
``` 
**Verify the certificate was issued:**
 
bash
 ```
kubectl get certificate -n celestacx
``` 
The `READY` column should show `True` within a few minutes. cert-manager will automatically renew this certificate before it expires.
 
> *(Add a yellow Note Panel macro around the following)*
> ⚠️ **Let's Encrypt requires your FQDN to be publicly accessible on port 80** for the HTTP-01 challenge to work. If your deployment is on a private network without public internet access, use Option B or Option C instead.
 
---
 
#### Option B — Commercial Certificate
 
If you have purchased an SSL certificate from a certificate authority (CA) such as DigiCert, Sectigo, or your organization's internal CA, follow these steps.
 
**You should have received from your CA:**
 
- A certificate file — e.g. `celestacx.crt`
- A private key file — e.g. `celestacx.key`
- Optionally, an intermediate/chain certificate — e.g. `ca-bundle.crt`
 
**If you have an intermediate certificate, combine it with your certificate:**
 
bash
 ```
cat celestacx.crt ca-bundle.crt > celestacx-full.crt
``` 
**Create the Kubernetes TLS secret:**
 
bash
 ```
kubectl create secret tls cx-tls \
 --cert=celestacx-full.crt \
 --key=celestacx.key \
 --namespace celestacx
``` 
**Verify the secret was created:**
 
bash
 ```
kubectl get secret cx-tls -n celestacx
``` 
> 💡 **Certificate renewal** for commercial certificates must be done manually. Set a calendar reminder 30 days before your certificate expires. When you receive the renewed certificate, delete the old secret and create a new one using the same command above — Kubernetes will pick up the new certificate automatically.
 
---
 
#### Option C — Self-Signed Certificate
 
Self-signed certificates are suitable for development, testing, and air-gapped environments where a trusted CA is not required. Browsers will show a security warning for self-signed certificates — this is expected and acceptable in non-production environments.
 
**Generate a self-signed certificate:**
 
bash
 ```
openssl req -x509 -nodes -days 365 \
 -newkey rsa:2048 \
 -keyout celestacx-selfsigned.key \
 -out celestacx-selfsigned.crt \
 -subj "/CN=celestacx.yourcompany.com/O=OctaveBytes" \
 -addext "subjectAltName=DNS:celestacx.yourcompany.com"
``` 
**Create the Kubernetes TLS secret:**
 
bash
 ```
kubectl create secret tls cx-tls \
 --cert=celestacx-selfsigned.crt \
 --key=celestacx-selfsigned.key \
 --namespace celestacx
``` 
> *(Add a yellow Note Panel macro around the following)*
> ⚠️ **Do not use self-signed certificates in production.** Users will see browser security warnings, and some integrations — particularly mobile apps and third-party webhooks — will refuse to connect to endpoints with untrusted certificates.
 
---
 
### Part 2 — Post-Installation Security Hardening
 
After installing CelestaCX and configuring your certificate, apply the following security hardening steps before going live.
 
---
 
#### 2a — Change All Default Passwords
 
CelestaCX installs with auto-generated or default credentials for several components. Change all of them immediately after installation.
 
**Unified Admin password:** Log in to the Unified Admin at `https://your-fqdn/admin` and change the admin password via Profile → Change Password.
 
**IAM (Keycloak) admin password:**
 
bash
 ```
# Access the Keycloak admin console
# Navigate to: https://your-fqdn/auth/admin
# Log in with the initial credentials retrieved during installation
# Go to: Admin user → Credentials → Reset Password
``` 
**Database passwords:** If you used the default passwords during installation, update them now and recreate the Kubernetes secrets:
 
bash
 ```
# Delete old secret
kubectl delete secret cx-db-credentials -n celestacx

# Recreate with new passwords
kubectl create secret generic cx-db-credentials \
 --from-literal=mongodb-password=<NEW_STRONG_PASSWORD> \
 --from-literal=postgresql-password=<NEW_STRONG_PASSWORD> \
 --from-literal=redis-password=<NEW_STRONG_PASSWORD> \
 --namespace celestacx

# Restart affected pods to pick up new credentials
kubectl rollout restart deployment -n celestacx
``` 
**ActiveMQ password:**
 
bash
 ```
kubectl create configmap activemq-config \
 --from-literal=admin-password=<NEW_STRONG_PASSWORD> \
 --namespace celestacx \
 --dry-run=client -o yaml | kubectl apply -f -
``` 
---
 
#### 2b — Configure API Gateway Rate Limiting
 
The API Gateway (APISIX) protects CelestaCX APIs from abuse. Enable IP-based rate limiting to prevent brute force and denial-of-service attempts.
 
Rate limiting is configured per route in APISIX. The recommended baseline limits are:
 | Route Type | Recommended Limit |
| --- | --- |
| Authentication endpoints | 10 requests per minute per IP |
| Agent Desk API | 300 requests per minute per agent |
| Customer Widget API | 60 requests per minute per customer session |
| Admin API | 100 requests per minute per IP | 
Contact your OctaveBytes account manager for the APISIX configuration file with these limits pre-configured for CelestaCX routes.
 
---
 
#### 2c — Enable Two-Factor Authentication
 
For admin and supervisor accounts, enable two-factor authentication (2FA) to add a second layer of login security.
 
**Configure 2FA via email OTP:**
 
Navigate to Unified Admin → Security Settings → Two-Factor Authentication and enable 2FA for the Admin and Supervisor roles.
 
Users will be prompted to verify a one-time password sent to their registered email address on each login. Ensure your SMTP configuration is working before enabling this — users will be locked out if email delivery fails.
 
> 👉 For full 2FA setup instructions see [Identity & Access Management →](#)
 
---
 
#### 2d — Configure PII Data Masking
 
If your deployment handles personal customer data — names, phone numbers, email addresses, national IDs — enable PII masking to prevent sensitive data from appearing in logs and on-screen in unauthorized contexts.
 
PII masking is configured in the Unified Admin:
 
Navigate to Unified Admin → Security → PII Data Masking and configure the fields you want masked. Two masking levels are available:
 | Level | Behaviour | Use Case |
| --- | --- | --- |
| Log masking | PII is redacted from all system logs | Required for GDPR and HIPAA compliance |
| UI masking | PII is masked on screen for agents without the "View PII" permission | Required when agents should not see full customer details | 
---
 
#### 2e — Review User Roles and Permissions
 
Before going live, audit the roles and permissions configured in IAM to ensure the principle of least privilege is applied — users should only have access to what they need for their role.
 
CelestaCX default roles and their intended scope:
 | Role | Intended For | Key Permissions |
| --- | --- | --- |
| Agent | Contact center agents | Handle conversations, view own performance |
| Supervisor | Team supervisors | Monitor agents, access dashboards, manage team |
| Quality Manager | QA staff | Access recordings, run evaluations |
| Admin | Platform administrators | Full configuration access |
| API Admin | System integrations | Programmatic API access | 
> 👉 For detailed role configuration see [Identity & Access Management →](#)
 
---
 
#### 2f — Network Security
 
Apply the following network-level security measures:
 
**Restrict Kubernetes API access:**
 
bash
 ```
# The Kubernetes API (port 6443) should only be accessible
# from known management IPs — not from the public internet
# Configure your firewall accordingly
``` 
**Restrict etcd access:**
 
bash
 ```
# etcd (ports 2379-2380) should only be accessible
# between control plane nodes — never exposed externally
``` 
**Close unused ports:** Ensure your firewall only allows the following external-facing ports:
 | Port | Purpose | Who Needs Access |
| --- | --- | --- |
| 443 | HTTPS — all CelestaCX interfaces | All users |
| 80 | HTTP — redirects to HTTPS | All users |
| 22 | SSH — server management | IT admins only, from known IPs | 
All other ports should be closed to external traffic.
 
---
 
### Part 3 — Certificate Renewal
 
#### Let's Encrypt (cert-manager)
 
No action required. cert-manager automatically renews Let's Encrypt certificates 30 days before expiry. Monitor certificate status with:
 
bash
 ```
kubectl get certificate -n celestacx
kubectl describe certificate celestacx-tls -n celestacx
``` 
#### Commercial Certificates
 
Renew manually before expiry. When you receive the renewed certificate from your CA:
 
bash
 ```
# Delete the existing secret
kubectl delete secret cx-tls -n celestacx

# Create a new secret with the renewed certificate
kubectl create secret tls cx-tls \
 --cert=renewed-celestacx-full.crt \
 --key=celestacx.key \
 --namespace celestacx

# Restart ingress controller to pick up new certificate
kubectl rollout restart deployment ingress-nginx-controller \
 -n ingress-nginx
```

### Self-Signed Certificates
Self-signed certificates generated with the command in Option C expire after 365 days. Regenerate them before expiry using the same `openssl` command with a new expiry date, then recreate the Kubernetes secret.

---

## Security Compliance

CelestaCX includes built-in support for the following compliance frameworks. These require specific configuration beyond the steps on this page — see the linked pages for details.

| Framework | Relevant For | Configuration Guide |
|---|---|---|
| **GDPR** | EU customer data | PII masking, data retention policies |
| **HIPAA** | Healthcare organizations | Encryption at rest, access controls, audit logging |
| **PCI DSS** | Payment card data handling | Voice recording controls, data isolation |

> 👉 Full compliance configuration guidance is covered in [Configuration & Administration → Security & Compliance →](#)

---

```
