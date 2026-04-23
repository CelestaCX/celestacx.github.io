# Evaluator Guide

This guide is written for Evaluators — the people who listen to, read, and score completed agent interactions as part of the contact centre's quality assurance process.

Your role is focused and specific: you are given a queue of conversations to review, and you assess each one using the evaluation form your Quality Manager has designed. Your scoring feeds directly into quality reports, coaching decisions, and continuous improvement programmes — so accuracy, consistency, and thoroughness matter.

This guide covers everything you need to do your job effectively in CelestaCX: finding your work queue, reviewing conversations, scoring interactions correctly, and submitting evaluations on time.

> **Note:** The Evaluator role is assigned to your account by your administrator. If you cannot see the Quality Management section in your interface, contact your administrator to confirm your role is correctly configured.

---

#### 1. Understanding Your Role

As an Evaluator you work within a structured QA workflow that is coordinated by your Quality Manager. The key relationships to understand:

| Role | What They Do for You |
| --- | --- |
| Quality Manager | Designs evaluation forms, assigns conversations to your review queue, sets deadlines, monitors your progress, and reviews your completed evaluations |
| You (Evaluator) | Open assigned conversations, review the full interaction, score against the evaluation form criteria, add coaching notes, and submit |
| Agent | The person whose interaction you are evaluating — your scores and notes feed into their coaching and performance records |

You do not choose which conversations to evaluate — they are assigned to you either by the Quality Manager directly or through a scheduled bulk assignment. Your job is to work through your queue accurately and on time.

---

#### 2. Your Work Queue

All of your evaluation work lives in the **Reviews** tab within the Quality Management section. This is your personal task list — every conversation assigned to you appears here.

#### Reading Your Queue

| Column | What It Shows |
| --- | --- |
| Conversation | The interaction ID and basic details — agent, channel, date |
| Questionnaire | The evaluation form assigned to this review — check this before you start so you know the scoring criteria |
| Due Date | The deadline by which this evaluation must be submitted |
| Status | Pending (not yet started), In Progress (opened but not submitted), or Completed |

**Always sort by Due Date** to ensure you are prioritising correctly. Evaluations that are overdue or approaching deadline should be addressed first.

#### Filtering Your Queue

Use the status filter to toggle between views:

- **Pending** — work still to be done
- **In Progress** — evaluations you have opened but not yet submitted
- **Completed** — your submitted evaluations and their recorded scores

---

#### 3. Reviewing a Conversation

Click the **Start Review** icon on any pending item to open the evaluation workspace. The workspace is designed to show everything you need in one place — you should not need to switch between screens while evaluating.

#### What You Will See

The workspace is split into two panels displayed side by side:

**Left panel — the conversation:**

| Content Type | What Is Available |
| --- | --- |
| Chat transcript | Full text of the digital interaction — every message, in order, with timestamps |
| Voice recording | Integrated audio player for voice calls — play, pause, scrub, and adjust playback speed |
| Screen recording | Where screen capture is enabled, a video playback of the agent's screen during the call |
| Metadata sidebar | Agent name, queue, channel, interaction duration, wrap-up codes, and timestamps |

**Right panel — the evaluation form:**

The form your Quality Manager configured, displayed section by section. You work through it as you review the conversation, scoring each question as you go.

#### How to Review Effectively

Work through the conversation from start to finish before scoring, or score section by section as you go — whichever approach your Quality Manager's calibration guidelines recommend. The key principles:

- **Be objective.** Score based strictly on the rubric criteria, not on your personal impression of the agent. If the criteria say "Yes" applies when the agent verified the customer's identity, score Yes if they did it — regardless of anything else in the call.
- **Use the full conversation.** Do not score based on a single moment. Listen to or read the entire interaction before finalising any scores.
- **Check the metadata.** Wrap-up codes, call duration, and queue information can provide useful context — for example, an unusually long call may be explained by a complex multi-issue interaction rather than poor handling.

---

#### 4. Scoring the Interaction

As you work through the evaluation form, you will encounter three types of questions:

| Question Type | How to Answer | Affects Score? |
| --- | --- | --- |
| Weighted Single Select | Choose from the defined options (e.g. Yes / No, or a 1–5 scale) | ✅ Yes — this is the primary scored element |
| Non-weighted Dropdown | Select a category to classify the issue | ❌ No |
| Text Input | Write a note, observation, or coaching comment | ❌ No |

#### Adding Coaching Notes

For every question where you deduct points, add a specific note explaining your reasoning. These notes are the most valuable part of your evaluation for the agent — vague deductions without explanation do not help anyone improve. Be specific and constructive:

> ✅ Good: *"Agent did not confirm the customer's account number before discussing account details — verification step was skipped entirely."*

> ❌ Not useful: *"Compliance issue."*

Your notes are visible to the Quality Manager and may be shared with the agent as part of their coaching feedback. Write as if you are speaking directly to the agent in a coaching session.

#### How Scoring Works

The platform calculates the final score automatically when you submit. You do not need to do any manual calculation. The score is weighted based on the form structure your Quality Manager configured:

```
Final Score = Sum of (question score × question weight × section weight)
```

For example, if a section is weighted at 40% and a question within it is weighted at 50% of that section, a full-mark answer on that question contributes 20% to the total score. A zero on the same question deducts 20%.

This is why it is important to understand the form's weighting before you evaluate — some questions carry much more weight than others.

---

#### 5. Submitting Your Evaluation

Once you have answered all required questions and added notes where needed:

1. Review your answers quickly to check for anything you may have missed.
2. Click **Submit**.
3. The platform calculates and records the final score automatically.
4. The evaluation moves to your **Completed** list.
5. The Quality Manager is notified that the evaluation is done.

You cannot edit a submitted evaluation. If you submitted an evaluation in error or need to correct a score, contact your Quality Manager to have it reassigned.

---

#### 6. Staying on Top of Deadlines

The platform will notify you of new assignments and approaching deadlines through the **Notifications bell** in the top right corner of your interface.

| Notification Type | When It Appears |
| --- | --- |
| New Assignment | When your Quality Manager assigns a new batch of conversations to you |
| Deadline Reminder | When a pending evaluation is approaching its due date |

Check notifications regularly during your shift. If you have a large queue and are at risk of missing a deadline, notify your Quality Manager so they can reassign or extend as needed — do not simply let deadlines pass without communicating.

---

#### 7. Working with Voice and Screen Recordings

For voice interactions, the evaluation workspace includes an integrated audio player. Key controls:

| Control | What It Does |
| --- | --- |
| Play / Pause | Start and stop playback |
| Scrub bar | Jump to any point in the recording |
| Playback speed | Increase speed (e.g. 1.25×, 1.5×) to review calls more efficiently |
| Speaker labels | Where available, identifies which audio track is the agent and which is the customer |

For screen recordings (where enabled by your administrator), a video player appears alongside the audio. Use this to verify what the agent was doing on their screen during the call — particularly useful for compliance evaluations involving data handling or system navigation.

> **Compatibility note:** Voice and screen recording playback requires a modern browser with audio support. If recordings fail to load, check your browser permissions and contact your administrator if the issue persists.

---

#### 8. Calibration and Consistency

Calibration is the process of aligning how all evaluators interpret and apply the rubric criteria. Your Quality Manager will periodically run calibration sessions — these typically involve all evaluators scoring the same conversation independently, then comparing and discussing the results.

Participating actively in calibration sessions is important because:

- Inconsistency between evaluators undermines the fairness of the QA process
- Agents who are scored differently by different evaluators on the same type of interaction will lose trust in the system
- Your Quality Manager uses calibration to identify and correct rubric ambiguities

If you are ever unsure how to score a particular scenario, raise it with your Quality Manager before submitting — do not guess and do not skip the question.

---

#### Quick Reference: Evaluator Workflow

```
1. Open Reviews tab → sort by Due Date
2. Click Start Review on a Pending item
3. Read the assigned questionnaire before starting
4. Review the full conversation (transcript / recording / metadata)
5. Score each Weighted Single Select question
6. Add specific coaching notes for any deductions
7. Complete all required fields
8. Click Submit → score is calculated automatically
9. Check Notifications regularly for new assignments and reminders
```

---

#### What's Next

- [**Quality Manager Guide**](#) — understand how your work fits into the broader QM workflow and how evaluations are assigned and reviewed
- [**Reports & Analytics**](#) — how quality scores feed into team-level and agent-level reports
- [**Unified Admin Guide**](#) — if you also have administrative responsibilities in your role
