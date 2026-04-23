# Channels & Routing Configuration

Routing is the engine of CelestaCX. It determines how every incoming interaction — whether a phone call, WhatsApp message, email, or webchat — is received, classified, and delivered to the right queue and agent. This page covers the core routing model, how to configure routing rules, and the settings that apply across all channels.

Channel-specific setup (WhatsApp, Voice, Email, etc.) is covered in the [Channels & Integrations](#) section. This page focuses on the routing layer that sits above individual channels.

---

### How Routing Works in CelestaCX

When an interaction arrives, CelestaCX processes it through a sequence of steps before it reaches an agent:

```
Incoming Interaction
        │
        ▼
  Channel Connector
  (receives & normalises the interaction)
        │
        ▼
  Routing Engine
  (evaluates rules: skills, priority, availability)
        │
        ▼
  Queue
  (holds the interaction until an agent is available)
        │
        ▼
  Agent Desk
  (agent accepts and handles the interaction)
```

The routing engine evaluates rules in real time based on interaction attributes (channel, language, topic, customer data), agent availability, and queue state.

---

### Key Routing Concepts

| Concept | Description |
| --- | --- |
| Channel Connector | The integration point that receives interactions from an external channel (e.g. WhatsApp, telephony switch, email server) and passes them into the routing engine. |
| Routing Rule | A logical condition that determines which queue an interaction is sent to, based on attributes such as channel type, DNIS, language, or custom tags. |
| Queue | A holding area for interactions waiting to be handled. Each queue is served by one or more agent teams. See Agent Teams & Queues for queue configuration. |
| Skill | A tag assigned to agents and routing rules to enable skill-based routing — ensuring interactions requiring specific expertise reach appropriately skilled agents. |
| Priority | A numeric value assigned to an interaction or queue that determines its position relative to others. Higher-priority interactions are offered to agents first. |
| Overflow | A fallback action triggered when a queue exceeds a defined wait threshold — for example, redirecting to a different queue, playing a callback offer, or sending an automated message. |
| Business Calendar | Routing rules can reference a business calendar to apply different logic during open hours, closed hours, and public holidays. See Business Calendars . |

---

### Routing Rule Configuration

Routing rules are configured under **Administration → Routing → Routing Rules**.

#### Rule Structure

Each routing rule consists of:

1. **Trigger** — the condition that activates the rule (e.g. interaction arrives on a specific channel, DNIS, or with a specific attribute value)
2. **Conditions** — optional additional filters (e.g. language = English, customer segment = VIP)
3. **Action** — what happens when the rule matches (e.g. send to Queue A, apply priority level 2, attach a skill requirement)

Rules are evaluated in priority order. The first matching rule wins. If no rule matches, the interaction is sent to the configured default queue.

#### Creating a Routing Rule

1. Navigate to **Administration → Routing → Routing Rules → Add Rule**.
2. Give the rule a descriptive name (e.g. "VIP Customers — English Voice").
3. Define the trigger: select the channel and any channel-specific identifier (e.g. DNIS number, WhatsApp number, email address).
4. Add conditions using the condition builder. Conditions can reference:

  - Interaction attributes (channel, language, topic)
  - Customer data fields (if CRM integration is active)
  - Time/date (using a linked Business Calendar)
  - Custom tags set by a bot or IVR
5. Set the action: select the target queue, assign a priority level (1–10), and optionally require one or more skills.
6. Set the rule status to **Active** and save.
7. Use the **drag handle** on the rules list to adjust evaluation order.

#### Priority Levels

| Priority Level | Typical Use |
| --- | --- |
| 1 | Standard interactions |
| 2–4 | Escalations or callbacks |
| 5–7 | VIP or SLA-sensitive customers |
| 8–10 | Emergency or compliance-critical interactions |

---

### Skill-Based Routing

Skill-based routing ensures that interactions requiring specific agent capabilities — language, product knowledge, technical tier — are only offered to agents who hold the relevant skill.

#### Setting Up Skills

1. Navigate to **Administration → Routing → Skills → Add Skill**.
2. Create skills that reflect your operational needs (e.g. "Spanish", "Billing", "Technical Tier 2").
3. Assign skills to agents via **Administration → Users → [User] → Skills**.
4. Reference skills in routing rules using the **Required Skills** field.

An interaction with a required skill will only be offered to agents who hold that skill. If no skilled agent is available, the interaction waits in queue until one becomes available, or until an overflow threshold is reached.

#### Skill Proficiency Levels

Each skill assignment includes a proficiency level (1–5). When multiple agents hold the required skill, the routing engine prefers agents with higher proficiency, subject to availability.

---

### Overflow & Fallback Handling

Overflow rules define what happens when interactions wait too long or a queue becomes unavailable. Overflow is configured per queue under **Administration → Queues → [Queue] → Overflow Settings**.

| Overflow Trigger | Available Actions |
| --- | --- |
| Wait time exceeds threshold | Transfer to another queue |
| Wait time exceeds threshold | Offer callback (voice) |
| Wait time exceeds threshold | Send automated message (digital channels) |
| Queue is closed (outside calendar hours) | Play closed message / send out-of-hours reply |
| No agents available | Transfer to voicemail (voice) or bot (digital) |
| All agents busy for X minutes | Escalate priority of waiting interactions |

Overflow rules are evaluated every 30 seconds against live queue state.

---

### Out-of-Hours Routing

CelestaCX can apply different routing behaviour outside of business hours by linking routing rules to a **Business Calendar**. When the calendar indicates the queue is closed:

- Voice interactions can be routed to a voicemail or an out-of-hours IVR flow.
- Digital interactions can receive an automated reply informing the customer of operating hours.
- Interactions can be queued and held for the next available business period.

Out-of-hours calendar linking is configured per routing rule under the **Time Condition** section of the rule builder. See [Business Calendars](#) for how to set up calendars.

---

### Interaction Attributes & Tagging

The routing engine can make decisions based on custom attributes attached to an interaction. These attributes can be set by:

- A **bot or virtual agent** that pre-qualifies the customer before handoff
- An **IVR flow** that collects input from the caller
- A **CRM connector** that enriches the interaction with customer data on arrival
- Manually by an agent (for rerouting or transfer purposes)

Custom attributes are defined under **Administration → Routing → Interaction Attributes**. Once defined, they are available as conditions in routing rule builders and visible to agents on the Agent Desk interaction panel.

---

### Viewing Active Routing Configuration

To get an overview of all active routing rules and their current queue targets, navigate to **Administration → Routing → Overview**. This view shows:

- All active rules in evaluation order
- The queue each rule routes to
- Whether each queue is currently open (based on its linked calendar)
- A count of interactions currently in each queue

This overview is read-only and does not require Supervisor or Admin access to view — it is accessible to all admin roles.

---

*What's next: With routing rules in place, configure the queues and teams that will receive that traffic in*[*Agent Teams & Queues*](#)*.*
