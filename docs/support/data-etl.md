# Data Platform & ETL

#### Overview

This page explains how interaction data flows through the CelestaCX reporting pipeline — from raw conversation events in MongoDB to the structured SQL tables that power Superset dashboards and historical reports. It covers the ETL architecture, how to set up and verify the pipeline, how to manage data freshness, and how to access the underlying data for custom reporting or external BI integrations.

Understanding this pipeline is essential for platform operators responsible for keeping historical reports current, and for IT teams or developers who need to build custom reports or integrate CelestaCX data with external analytics tools.

---

#### 1. How the Data Pipeline Works

CelestaCX uses a two-layer data architecture:

**Layer 1 — Operational data (MongoDB):** All live interaction data is written to MongoDB in real time as conversations occur. This is the primary operational database — it powers the Agent Desk, routing engine, and real-time dashboards. It is optimised for high-frequency writes and real-time reads, not for the complex analytical queries that reporting requires.

**Layer 2 — Reporting data (relational SQL):** A separate reporting database — MySQL or Microsoft SQL Server — holds structured, query-optimised data derived from MongoDB. This is what Superset reads to generate historical reports.

**The bridge between the two is the CX Analyser ETL pipeline:**

```
MongoDB (Operational)
  └── Interaction events: conversations, tasks,
      agent states, wrap-up, queue activity
        │
        ▼
  CX Analyser (ETL Engine)
  ├── Reporting Connector: reads from MongoDB
  ├── Reporting Scheduler: runs on a configurable cron schedule
  │   (default: every 5 minutes)
  └── Transforms and writes to relational SQL tables
        │
        ▼
  Relational SQL Database (MySQL / MSSQL)
  └── Structured tables: ConversationActivities,
      AgentStates, QueueStats, WrapupCodes, etc.
        │
        ▼
  Apache Superset
  └── Reads SQL tables → renders historical reports and dashboards
```

**Key consequence:** Historical reports only include interactions that have been fully closed **and** processed by the ETL pipeline. The pipeline runs every 5 minutes by default, so there is always a short lag between a conversation closing and it appearing in reports. Very recent data will not be visible until the next ETL cycle completes.

---

#### 2. Data Platform Components

| Component | Role | Technology |
| --- | --- | --- |
| MongoDB | Primary operational data store — all live interaction data | MongoDB with Replica Set (recommended) |
| Reporting Connector | Reads closed interaction data from MongoDB and passes it to the scheduler | Java-based service, deployed as a Kubernetes pod |
| Reporting Scheduler | Orchestrates ETL jobs on a cron schedule; transforms and writes data to SQL | Java-based service, deployed as a Kubernetes pod |
| Relational SQL Database | Stores structured reporting data in query-optimised tables | MySQL or Microsoft SQL Server |
| Apache Superset | Reads from the SQL database; renders dashboards, charts, and reports for users | Python-based, deployed as a Kubernetes pod |

---

#### 3. Setting Up the Data Pipeline

If the ETL pipeline is not yet running — for example on a new deployment or after adding a new tenant — follow this sequence.

#### Step 1: Infrastructure Prerequisites

Before deploying the ETL components, the following must be in place:

- **TLS certificates** for secure communication between the Reporting Connector and the SQL database. Obtain the MySQL `.jks` and `.cert` files and create a Kubernetes ConfigMap:

bash

```
kubectl create configmap celesta-reporting-connector-keystore-cm \
  --from-file=mykeystore.jks \
  -n <tenant-namespace>
```

- **SQL database schema initialised** for each tenant. Navigate to the `pre-deployment/reportingConnector/dbScripts/` directory in the release package and run the creation script for your target DBMS (MySQL or MSSQL). The database user must have permissions to create tables and execute stored procedures.

#### Step 2: Configure the Reporting Connector

Create the `reporting-connector.conf` configuration file for each tenant. Key parameters:

| Parameter | Description |
| --- | --- |
| tenant_id | Must match the tenant's logical ID exactly |
| db_host | IP address or hostname of the SQL database |
| db_port | SQL database port (default: 3306 for MySQL, 1433 for MSSQL) |
| db_username | Database user with read/write access to the reporting schema |
| db_password | Database password |
| service_url | URL of the historical reports service (e.g. http://celesta-cx-historical-reports-svc:8081 ) |

#### Step 3: Deploy the Reporting Scheduler

Once ConfigMaps and databases are ready, deploy via Helm:

bash

```
helm upgrade --install \
  --namespace <tenant-namespace> \
  cx-reporting \
  helm/Reporting \
  --values=values.yaml
```

#### Step 4: Configure ETL Schedule

In `values.yaml`, configure the ETL cron intervals. The default — and recommended minimum — is every 5 minutes:

yaml

```
reportingScheduler:
  cronExpression: "0 */5 * * * *"   # Every 5 minutes
  batchSize: 1000                    # Records per ETL batch
```

For high-volume deployments processing many thousands of interactions per hour, consider tuning the batch size to prevent individual ETL runs from taking longer than the cron interval.

#### Step 5: Verify the Pipeline

Confirm the ETL pipeline is working correctly:

bash

```
# Check the reporting scheduler pod is running
kubectl get pods -n <tenant-namespace> | grep reporting-scheduler

# Stream logs and confirm ETL jobs are completing without errors
kubectl logs -f <reporting-scheduler-pod> -n <tenant-namespace>

# Look for successful job completion messages
kubectl logs <reporting-scheduler-pod> -n <tenant-namespace> | grep -E "completed|ERROR|connection"
```

**Successful pipeline indicators:**

- The SQL table `ConversationActivities` is populating with new interaction records
- Superset dashboards show data for the current day
- No `connection_refused` errors appear in the Reporting Connector logs

---

#### 4. Setting Up Superset Reports

Once data is flowing into the SQL database, import the CelestaCX report definitions into Superset.

#### Importing Report Templates

1. Obtain the report definition files (`.zip` or `.yaml`) from the CelestaCX release package provided by OctaveBytes
2. Log in to the Superset UI at `https://<your-domain>/superset`
3. Navigate to **Dashboards**
4. Click the **Import Dashboard** icon (up arrow)
5. Upload the template file
6. Select the correct SQL database as the data source when prompted
7. Confirm the import

#### Configuring the Database Connection in Superset

If Superset is not yet connected to the reporting SQL database:

1. Navigate to **Data → Databases → + Database**
2. Select your database type (MySQL or MSSQL)
3. Enter the connection string:

```
mysql://<user>:<password>@<host>:<port>/<database>
```

1. Test the connection and save

#### Configuring UTC Offset

Reports default to the server's timezone. If your operation is in a different timezone, configure the UTC offset so report timestamps match your local time:

1. In Superset, open the relevant dashboard or report
2. Navigate to the edit view for the affected chart
3. Apply the UTC offset in the time filter configuration (e.g. `+5:00` for UTC+5)

This setting must be applied per report — it is not a global setting. For multi-timezone deployments, create separate report views per timezone.

#### Assigning Report Permissions

After importing reports, ensure the correct user groups can access them:

1. In Superset, navigate to **Security → List Roles**
2. Assign the relevant dashboards and datasets to the appropriate role (e.g. Supervisor, Quality Manager)
3. Confirm that users in those roles can see the dashboards when they log in

---

#### 5. Managing Data Freshness

#### Default ETL Cadence

| Data Type | Default Refresh | Notes |
| --- | --- | --- |
| Conversation records | Every 5 minutes | After conversation closes and ETL runs |
| Agent state history | Every 5 minutes | Aggregated from state change events |
| Queue statistics | Every 5 minutes | Derived from task and queue event data |
| Quality evaluation scores | On submission | Written directly when evaluator submits |
| Campaign results | On call completion + ETL | Combined with per-call outcome data |

#### Adjusting ETL Frequency

For operations that need more frequent report updates, reduce the cron interval:

yaml

```
# Every 2 minutes — higher CPU and I/O impact
cronExpression: "0 */2 * * * *"

# Every minute — use only if absolutely necessary and system has capacity
cronExpression: "0 * * * * *"
```

Be cautious when reducing the interval on high-volume deployments. Each ETL run queries MongoDB and writes to the SQL database — too-frequent runs can cause resource contention if individual runs take longer than the interval.

#### Diagnosing Stale Reports

If reports are not updating as expected:

bash

```
# Step 1: Confirm the reporting scheduler pod is running
kubectl get pods -n <tenant-namespace> | grep reporting

# Step 2: Check when the last ETL job ran
kubectl logs <reporting-scheduler-pod> -n <tenant-namespace> | grep "completed"

# Step 3: Check for connection errors
kubectl logs <reporting-scheduler-pod> -n <tenant-namespace> | grep "ERROR"

# Step 4: Verify the SQL database is reachable from the pod
kubectl exec -it <reporting-scheduler-pod> -n <tenant-namespace> -- \
  curl -v telnet://<db-host>:<db-port>

# Step 5: Restart the scheduler if no errors are visible but jobs have stopped
kubectl rollout restart deployment/reporting-scheduler -n <tenant-namespace>
```

---

#### 6. Accessing Raw Data for Custom Reporting

For teams building custom reports in Superset or integrating CelestaCX data with an external BI tool (Power BI, Tableau, Looker, etc.), the underlying SQL tables are directly accessible.

#### Primary Reporting Tables

| Table | What It Contains |
| --- | --- |
| ConversationActivities | One record per conversation — direction, queue, channel, agent, duration, disposition, timestamps |
| ChannelSessions | One record per channel session within a conversation |
| AgentStateLogs | Time-series record of agent state transitions with timestamps and durations |
| TaskActivities | Individual task records — routing, assignment, acceptance, closure |
| QueueStats | Aggregated queue metrics per interval — offered, answered, abandoned, SLA |
| WrapupCodes | Wrap-up category and reason code selections per conversation |
| EvaluationResults | QM evaluation scores by agent, form, section, and question |
| OutboundCampaignCalls | Per-call records for outbound campaign activity |

> For the complete table schema with column definitions and data types, see the **Reporting Database Schema** page.

#### Connecting an External BI Tool

Any BI tool that supports MySQL or MSSQL connections can read directly from the CelestaCX reporting database. Provide your BI team with:

- Database host, port, and database name
- A read-only database user — create one specifically for BI access rather than using the ETL user
- The table schema (see Reporting Database Schema)

For Power BI, use the MySQL or SQL Server native connector. For Tableau, use the corresponding database driver. Configure the connection to use SSL if the database is accessible over a network.

---

#### 7. Data Archival and Retention

As the reporting database grows over time, older data can be archived to maintain query performance. CelestaCX supports configurable archival of interaction records older than a defined retention period.

See [Platform Administration → Data Archival](#) for archival configuration steps, retention policy recommendations, and storage target options.

**General guidance:**

- Keep at least 90 days of data in the primary reporting database for operational reporting
- Archive data older than 90 days to secondary storage if database size is becoming a concern
- Never delete archived data without confirming your regulatory and contractual retention obligations

---

#### Quick Reference: ETL Troubleshooting

| Symptom | First Check | Resolution |
| --- | --- | --- |
| Reports show no data for today | Is the scheduler pod running? | `kubectl get pods |
| Reports stopped updating | Last successful ETL job time | Check logs for completed then ERROR entries |
| SQL connection errors | Database reachability | Test connectivity from pod; verify credentials in ConfigMap |
| ETL runs but data missing | MongoDB connectivity | Check Reporting Connector logs for MongoDB connection errors |
| Superset shows wrong timezone | UTC offset setting | Configure UTC offset per report in Superset edit view |
| High database load | ETL frequency too high | Increase cron interval; check batch size |

---

#### What's Next

- [**FAQs**](#) — quick answers to common questions from administrators and IT teams
- [**Platform Administration**](#) — backup, upgrade, and credential rotation procedures
- [**Historical Reports**](#) — the full library of reports powered by this data pipeline
