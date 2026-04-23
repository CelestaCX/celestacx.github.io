# Agent Guide

#### Overview

This guide is written for contact centre agents — the people who handle conversations with customers every day. It covers everything you need to use CelestaCX Agent Desk effectively: logging in, managing your availability, handling conversations across voice and digital channels, collaborating with colleagues, and wrapping up interactions correctly.

If you are new to CelestaCX, start with the [Getting Started](#) section below and work through it in order before exploring the rest of the guide.

---

#### 1. Getting Started

Before you can receive any conversations, two things need to be in place: you must be logged in, and you must be in the **Ready** state. CelestaCX will not route anything to you until both conditions are met.

**To log in and become available:**

1. Open Agent Desk in your browser and sign in with your credentials.
2. After login, your global state will automatically be set to **Not Ready**. No conversations will be routed to you yet.
3. Use the state selector at the top of the Agent Desk to switch to **Ready**.
4. Enable the channel toggles for each channel type you handle (e.g., Chat, Voice, Email). Each toggle turns green when active.

You are now available to receive routed conversations.

> **Browser requirements:** CelestaCX Agent Desk runs in modern browsers. Enable browser notifications when prompted — these are used to alert you to incoming conversations when you are on another tab.

---

#### 2. Managing Your Availability

Your availability is controlled through a two-level state system: a **Global State** that applies across all channels, and individual **Channel Category States** that let you fine-tune availability per channel type.

#### Global States

| State | What It Means |
| --- | --- |
| Not Ready | Default after login. No conversations are routed to you. Select a reason code if prompted. |
| Ready | You are available. Conversations may be routed depending on your channel toggles. |
| Logout | You are signing out. Active conversations are re-routed before the logout completes. |
| Unknown | Rare failover state. Recover by switching to Ready or Not Ready. |

A **state timer** on the Agent Desk shows how long you have been in your current global state. It resets every time your global state changes.

#### Channel Category States

Within each channel (Chat, Voice, Email), you will see a toggle that controls availability for that specific channel type independently of your global state.

| Toggle Colour | State | Meaning |
| --- | --- | --- |
| Grey (off) | Not Ready | Not receiving conversations on this channel |
| Green (on) | Ready | Available to receive conversations on this channel |
| Orange (on) | Active | Currently handling one or more conversations on this channel |
| Red (on) | Busy | At maximum task limit — no new conversations routed |
| Purple (off) | Pending Not Ready | Waiting for active tasks to clear before going off |

> **Push vs. Pull mode:** The state system applies to Push mode routing, where conversations are assigned to you automatically. In Pull mode, you can pick up conversations from a queue list regardless of state, as long as you are logged in.

#### Not Ready Reason Codes

When switching from Ready to Not Ready, you may be prompted to select a reason code (e.g., Break, Training, Meeting). These are configured by your administrator and are used for adherence reporting. Always select the most accurate reason — your supervisor can see these in real time.

---

#### 3. Receiving & Accepting Conversations

When a new conversation is routed to you, a notification appears on your Agent Desk.

- **Green customer name** — the customer has been recognised (matched by phone number, email, or other identifier)
- **Red / "Anonymous"** — no matching customer record found; you may need to update the profile after the conversation

Click **Accept** to join the conversation. Once accepted, the main conversation view expands to show:

- **Conversation history** — all past messages, calls, and wrap-up notes for this customer
- **Customer profile** (right sidebar) — contact details and any linked records
- **Active sessions** — other channels the customer currently has open

#### Customer Profile Matching

CelestaCX automatically tries to match the customer by their channel identifier (phone number, email address, etc.).

| Scenario | What to Do |
| --- | --- |
| Single match found | Start responding — customer context is loaded automatically |
| Multiple matches found | Use the Link Profile tab in the sidebar to select the correct record |
| No match found | Update the anonymous profile with the correct customer details after the conversation |

> **Tip:** Always scroll through conversation history before typing your first response. Use **Load More** to see older messages and **Jump to Bottom** to return to the latest one quickly.

---

#### 4. Handling Conversations

#### Responding to Messages

Type your response in the message input area and press Enter or click Send. Depending on the channel, you may be able to send attachments, images, and quick reply options.

#### Pausing a Conversation

If you need to step away from a conversation temporarily without closing it, you can pause it. The conversation is held in a paused state and will not be re-routed while paused. Note that pause/resume behaviour varies slightly by channel — consult your administrator if you are unsure whether it is enabled for your channels.

#### Understanding Response SLA

CelestaCX tracks how quickly you respond to customers. A **response SLA timer** is visible in each active conversation, indicating how long the customer has been waiting for your reply. When the timer turns red, you are approaching or have exceeded the target response time for that channel.

Your supervisor can see SLA status for all your active conversations in their real-time dashboard.

---

#### 5. Consult, Transfer & Conference

When you need help with a conversation or need to move it to a different team, CelestaCX gives you three options:

#### Consult

Bring in a second agent as an **Assistant** without the customer knowing. The assistant can see the conversation and advise you privately. To consult:

1. Click **Agent Assistance** on the conversation toolbar.
2. Select a queue (you will see how many agents are available in each queue) or expand a queue to pick a specific agent.
3. Click **Consult**. The agent joins your conversation as an Assistant once they accept.

You can also consult on an external phone number using the Dialpad in the same menu.

#### Transfer

Move the conversation to a different queue or agent. You are removed from the conversation once the transfer completes.

| Transfer Type | How |
| --- | --- |
| To a queue | Agent Assistance → select queue → Transfer |
| To a specific agent | Agent Assistance → expand queue → hover agent → Transfer |
| To an external number | Agent Assistance → Dialpad → enter number → Transfer |

The customer sees a message telling them they are being transferred and which queue they are going to.

#### Consult Transfer & Conference

Once a consult is active, you have two further options:

- **Consult Transfer** — hand the conversation fully to the assistant. You leave, they become the primary agent.
- **Consult Conference** — bring the assistant in alongside you so both of you can interact with the customer simultaneously. Up to 4 agents can participate in a conference at once.

---

#### 6. Getting Help from Your Supervisor

If you need assistance during a conversation, you can raise a **hand raise** signal visible to your supervisor on their dashboard. Your supervisor can then:

- **Whisper** — send you a private message the customer cannot see
- **Barge in** — join the conversation directly to assist

To raise your hand: click the **Hand Raise** button on the conversation toolbar. Lower it once your issue is resolved.

---

#### 7. Wrap-up & Post-Conversation Work

Properly completing wrap-up is important — it feeds into reporting, quality reviews, and customer history. Do not skip it.

#### What Wrap-up Involves

After a conversation ends, you will be prompted to complete wrap-up before becoming available for new conversations (especially on voice channels, where your state moves to **Wrap-up** automatically).

1. Click the **Notes** icon on the control toolbar.
2. Select a **Category** (e.g., Billing, Technical Support).
3. Select a **Reason** (e.g., Issue Resolved, Escalated).
4. You can apply up to **5 wrap-up codes** per conversation.
5. Add any relevant notes in the **Notes** field — these are internal only, not visible to the customer, and will appear in the customer's history for future agents.

#### The Wrap-up Timer

Your administrator may set a wrap-up timer (commonly 60 seconds). When the timer expires, the conversation is closed regardless of whether wrap-up was completed. If this happens, the interaction is marked as **Closed without Wrap-up** in reports.

> **Tip:** Be consistent with your wrap-up code selection. Inconsistent codes make your supervisor's reporting less accurate and may affect team performance metrics.

---

#### 8. AI Co-Pilot

CelestaCX includes an AI Co-Pilot that works alongside you during conversations. It can surface relevant knowledge base articles, suggest responses, summarise long conversation histories, and flag sentiment changes in real time.

Co-Pilot suggestions appear in a panel on the right side of the Agent Desk. You can accept, edit, or dismiss any suggestion — Co-Pilot never sends anything on your behalf without your confirmation.

Your administrator controls which Co-Pilot features are enabled for your team.

---

#### Quick Reference: Agent Desk Controls

| Control | What It Does |
| --- | --- |
| State selector | Switch between Ready, Not Ready, and Logout |
| Channel toggles | Enable / disable availability per channel type |
| Accept button | Accept an incoming conversation |
| Agent Assistance | Open consult, transfer, or conference options |
| Hand Raise | Signal to your supervisor that you need help |
| Notes icon | Open wrap-up codes and notes panel |
| Pause / Resume | Pause an active conversation temporarily |
| Load More | Load older conversation history |
| Jump to Bottom | Return to the latest message in a long conversation |

---

#### What's Next

Once you are comfortable with day-to-day conversation handling, explore these related areas:

- [**Supervisor Guide**](#) — understand what your supervisor can see and do
- [**Reports & Analytics**](#) — how your performance metrics are calculated
- [**Channels & Integrations**](#) — channel-specific behaviour for WhatsApp, Email, Voice, and more
