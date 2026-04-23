#### Overview
 
This page covers how to access and interpret logs from a running CelestaCX deployment, how to monitor platform health across key infrastructure components, and how to set up alerting so problems are caught proactively rather than reactively.
 
CelestaCX runs on Kubernetes (RKE2) and all application services emit OTLP-formatted logs accessible via `kubectl` . For deployments with the observability stack enabled, logs are also aggregated in OpenSearch and visualisable through Grafana.
 
**Prerequisites for this page:**
 
- `kubectl` configured and authenticated against the target cluster
- Access to the `expertflow` namespace (or the namespace your deployment uses)
- For Grafana and Superset alerting: admin access to those interfaces
 
---
 
#### 1. Accessing Application Logs
 
#### Finding the Right Pod
 
Before viewing logs, identify which pod is responsible for the component you want to investigate.
 
bash
 ```
# List all running pods in the expertflow namespace
kubectl get pods -n expertflow

# List pods across all namespaces (useful for external components)
kubectl get pods -A

# Find pods that are not Running
kubectl get pods -A | grep -v Running
``` 
#### Common Pod Name Prefixes
 | Pod  Prefix | Component |
| --- | --- |
| ef-ccm-* | Core  Conversation  Manager  —  the  heart  of  routing  and  conversation  lifecycle |
| ef-agent-manager-* | Agent  state  management  and  session  handling |
| ef-unified-admin-* | Unified  Admin  interface  backend |
| ef-cx-router-* | Routing  engine  —  queue  and  agent  matching |
| ef-cx-otp-manager-* | 2FA  OTP  manager  service |
| *-connector-* | Individual  channel  connectors  (WhatsApp,  email,  Facebook,  etc.) |
| ef-media-server-* | Voice  media  server |
| ef-cx-analyser-* | ETL  pipeline  and  reporting  data  processor |
| keycloak-* | Identity  and  access  management |
| mongo-* | MongoDB  (if  deployed  in-cluster) |
| redis-* | Redis  (if  deployed  in-cluster) |
| superset-* | Apache  Superset  reporting  interface | 
#### Viewing Live Logs
 
Stream logs in real time for a specific pod:
 
bash
 ```
# Stream live logs
kubectl logs -f <pod-name> -n expertflow

# View the last 100 lines without streaming
kubectl logs <pod-name> -n expertflow --tail=100

# View logs from the previous (crashed) container instance
kubectl logs <pod-name> -n expertflow --previous

# View logs from the last hour
kubectl logs <pod-name> -n expertflow --since=1h
``` 
#### Filtering Logs
 
Use `grep` to isolate specific event types within a log stream:
 
bash
 ```
# Filter for audit log events
kubectl logs <pod-name> -n expertflow | grep "audit_logging"

# Filter for tracing events
kubectl logs <pod-name> -n expertflow | grep "tracing"

# Filter for ERROR level logs only
kubectl logs <pod-name> -n expertflow | grep "ERROR"

# Filter for a specific conversation or customer ID
kubectl logs <pod-name> -n expertflow | grep "<conversation-id>"

# Combine: stream live errors from the CCM
kubectl logs -f <ef-ccm-pod-name> -n expertflow | grep "ERROR"
``` 
#### Viewing Pod Events
 
When a pod is behaving unexpectedly — crashing, failing to start, or showing resource pressure — the pod events provide valuable context:
 
bash
 ```
# Describe a pod for events and resource information
kubectl describe pod <pod-name> -n expertflow

# View all recent events in the namespace, sorted by time
kubectl get events -n expertflow --sort-by='.lastTimestamp'
``` 
---
 
#### 2. Key Log Streams to Monitor
 
Not all logs are equally important. The following services produce the most operationally significant log output and should be checked first when investigating platform issues.
 
#### CX Router ( `ef-cx-router-*` )
 
The routing engine is the first place to check when conversations are not being routed correctly. Look for:
 
- `ERROR` entries indicating routing rule evaluation failures
- Queue assignment failures
- Agent state synchronisation errors
 
bash
 ```
kubectl logs -f <ef-cx-router-pod> -n expertflow | grep -E "ERROR|WARN|routing"
``` 
#### Core Conversation Manager ( `ef-ccm-*` )
 
The CCM manages the full conversation lifecycle. Its logs are the most comprehensive source for diagnosing conversation-level issues — failed channel handoffs, missing events, bot escalation failures.
 
bash
 ```
kubectl logs -f <ef-ccm-pod> -n expertflow | grep -E "ERROR|conversation|session"
``` 
#### Channel Connectors ( `*-connector-*` )
 
Each active channel has its own connector pod. When a channel stops receiving or sending messages, start here. Look for webhook failures, API authentication errors, and connection timeouts.
 
bash
 ```
# Find all connector pods
kubectl get pods -n expertflow | grep connector

# Check a specific connector
kubectl logs -f <whatsapp-connector-pod> -n expertflow | grep -E "ERROR|webhook|token"
``` 
#### Agent Manager ( `ef-agent-manager-*` )
 
Logs agent state transitions, session establishment, and login events. Check here when agents are experiencing state issues or cannot be seen on dashboards.
 
bash
 ```
kubectl logs -f <ef-agent-manager-pod> -n expertflow | grep -E "ERROR|state|login"
``` 
#### ETL / CX Analyser ( `ef-cx-analyser-*` )
 
The ETL pipeline that feeds historical reports. If reports are not updating, check this pod for pipeline failures or database connection errors.
 
bash
 ```
kubectl logs -f <ef-cx-analyser-pod> -n expertflow | grep -E "ERROR|pipeline|mongo"
``` 
---
 
#### 3. Cluster Health Monitoring
 
#### Node and Pod Status
 
Run these commands as a routine health check or at the start of an incident investigation:
 
bash
 ```
# Node status — all nodes should show Ready
kubectl get nodes

# Node resource utilisation
kubectl top nodes

# Pod resource utilisation (highest consumers first)
kubectl top pods -n expertflow --sort-by=memory

# All pods — quickly identify anything not in Running state
kubectl get pods -A | grep -v Running

# Count of pods by state (quick summary)
kubectl get pods -A | awk '{print $4}' | sort | uniq -c
``` 
#### Routine Health Check Checklist
 
Run through this checklist at the start of each operational day and after any maintenance or platform change:
 ```
☐ All pods in Running state (kubectl get pods -A | grep -v Running returns nothing)
☐ No pods with high restart counts (kubectl get pods -A | grep -v "0 " in RESTARTS column)
☐ Node CPU and memory within normal range (kubectl top nodes)
☐ MongoDB replica set healthy (see Section 4)
☐ Redis memory within limits (see Section 5)
☐ No recurring ERROR patterns in CCM or router logs
☐ SIP registrations stable (if voice is enabled)
☐ Disk usage below 75% on all nodes
☐ SSL certificates not approaching expiry
``` 
---
 
#### 4. MongoDB Monitoring
 
MongoDB is the primary data store for CelestaCX. Degraded MongoDB performance directly impacts conversation processing, routing, and reporting.
 
#### Checking Replica Set Health
 
bash
 ```
# Access the MongoDB primary pod
kubectl exec -it <mongo-pod> -n <mongo-namespace> -- mongosh

# Check replica set status — all members should show as PRIMARY or SECONDARY
rs.status()

# Check replication lag
rs.printReplicationInfo()
rs.printSecondaryReplicationInfo()
``` 
A healthy replica set shows:
 
- One PRIMARY member
- All SECONDARY members with low replication lag (under 5 seconds)
- No members in RECOVERING or UNKNOWN state
 
#### Enabling Slow Query Logging
 
When MongoDB performance is degraded — slow report generation, delayed conversation updates — enable the profiler to identify slow queries.
 **Warning:** Profiling adds performance overhead and disk usage. Use it for targeted investigation, not as a permanent setting. Always disable it when the investigation is complete. 
bash
 ```
# Connect to MongoDB primary
kubectl exec -it <mongo-pod> -n <mongo-namespace> -- mongosh

# Switch to the CelestaCX database
use ccm_db

# Enable profiling — log queries slower than 200ms
db.setProfilingLevel(1, { slowms: 200 })

# Enable with sample rate to reduce overhead (logs 50% of slow queries)
db.setProfilingLevel(1, { slowms: 200, sampleRate: 0.50 })
``` 
**Querying the slow query log:**
 
bash
 ```
# View 10 most recent slow query entries
db.system.profile.find().limit(10).sort({ ts: -1 }).pretty()

# Filter by execution time — queries over 5ms
db.system.profile.find({ millis: { $gt: 5 } }).pretty()

# Filter by collection
db.system.profile.find({ ns: 'ccm_db.ChannelConnector' }).pretty()

# Filter by time range
db.system.profile.find({
  ts: {
    $gt: new ISODate("2025-01-10T03:00:00Z"),
    $lt: new ISODate("2025-01-10T04:00:00Z")
  }
}).sort({ millis: -1 }).pretty()

# Quick view of 5 most recent slow operations
show profile
``` 
**Disable profiling when done:**
 
bash
 ```
db.setProfilingLevel(0)
``` 
---
 
#### 5. Redis Monitoring
 
Redis holds real-time session state and queue data. Redis health directly affects agent state visibility, conversation routing, and dashboard data.
 
#### Checking Redis Health
 
bash
 ```
# Access the Redis pod
kubectl exec -it <redis-pod> -n <redis-namespace> -- redis-cli

# Check memory usage
INFO memory

# Check connected clients
INFO clients

# Check keyspace statistics
INFO keyspace

# Check replication status (if Redis Sentinel or Cluster is deployed)
INFO replication
``` 
Key metrics to watch:
 
- `used_memory_human` — current memory in use
- `maxmemory_human` — the memory limit
- `evicted_keys` — any non-zero value indicates Redis is evicting data under memory pressure, which can cause session loss
- `connected_clients` — high connection counts may indicate a connection leak
 
#### Redis Slow Log
 
The Redis SLOWLOG records commands that exceeded the configured execution time threshold.
 
bash
 ```
# View the last 10 slow commands
SLOWLOG GET 10

# Check how many slow log entries exist
SLOWLOG LEN

# Reset the slow log
SLOWLOG RESET

# View and set the slow log threshold (in microseconds — default 10000 = 10ms)
CONFIG GET slowlog-log-slower-than
CONFIG SET slowlog-log-slower-than 5000
``` 
Each SLOWLOG entry shows the command, execution time in microseconds, and timestamp — allowing you to identify which operations are causing Redis latency.
 
---
 
#### 6. Superset Alerting
 
CelestaCX uses Apache Superset for historical reporting. Superset's built-in alerts and scheduled reports feature can notify your team when KPI thresholds are breached in historical data.
 
#### Enabling Superset Alerts
 
Superset alerts must be enabled by an administrator before they can be configured. Navigate to **Superset → Settings → Feature Flags** and confirm the alerts and reports feature is enabled. If it is not, your administrator needs to enable it via the Superset deployment configuration.
 
#### Configuring an Alert
 
1. In Superset, navigate to **Alerts & Reports → Alerts → + Alert** .
2. Give the alert a name and set the **Owner** .
3. Select the **Alert Condition** — choose a chart or dataset to monitor.
4. Set the **Condition Type** (e.g. value crosses a threshold).
5. Set the **Threshold** value that triggers the alert.
6. Configure the **Schedule** — how frequently Superset checks the condition (e.g. every 15 minutes).
7. Set the **Notification Method** — email recipients or Slack webhook.
8. Save and enable the alert.
 
#### Recommended Alerts for Operations Teams
 | Alert | Condition | Suggested  Threshold |
| --- | --- | --- |
| High  abandonment  rate | Queue  abandonment  % | Greater  than  10% |
| SLA  breach | Service  Level  % | Below  80% |
| Low  agent  availability | Ready  agents  count | Below  2  per  queue |
| ETL  pipeline  stale | Last  report  update  time | Greater  than  30  minutes  ago |
| High  RONA  rate | RONA  count  per  hour | Greater  than  5 | 
---
 
#### 7. Grafana Dashboards (Voice Monitoring)
 
For deployments with CX Voice enabled, Grafana provides real-time infrastructure dashboards for the EFSwitch voice server — SIP registration counts, active call counts, call success rates, and media server resource utilisation.
 
Access Grafana at `https://<your-domain>/grafana` using your platform operator credentials.
 
Key dashboards to monitor for voice deployments:
 
- **EFSwitch Overview** — SIP registrations, active calls, call rates
- **Media Server** — CPU, memory, active sessions, recording storage
- **Kubernetes Node Overview** — cluster resource consumption across all nodes
 
The Supervisor interface also embeds Grafana dashboards directly — the Summary Dashboard and Team Stats Dashboard are both Grafana-powered and refresh automatically.
 
---
 
#### 8. Log Retention and Archival
 
By default, Kubernetes pod logs are stored on the node where the pod runs and are subject to the node's log rotation settings. Logs are not persisted beyond pod restarts without a centralised log aggregation stack.
 
**For production deployments, configure log aggregation:**
 
CelestaCX supports integration with OpenSearch (formerly Elasticsearch) for centralised log collection, indexing, and retention. When OpenSearch is deployed as part of the observability stack, logs from all pods are shipped automatically via a log collector (Fluentd or Fluent Bit) and are searchable through the OpenSearch Dashboards interface.
 
Contact OctaveBytes if you need assistance setting up centralised log aggregation for your deployment.
 
---
 
#### Quick Reference: Log Commands
 
bash
 ```
# Health check — pods not in Running state
kubectl get pods -A | grep -v Running

# Health check — node resources
kubectl top nodes

# Stream live logs from a pod
kubectl logs -f <pod-name> -n expertflow

# Stream live errors only
kubectl logs -f <pod-name> -n expertflow | grep ERROR

# Pod events and resource info
kubectl describe pod <pod-name> -n expertflow

# Last hour of logs
kubectl logs <pod-name> -n expertflow --since=1h

# Previous container logs (after crash)
kubectl logs <pod-name> -n expertflow --previous

# All namespace events sorted by time
kubectl get events -n expertflow --sort-by='.lastTimestamp'
``` 
---
 
#### What's Next
 
- [**Platform Administration**](#) — backup and restore, upgrades, certificate renewal, and credential rotation
- [**Troubleshooting Guide**](#) — symptom-based diagnosis for common platform issues
- [**FAQs**](#) — quick answers to common operational questions