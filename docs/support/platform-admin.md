# Platform Administration
#### Overview
 
This page covers the ongoing operational tasks that keep a CelestaCX deployment healthy over its lifetime — backup and restore, platform upgrades, SSL certificate maintenance, credential rotation, license management, and data archival. These tasks are performed by platform operators and IT administrators, typically on a scheduled basis or in response to specific operational triggers.
 
**Prerequisites:**
 
- `kubectl` configured and authenticated against the cluster
- Helm installed and configured for the deployment
- Access to Unified Admin for license and archival tasks
- Keycloak admin access for IAM-related operations
 
---
 
#### 1. Backup & Restore
 
Regular backups are the single most important operational practice for a production CelestaCX deployment. A backup that has not been tested is not a backup — always verify that restores work before you need them.
 
#### What Needs to Be Backed Up
 | Component | What It Holds | Backup Method |
| --- | --- | --- |
| MongoDB | All conversation data, tenant configuration, user data, routing rules | mongodump or managed backup service |
| Redis | Active session state and queue data (if persistence is enabled) | RDB snapshot |
| PostgreSQL | Vault secrets, Superset metadata, reporting configuration | pg_dump |
| Kubernetes cluster state | Helm values, Kubernetes secrets, ingress rules, ConfigMaps | etcd snapshot or Velero |
| Media files | Voice recordings, file attachments | Object storage backup (S3-compatible) | 
> **Critical note:** Redis, MongoDB, ActiveMQ, and PostgreSQL deployed as non-HA single instances are single points of failure. If any of these fail without a recent backup, active conversation data may be permanently lost. See [Deployment & Infrastructure → High Availability](#) for HA configuration guidance.
 
#### MongoDB Backup
 
bash
 ```
# Full backup to a dated directory
mongodump \
 --uri="mongodb://<user>:<password>@<host>:27017" \
 --out=/backups/mongo/$(date +%Y%m%d)

# Backup a specific database only
mongodump \
 --uri="mongodb://<user>:<password>@<host>:27017/ccm_db" \
 --out=/backups/mongo/ccm_db_$(date +%Y%m%d)
``` 
**Recommended frequency:** Daily full backup, retained for 30 days minimum.
 
#### MongoDB Restore
 
bash
 ```
# Restore from a specific backup
mongorestore \
 --uri="mongodb://<user>:<password>@<host>:27017" \
 /backups/mongo/<date>

# Restore a specific database
mongorestore \
 --uri="mongodb://<user>:<password>@<host>:27017" \
 --nsInclude="ccm_db.*" \
 /backups/mongo/<date>
``` 
After restore, verify data integrity by checking recent conversation records and confirming tenant configuration is intact.
 
#### Redis Backup
 
Redis persistence must be enabled for backups to be meaningful. CelestaCX deployments should use RDB persistence as a minimum.
 
bash
 ```
# Trigger a manual RDB snapshot from within the Redis pod
kubectl exec -it <redis-pod> -n <namespace> -- redis-cli BGSAVE

# Check snapshot status
kubectl exec -it <redis-pod> -n <namespace> -- redis-cli LASTSAVE
``` 
The RDB file ( `dump.rdb` ) is written to the Redis data directory. Copy this file to off-node storage as part of your backup routine.
 
#### PostgreSQL Backup
 
bash
 ```
# Full database dump
pg_dump \
 -h <host> \
 -U <user> \
 <database> > /backups/pg/$(date +%Y%m%d).sql

# Restore
psql \
 -h <host> \
 -U <user> \
 <database> < /backups/pg/<date>.sql
``` 
#### Kubernetes State Backup
 
bash
 ```
# RKE2 etcd snapshot
rke2 etcd-snapshot save --name cx-backup-$(date +%Y%m%d)

# List available snapshots
rke2 etcd-snapshot list

# Store snapshots off-node — copy to your backup storage
``` 
For more comprehensive Kubernetes backup including Persistent Volume Claims, consider Velero for backup and restore of the full cluster state.
 
#### Backup Verification Checklist
 
Run this checklist at least monthly — ideally in a staging environment:
 ```
☐ MongoDB backup completed without errors
☐ PostgreSQL backup completed without errors
☐ etcd snapshot saved and stored off-node
☐ Redis RDB file copied to backup storage
☐ Test restore performed in staging environment
☐ Post-restore: agents can log in
☐ Post-restore: inbound test conversation routes correctly
☐ Post-restore: tenant configuration intact
☐ Post-restore: reporting data accessible
``` 
---
 
#### 2. Platform Upgrades
 
CelestaCX upgrades are delivered as new Helm chart versions. Components must be upgraded in a specific order to avoid dependency issues. Always upgrade in a staging environment first and have a tested rollback plan before upgrading production.
 
#### Pre-Upgrade Checklist
 ```
☐ Current cluster health is green — all pods Running, no alerts firing
☐ Full backup completed and restore tested
☐ Maintenance window agreed with tenant administrators
☐ Release notes reviewed for breaking changes or migration steps
☐ kubectl and Helm versions compatible with the target release
☐ Sufficient free disk space on all nodes for new container images
☐ Rollback procedure documented and ready
``` 
#### Upgrade Order
 
Components must be upgraded in this sequence to respect dependencies:
 ```
1. Infrastructure layer
 └── RKE2 (Kubernetes) → cert-manager → ingress controller

2. Data layer
 └── MongoDB → Redis → PostgreSQL

3. Core CX services
 └── IAM (Keycloak) → CX Router → Core Conversation Manager (CCM)

4. Supporting services
 └── Agent Manager → Unified Admin → ETL / CX Analyser

5. Channel connectors
 └── Each active connector (WhatsApp, email, voice, etc.)

6. Media server (if voice is deployed)

7. Frontend
 └── Agent Desk → Customer Widget
``` 
#### Upgrading a Component via Helm
 
bash
 ```
# Update Helm repositories
helm repo update

# Check current deployed version
helm list -n expertflow

# Upgrade a specific chart to a new version
helm upgrade <release-name> <chart-name> \
 -n expertflow \
 --version <target-version> \
 -f values.yaml

# Monitor the rollout
kubectl rollout status deployment/<deployment-name> -n expertflow
``` 
#### Post-Upgrade Validation
 
After each component upgrade, confirm the rollout succeeded before proceeding to the next:
 
bash
 ```
# Confirm all pods are Running after upgrade
kubectl get pods -n expertflow | grep -v Running

# Check for new ERROR patterns introduced by the upgrade
kubectl logs <pod-name> -n expertflow --since=10m | grep ERROR

# Verify the component version
kubectl describe pod <pod-name> -n expertflow | grep Image
``` 
**End-to-end smoke test after full upgrade:**
 ```
☐ Agents can log in to Agent Desk
☐ Agent state changes work correctly
☐ Inbound test conversation routes to the correct queue and agent
☐ Supervisor dashboards show live data
☐ Historical reports load in Superset
☐ All channel connectors show as Connected in Unified Admin
☐ No new ERROR patterns in CCM or router logs
``` 
#### Rollback Procedure
 
If an upgrade introduces a critical issue, roll back the affected component:
 
bash
 ```
# Roll back a Helm release to the previous version
helm rollback <release-name> -n expertflow

# Monitor the rollback
kubectl rollout status deployment/<deployment-name> -n expertflow

# If rollback is insufficient, restore from pre-upgrade backup
# Follow the MongoDB and PostgreSQL restore procedures above
``` 
---
 
#### 3. SSL Certificate Management
 
CelestaCX uses SSL/TLS for all external and inter-service communication. Expired certificates will cause service failures and browser security warnings.
 
#### Checking Certificate Expiry
 
bash
 ```
# List all TLS secrets in the cluster
kubectl get secrets -A | grep tls

# Check the expiry date of a specific certificate
kubectl get secret <tls-secret-name> -n <namespace> \
 -o jsonpath='{.data.tls\.crt}' | \
 base64 --decode | \
 openssl x509 -noout -dates

# Check the RKE2 cluster certificate expiry
rke2 certificate check
``` 
Set a reminder to check certificate expiry at least 60 days before the expiry date. The platform will start showing warnings at 30 days remaining.
 
#### Extending RKE2 SSL Certificates
 
RKE2 cluster certificates have a default validity of one year. To extend them before they expire:
 
bash
 ```
# Rotate RKE2 certificates
rke2 certificate rotate

# After rotation, restart RKE2 on all nodes
systemctl restart rke2-server # on control plane nodes
systemctl restart rke2-agent # on worker nodes

# Verify the new expiry date
rke2 certificate check
``` 
> **Important:** Certificate rotation requires a brief restart of the RKE2 service. Coordinate with tenant administrators and schedule a maintenance window. Active conversations may be interrupted during the restart.
 
#### Renewing Application SSL Certificates
 
For certificates managed by cert-manager (the recommended approach):
 
bash
 ```
# Check certificate status
kubectl get certificates -A

# Force renewal of a specific certificate
kubectl annotate certificate <cert-name> -n <namespace> \
 cert-manager.io/force-renewal="true"

# Monitor renewal progress
kubectl describe certificate <cert-name> -n <namespace>
``` 
For certificates managed externally (e.g. from a corporate CA), update the Kubernetes TLS secret with the new certificate and key:
 
bash
 ```
kubectl create secret tls <tls-secret-name> \
 --cert=path/to/cert.pem \
 --key=path/to/key.pem \
 -n <namespace> \
 --dry-run=client -o yaml | kubectl apply -f -
``` 
---
 
#### 4. Credential Rotation
 
Credentials should be rotated regularly as part of security hygiene, and immediately if a credential is suspected to have been compromised.
 
#### Rotating ActiveMQ Passwords
 
ActiveMQ ships with default credentials that **must** be changed before go-live. Rotation is managed via Kubernetes ConfigMap:
 
bash
 ```
# Edit the ActiveMQ ConfigMap with the new credentials
kubectl edit configmap activemq-config -n expertflow

# Restart the ActiveMQ pod to pick up the new credentials
kubectl rollout restart deployment/activemq -n expertflow

# Update any services that connect to ActiveMQ with the new credentials
# (typically managed via Helm values — update and redeploy affected services)
``` 
#### Rotating Vault Dynamic Credentials
 
If HashiCorp Vault is deployed for secrets management, dynamic credentials for MongoDB, Redis, and ActiveMQ are auto-rotated by Vault according to the configured lease duration. To force an immediate rotation:
 
bash
 ```
# Revoke the current dynamic credential lease
vault lease revoke -prefix <lease-path>

# Services will automatically request new credentials on their next connection
``` 
#### Rotating Keycloak Admin Credentials
 
1. Log in to the Keycloak admin console at `https://<your-domain>/auth`
2. Navigate to the **Master** realm → **Users** → select the admin user
3. Go to the **Credentials** tab and click **Set Password**
4. Enter the new password, set Temporary to **Off** , and save
5. Update any automation scripts or monitoring tools that use these credentials
 
---
 
#### 5. License Management
 
CelestaCX licensing is based on concurrent agent seats. Each tenant has a purchased license capacity and an expiry date.
 
#### Accessing the License Dashboard
 
1. Log in to Unified Admin as an administrator
2. Navigate to **General Settings → License Info**
3. The table shows all active products for your tenant
 
#### License Metrics
 | Metric | What It Means | Action Required |
| --- | --- | --- |
| Purchased Licenses | Total concurrent agent seats allocated to this tenant | Contact OctaveBytes to increase if needed |
| Consumed Licenses | Current number of agents logged in across all MRDs | Monitor during peak hours — approaching the limit may affect routing |
| Expiry Date | Date the license key deactivates | Initiate renewal at least 30 days before this date |
| Activation Status | Active or Suspended | If Suspended, check subscription status with OctaveBytes | 
#### Automated Warnings
 
The Unified Admin interface shows:
 
- A **yellow banner** when consumption reaches 90% of the purchased limit
- A **days remaining counter** starting 30 days before expiry
 
#### If a Product Does Not Appear in License Info
 
1. Confirm at least one agent has successfully logged into that product at least once
2. Refresh the License Info page manually to sync with the licence engine
3. If the issue persists, verify the Master License Key in the platform configuration and contact OctaveBytes support
 
---
 
#### 6. Data Archival
 
CelestaCX supports configurable archival of older interaction data — moving it from the primary operational database to long-term storage to manage MongoDB size and maintain query performance over time.
 
#### Archival Configuration
 
Navigate to **Unified Admin → System → Archival Configuration** to configure:
 | Setting | Description |
| --- | --- |
| Retention Period | How long interactions remain in the primary database before being archived (e.g. 90 days) |
| Storage Target | The destination for archived data — an S3-compatible object store or a secondary MongoDB instance |
| Archival Schedule | When the archival job runs — typically nightly during off-peak hours |
| Include Transcripts | Whether full conversation transcripts are archived alongside metadata |
| Include Recordings | Whether voice recordings are included in the archival job | 
#### Archival Best Practices
 
- Set the retention period based on your regulatory and business requirements. Common configurations are 90 days for primary storage and 7 years for long-term archive.
- Test archival and retrieval in a staging environment before enabling in production.
- Monitor the archival job log for failures — a failed archival job will not alert by default unless Superset alerting is configured.
- Confirm that archived data is still accessible for compliance reviews before deleting it from primary storage.
 
---
 
#### 7. Routine Administration Calendar
 
Use this schedule as a baseline for planned operational activities:
 | Frequency | Task |
| --- | --- |
| Daily | Check pod health (`kubectl get pods -A |
| Daily | Review ERROR logs from CCM and CX Router |
| Weekly | Check MongoDB replication lag and Redis memory |
| Weekly | Verify backup jobs completed successfully |
| Monthly | Test restore from backup in staging |
| Monthly | Check SSL certificate expiry dates |
| Monthly | Review license consumption trends |
| Quarterly | Rotate credentials (ActiveMQ, admin accounts) |
| Annually | Renew RKE2 cluster certificates (before expiry) |
| As released | Review release notes and plan upgrade cycle | 
---
 
#### What's Next
 
- [**Data Platform & ETL**](#) — how interaction data flows through the reporting pipeline and how to manage it
- [**Troubleshooting Guide**](#) — symptom-based diagnosis for common platform issues
- [**Logging & Monitoring**](#) — log access, cluster health, and alerting setup
- [**FAQs**](#) — quick answers to common operational questions
