CelestaCX's real-time dashboards give supervisors a live view of everything happening in their contact centre — queue health, agent availability, active conversations, bot activity, and AI-powered sentiment signals — all updating continuously throughout the shift.
 
The real-time layer is embedded directly in the Agent Desk interface, so supervisors do not need to open a separate tool. Dashboards are built on Grafana and refresh automatically, with detailed dashboards updating every 10 seconds.
 
This page documents every real-time dashboard available to supervisors and agents, what each one shows, how to filter it, and what actions can be taken from within it.
 **Real-time vs historical:** The dashboards on this page show live data for active and current-shift interactions. For completed interactions and trend analysis over time, see [Historical Reports](#) . Some metrics in real-time dashboards — such as Service Level % and AHT — are derived from the historical ETL pipeline and update every 5 minutes rather than in real time. 
---
 
#### Dashboard Map
 
CelestaCX provides two tiers of real-time dashboards:
 | Tier | Dashboards | Refresh |
| --- | --- | --- |
| Summary | Summary  Dashboard,  Team  Stats  Dashboard | Automatic  (Grafana) |
| Detailed | Queued  Conversations,  Ongoing  Conversations,  Available  Agents,  My  Past  Conversations | Every  10  seconds | 
---
 
#### 1. Summary Dashboard
 
The Summary Dashboard is the default view shown after login. It provides a high-level snapshot of queue health and agent state distribution across the supervisor's assigned teams and queues — the starting point for every shift.
 
#### Who Sees What
 | Role | What  They  See |
| --- | --- |
| Supervisor | Must  select  a  Team  first,  then  one  or  more  queues  from  that  team.  Sees  only  teams  they  supervise  —  not  any  team  they  belong  to  as  an  agent. |
| Agent | Sees  only  the  Queues  dropdown  with  queues  they  are  assigned  to. | A supervisor must have at least one team with at least one agent who has logged in at least once before team and queue data becomes visible on this dashboard. 
#### Filters
 
Apply filters in this sequence to scope the data:
 | Filter | What  It  Does |
| --- | --- |
| Team  Name | Selects  the  team  to  monitor.  Drives  which  queues  appear  in  the  Queue  dropdown. |
| Queue  Name | Filters  queue  stats  to  selected  queue(s).  Supports  multi-select. |
| MRD  Name | Filters  the  agent  state  pie  chart  to  agents  in  the  selected  MRD. |
| Bot  Name | Filters  bot  activity  to  a  specific  bot  connector. | 
#### Panels
 
**Queue Stats Row** — four headline metrics across selected queues:
 | Metric | What  It  Shows | Data  Source |
| --- | --- | --- |
| Service  Level  % | Percentage  of  conversations  answered  within  the  SL  threshold | Historical  ETL  (updates  every  ~5  min) |
| Active  with  Agents | Conversations  currently  being  handled  by  agents | Live |
| Average  Wait  Time | Average  time  conversations  have  spent  queued  before  being  answered  (  HH:MM:SS  ) | Live |
| Average  Handle  Time  (AHT) | Average  duration  of  closed  tasks  including  wrap-up  (  Total  Duration  ÷  Total  Tasks  ) | Historical  ETL  (updates  every  ~5  min) | 
**Queue Summary Statistics Table** — one row per queue in the selected team:
 | Column | What  It  Shows |
| --- | --- |
| Queue  Name | Name  of  the  queue |
| MRD  Name | Media  Routing  Domain  associated  with  this  queue |
| Total  Queued | Conversations  currently  waiting  in  this  queue |
| Oldest  in  Queue | Longest  current  wait  time  in  the  queue  (  HH:MM:SS  ;  shows  00:00:00  if  empty) |
| Not  Ready | Agents  in  NOT_READY  state  on  this  queue's  MRD |
| Ready | Agents  in  READY  state |
| Active | Agents  in  ACTIVE  state |
| Pending  Not  Ready | Agents  in  PENDING_NOT_READY  state |
| Busy | Agents  at  maximum  task  limit | 
**Agent States Row** — a pie chart showing how many agents are in each MRD state (Not Ready, Ready, Active, Busy, Pending Not Ready), filtered by MRD.
 
**Bot Stats Row** — number of conversations currently being handled by the bot, filtered by Bot Name. A conversation counts as active with the bot until an agent joins it.
 
#### Notes
 
- Theme changes applied elsewhere in Agent Desk automatically apply to the Summary Dashboard on next open. Changing the theme while on the dashboard causes it to reload.
- Agent attribute or team membership changes made in the IAM system take effect on the agent's **next login** , not immediately on the dashboard.
 
---
 
#### 2. Team Stats Dashboard
 
The Team Stats Dashboard provides a per-agent breakdown within the selected team — showing individual agent states, conversation counts, and handle time side by side. Use this when the Summary Dashboard signals a problem and you need to identify which specific agents are overloaded or idle.
 
Navigate to the Team Stats Dashboard from the supervisor interface dashboard menu. Apply the same Team and Queue filters as the Summary Dashboard to scope the view to the right agents.
 
---
 
#### 3. Realtime Detailed Dashboards
 
The detailed dashboards are embedded in Agent Desk and refresh every **10 seconds** . They complement the Summary Dashboard's aggregated statistics by showing individual conversation and agent records that supervisors can act on directly.
 
#### Saved Filters
 
Filter selections are shared across all four detailed dashboards. Selecting Team A and Queue B on the Queued Conversations dashboard automatically applies those filters when you navigate to Ongoing Conversations or Available Agents. Filters are stored in the browser cache.
 
---
 
#### 3a. Queued Conversations Detail
 
Lists every conversation currently waiting in a queue — not yet assigned to an agent.
 
**Access:** Supervisors only. Agents cannot view this dashboard.
 
**Filters:** Select Team first, then Queue(s).
 | Column | What  It  Shows |
| --- | --- |
| Customer | Customer  name  (if  identified),  channel  identifier,  and  channel  type  icon |
| Waiting  Since | How  long  the  conversation  has  been  in  queue  (e.g.  "4  minutes  ago") |
| Queue  Name | The  queue  where  this  conversation  is  waiting | 
**Notes:**
 
- Shows all queues in the selected team when no specific queue is chosen
- Queue configuration changes take effect on the agent's next login
 
---
 
#### 3b. Ongoing Conversations Detail
 
Lists every inbound conversation currently active with an agent, supervisor, or bot. This is the primary dashboard for live monitoring — and the entry point for silent monitoring and barge-in.
 
**Access:** Supervisors only. Agents cannot view this dashboard.
 
**Filters:**
 | Filter | Description |
| --- | --- |
| Answered  by | Toggle  between  Agents  (default)  and  Bots |
| Team | Filter  to  the  supervisor's  assigned  team |
| Queue(s) | Filter  by  one  or  more  queues  within  the  selected  team | 
For supervisors with secondary supervisor assignments: by default you see only your assigned agents. Use the team filter to expand the view to other secondary supervisors' agents or all team agents.
 | Column | What  It  Shows |
| --- | --- |
| Direction | Inbound  or  Outbound |
| Channel | Channel  type  (WhatsApp,  Facebook,  Webchat,  Email,  Voice,  etc.) |
| Customer | Name  if  identified,  plus  channel  identity |
| Active  Since | How  long  the  conversation  has  been  active |
| Agent | Agent  name  and  username  handling  the  conversation |
| Queue  Name | Queue  the  conversation  was  routed  through | 
**From this dashboard you can:**
 
- Click the **Monitor** (eye) icon to enter Silent Monitoring on any conversation
- Escalate to **Barge-in** from within the monitored conversation
 
**Known limitations:**
 
- Only inbound conversations appear — outbound is not shown
- A conversation active with agents from different queues appears as two rows; same queue appears as one row
- Conversations involving agents not in the supervisor's team are not visible
- After a consult transfer, the new primary agent's conversation does not appear until the dashboard next refreshes
- After a direct named agent transfer, the conversation is removed from the dashboard
 
---
 
#### 3c. Available Agents Detail
 
Displays all agents in the selected team with their current global state and active conversation count. The primary dashboard for team availability monitoring and the only place supervisors can change agent state or force logout remotely.
 
**Access:** Supervisors and Agents. Agents see only their own team's data.
 
**Filters:** Search by agent name, or filter by Team.
 | Column | What  It  Shows |
| --- | --- |
| Agent | Agent  name  and  current  global  state  (Ready  /  Not  Ready) |
| Active  Conversations | Number  of  conversations  currently  assigned  to  this  agent |
| MRD  State | Agent's  state  per  MRD,  shown  as  colour-coded  indicators | 
**Supervisor actions available from this dashboard:**
 | Agent's  Current  State | Available  Action |
| --- | --- |
| Not  Ready | Change  to  Ready |
| Any  state | Force  Log  Out | State changes from this dashboard affect the agent's **global** state only, not individual MRD states. Force logout re-routes all active conversations before completing. 
**Known limitations:**
 
- The Reserved MRD state is not currently shown — the Force Logout option appears even for agents in Reserved state
- Team membership changes in Keycloak take effect on the agent's next login
 
---
 
#### 3d. My Past Conversations
 
Allows agents to view and reopen their recently handled conversations for follow-up messages, deferred replies, or re-engagement — particularly useful for email and asynchronous digital channels.
 
**Access:** Agents only (each agent sees their own conversations only).
 
**Filters:**
 | Filter | Options |
| --- | --- |
| Search | By  customer  name |
| Channel | Filter  by  one  or  more  channel  types |
| Time  range | Last  24  hours  (default),  Last  3  days,  Last  7  days | | Column | What  It  Shows |
| --- | --- |
| Customer  Name | The  customer  from  the  past  conversation |
| Channel | Communication  channel  used |
| Conversation  End  Time | When  the  conversation  was  closed |
| Action | Opens  the  interaction  history;  enables  sending  a  follow-up  if  supported  by  the  channel | 
Conversations are sorted newest first, with 25 per page and pagination controls.
 
---
 
#### 4. AI-Powered Real-Time Intelligence
 
Beyond the dashboard panels, CelestaCX surfaces AI-generated intelligence during live interactions to enable proactive supervisor action.
 
#### Real-Time Sentiment Detection
 
The platform tracks the customer's emotional tone as it shifts during an active conversation, flagging negative sentiment escalations as they happen. Supervisors see sentiment signals directly in the Ongoing Conversations Detail view, allowing them to prioritise which conversations need attention before an agent raises a hand raise.
 
#### Live AI Summaries
 
For conversations involving a bot or a long agent interaction, the platform generates a running AI summary of key discussion points. When a supervisor enters Silent Monitoring or prepares to Barge In, the summary gives them immediate context — what the customer's issue is, what has been tried, and what the current state is — without needing to read the full transcript.
 
#### KPI Alerts
 
A rule-based alerting system generates real-time notifications when configured KPI thresholds are breached. Common alert triggers include:
 | Alert | Example  Threshold |
| --- | --- |
| Queue  wait  time | Exceeds  SLA  threshold |
| Agent  Not  Ready  duration | Agent  in  Not  Ready  for  more  than  X  minutes |
| Customer  sentiment | Drops  to  negative  and  stays  there |
| Bot  dead-end | Bot  fails  to  resolve  after  N  turns | 
Alerts appear in the supervisor interface so action can be taken immediately — before the situation escalates to a formal hand raise or complaint.
 
#### Bot-to-Agent Escalation Context
 
When the AI detects that a bot is unable to resolve a customer's issue — repeated misunderstanding, explicit escalation request, or configured trigger — it initiates a warm transfer to a human agent. The receiving agent automatically gets the AI-generated conversation summary, the full bot session history, and the customer's current sentiment state. The customer does not need to repeat themselves.
 
---
 
#### Quick Reference: Which Dashboard for What
 | I  need  to… | Use  this  dashboard |
| --- | --- |
| Get  a  shift-start  overview  of  queue  and  agent  health | Summary  Dashboard |
| See  which  specific  agents  are  idle  or  overloaded | Team  Stats  Dashboard |
| See  which  conversations  are  waiting  in  queue | Queued  Conversations  Detail |
| Monitor  a  live  conversation  or  barge  in | Ongoing  Conversations  Detail |
| Change  an  agent's  state  or  force  logout | Available  Agents  Detail |
| Follow  up  on  a  past  conversation | My  Past  Conversations  (agent) | 
---
 
#### What's Next
 
- [**Historical Reports**](#) — the full library of closed-interaction reports available in Superset
- [**Key Reporting Metrics**](#) — definitions of every metric referenced in dashboards and reports
- [**Supervisor Guide**](#) — how to use dashboards as part of your day-to-day supervision workflow