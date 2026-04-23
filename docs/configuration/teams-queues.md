# Agent Teams & Queues
Agent teams and queues are the organisational backbone of CelestaCX. Teams group your agents for management and reporting purposes. Queues define where interactions wait and which agents are available to handle them. This page covers how to create and configure both.
 
---
 
### Teams vs. Queues — Understanding the Difference
 
These two concepts are related but serve different purposes:
 | Concept | Purpose | Managed By |
| --- | --- | --- |
| Team | Groups agents together for supervisory, reporting, and management purposes. A team has a supervisor and a set of members. | Tenant Admin, Supervisor |
| Queue | Holds incoming interactions until an agent is available. A queue is served by one or more teams. | Tenant Admin | 
A single agent can belong to one team but serve multiple queues. A single queue can be served by agents from multiple teams. This flexibility allows you to model complex operational structures — for example, a blended team handling both voice and chat queues simultaneously.
 
---
 
### Agent Teams
 
#### Creating a Team
 
1. Navigate to **Administration → Teams → Add Team** .
2. Enter a team name and optional description.
3. Assign a **Supervisor** — the user who will monitor and manage this team's agents in real time. A team must have at least one supervisor.
4. Add team members by searching for and selecting users with the Agent role.
5. Set the team status to **Active** and save.
 
#### Team Settings
 | Setting | Description |
| --- | --- |
| Team Name | Display name used in reports, dashboards, and the supervisor view. |
| Supervisor(s) | One or more users with the Supervisor role assigned to oversee this team. |
| Members | Agents who belong to this team. An agent can only belong to one team at a time. |
| Default Queue | The queue interactions are offered to agents in this team by default, if no other routing rule applies. |
| Status | Active or Inactive. Inactive teams are excluded from routing and reporting. | 
#### Managing Team Membership
 
Team membership can be updated at any time. Adding or removing an agent from a team takes effect immediately — there is no need to restart or republish configuration. Changes are reflected in the supervisor live view within one refresh cycle.
 
When an agent is removed from a team, any interactions currently assigned to them are not automatically transferred. The supervisor should manage active work before removing an agent from a live team.
 
---
 
### Queues
 
#### Creating a Queue
 
1. Navigate to **Administration → Queues → Add Queue** .
2. Enter a queue name and optional description.
3. Select the **channel type(s)** this queue will handle: voice, chat, email, or all digital. A queue can be omnichannel or restricted to specific channel types.
4. Assign one or more **teams** to serve this queue.
5. Configure capacity and behaviour settings (see below).
6. Link a **Business Calendar** to define when this queue is open. See [Business Calendars](#) .
7. Configure **Overflow Settings** to define fallback behaviour when wait thresholds are exceeded. See [Channels & Routing Configuration](#) .
8. Set the queue status to **Active** and save.
 
#### Queue Capacity Settings
 | Setting | Description | Default |
| --- | --- | --- |
| Max Queue Size | Maximum number of interactions allowed to wait in queue simultaneously. Interactions above this limit are rejected or overflow. | 50 |
| Max Wait Time | Time (in seconds) after which a waiting interaction triggers overflow handling. | 300 |
| Agent Concurrent Capacity | Maximum number of simultaneous interactions a single agent in this queue can handle. Applies to digital/chat channels. | 3 |
| Wrap-Up Time | Time (in seconds) an agent has after completing an interaction before they are offered the next one. | 30 |
| Auto-Accept | If enabled, interactions are automatically assigned to available agents without requiring them to manually accept. | Disabled | 
#### Wrap-Up Behaviour
 
Wrap-up time gives agents a period after an interaction ends to complete after-call or after-contact work — updating records, completing forms, or adding notes — before the next interaction is offered.
 
During wrap-up, the agent's status is set to **Wrap-Up** and they are excluded from routing. Wrap-up ends automatically when the timer expires, or the agent can end it manually by changing their status to **Available** .
 
Supervisors can override an agent's wrap-up state from the live supervisor view if needed.
 
#### Queue Priority Weighting
 
When an agent becomes available and multiple queues they serve have waiting interactions, CelestaCX selects the next interaction based on:
 
1. **Interaction priority level** (set by routing rules — higher priority wins)
2. **Wait time** (longest-waiting interaction at the same priority level is offered first)
3. **Queue weighting** (if two queues have interactions of equal priority and wait time, the queue with the higher configured weight is preferred)
 
Queue weighting is set under **Administration → Queues → [Queue] → Advanced Settings → Queue Weight** (scale of 1–10).
 
---
 
### Agent States
 
Understanding agent states is important when configuring queues, as routing only offers interactions to agents in the **Available** state.
 | State | Description | Routable |
| --- | --- | --- |
| Available | Agent is logged in and ready to receive interactions. | Yes |
| Busy | Agent is actively handling one or more interactions. | Only if concurrent capacity allows |
| Wrap-Up | Agent is completing after-contact work following an interaction. | No |
| Away | Agent has set themselves as temporarily unavailable (e.g. break, meeting). | No |
| Offline | Agent is not logged in. | No | 
Supervisors can view and override agent states from the live supervisor dashboard. Away reason codes can be customised — see [Supervisor Guide](#) for details.
 
---
 
### Omnichannel Blending
 
CelestaCX supports omnichannel blending, where a single agent can handle interactions from multiple channel types simultaneously — for example, two live chats and one email at the same time.
 
Blending is controlled by the **Agent Concurrent Capacity** setting per queue, combined with the agent's own channel capacity limits set at the user level.
 
To configure per-agent channel capacity:
 
1. Navigate to **Administration → Users → [User] → Channel Capacity** .
2. Set the maximum concurrent interactions for each channel type (voice, chat, email).
 
> **Best practice:** Voice interactions should always have a concurrent capacity of 1. Chat and messaging channels typically support 2–4 concurrent interactions depending on agent experience and interaction complexity.
 
---
 
### Viewing Queue Performance
 
Live queue metrics are available to supervisors and admins in the **Real-Time Dashboard** . Key metrics visible per queue include:
 
- Interactions waiting and average wait time
- Number of agents available, busy, and in wrap-up
- Longest wait in queue
- Service level percentage (interactions answered within target time)
 
For historical queue performance, see [Historical Reports](#) in the Reports & Analytics section.
 
---
 
*What's next: Configure* [*Business Calendars*](#) *to define when your queues are open and apply time-based routing rules.*
