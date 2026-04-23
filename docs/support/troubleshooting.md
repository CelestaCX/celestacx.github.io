#### Overview
 
This guide covers the most common issues encountered in live CelestaCX deployments — organised by the area of the platform where the problem appears. For each issue you will find the typical symptoms, likely causes, and resolution steps.
 
This guide is intended for platform operators, IT administrators, and reseller support staff. It assumes you have access to the Kubernetes cluster, Unified Admin, and the Keycloak administration console.
 
If you cannot resolve an issue using this guide, contact OctaveBytes support with the diagnostic information described at the end of this page.
 
---
 
#### How to Use This Guide
 
Find the category that matches the area where the issue is occurring, then locate the symptom that best describes what you are seeing. Work through the resolution steps in order — they are sequenced from the most common and simplest fix to the less common and more complex.
 
**Category index:**
 
1. [Authentication & Login](https://octavebytes-docs.atlassian.net/wiki/spaces/CelestaCX/pages/135462948/Troubleshooting+Guide#1.-Authentication-%26-Login)
2. [Agent Desk & Agent State](https://octavebytes-docs.atlassian.net/wiki/spaces/CelestaCX/pages/135462948/Troubleshooting+Guide#2.-Agent-Desk-%26-Agent-State)
3. [Routing & Queues](https://octavebytes-docs.atlassian.net/wiki/spaces/CelestaCX/pages/135462948/Troubleshooting+Guide#3.-Routing-%26-Queues)
4. [Digital Channels & Connectors](https://octavebytes-docs.atlassian.net/wiki/spaces/CelestaCX/pages/135462948/Troubleshooting+Guide#4.-Digital-Channels-%26-Connectors)
5. [Voice & WebRTC](https://octavebytes-docs.atlassian.net/wiki/spaces/CelestaCX/pages/135462948/Troubleshooting+Guide#5.-Voice-%26-WebRTC)
6. [Real-Time Dashboards](https://octavebytes-docs.atlassian.net/wiki/spaces/CelestaCX/pages/135462948/Troubleshooting+Guide#6.-Real-Time-Dashboards)
7. [Historical Reports & Superset](https://octavebytes-docs.atlassian.net/wiki/spaces/CelestaCX/pages/135462948/Troubleshooting+Guide#7.-Historical-Reports-%26-Superset)
8. [Platform & Infrastructure](https://octavebytes-docs.atlassian.net/wiki/spaces/CelestaCX/pages/135462948/Troubleshooting+Guide#8.-Platform-%26-Infrastructure)
9. [Failover & Recovery](https://octavebytes-docs.atlassian.net/wiki/spaces/CelestaCX/pages/135462948/Troubleshooting+Guide#9.-Failover-%26-Recovery)
 
---
 
#### 1. Authentication & Login
 
#### Agents cannot log in
 
**Symptoms:** Login page loads but credentials are rejected. Agent sees an error or is redirected back to the login page.
 | Likely  Cause | Resolution |
| --- | --- |
| Incorrect  password | Reset  the  agent's  password  in  Keycloak:  Users  →  [User]  →  Credentials  →  Set  Password.  Ensure  Temporary  is  set  to  Off. |
| User  account  disabled  or  not  created | Verify  the  user  exists  in  Keycloak  and  is  enabled.  Check  the  Enabled  toggle  on  the  user's  account  page. |
| Wrong  realm | Confirm  the  agent  is  logging  in  at  the  correct  URL  for  their  tenant.  Each  tenant  uses  a  separate  Keycloak  realm. |
| Role  not  assigned | Check  that  the  agent  has  the  agent  role  assigned  in  Keycloak,  along  with  offline_access  and  uma_authorization  . |
| Agent  not  assigned  to  a  team | Agents  must  be  assigned  to  a  team  in  Unified  Admin  in  addition  to  having  their  Keycloak  role.  Navigate  to  Teams  and  confirm  membership. |
| SSO  misconfiguration | If  SSO  is  enabled,  verify  the  identity  provider  settings  in  Keycloak  →  Identity  Providers.  Test  with  a  local  account  to  isolate  the  issue. | 
---
 
#### 2FA not working
 
**Symptoms:** Agent enters their password successfully but the OTP step fails or the OTP is never received.
 | Likely  Cause | Resolution |
| --- | --- |
| 2FA  not  properly  enabled  in  tenant  settings | Verify  the  twoFAConfigs  object  is  correctly  set  in  the  tenant  settings  via  the  Update  Tenant  API.  Confirm  is2FAEnabled  is  true  . |
| Email  channel  not  configured | 2FA  via  email  requires  a  working  IMAP/SMTP  email  channel.  Confirm  the  channel  is  active  in  Unified  Admin  and  the  channelServiceIdentifier  matches  the  sender  address. |
| OTP  expired | The  default  OTP  lifetime  is  60  seconds.  If  users  are  slow  to  enter  it,  increase  otpExpiry  in  tenant  settings  (recommended  maximum:  120  seconds). |
| OTP  max  attempts  exceeded | The  user  account  may  be  in  cooldown  after  too  many  failed  attempts.  Check  otpMaxAttempts  setting  and  reset  the  user's  2FA  state  in  Keycloak  if  needed. |
| OTP  manager  service  not  running | Check  the  ef-cx-otp-manager-svc  pod  status:  `kubectl  get  pods  -A | 
---
 
#### User role changes not taking effect
 
**Symptoms:** A user's permissions do not change after their role is updated in Keycloak.
 
**Resolution:** Role changes in Keycloak require the affected user to log out and log back in. Token refresh alone is not sufficient — the user must complete a full new login session.
 
---
 
#### 2. Agent Desk & Agent State
 
#### Agent is stuck in Not Ready and cannot switch to Ready
 
**Symptoms:** Agent clicks Ready but the state does not change, or reverts immediately back to Not Ready.
 | Likely  Cause | Resolution |
| --- | --- |
| Active  task  preventing  state  change | Check  if  the  agent  has  an  active  or  pending-wrap-up  conversation.  Agent  state  cannot  change  while  tasks  are  active  on  some  MRDs. |
| Browser  session  issue | Ask  the  agent  to  hard-refresh  the  browser  (Ctrl+Shift+R  /  Cmd+Shift+R)  or  clear  the  browser  cache  and  log  in  again. |
| WebSocket  connection  lost | The  agent  desk  uses  WebSocket  for  real-time  state  updates.  Ask  the  agent  to  check  their  network  connection,  then  log  out  and  back  in. |
| Supervisor  force-state  conflict | Verify  no  supervisor  has  recently  changed  the  agent's  state  remotely.  Check  the  Available  Agents  Detail  dashboard  for  the  agent's  current  state. | 
---
 
#### Agent shows as Available on dashboard but receives no conversations
 
**Symptoms:** Agent is in Ready state and channel toggles are green, but no conversations are being routed to them.
 | Likely  Cause | Resolution |
| --- | --- |
| Agent  not  mapped  to  any  queue | Verify  the  agent's  team  is  linked  to  at  least  one  queue  in  Unified  Admin  →  Routing  →  Agent-Queue  Mapping. |
| No  conversations  in  queue | Check  the  Queued  Conversations  Detail  dashboard  to  confirm  conversations  are  actually  waiting. |
| Channel  toggle  is  off  for  the  relevant  MRD | Verify  the  agent's  individual  MRD  channel  toggles  are  green  —  it  is  possible  for  global  state  to  be  Ready  while  specific  channel  toggles  are  off. |
| Routing  rule  not  matching | Check  that  a  routing  rule  exists  directing  interactions  from  the  relevant  channel  to  a  queue  the  agent  is  mapped  to. |
| Pull  mode  active | If  the  queue  uses  Pull  mode,  conversations  are  not  pushed  to  agents  —  agents  must  actively  select  them  from  the  subscribed  list. | 
---
 
#### Wrap-up timer expired before agent completed wrap-up
 
**Symptoms:** Conversation was closed automatically and marked as "Closed without Wrap-up" in reports.
 
**Resolution:** This is expected behaviour when the wrap-up timer is configured. If it is happening too frequently, increase the wrap-up timer duration in Unified Admin → Queues → [Queue] → Wrap-up Settings. Coach agents to complete wrap-up promptly. Note that the timer cannot be extended mid-wrap-up once it has started counting.
 
---
 
#### Agent desk shows blank screen or fails to load
 
**Symptoms:** Agent logs in but the Agent Desk interface does not render, shows a white screen, or shows a loading spinner indefinitely.
 | Likely  Cause | Resolution |
| --- | --- |
| Browser  compatibility | Confirm  the  agent  is  using  a  supported  modern  browser  (Chrome  or  Edge  recommended).  Check  for  browser  console  errors  (F12  →  Console). |
| Keycloak  token  issue | Log  out,  clear  browser  cookies  for  the  domain,  and  log  back  in. |
| CX  Router  service  down | Check  the  routing  service  pods:  `kubectl  get  pods  -A |
| Network/firewall  blocking  WebSocket | Confirm  the  agent's  network  allows  WebSocket  connections  to  the  CelestaCX  domain  on  port  443. | 
---
 
#### 3. Routing & Queues
 
#### Conversations are not being routed to agents
 
**Symptoms:** Customers make contact but conversations accumulate in the queue and are not assigned to any available agent.
 | Likely  Cause | Resolution |
| --- | --- |
| No  agents  in  Ready  state  on  the  relevant  MRD | Check  the  Summary  Dashboard  —  confirm  agents  are  Ready  and  channel  toggles  are  active  for  the  correct  MRD. |
| No  routing  rule  matches  the  incoming  interaction | Review  routing  rules  in  Unified  Admin  →  Routing  →  Routing  Rules.  Confirm  a  rule  exists  for  the  channel  and  interaction  type,  and  that  a  default  queue  is  set. |
| Queue  has  no  agents  mapped  to  it | Confirm  agent  teams  are  linked  to  the  queue  in  Unified  Admin  →  Routing  →  Agent-Queue  Mapping. |
| Business  calendar  blocking  routing | If  the  queue  is  linked  to  a  business  calendar  and  the  current  time  is  outside  operating  hours,  interactions  will  be  held  or  overflow.  Check  the  calendar  configuration. |
| CX  Router  pod  not  running | Check:  `kubectl  get  pods  -A | 
---
 
#### High RONA rate — agents assigned conversations but not accepting
 
**Symptoms:** Conversations are being offered to agents (ringing) but agents are not accepting, causing them to be re-routed.
 | Likely  Cause | Resolution |
| --- | --- |
| Agents  are  in  Ready  state  but  away  from  their  desk | Check  if  RONA  events  correlate  with  specific  agents  or  times  of  day.  Coach  agents  on  staying  present  when  in  Ready  state. |
| Browser  notifications  disabled | Agents  may  not  be  hearing  the  incoming  conversation  alert.  Confirm  browser  notifications  are  enabled  for  the  CelestaCX  domain. |
| Agent  Disconnect  timer  too  short | If  the  RONA  timeout  is  very  short,  agents  may  not  have  enough  time  to  accept.  Review  the  alert  timeout  setting  in  Unified  Admin  →  Agent  Desk  Settings. | 
---
 
#### Conversations not routing out-of-hours as expected
 
**Symptoms:** Outside configured working hours, conversations are being accepted and routed to agents rather than following the out-of-hours overflow rule.
 | Likely  Cause | Resolution |
| --- | --- |
| Business  calendar  not  linked  to  queue | Confirm  the  queue  has  a  business  calendar  assigned  in  Unified  Admin  →  Queues  →  [Queue]  →  Business  Calendar. |
| Calendar  timezone  incorrect | Verify  the  calendar's  timezone  matches  the  operation's  local  timezone.  A  misconfigured  timezone  is  the  most  common  cause  of  this  issue. |
| Public  holidays  not  entered | Check  that  current  public  holidays  are  listed  in  the  calendar.  They  are  not  added  automatically. | 
---
 
#### 4. Digital Channels & Connectors
 
#### WhatsApp messages not arriving in Agent Desk
 
**Symptoms:** Customers send WhatsApp messages but they do not appear as conversations in CelestaCX.
 | Likely  Cause | Resolution |
| --- | --- |
| Connector  pod  not  running | Check:  `kubectl  get  pods  -A |
| Webhook  not  registered  or  expired | Meta  webhooks  require  periodic  revalidation.  Verify  the  webhook  URL  and  verification  token  in  the  Meta  Business  Manager  match  the  connector  configuration. |
| Phone  number  not  approved | Confirm  the  WhatsApp  Business  phone  number  is  approved  and  active  in  Meta  Business  Manager. |
| Channel  not  linked  to  a  routing  rule | Verify  a  routing  rule  exists  in  Unified  Admin  that  directs  WhatsApp  interactions  to  a  queue. |
| API  token  expired | Meta  Cloud  API  tokens  can  expire.  Regenerate  the  token  in  Meta  Business  Manager  and  update  it  in  the  connector  configuration. | 
---
 
#### Email channel not fetching new messages
 
**Symptoms:** Emails sent to the configured address are not appearing as conversations in CelestaCX.
 | Likely  Cause | Resolution |
| --- | --- |
| IMAP/SMTP  credentials  changed  or  expired | Verify  the  email  account  credentials  are  still  valid.  Some  providers  require  app-specific  passwords  —  confirm  the  correct  password  is  configured. |
| IMAP  access  disabled  on  the  mailbox | Check  that  IMAP  is  enabled  for  the  email  account  in  the  provider  settings  (Gmail,  Microsoft  365,  etc.). |
| Email  connector  pod  not  running | Check:  `kubectl  get  pods  -A |
| Firewall  blocking  IMAP  port | Confirm  the  platform  can  reach  the  mail  server  on  port  993  (IMAP  over  SSL). |
| Fetch  timestamp  issue  after  downtime | After  a  system  recovery,  emails  received  during  downtime  may  not  be  fetched  automatically.  Manually  trigger  a  re-fetch  or  restart  the  email  connector  pod. | 
---
 
#### Channel connector shows as disconnected in Unified Admin
 
**Symptoms:** The channel status indicator in Unified Admin shows red or disconnected.
 | Resolution  Steps |
| --- |
| Check  the  connector  pod  is  running:  `kubectl  get  pods  -A |
| Review  pod  logs  for  error  messages:  kubectl  logs  <pod-name>  -n  <namespace> |
| Verify  the  channel  credentials  and  webhook  URLs  are  still  valid  with  the  upstream  provider |
| Restart  the  connector  pod:  kubectl  rollout  restart  deployment/<connector-name>  -n  <namespace> |
| If  the  issue  persists,  check  for  upstream  provider  API  outages  or  rate  limiting | 
---
 
#### 5. Voice & WebRTC
 
#### Inbound calls not reaching agents
 
**Symptoms:** Customers call the contact centre number but the call is not presented to any agent.
 | Likely  Cause | Resolution |
| --- | --- |
| SIP  registration  failure | Check  EFSwitch  SIP  registrations.  Verify  the  SIP  trunk  credentials  and  that  the  SIP  proxy  is  reachable  from  the  voice  gateway. |
| Voice  connector  pod  not  running | Check:  `kubectl  get  pods  -A |
| No  routing  rule  for  voice  MRD | Confirm  a  routing  rule  exists  for  the  voice  channel  directing  calls  to  a  queue. |
| No  agents  in  Ready  state  for  the  voice  MRD | Verify  at  least  one  agent  has  their  Voice  MRD  channel  toggle  enabled  and  is  in  Ready  state. |
| Media  server  not  running | Check  the  media  server  pod  status.  Without  a  running  media  server,  calls  cannot  be  established. | 
---
 
#### Poor call quality or one-way audio
 
**Symptoms:** Calls connect but audio is choppy, delayed, or heard in only one direction.
 | Likely  Cause | Resolution |
| --- | --- |
| NAT  traversal  issue | Confirm  STUN/TURN  server  settings  in  the  WebRTC  configuration.  One-way  audio  is  typically  caused  by  NAT  not  being  correctly  traversed. |
| Firewall  blocking  UDP  media  ports | Voice  media  uses  UDP  on  a  range  of  ports  (typically  10000–20000).  Confirm  these  are  open  between  the  client  and  the  media  server. |
| Codec  mismatch | Check  that  the  SIP  trunk  and  EFSwitch  are  configured  to  use  a  common  codec  (G.711  is  the  most  compatible). |
| Network  bandwidth  insufficient | Minimum  recommended  bandwidth  per  voice  call  is  100  Kbps  upload  and  download.  Test  the  agent's  connection  quality. | 
---
 
#### 6. Real-Time Dashboards
 
#### Dashboard shows no data or all zeros
 
**Symptoms:** The Summary Dashboard or detailed dashboards load but all counters show zero or "No data".
 | Likely  Cause | Resolution |
| --- | --- |
| Team  or  queue  not  selected | The  Summary  Dashboard  requires  a  Team  to  be  selected  before  queue  data  appears.  Confirm  the  supervisor  has  selected  their  team  from  the  filter. |
| No  agents  have  logged  in  yet | Team  and  queue  data  only  appears  once  at  least  one  agent  in  that  team  has  logged  in. |
| Grafana  data  source  disconnected | Check  the  Grafana  data  source  connection  in  the  Grafana  admin  panel.  Reconnect  if  the  source  is  showing  as  disconnected. |
| Agent  team  membership  changed | Team  membership  changes  in  Keycloak  take  effect  on  the  agent's  next  login.  The  dashboard  will  not  update  until  the  agent  re-logs. | 
---
 
#### Supervisor cannot see certain agents on the dashboard
 
**Symptoms:** Some agents are missing from the Available Agents Detail or Ongoing Conversations dashboards.
 | Likely  Cause | Resolution |
| --- | --- |
| Agent  not  in  supervisor's  team | Confirm  the  agent  is  a  member  of  the  supervisor's  team  in  Unified  Admin  →  Teams. |
| Secondary  supervisor  scope | Secondary  supervisors  see  only  their  assigned  agents  by  default.  Use  the  team  filter  to  expand  the  view. |
| Agent  has  not  logged  in  since  team  membership  changed | Team  changes  take  effect  on  the  agent's  next  login.  Ask  the  agent  to  log  out  and  back  in. | 
---
 
#### 7. Historical Reports & Superset
 
#### Reports show no data for a recent period
 
**Symptoms:** A report is opened for today or yesterday but returns no results, even though conversations occurred.
 | Likely  Cause | Resolution |
| --- | --- |
| ETL  pipeline  delay | Historical  reports  update  every  5  minutes  after  a  conversation  closes.  Very  recently  closed  conversations  may  not  yet  appear  —  wait  and  refresh. |
| UTC  offset  misconfigured | If  the  report  timezone  does  not  match  the  deployment  timezone,  data  may  appear  to  be  missing  when  it  is  actually  in  a  different  time  window.  Adjust  the  UTC  offset  filter. |
| Superset  date  filter  set  incorrectly | Confirm  the  date  range  filter  in  Superset  is  set  to  the  correct  period.  The  default  may  be  set  to  a  range  that  excludes  the  period  you  are  investigating. |
| ETL  service  not  running | Check  the  CX  Analyser  /  ETL  pod  status:  `kubectl  get  pods  -A | 
---
 
#### Superset is inaccessible or returns a 500 error
 | Resolution  Steps |
| --- |
| Check  the  Superset  pod:  `kubectl  get  pods  -A |
| Review  Superset  logs:  kubectl  logs  <superset-pod>  -n  <namespace> |
| Verify  the  PostgreSQL  database  connection  from  Superset  is  healthy |
| Restart  Superset:  kubectl  rollout  restart  deployment/superset  -n  <namespace> |
| If  the  database  connection  is  the  issue,  check  PostgreSQL  pod  health  and  credentials | 
---
 
#### 8. Platform & Infrastructure
 
#### Pods are crashing or restarting repeatedly
 
**Symptoms:** `kubectl get pods -A` shows pods in CrashLoopBackOff or with high restart counts.
 
bash
 ```
# Check pod status
kubectl get pods -A | grep -v Running

# View logs for a crashing pod
kubectl logs <pod-name> -n <namespace> --previous

# Describe the pod for events and resource issues
kubectl describe pod <pod-name> -n <namespace>
``` | Likely  Cause | Resolution |
| --- | --- |
| Insufficient  memory  or  CPU | Check  node  resource  utilisation:  kubectl  top  nodes  .  Verify  pods  are  within  the  resource  limits  defined  in  Helm  values. |
| Missing  ConfigMap  or  Secret | Describe  the  pod  and  look  for  Failed  to  pull  secret  or  ConfigMap  not  found  events.  Verify  all  required  Kubernetes  secrets  and  ConfigMaps  are  present. |
| Database  connection  failure | Many  CelestaCX  services  will  crash-loop  if  they  cannot  connect  to  MongoDB  or  Redis.  Verify  those  pods  are  healthy  first. |
| Expired  SSL  certificate | Some  services  validate  certificates  on  startup.  Check  certificate  expiry  dates  and  renew  if  needed. | 
---
 
#### MongoDB replication lag or primary election issues
 
**Symptoms:** MongoDB replica set is unhealthy, logs show election events, or replication lag warnings appear.
 
bash
 ```
# Check replica set status
kubectl exec -it <mongodb-pod> -n <namespace> -- mongosh --eval "rs.status()"
``` | Likely  Cause | Resolution |
| --- | --- |
| Network  partition  between  replicas | Check  connectivity  between  all  MongoDB  replica  nodes.  Ensure  network  policies  allow  inter-pod  communication  on  the  MongoDB  port. |
| One  replica  significantly  behind | Resync  the  lagging  secondary  by  stepping  it  down  and  allowing  it  to  re-sync  from  the  primary. |
| Disk  I/O  saturation | Check  disk  performance  on  the  node  hosting  the  primary.  High  I/O  wait  times  cause  replication  lag.  Consider  moving  to  a  faster  storage  class. | 
---
 
#### SSL certificate expired
 
**Symptoms:** Browser shows certificate warnings when accessing the platform. Services may fail to start or communicate.
 
bash
 ```
# Check RKE2 cluster certificate expiry
kubectl get secret -n kube-system | grep tls

# Extend RKE2 SSL certificate expiry
# Follow the procedure in Deployment & Infrastructure → Security & Certificates
``` 
---
 
#### 9. Failover & Recovery
 
#### Conversations lost after system recovery
 
**Symptoms:** After a platform outage, active conversations that were in progress are gone — agents cannot see them and customers may be disconnected.
 
**Important:** This is expected behaviour in certain failure scenarios. The recoverability of active conversations depends on which components failed and whether they are deployed in HA mode.
 | Component  that  Failed | Impact  on  Active  Conversations |
| --- | --- |
| Application  pods  (CX  Router,  connectors) | Conversations  are  restored  on  pod  recovery  —  routing  engine  re-queues  or  restores  active  sessions |
| Redis  (non-HA  deployment) | All  active  conversations  are  permanently  lost  —  Redis  holds  session  state  and  cannot  be  recovered  after  failure |
| MongoDB  (non-HA  deployment) | All  conversation  data  is  lost  —  MongoDB  is  the  primary  data  store |
| ActiveMQ  /  PostgreSQL  (non-HA) | Active  conversations  in  transit  are  lost  —  messages  in  flight  cannot  be  recovered | 
**Recommendation:** Deploy Redis, MongoDB, ActiveMQ, and PostgreSQL as HA StatefulSets or managed HA services to eliminate these single points of failure. See [Deployment & Infrastructure → High Availability](#) for configuration guidance.
 
#### Email messages not fetched after recovery
 
After an email channel outage, emails received during downtime may not be automatically fetched when the connector recovers. Restart the email connector pod to trigger a re-fetch of recent messages.
 
#### Conversations stuck in queue after recovery
 
If conversations appear stuck in queue (not being routed) after a recovery event, restart the CX Router pod:
 
bash
 ```
kubectl rollout restart deployment/cx-router -n <namespace>
``` 
---
 
#### Collecting Diagnostic Information for Support
 
When raising a support ticket with OctaveBytes, collect the following before contacting support:
 
bash
 ```
# 1. Platform version
kubectl get configmap cx-version -n <namespace> -o yaml

# 2. Pod health snapshot
kubectl get pods -A > pod-status.txt

# 3. Recent logs from the affected service
kubectl logs <affected-pod> -n <namespace> --since=1h > service-logs.txt

# 4. Events in the namespace
kubectl get events -n <namespace> --sort-by='.lastTimestamp' > events.txt

# 5. Node resource usage
kubectl top nodes > node-resources.txt
kubectl top pods -A > pod-resources.txt
``` 
Include these files in your support ticket along with a description of the issue, when it started, and which users or tenants are affected.
 
---
 
#### What's Next
 
- [**Logging & Monitoring**](#) — how to access logs, set up monitoring, and configure alerts
- [**Platform Administration**](#) — backup, restore, upgrades, and certificate renewal
- [**FAQs**](#) — quick answers to common questions