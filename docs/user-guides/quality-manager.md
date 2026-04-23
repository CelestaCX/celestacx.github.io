# Quality Manager Guide

#### Overview

This guide is written for Quality Managers — the people responsible for defining what good service looks like, building the frameworks that measure it, and driving continuous improvement across agent teams.

In CelestaCX, the Quality Manager sits at the centre of the quality management (QM) module. You design the evaluation forms that set the standard, schedule and assign reviews, monitor progress, analyse results, and use AI-driven tooling to scale your QA coverage far beyond what manual evaluation alone can achieve.

The [Evaluator Guide](#) covers the parallel workflow for the evaluators who do the day-to-day conversation scoring — read both guides if you perform both roles.

---

#### 1. What Quality Managers Can Do

| Capability | Description |
| --- | --- |
| Design evaluation forms | Build structured scoring forms with weighted sections and questions that define your quality standard |
| Schedule bulk reviews | Assign batches of conversations to evaluators based on agent, team, channel, and date filters — one-time or recurring |
| Assign individual reviews | Manually assign a specific conversation to a specific evaluator for ad-hoc review |
| Review conversations directly | Open and review any conversation yourself without going through the evaluator workflow |
| Monitor evaluation progress | Track the status of all evaluation jobs — Pending, In Progress, Completed — across your team |
| Analyse QA reports | View scores by agent, team, form, and time period to identify trends and coaching opportunities |
| Configure AI-driven QM | Enable and configure automated AI scoring to achieve 100% interaction coverage |
| Override AI scores | Review AI-generated evaluations and manually override them where needed |

---

#### 2. Setting Up: QM Configuration

Before evaluations can begin, two things need to be configured. Your administrator handles the system-level setup; you handle the evaluation framework itself.

#### Administrator Prerequisites

Your administrator must enable the Quality Management module and assign the **Quality Manager** role to your user account via the IAM system. Without this role, the QM interface will not be accessible.

The administrator also configures:

- Which agents and teams are in scope for QM
- Whether AI-driven evaluation is enabled and in which mode
- Access permissions for evaluators

Contact your administrator if any of these are not in place.

#### QM Module Configuration (Quality Manager)

Once access is confirmed, the first configuration task is navigating to **Quality Management → QM Admin** to verify the module settings for your deployment — evaluator assignments, team scope, and AI integration settings if applicable.

---

#### 3. Designing Evaluation Forms

Evaluation forms are the foundation of your quality framework. They translate your definition of "good service" into a structured, objective scoring instrument that evaluators (and the AI engine) use to assess every interaction.

#### Form Structure

Each evaluation form is organised into sections, and each section contains questions. Every element carries a weight that contributes to the overall score.

| Level | What It Defines | Must Sum To |
| --- | --- | --- |
| Form | The overall evaluation | 100% across all sections |
| Section | A theme or business priority (e.g. Communication, Compliance, Problem Solving) | 100% across all questions in that section |
| Question | A specific assessment point within a section | Contributes to section % |

#### Question Types

| Type | Affects Score? | Best Used For |
| --- | --- | --- |
| Weighted Single Select | ✅ Yes | Pass/fail checks, scaled ratings (e.g. 1–5, Yes/No) |
| Non-weighted Dropdown | ❌ No | Categorising issues without affecting the score |
| Text Input | ❌ No | Qualitative coaching notes, mandatory comments |

Only **Weighted Single Select** questions contribute to the mathematical score. All other question types are for context and coaching.

#### Building a Form

1. Navigate to **Quality Management → Evaluation Forms**.
2. Click **New Form** and give it a descriptive name.
3. Add sections that reflect your business priorities. A typical structure might be:

  - **Communication** (30%) — tone, clarity, empathy
  - **Compliance** (40%) — script adherence, verification steps, regulatory requirements
  - **Problem Solving** (30%) — accuracy of information, resolution speed, follow-through
4. Add questions within each section and assign percentage weights. Questions within a section must sum to 100% of that section's weight.
5. The Form Builder highlights any validation errors automatically — fix these before saving.
6. Click **Save** to publish the form and make it available for scheduling.

#### Best Practices

- **Keep forms focused.** A large number of questions slows evaluators down and reduces consistency. Aim for depth in a smaller number of meaningful criteria rather than breadth across many.
- **Weight critical failures heavily.** A compliance breach (e.g. failing a security verification) should carry significantly more weight than a minor communication issue.
- **Calibrate regularly.** Periodically review completed evaluations with your evaluators to ensure everyone is interpreting the rubric criteria the same way. Calibration sessions prevent evaluator drift over time.
- **Version your forms.** When you make significant changes to a form, create a new version rather than editing the existing one. This preserves historical score integrity.

---

#### 4. Scheduling & Assigning Evaluations

#### Scheduling Bulk Reviews

The Review Scheduler lets you assign evaluation work to evaluators in bulk, with filters that target specific conversations.

1. Navigate to **Quality Management → Review Scheduler**.
2. Set your filters:

| Filter | Options |
| --- | --- |
| Agent | Specific agent or all agents |
| Team | Specific team or all teams |
| Evaluator | The person who will complete the reviews |
| Channel | Voice, chat, email, or all |
| Date range | The period of conversations to include |
| Call properties | Duration, outcome, wrap-up code, or other attributes |

1. Choose the schedule type:

  - **One-time** — a single batch of reviews with a completion deadline
  - **Recurring** — automatically generates new batches on a defined cadence (daily, weekly, monthly)
2. Select the evaluation form to be used.
3. Confirm and save. Assigned conversations appear in the evaluator's work queue.

#### Assigning Individual Reviews

For ad-hoc or targeted reviews — a specific complaint, an escalation, a coaching follow-up — you can assign individual conversations directly.

1. Navigate to **Quality Management → Conversation List**.
2. Find the conversation using search and filters.
3. Select the conversation and click **Assign to Evaluator**.
4. Choose the evaluator and confirm.

#### Reviewing Conversations Directly

As a Quality Manager you are not restricted to the evaluator workflow. You can open and review any conversation in the system yourself at any time.

1. Navigate to **Quality Management → Conversation View**.
2. Open the target conversation.
3. Review the full interaction content — transcript, recordings, wrap-up notes.
4. Optionally assign it to an evaluator from this view for formal scoring.

---

#### 5. Monitoring Evaluation Progress

The Review Screen gives you a live view of all evaluation activity across your team.

Navigate to **Quality Management → Review Screen** to see:

| Status | Meaning |
| --- | --- |
| Pending | Assigned but not yet started by the evaluator |
| In Progress | Evaluator has opened and begun scoring the conversation |
| Completed | Evaluation submitted and score recorded |

Use this view to identify bottlenecks — evaluators with large pending queues, deadlines at risk, or evaluations that have stalled. You can reassign evaluations from this screen if needed.

---

#### 6. AI-Driven Quality Management

Manual evaluation can realistically cover only a small percentage of all interactions. CelestaCX's AI-driven QM module addresses this by automatically scoring interactions at scale using the same evaluation forms your human evaluators use.

#### How It Works

The AI engine processes closed interaction data — transcripts, metadata, and conversation context — and applies your configured evaluation form to produce a score. Results are available immediately after a conversation closes, without any evaluator involvement.

#### Operating Modes

| Mode | Description | Best For |
| --- | --- | --- |
| Full Automation | AI evaluates 100% of closed interactions | High-volume teams where manual coverage is impractical |
| Hybrid | A configured percentage goes to AI; the remainder to human evaluators | Teams that want AI coverage with human oversight for complex cases |

#### Benefits Over Manual-Only QA

- **Full coverage** — every interaction is scored, not just a random sample
- **Consistency** — the same criteria are applied identically to every interaction, removing evaluator variability
- **Speed** — scores are available the moment a conversation closes, not days later
- **Efficiency** — your evaluators focus on exception handling, coaching, and calibration rather than routine scoring

#### Language Support

The AI evaluation engine currently supports **English** and **Arabic** (Egyptian dialect).

#### Working with AI Scores

AI-generated evaluations appear in the same QM interface as human evaluations. As Quality Manager you can:

- Review any AI-scored evaluation
- Override the AI score where you disagree with its assessment
- Compare AI scores with human evaluator scores to calibrate the engine's accuracy over time
- Use AI-flagged interactions to identify systemic issues or coaching opportunities that would otherwise go undetected in a sampled approach

---

#### 7. Analysing Quality Reports

The QM reporting suite gives you visibility into quality performance across agents, teams, forms, and time periods.

| Report | What It Shows |
| --- | --- |
| Quality Assurance Reports | Aggregate and individual scores by agent, team, and evaluation form |
| Evaluator Activity | How many evaluations each evaluator has completed, with scores and time taken |
| Score Trends | How quality scores are moving over time — improving, declining, or stable |
| Form Performance | Which questions or sections agents consistently score low on — signals coaching priorities |

Use these reports to:

- Identify agents who need targeted coaching
- Spot systemic training gaps affecting whole teams
- Demonstrate quality improvement trends to senior stakeholders
- Validate whether changes to processes or training are having the intended effect

---

#### Quick Reference: Quality Manager Workflow

```
1. Configure
   └── QM Admin settings → assign evaluators → enable AI (if applicable)

2. Design
   └── Build evaluation forms → set section and question weights → validate → publish

3. Schedule
   └── Review Scheduler → set filters → assign to evaluators → set deadlines

4. Monitor
   └── Review Screen → track Pending / In Progress / Completed → reassign if needed

5. Analyse
   └── QA Reports → scores by agent / team / form → identify coaching needs

6. Improve
   └── Calibration sessions → form updates → coaching → track impact in reports
```

---

#### What's Next

- [**Evaluator Guide**](#) — the day-to-day conversation scoring workflow from the evaluator's perspective
- [**Reports & Analytics**](#) — full reference for all QA report types and metrics
- [**Supervisor Guide**](#) — how supervisors interact with quality management data in their dashboards

---
