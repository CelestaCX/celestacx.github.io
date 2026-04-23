# Key Reporting Metrics
#### Overview
 
This page defines every core metric used across CelestaCX historical reports and real-time dashboards. Use these definitions to interpret report data consistently, configure queue service level thresholds correctly, and ensure that everyone in your operation — supervisors, quality managers, and administrators — is working from the same understanding of what each number means.
 
Metrics are organised into four groups: conversation state, service level, service level calculation types, and derived performance metrics.
 
---
 
#### 1. Conversation State Metrics
 
These metrics define the fundamental outcome categories for every interaction that enters CelestaCX. Every conversation ends in one of these states, and this classification drives how it is counted in all downstream reports.
 | Metric | Definition |
| --- | --- |
| Handled Conversation | A conversation where the final task disposition is CLOSED with reason code DONE . The agent completed the interaction and submitted wrap-up. This is the ideal outcome. |
| Abandoned Conversation | A conversation where the customer disconnected while waiting in the queue or during agent alerting (ringing — before the agent accepted). The agent task closes with state CLOSED and reason code CANCELLED . |
| RONA (Redirect on No Answer) | A conversation that was assigned to an agent but the agent did not accept it within the configured alert timeout. The task is re-routed back to the queue or to another available agent. RONA events are tracked separately from abandonment. |
| No Agent Available | A conversation that entered the queue but could not be routed because no eligible agent was available. Tracked as a distinct category — does not contribute to SLA metrics. |
| Transferred Conversation | A conversation that was moved from one agent or queue to another before final closure. Transfer counts are recorded at the conversation and task level. |
| Flushed Conversation | A conversation forcefully closed by a supervisor or administrator rather than by natural completion or customer disconnection. | 
---
 
#### 2. Service Level Metrics
 
Service Level (SL%) measures the percentage of incoming conversations that an agent answers within a defined time threshold. The threshold timer starts the moment the conversation enters a precision queue — not when it first arrives in the system.
 
**Example:** If your SL threshold is 120 seconds and your SL target is 80%, you are aiming to answer 80% of all conversations within 2 minutes of them entering the queue.
 
The SL threshold is configured per queue in Unified Admin. Different queues can have different thresholds — for example, a VIP queue might use a 30-second threshold while a general enquiries queue uses 90 seconds.
 | Metric | Definition |
| --- | --- |
| chats_offered | Total conversations offered to the queue — the denominator for all SLA calculations. |
| chats_answered | Conversations accepted by an agent, regardless of whether they were answered within or after the SL threshold. |
| chats_abandoned | Conversations where the customer left before connecting to an agent. |
| chats_rona | Conversations where an agent was assigned but did not accept within the alert timeout. |
| NoAgentAvailable | Conversations where no eligible agent was available in the queue. Tracked separately — does not count toward SL metrics. |
| service_level_offered | AnsweredWithinSL + AbandonedWithinSL — all conversations that were still in the SL window when their outcome was determined. |
| service_level_answered | Conversations accepted by an agent within the SL threshold. The primary numerator for SL% calculations. |
| service_level_abandoned | Conversations abandoned by the customer before the SL threshold elapsed. |
| service_level_rona | RONA events that occurred within the SL threshold window. | 
#### SLA by Scenario
 
The table below shows exactly how each metric is counted across the eight possible conversation scenarios.
 | # | Scenario | offered | answered | abandoned | rona | NoAgent | sl_offered | sl_answered | sl_abandoned | sl_rona |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | Routed to agent within SL — agent accepts | 1 | 1 | 0 | — | — | 1 | 1 | 0 | — |
| 2 | Customer abandons before routing , within SL | 1 | 0 | 1 | — | — | 1 | 0 | 1 | — |
| 3 | Routed to agent within SL — customer abandons before agent accepts | 1 | 0 | 1 | — | — | 1 | 0 | 1 | — |
| 4 | Routed to agent after SL — customer abandons before agent accepts | 1 | 0 | 1 | — | — | 0 | 0 | 0 | — |
| 5 | Routed to agent after SL — agent accepts | 1 | 1 | 0 | — | — | 0 | 0 | 0 | — |
| 6 | Routed to agent after SL — agent does not accept (RONA) | 1 | 0 | 0 | 1 | — | 0 | 0 | 0 | 0 |
| 7 | Routed to agent within SL — agent does not accept (RONA) | 1 | 0 | 0 | 1 | — | 1 | 0 | 0 | 1 |
| 8 | Conversation queued — no agents available | 1 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 
**Key observations:**
 
- **Scenario 1** is the ideal outcome — offered, answered, and within threshold.
- **Scenarios 2 and 3** count as abandoned within the SL window — the customer left while still eligible for service. Whether these count against SL% depends on the SL Type configured (see below).
- **Scenario 4** — abandonment after the threshold — does not contribute to SL metrics at all.
- **Scenario 5** — answered after the threshold — counts as answered but not within service level.
- **Scenario 7** — RONA within threshold — counts as an SL-offered RONA event.
- **Scenario 8** — no agent available — is tracked separately via `NoAgentAvailable` and does not affect SL metrics.
 
---
 
#### 3. Service Level Calculation Types
 
The SL% formula applied in reports depends on the **Service Level Type** configured per queue. There are three types, each treating abandoned conversations differently. Your choice of type has a significant impact on how your SLA numbers appear, so it is important to select the one that reflects your operation's philosophy on abandoned contacts.
 | Type | Name | Formula |
| --- | --- | --- |
| Type 1 | Ignore Abandoned | AnsweredWithinSL ÷ (TotalOffered – AbandonedWithinSL) |
| Type 2 | Abandoned = Negative Impact | AnsweredWithinSL ÷ TotalOffered |
| Type 3 | Abandoned = Positive Impact | (AnsweredWithinSL + AbandonedWithinSL) ÷ TotalOffered | 
#### Choosing the Right Type
 | If… | Use |
| --- | --- |
| Your team is not responsible for customers who abandon quickly — e.g. customers who hang up within seconds before you could realistically answer | Type 1 — removes early abandons from the calculation so they do not unfairly drag down your SL% |
| You want the strictest possible SLA measurement — every abandon counts against you regardless of when it occurred | Type 2 — the most conservative approach, often required by enterprise contracts or regulators |
| Early abandons (customers who hang up very quickly) should be treated as self-served or non-events rather than failures | Type 3 — credits the queue for these abandons, producing the highest SL% numbers | 
The Service Level Type is configured per queue in **Unified Admin → Queues → [Queue] → Service Level Settings** . Different queues in the same operation can use different types if their SLA obligations differ.
 
---
 
#### 4. Derived Performance Metrics
 
These metrics are calculated from raw interaction data and appear across multiple reports. They are the most commonly referenced metrics in supervisor reviews, management reporting, and workforce planning.
 | Metric | Definition | Formula |
| --- | --- | --- |
| AHT (Average Handle Time) | The average total time an agent spends on a conversation from acceptance to wrap-up completion. Includes talk time, hold time, and wrap-up time. | Total Handle Duration ÷ Total Handled Tasks |
| ATT (Average Talk Time) | Average time spent in active conversation with the customer, excluding hold and wrap-up. | Total Talk Duration ÷ Total Handled Tasks |
| AWT (Average Wait Time) | Average time conversations spend in queue before being answered or abandoned. | Total Queue Wait Duration ÷ Total Offered |
| FCR (First Contact Resolution) | The percentage of conversations resolved without the customer needing to contact again within a defined period (typically 24–72 hours). Measured via the Repeated Caller Report. | (Total Contacts – Repeat Contacts) ÷ Total Contacts × 100 |
| Occupancy | The percentage of logged-in time an agent spends handling interactions (talk + wrap-up), versus idle or available time waiting for a task. | (Talk Time + Wrap-up Time) ÷ (Logged-in Time) × 100 |
| Shrinkage | The percentage of scheduled time during which agents are unavailable for interactions — covering training, breaks, meetings, system issues, and unplanned absences. | Unavailable Time ÷ Scheduled Time × 100 |
| CSAT (Customer Satisfaction Score) | Average customer satisfaction rating collected via post-interaction surveys. Typically expressed as a percentage or average score on a defined scale. | Derived from survey responses — see Survey Reports |
| Abandonment Rate | The percentage of offered conversations where the customer disconnected before connecting to an agent. | Abandoned ÷ Offered × 100 |
| RONA Rate | The percentage of routed conversations where the assigned agent did not accept. High RONA rates indicate agents going Not Ready without updating their state, or availability issues. | RONA Events ÷ Offered × 100 |
| Transfer Rate | The percentage of handled conversations that were transferred to a different agent or queue. High rates may indicate routing misconfiguration or skill gaps. | Transfers ÷ Handled × 100 | 
---
 
#### 5. Agent State Metrics
 
Agent state metrics appear in availability and adherence reports. They measure how agents are distributing their logged-in time across different states.
 | Metric | Definition |
| --- | --- |
| Logged-in Time | Total time the agent was signed into the platform during the period |
| Ready Time | Time spent in Ready state — eligible to receive routed conversations |
| Not Ready Time | Time spent in Not Ready state — unavailable for routing, with or without a reason code |
| Active Time | Time spent actively handling one or more conversations |
| Busy Time | Time spent at the maximum task limit — no new conversations could be routed |
| Wrap-up Time | Time spent completing wrap-up after a conversation closes (voice channels; digital wrap-up is part of Active time) |
| Availability % | Ready Time ÷ Logged-in Time × 100 — what proportion of logged-in time the agent was available |
| Utilisation % | Active Time ÷ Logged-in Time × 100 — what proportion of logged-in time the agent was actually handling interactions | 
---
 
#### 6. Queue Performance Metrics
 
These metrics appear in queue productivity and interval reports.
 | Metric | Definition |
| --- | --- |
| Offered | Total conversations that entered the queue |
| Answered | Conversations accepted by an agent |
| Abandoned | Conversations where the customer left before connecting |
| Abandoned % | Abandoned ÷ Offered × 100 |
| Average Speed of Answer (ASA) | Average time from a conversation entering the queue to the moment an agent accepts it — for conversations that were answered |
| Oldest in Queue | The longest current wait time among conversations still waiting — a real-time metric visible in the Summary Dashboard |
| Queue Depth | Number of conversations currently waiting — a real-time metric |
| Throughput | Number of conversations fully handled and closed within the period | 
---
 
#### Quick Reference: Metric Lookup
 | I want to know… | Metric to use |
| --- | --- |
| Are we answering fast enough? | Service Level % (SL%), ASA |
| How long are agents spending per interaction? | AHT, ATT |
| Are customers having to call back? | FCR, Repeated Caller Report |
| How busy are my agents really? | Occupancy, Utilisation % |
| Are agents available enough? | Availability %, Ready Time |
| How many customers are giving up? | Abandonment Rate |
| Are agents missing assigned conversations? | RONA Rate |
| Are we routing correctly? | Transfer Rate |
| How satisfied are customers? | CSAT |
| What time is the queue busiest? | Interval Performance Report, Answered Chats in Time Intervals | 
---
 
#### What's Next
 
- [**Reporting Database Schema**](#) — the underlying data model for teams building custom Superset dashboards or integrating with external BI tools
- [**Historical Reports**](#) — the full report library where these metrics appear
- [**Real-Time Dashboards**](#) — where to see live values for queue and agent metrics during the shift
