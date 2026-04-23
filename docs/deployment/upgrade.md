This page covers how to upgrade CelestaCX from one version to the next. Whether you are upgrading a single customer deployment or rolling out a new version across multiple tenants, follow the process on this page to ensure a safe, predictable upgrade.
 *(Add a yellow Note Panel macro around the following)*
 
⚠️ **Always read the version-specific notes** for your upgrade path before starting. Some upgrades include database schema changes, configuration file updates, or component replacements that require additional steps beyond the standard process. 
---
 
### Upgrade Compatibility Matrix
 
Before upgrading, confirm your upgrade path is supported. Always upgrade one major version at a time — do not skip major versions.
 | From  Version | To  Version | Supported | Notes |
| --- | --- | --- | --- |
| 4.10.x | 5.0 | ✅  Yes | Includes  APISIX  reconfiguration  —  see  version  notes |
| 4.9.x | 4.10 | ✅  Yes | Includes  schema  migration  steps |
| 4.8.x | 4.9 | ✅  Yes | Standard  upgrade |
| 4.7.x | 4.8 | ✅  Yes | Standard  upgrade |
| 4.6.x | 4.7 | ✅  Yes | Standard  upgrade |
| 4.5.x | 4.6 | ✅  Yes | Standard  upgrade |
| 4.x | 5.0  (direct) | ❌  No | Must  upgrade  to  4.10  first |
| 3.x | 4.x | ❌  No | Contact  OctaveBytes  —  migration  assistance  required | 💡 **Patch upgrades** (e.g., 4.10.0 → 4.10.1) follow the same process as minor upgrades but carry significantly lower risk. They contain bug fixes only and do not include schema or configuration changes unless explicitly noted in the release notes. 
---
 
### Before You Upgrade — Pre-Upgrade Checklist
 
Complete every item on this checklist before starting any upgrade. Skipping pre-upgrade steps is the most common cause of upgrade failures.
 
*(Add a blue Info Panel macro around the following)*
 
**Back Up Everything**
 
- Take an etcd snapshot of the Kubernetes cluster state
- Back up the MongoDB database
- Back up the PostgreSQL reporting database
- Save a copy of your current `cx-values.yaml` Helm values file
- Save a copy of all custom configuration files
 
**Verify Current Health**
 
- All pods in the `celestacx` namespace are in `Running` or `Completed` state
- No active incidents or known issues on the current version
- All nodes show `Ready` status
 
**Plan Your Maintenance Window**
 
- Notify users of the planned maintenance window — upgrades cause a brief service interruption
- Schedule the upgrade during off-peak hours
- Confirm your rollback plan before starting
 
**Review the New Version**
 
- Read the release notes for the target version — see [Release Notes & Version History →](#)
- Read the version-specific upgrade notes on this page for your upgrade path
- Confirm your license file is valid for the new version
 
---
 
### Standard Upgrade Process
 
This process applies to all upgrades. Follow it in order without skipping steps.
 
---
 
#### Step 1 — Back Up the Cluster
 
**Take an etcd snapshot:**
 
bash
 ```
sudo rke2 etcd-snapshot save \
  --name pre-upgrade-$(date +%Y%m%d-%H%M)
``` 
**Back up MongoDB:**
 
bash
 ```
kubectl exec -n celestacx \
  $(kubectl get pods -n celestacx -l app=mongodb \
  -o jsonpath='{.items[0].metadata.name}') \
  -- mongodump \
  --username admin \
  --password <MONGODB_PASSWORD> \
  --out /tmp/mongodb-backup-$(date +%Y%m%d)

# Copy the backup out of the pod
kubectl cp celestacx/$(kubectl get pods -n celestacx \
  -l app=mongodb -o jsonpath='{.items[0].metadata.name}'):\
/tmp/mongodb-backup-$(date +%Y%m%d) \
  ./mongodb-backup-$(date +%Y%m%d)
``` 
**Back up PostgreSQL:**
 
bash
 ```
kubectl exec -n celestacx \
  $(kubectl get pods -n celestacx -l app=postgresql \
  -o jsonpath='{.items[0].metadata.name}') \
  -- pg_dumpall \
  -U postgres > ./postgresql-backup-$(date +%Y%m%d).sql
``` 
**Save your current Helm values:**
 
bash
 ```
helm get values celestacx -n celestacx > \
  cx-values-backup-$(date +%Y%m%d).yaml
``` 
---
 
#### Step 2 — Update Your Values File
 
Check if the new version requires any changes to your `cx-values.yaml` . The version-specific notes later on this page will tell you if any new fields are required or if any existing fields have changed.
 
Pull the new chart to review default values:
 
bash
 ```
helm repo update

helm show values celestacx/celestacx \
  --version <NEW_CHART_VERSION> > new-default-values.yaml
``` 
Compare your current values against the new defaults:
 
bash
 ```
diff cx-values.yaml new-default-values.yaml
``` 
Update your `cx-values.yaml` as needed based on the diff and the version-specific notes for your upgrade path.
 
---
 
#### Step 3 — Run the Helm Upgrade
 
bash
 ```
helm upgrade celestacx celestacx/celestacx \
  --namespace celestacx \
  --values cx-values.yaml \
  --version <NEW_CHART_VERSION> \
  --timeout 20m \
  --wait
``` 
Watch the upgrade progress in a separate terminal:
 
bash
 ```
kubectl get pods -n celestacx --watch
``` 
During the upgrade you will see pods terminating and new pods starting. This is normal. The upgrade typically takes **10–20 minutes** depending on your cluster speed and the number of components being updated.
 
---
 
#### Step 4 — Run Database Schema Migrations
 
Some upgrades include database schema changes. After the Helm upgrade completes, check whether schema migrations need to be run:
 
bash
 ```
# Check migration job status
kubectl get jobs -n celestacx | grep migration
``` 
If a migration job is present, wait for it to complete:
 
bash
 ```
kubectl wait job/cx-schema-migration \
  --for=condition=complete \
  --namespace celestacx \
  --timeout=10m
``` 
Check the migration logs if the job fails:
 
bash
 ```
kubectl logs -n celestacx \
  $(kubectl get pods -n celestacx -l job-name=cx-schema-migration \
  -o jsonpath='{.items[0].metadata.name}')
``` 
---
 
#### Step 5 — Verify the Upgrade
 
Run these checks after the upgrade completes:
 
**Check all pods are running:**
 
bash
 ```
kubectl get pods -n celestacx
``` 
All pods should show `Running` or `Completed` . No pods should be in `Error` or `CrashLoopBackOff` state.
 
**Verify the new version is running:**
 
bash
 ```
helm list -n celestacx
``` 
The `CHART` column should show the new version number.
 
**Functional verification:**
 
- Log in to Unified Admin — verify version number in the footer
- Log in to Agent Desk — verify it loads correctly
- Send a test message through at least one channel end-to-end
- Verify real-time dashboards are populating
- Verify historical reports are accessible
- Check IAM — verify user logins are working
 
---
 
#### Step 6 — Notify Users
 
Once verification is complete, notify your users that the maintenance window is over and CelestaCX is back online. Include a brief summary of what changed — link to the relevant section of [Release Notes & Version History →](#) .
 
---
 
### Rollback Process
 
If the upgrade fails verification or causes unexpected issues, roll back to the previous version using the Helm rollback command.
 
**View upgrade history:**
 
bash
 ```
helm history celestacx -n celestacx
``` 
**Roll back to the previous release:**
 
bash
 ```
helm rollback celestacx 0 -n celestacx --wait
``` 
The `0` refers to the previous revision. Use `helm history` to confirm the revision number if you need to roll back further.
 
**Verify the rollback:**
 
bash
 ```
helm list -n celestacx
kubectl get pods -n celestacx
``` *(Add a yellow Note Panel macro around the following)*
 
⚠️ **Database rollback warning** Helm rollback restores the application to its previous version but does **not** automatically reverse database schema migrations. If the upgrade included schema migrations, you may need to restore your database from the pre-upgrade backup to fully return to the previous state. This is why the pre-upgrade backup step is mandatory — never skip it. 
**Restore MongoDB from backup if needed:**
 
bash
 ```
# Copy backup back into the pod
kubectl cp ./mongodb-backup-<DATE> \
  celestacx/$(kubectl get pods -n celestacx \
  -l app=mongodb \
  -o jsonpath='{.items[0].metadata.name}'):/tmp/restore

# Run restore
kubectl exec -n celestacx \
  $(kubectl get pods -n celestacx -l app=mongodb \
  -o jsonpath='{.items[0].metadata.name}') \
  -- mongorestore \
  --username admin \
  --password <MONGODB_PASSWORD> \
  --drop /tmp/restore
``` 
---
 
### Version-Specific Upgrade Notes
 
#### Upgrading from 4.10.x to 5.0
 
**Estimated downtime:** 15–25 minutes
 
**Key changes requiring attention:**
 
1. **APISIX Reconfiguration** CelestaCX 5.0 introduces a new minified APISIX deployment mode. The existing APISIX installation must be removed and redeployed as part of the upgrade. This is handled automatically by the Helm upgrade but requires your current APISIX routes to be deleted first:
 
bash
 ```
# Delete all existing APISIX routes
kubectl -n celestacx delete ar \
  $(kubectl -n celestacx get ar \
  -o jsonpath='{.items[*]..metadata.name}')

# Uninstall existing APISIX
helm -n cx-external uninstall apisix
``` 
Then proceed with the standard Helm upgrade — the new APISIX configuration will be deployed automatically.
 
1. **Multi-Tenancy Initialization** If you are enabling multi-tenancy in 5.0 for the first time, run the tenant initialization job after the upgrade:
 
bash
 ```
kubectl create job cx-tenant-init \
  --from=cronjob/cx-tenant-setup \
  -n celestacx
``` 
1. **Values File Updates** The following new fields are required in `cx-values.yaml` for 5.0:
 
yaml
 ```
multitenancy:
  enabled: false    # Set to true only if enabling CCaaS multi-tenancy
  defaultTenant: default
``` 
1. **MongoDB Upgrade** CelestaCX 5.0 requires MongoDB 8.x. If you are running an older MongoDB version, upgrade it before running the Helm upgrade:
 
bash
 ```
# Check current MongoDB version
kubectl exec -n celestacx \
  $(kubectl get pods -n celestacx -l app=mongodb \
  -o jsonpath='{.items[0].metadata.name}') \
  -- mongod --version
``` 
Follow the [MongoDB upgrade guide →](#) in Platform Administration if your version is below 8.x.
 
---
 
#### Upgrading from 4.9.x to 4.10
 
**Estimated downtime:** 10–15 minutes
 
**Key changes requiring attention:**
 
1. **Schema Migration** CelestaCX 4.10 includes Alembic database schema migrations for the PostgreSQL reporting database. These run automatically as a Kubernetes job after the Helm upgrade — monitor the migration job as described in Step 4 of the standard process.
 
1. **New Values File Fields** The following new optional fields are available in 4.10:
 
yaml
 ```
formBuilder:
  revampEnabled: true    # Enables the redesigned Form Builder UI

linkedIn:
  enabled: false         # Set to true if deploying LinkedIn channel

qualityManagement:
  aiAssessments: false   # Set to true to enable AI-driven QM
``` 
1. **CX Voice** If you are running CX Voice components, voice has a separate upgrade process. After completing the main CelestaCX upgrade, follow the CX Voice upgrade steps in the [Voice & Video →](#) section of Channels & Integrations.
 
---
 
#### Upgrading Patch Versions (e.g., 4.10.0 → 4.10.1)
 
Patch upgrades are low-risk and follow the standard process with two simplifications:
 
- Schema migrations are rarely included in patch releases — check the release notes to confirm
- Values file changes are not required for patch releases unless explicitly noted
 
bash
 ```
# Update the repo
helm repo update

# Upgrade to the latest patch
helm upgrade celestacx celestacx/celestacx \
  --namespace celestacx \
  --values cx-values.yaml \
  --version <PATCH_VERSION> \
  --timeout 15m \
  --wait
```

---

## Upgrade Troubleshooting

| Symptom | Likely Cause | Resolution |
|---|---|---|
| Pods stuck in `Pending` after upgrade | New version requires more resources | Check `kubectl describe pod` for resource constraints — scale up nodes if needed |
| `ImagePullBackOff` after upgrade | Registry credentials expired | Recreate the `cx-registry` image pull secret with fresh credentials |
| Schema migration job fails | Database connectivity issue or insufficient permissions | Check migration job logs — verify database credentials are correct |
| Agent Desk shows blank screen after upgrade | Browser cache serving old JavaScript | Clear browser cache and hard reload (Ctrl+Shift+R) |
| IAM login fails after upgrade | Keycloak realm configuration mismatch | Check IAM pod logs — may require realm re-import for major version upgrades |
| Rollback leaves pods in bad state | Mixed version components | Run `helm rollback` again and then `kubectl rollout restart deployment -n celestacx` |

---

## Completing the Deployment & Infrastructure Section
```