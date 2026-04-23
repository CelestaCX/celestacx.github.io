# Historical Reports

#### Overview

Historical reports in CelestaCX cover every dimension of closed interaction data — agent performance, queue productivity, conversation outcomes, quality scores, campaign results, IVR journeys, WebRTC sessions, and customer survey responses. All reports are delivered through Apache Superset and are available to supervisors, quality managers, and administrators based on their role.

Historical reports only include interactions that have been fully closed and processed through the ETL pipeline. By default the pipeline runs every 5 minutes, so very recently closed conversations may not yet appear. For live data, use the [Real-Time Dashboards](#).

**Accessing reports:** Navigate to the **Analytics & Reporting** module in the CelestaCX interface to open Superset. All reports listed on this page are available there.

> **Timezone note:** Always confirm the UTC offset setting for your reports matches your operational timezone. Reports default to the server timezone. See your administrator if reports show timestamps that do not match your local time.

---

#### How to Use This Page

Reports are organised by category below. For each report you will find what it covers, who typically uses it, and the key fields or columns available. Use the category index to jump directly to the area you need.

**Category index:**

1. [Conversation & Channel Reports](#)
2. [Agent & Queue Reports](#)
3. [Quality & Evaluation Reports](#)
4. [Outbound & Campaign Reports](#)
5. [IVR Reports](#)
6. [WebRTC Reports](#)
7. [Survey Reports](#)
8. [Common Filters & Settings](#)

---

#### 1. Conversation & Channel Reports

These reports cover the full customer journey — from the moment an interaction enters the system to its final disposition. Use them to understand interaction volumes, channel mix, outcomes, and customer behaviour patterns.

#### Conversation Detail

The most granular conversation-level report. One row per conversation, containing everything that happened within it.

| Field | Description |
| --- | --- |
| Direction | Inbound or Outbound |
| Queue | Queue the conversation was routed through |
| Start / End Time | Timestamps for conversation open and close |
| Duration | Total conversation duration |
| Agent | Handling agent name and username |
| Customer | Customer identity and channel identifier |
| Routing Mode | Push or Pull |
| Transfer Count | Number of transfers that occurred |
| Bot Participation % | Percentage of the conversation handled by a bot |
| Transcript | Link to full conversation transcript |
| Disposition | Final outcome (Handled, Abandoned, etc.) |

**Best for:** Compliance reviews, escalation investigations, individual case lookups.

---

#### Historical Conversation Summary

Aggregated summary of all conversations over a selected period, across all channel types and queues.

**Best for:** Daily and weekly volume trending, shift performance reviews, management reporting.

---

#### Channel Stats Summary

Summarises all conversations opened for a particular channel during the selected period — total opened, handled, abandoned, and transferred by channel type.

**Best for:** Understanding which channels are driving the most volume and where abandonment is concentrated.

---

#### Channel Stats Graph

Visual representation showing the percentage of channel sessions closed due to each disposition type (handled, abandoned, transferred, bot-resolved, etc.).

**Best for:** Presentations, quick visual checks of channel health.

---

#### Multichannel Session Detail

Where a single conversation spans multiple channel sessions (e.g. a customer starts on webchat and then escalates to voice), this report shows each session within that conversation as a separate row.

**Best for:** Understanding cross-channel journeys and escalation patterns.

---

#### Channel Session Detail

Details of each individual channel session — one row per session regardless of how many sessions a conversation contains.

**Best for:** Detailed session-level analysis, channel-specific performance measurement.

---

#### Channel Session Audit Report

Session-level audit trail intended for compliance and quality reviews. Shows the full sequence of events within each channel session with timestamps.

**Best for:** Compliance audits, dispute resolution, regulatory evidence.

---

#### Conversation Volume by Disposition

Bar chart showing conversation volume broken down by who resolved it — Agent, Bot, or Network — over the selected period.

**Best for:** Tracking bot containment rates, understanding agent vs. automation resolution split.

---

#### Wrap-up Summary

Count of conversations grouped by wrap-up category and reason code for the selected period.

| Field | Description |
| --- | --- |
| Wrap-up Category | The broad topic selected by the agent (e.g. Billing, Technical Support) |
| Wrap-up Reason | The specific outcome selected (e.g. Issue Resolved, Escalated) |
| Count | Number of conversations with this wrap-up combination |

**Best for:** Identifying the most common contact reasons, tracking resolution patterns, informing training priorities.

---

#### Repeated Caller Report

Identifies customers who made more than one contact within a configurable timeframe. Used to measure First Contact Resolution (FCR) indirectly — customers who return are likely those whose issue was not fully resolved on the first contact.

**Best for:** FCR measurement, identifying customers who need follow-up, spotting systemic resolution issues.

---

#### 2. Agent & Queue Reports

These reports measure how effectively agents are working and how efficiently queues are operating. The primary reports for supervisor performance management and workforce planning.

#### Agent Performance Report

The most commonly used historical report for supervisors. Shows agent KPIs in 15-minute interval buckets across the selected period.

| Field | Description | Format |
| --- | --- | --- |
| Interval | Time window of the data (default 15 minutes) | HH:MM |
| Calls Answered | Interactions successfully accepted by the agent | Count |
| Calls Handled | Interactions answered and completed with a wrap-up | Count |
| Talk Time | Total time spent in active conversation | HH:MM:SS |
| Hold Time | Total time the agent placed interactions on hold | HH:MM:SS |
| Ring Time | Time the task was offered before the agent answered (latency) | HH:MM:SS |
| Not Ready Time | Total time spent in Not Ready state across all reasons | HH:MM:SS |
| Wait Time | Total time in Ready state waiting for a task | HH:MM:SS |
| External Transfer | Count of interactions transferred to an external party | Count |

**Best for:** Individual agent productivity reviews, coaching preparation, interval-level trend analysis.

---

#### Agent Productivity by Queue

Concise per-queue summary of each agent's productivity — total tasks assigned and completed per queue.

**Best for:** Understanding which queues each agent is contributing to and where capacity imbalances exist.

---

#### Agent Task Detail

All conversation tasks handled by an agent over the selected period, including RONA (Redirect on No Answer) events where the agent was assigned a task but did not accept it.

**Best for:** Identifying agents with high RONA rates, compliance investigations, detailed task-level review.

---

#### Agent Availability Report

MRD-level availability statistics per agent for the selected period — time logged in, time Ready, time Not Ready, time Active, time Busy, and overall availability percentage.

**Best for:** Measuring effective agent utilisation, identifying availability issues, adherence baseline data.

---

#### Agent State Analysis Report

Time-series breakdown of each agent's state transitions — ready, not-ready (with reason), active, busy — for monitoring and visualisation.

**Best for:** Identifying state patterns, coaching agents on availability behaviour, shift-level analysis.

---

#### Agent Not Ready Detail

Details of each individual Not Ready period per agent — timestamp, duration, and reason code selected.

**Best for:** Granular adherence investigation, verifying reason code accuracy, coaching conversations.

---

#### Agent Not Ready Summary

Aggregated summary of Not Ready reason durations per agent across the selected period.

**Best for:** Quick comparison of how agents are distributing their non-available time across reason codes.

---

#### Agent Adherence Report

Compares scheduled availability against actual login and ready time for each agent. Shows how closely agents are following their scheduled shifts.

**Best for:** Workforce management adherence tracking, identifying patterns of early logout or late login.

---

#### Agent Interaction Log

Searchable log of every interaction handled by agents — with timestamps, channel, queue, outcome, and wrap-up codes. The most complete audit trail of agent activity available.

**Best for:** Compliance audits, finding a specific interaction, agent activity verification.

---

#### Presence and Availability Reports

Agent state history showing how long each agent spent in each state (Ready, Not Ready, Away) during the selected period, across all MRDs.

**Best for:** Shift-level availability analysis, identifying consistently under-available agents.

---

#### Answered Chats in Time Intervals

Conversations answered by agents broken down into 15-minute time buckets — showing when during the day or week volume peaks and agents are answering the most interactions.

**Best for:** Staffing optimisation, identifying peak periods, scheduling adjustments.

---

#### Queue Stats Today

Queue statistics for the current day since midnight — offered, answered, abandoned, SLA, and average wait time per queue.

**Best for:** Same-day operational review, end-of-shift reporting.

---

#### Queue Productivity Report

Wait times, abandonment rates, and interaction throughput per queue over the selected period. One of the most important reports for assessing whether queue configuration and staffing levels are appropriate.

| Field | Description |
| --- | --- |
| Total Offered | All interactions that entered the queue |
| Answered | Interactions accepted by an agent |
| Abandoned | Interactions where the customer left before connecting |
| Abandoned % | Abandonment as a percentage of total offered |
| Average Wait Time | Mean time in queue before answer or abandon |
| Longest Wait Time | The single longest queue wait in the period |
| SL% | Percentage answered within the configured SL threshold |

**Best for:** Queue health assessment, SLA compliance review, staffing gap identification.

---

#### Queue-wise Stats Summary

Count of requests per queue broken down by outcome — offered, done, cancelled, transferred, and other dispositions.

**Best for:** High-level queue comparison, identifying queues with high transfer or cancellation rates.

---

#### Transferred Tasks per Queue

Bar chart showing how many tasks were transferred out of each queue during the selected period.

**Best for:** Identifying queues that agents frequently transfer away from — may signal routing misconfiguration or skill gaps.

---

#### Queue Flushed Conversation Count

Count of conversations forcefully closed per queue by an administrator or supervisor (rather than by natural completion).

**Best for:** Operations reviews, tracking manual interventions, identifying queues that regularly need flushing.

---

#### Interval Performance Report

Key metrics broken into configurable time intervals — 15-minute, 30-minute, or hourly — across the selected period. Shows how performance varies throughout the day.

**Best for:** Intraday trend analysis, staffing optimisation, identifying time-of-day patterns in SLA breaches.

---

#### Agent & Team Leaderboard

Ranked performance comparison across agents and teams based on key metrics — interactions handled, average handle time, SLA compliance, and quality scores where available.

**Best for:** Team performance reviews, recognition programmes, identifying high and low performers.

---

#### Team Comparison Report

Side-by-side comparison of key metrics across multiple teams over the selected period.

**Best for:** Multi-team operations management, identifying structural performance differences between teams.

---

#### Digital Channel Traffic Analysis

Volume and trend analysis across digital channels — showing how interaction volumes are distributed across WhatsApp, email, webchat, social, and other channels over time.

**Best for:** Channel strategy decisions, resource allocation across channel types, trend spotting.

---

#### 3. Quality & Evaluation Reports

These reports are used by quality managers and supervisors to understand QA outcomes, evaluator activity, and agent quality trends over time.

#### Quality Assurance Reports

Aggregate and individual scores by agent, team, and evaluation form. The primary report for quality managers reviewing programme outcomes.

**Best for:** Monthly and quarterly quality reviews, identifying coaching targets, demonstrating quality improvement to stakeholders.

---

#### Team Comparison (Quality)

Visual and tabular comparison of team-level evaluation scores across a selected time range, with filters by team, evaluation form, section, and individual question.

**Best for:** Identifying which teams or form sections are consistently scoring low, directing coaching investment.

---

#### Agent & Team Leaderboard (Quality)

Ranked performance data based on evaluation results — showing number of reviews completed, average scores, and score variance per agent and team.

**Best for:** Performance management, recognising high-quality agents, spotting consistency issues.

---

#### Evaluator Comparison

Compares how multiple evaluators scored the same agent using the same evaluation form — the primary tool for calibration sessions.

**Best for:** Identifying evaluator drift, running calibration sessions, ensuring consistent scoring across the QA team.

---

#### Evaluation Volume

Counts of evaluations by status (planned, in progress, completed) shown in both tabular and graphical views.

**Best for:** Monitoring QA programme throughput, identifying bottlenecks in the evaluation workflow.

---

#### Skills Assessment

Agent or team performance based on evaluation form scores, broken down by skill or competency area, with graphical and tabular output.

**Best for:** Targeted training programme design, skill-specific coaching prioritisation.

---

#### 4. Outbound & Campaign Reports

These reports cover the performance, efficiency, and effectiveness of outbound dialling campaigns.

#### Campaign Summary

Campaign contact status grouped by call result — connected, no answer, busy, failed, Do Not Contact, and other outcomes.

**Best for:** Campaign health assessment, comparing contact rates across campaigns.

---

#### Dialing & Success Rate Summary

Dial rate and success rate per campaign — showing how efficiently the dialler is connecting to customers relative to total dial attempts.

**Best for:** Campaign efficiency monitoring, identifying campaigns with poor contact rates.

---

#### Connected Calls Detail

Details of successfully connected outbound calls — agent, customer, duration, disposition, and outcome. Excludes abandoned calls.

**Best for:** Reviewing agent performance on outbound, compliance reviews of connected call handling.

---

#### Campaign Calls Detail Report

All calls attempted by the dialler that did not connect — including agent-based and IVR-based attempts — with outcome codes.

**Best for:** Understanding why calls are not connecting, diagnosing dialler configuration issues.

---

#### 5. IVR Reports

These reports cover customer interactions with the IVR (Interactive Voice Response) system before reaching an agent.

#### IVR Summary Report

Aggregated summary of IVR activity per day — total calls entering IVR, self-service completions, transfers to agent, abandonments within IVR, and average IVR journey duration.

**Best for:** Measuring IVR self-service containment rate, daily volume trending.

---

#### IVR Detail Report

Granular view of each individual IVR journey — showing the path each customer took through the IVR menu tree, where they exited, and what happened next.

**Best for:** IVR design optimisation, identifying menu options with high drop-off, compliance review of IVR flows.

---

#### 6. WebRTC Reports

These reports cover browser-based voice and video sessions initiated via WebRTC links.

#### WebRTC Summary Report

Summary of WebRTC-generated links and sessions — number of links generated, sessions completed, and session outcomes over the selected period.

**Best for:** Measuring adoption and usage of WebRTC-based calling, capacity planning.

---

#### WebRTC Detail Report

Granular view of each individual WebRTC link and session — customer, agent, duration, link generation time, session start and end, and quality indicators where available.

**Best for:** Troubleshooting specific WebRTC session issues, compliance and audit reviews.

---

#### 7. Survey Reports

These reports cover customer feedback and satisfaction data collected through CelestaCX's survey capability.

#### Agents' Summary Report

Agent performance based on customer survey ratings — average CSAT score per agent, number of surveys received, and score distribution.

**Best for:** Customer satisfaction measurement at the agent level, identifying agents who consistently receive low or high satisfaction ratings.

---

#### Questions' Summary Report

Overview of customer responses to each individual survey question across all interactions in the selected period — showing response distribution and average scores per question.

**Best for:** Identifying which aspects of the customer experience (communication, resolution speed, professionalism) score lowest, informing training design.

---

#### 8. Common Filters & Settings

All historical reports in Superset support a standard set of filters. Apply these before reading any report to ensure the data is scoped correctly.

| Filter | What It Does |
| --- | --- |
| Date / Time Range | Scope the report to a specific period — yesterday, last 7 days, last 30 days, or a custom range |
| Agent Name | Filter to one or more specific agents |
| Team | Filter to a specific team |
| Queue Name | Filter to one or more queues |
| Channel Type | Filter by interaction channel (voice, chat, email, WhatsApp, etc.) |
| UTC Offset | Adjust report timestamps to match your local timezone |

#### Interval Settings

Some reports (particularly Agent Performance and Interval Performance) allow you to adjust the interval granularity:

| Interval | Best For |
| --- | --- |
| 15 minutes | Intraday analysis, peak period identification |
| 30 minutes | Shift-level analysis |
| Hourly | Daily trend analysis |
| Daily | Weekly and monthly trend reporting |

#### Scheduled Report Delivery

Superset supports scheduled delivery of reports by email. Ask your administrator to configure report schedules and alert thresholds. See the [Unified Admin Reference → Reporting & Analytics](#) for configuration steps.

---

#### What's Next

- [**Key Reporting Metrics**](#) — precise definitions of every metric referenced in these reports
- [**Reporting Database Schema**](#) — the underlying data model for teams building custom reports or integrating with external BI tools
- [**Real-Time Dashboards**](#) — for live shift monitoring that complements this historical data
