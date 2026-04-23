# Supervisor Guide

#### Overview

This guide is written for contact centre supervisors — the people responsible for keeping their team running smoothly, maintaining service levels, and supporting agents through their shift. It covers the full range of supervisor capabilities in CelestaCX: managing your team, monitoring conversations in real time, intervening when needed, tracking SLA, and analysing performance through historical reports.

Supervisors in CelestaCX have access to everything agents do, plus a dedicated set of monitoring, management, and reporting tools that are not available to standard agent accounts.

If you are also responsible for quality evaluation, see the [Quality Manager Guide](#) for the QM-specific workflows that sit alongside this guide.

---

#### 1. What Supervisors Can Do

| Capability | Description |
| --- | --- |
| Real-time dashboards | Monitor queue health, agent states, and live conversations across the team |
| Silent monitoring | Join an active conversation as an invisible observer — agent and customer are unaware |
| Barge-in | Enter a live conversation to assist the agent or speak directly to the customer |
| Whisper | Send a private message to an agent mid-conversation, visible only to them |
| Respond to hand raises | Receive and act on help requests raised by agents during conversations |
| Team management | Add agents and supervisors to teams, adjust membership, manage assignments |
| Force logout | Remotely log out any agent regardless of their current state |
| Team announcements | Broadcast messages visible to all agents on the team |
| Customer labels | Create labels to categorise customers (e.g. VIP, Language, Tier) — labels can also drive priority routing |
| Historical reports | Access full reporting suite: agent performance, queue productivity, adherence, SLA, and more |
| Workforce management | Use the WFM module for scheduling, forecasting, and shift management |
| Limited admin access | Perform certain configuration tasks in Unified Admin (see Supervisor Access to Unified Admin ) |

---

#### 2. Real-Time Monitoring

The Supervisor interface includes several dashboards that update live throughout the shift. These are your primary tool for understanding what is happening across your team at any moment.

#### Summary Dashboard

The top-level view. Shows queue health at a glance — conversations waiting, agents available, SLA status, and key counters for the shift. Use this as your default view during normal operations.

#### Team Stats Dashboard

Breaks down performance by team. Shows per-agent state, conversation count, and handle time side by side. Use this to spot imbalances — agents who are overloaded or agents who have gone idle.

#### Realtime Detailed Dashboards

Drill-down views into active conversations, individual agent states, and live queue depths. From here you can see exactly what each agent is handling and take action on specific conversations.

#### Ongoing Conversations Detail

Lists every active conversation with the handling agent, channel, customer identity, and duration. This is the entry point for silent monitoring and barge-in.

> **Team structure matters:** The dashboards show data scoped to your assigned team. If an agent is not a member of your team, they will not appear in your dashboards. See [Impact of Team Structure on Dashboards](#) for details.

---

#### 3. Monitoring & Intervening in Live Conversations

#### Silent Monitoring

Silent Monitoring lets you join any active conversation as an invisible observer. Neither the agent nor the customer knows you are present.

**To silently monitor a conversation:**

1. Open the **Ongoing Conversations Detail** dashboard.
2. Locate the conversation you want to observe.
3. Click the **Monitor** (eye) icon next to it.
4. You are now in the conversation view, reading the live interaction in real time.

While monitoring, you can:

- Read the full conversation history
- Send a **Whisper** message to the agent (chat only) — the customer cannot see this
- Escalate to **Barge-in** at any time

**Key limitations to be aware of:**

- Only one supervisor can silently monitor a given conversation at a time
- Whisper is available for chat sessions only, not voice
- If the primary agent leaves the conversation, you are automatically removed as a monitor
- Silent Monitoring is not available during outbound, named agent transfer, or consult transfer scenarios

#### Barge-in

When you need to step in actively — to correct a situation, assist a struggling agent, or address the customer directly — use Barge-in. You become a full participant in the conversation and can send messages the customer can see.

To barge in: while in Silent Monitoring mode, click the **Barge-in** button in the Conversation View.

#### Responding to Agent Hand Raises

When an agent clicks the Hand Raise button, a notification appears in your supervisor interface. From there you can:

- Open the agent's conversation to review context
- Send a Whisper message with guidance
- Barge in if the situation requires direct involvement

Acknowledge the hand raise once you have responded so the agent knows their signal was received.

#### Force Logout

If an agent is unreachable, stuck in an incorrect state, or needs to be removed from the queue immediately, you can force log them out remotely. This works regardless of their current state — including while they are active in conversations. Active conversations are re-routed before the logout completes.

**To force logout an agent:** Navigate to the **Available Agents Detail** dashboard, locate the agent, and select Force Logout.

---

#### 4. Team Management

#### Managing Team Membership

Agents and supervisors must be assigned to a team before they appear in your dashboards or can receive routed conversations. Team membership is managed through the Unified Admin.

**To add a member to your team:**

1. Open Unified Admin and navigate to **Teams**.
2. Select your team and click **Add Member**.
3. Search for the agent or supervisor by name and confirm.

Changes take effect immediately. Note that if you make role changes in the IAM system (Keycloak), the affected user will need to re-login for changes to take effect.

#### Team Announcements

Use team announcements to broadcast information to all agents on your team — shift updates, process reminders, system notices, or anything your team needs to know. Announcements are visible in the agent's Agent Desk interface.

**To create an announcement:** Go to the Team Announcements section in your supervisor interface, click **New Announcement**, enter your message, and publish.

---

#### 5. SLA Management

CelestaCX tracks Service Level (SL) metrics per queue. As a supervisor, understanding how SLA is calculated helps you interpret your dashboards accurately and take the right actions when targets are at risk.

#### How SLA Is Calculated

SLA is measured against a configurable threshold (set per queue by your administrator). For each conversation that enters the queue, the platform records whether it was answered, abandoned, or timed out relative to that threshold.

| Scenario | Counts Toward SL? |
| --- | --- |
| Offered and answered within threshold | ✅ Yes — ideal outcome |
| Customer abandons before threshold expires | ✅ Counted as abandoned within SL window |
| Answered after threshold | ❌ No — answered but outside service level |
| Customer abandons after threshold | ❌ No — not counted in SL metrics |
| Agent assigned but does not accept (RONA) within threshold | ✅ Counted as SL RONA event |
| No agents available | ❌ Tracked separately — does not affect SL metrics |

The key SLA metrics you will see in dashboards and reports are:

| Metric | What It Means |
| --- | --- |
| chats_offered | Total conversations that entered the queue |
| chats_answered | Conversations accepted by an agent |
| chats_abandoned | Conversations where the customer left before connecting |
| chats_rona | Conversations where the assigned agent did not accept |
| service_level_answered | Conversations answered within the SL threshold |
| NoAgentAvailable | Conversations where no agent was available at all |

> **Customer Inactivity SLA:** Separately from queue SLA, CelestaCX also tracks customer-side inactivity. If a customer stops responding for a configured period, the conversation may be automatically closed. This is configured by your administrator per channel.

---

#### 6. Historical Reporting

CelestaCX provides a comprehensive reporting suite accessible to supervisors. Reports cover agent performance, queue productivity, SLA, adherence, campaign results, and more.

#### Key Reports for Supervisors

| Report | What It Shows |
| --- | --- |
| Agent Performance Dashboard | Per-agent handle time, response time, conversations handled, and SLA compliance |
| Agent Interaction Log | Searchable log of every interaction with timestamps, channel, outcome, and wrap-up codes |
| Presence & Availability Report | How long each agent spent in each state (Ready, Not Ready, Away) during the shift |
| Agent Adherence Report | Scheduled vs. actual login and availability times |
| Agent & Team Leaderboard | Ranked comparison of agents and teams by key metrics |
| Team Comparison Report | Side-by-side metrics across multiple teams |
| Queue Productivity Report | Wait times, abandonment rate, and throughput per queue |
| Interval Performance Report | Metrics broken into time intervals (15-min, 30-min, hourly) for trend analysis |
| Digital Channel Traffic Analysis | Volume and trends across digital channels |
| Channel Session Audit Report | Session-level audit trail for compliance and quality reviews |
| Campaign Performance Report | Outbound campaign results — contact rates, dispositions, agent activity |

> Reports are powered by the CelestaCX analytics layer. Some reports may require specific configuration by your administrator. See the [Reports & Analytics](#) section for full field-level reference documentation.

---

#### 7. Workforce Management (WFM)

CelestaCX includes a Workforce Management module for supervisors who handle scheduling and shift planning. Through WFM you can:

- Forecast contact volumes by channel and time period
- Build and publish agent schedules
- Track schedule adherence in real time
- Manage shift exceptions and time-off requests

WFM is an optional module. If it is not available in your interface, contact your administrator to confirm whether it is enabled for your deployment.

---

#### 8. Supervisor Access to Unified Admin

Supervisors have limited access to Unified Admin — the platform's configuration interface. The actions available to supervisors (as opposed to full admins) are focused on team operations rather than system-level configuration.

Typical supervisor actions in Unified Admin include:

- Adding and removing team members
- Viewing queue configurations (read-only in most deployments)
- Managing customer labels and priorities

Your administrator controls which Unified Admin sections are accessible to your supervisor account.

---

#### Quick Reference: Supervisor Tools

| Task | Where to Do It |
| --- | --- |
| See live queue health | Summary Dashboard |
| See individual agent states | Team Stats Dashboard |
| View all active conversations | Ongoing Conversations Detail |
| Silently monitor a conversation | Ongoing Conversations Detail → Monitor icon |
| Barge into a conversation | While in silent monitoring → Barge-in button |
| Respond to a hand raise | Hand Raise notification → open conversation |
| Force logout an agent | Available Agents Detail Dashboard → Force Logout |
| Send a team announcement | Team Announcements → New Announcement |
| Add an agent to your team | Unified Admin → Teams → Add Member |
| View agent performance reports | Reports & Analytics → Agent Performance |

---

#### What's Next

- [**Quality Manager Guide**](#) — if you are also responsible for quality evaluations and scoring
- [**Reports & Analytics**](#) — full reference for all report types, fields, and filters
- [**Agent Guide**](#) — understand what your agents see and experience on their side
