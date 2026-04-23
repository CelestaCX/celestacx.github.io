# Admin Setup Guide
This guide provides a structured, task-oriented walkthrough for administrators setting up CelestaCX for the first time, or performing common ongoing administration tasks. Rather than navigating between reference pages, this guide gives you a sequential path through the platform — from a freshly deployed system to a fully operational contact centre.
 
Use the reference pages in this section for detailed configuration options. Use this guide to understand the order of operations and how each step connects to the next.
 
---
 
### Before You Begin
 
Confirm the following prerequisites are in place before starting configuration:
 | Prerequisite | Where to Check |
| --- | --- |
| CelestaCX is deployed and accessible via browser | Deployment & Infrastructure |
| You have Platform Admin or Tenant Admin credentials | Provided during deployment |
| DNS, SSL certificates, and network access are confirmed | Security & Certificates |
| You know the channels your operation will use (voice, WhatsApp, email, etc.) | Internal requirements |
| You have an organisational structure: teams, supervisors, agent headcount | Internal requirements |
| You have defined working hours and public holiday schedules | Internal requirements | 
---
 
### Phase 1: Platform & Tenant Setup
 
These steps are performed by a **Platform Admin** and only need to be completed once per deployment.
 
#### Step 1.1 — Create the Tenant
 
If your deployment serves a single organisation, a default tenant may already be created during installation. Verify this before creating a new one.
 
1. Log in as Platform Admin.
2. Navigate to **Platform Administration → Tenants** .
3. Confirm a tenant exists for your organisation. If not, select **New Tenant** and complete the tenant creation form.
4. Note the tenant identifier — you will need it when configuring SSO and channel connectors.
 
#### Step 1.2 — Create the First Tenant Admin User
 
1. Navigate to **Platform Administration → Users → Add User** .
2. Create a user with the **Tenant Admin** role scoped to your tenant.
3. Provide the credentials to the appropriate person in your organisation.
4. From this point forward, day-to-day administration is performed by the Tenant Admin, not the Platform Admin.
 
#### Step 1.3 — Configure SSO (If Required)
 
If your organisation uses Single Sign-On, configure it before provisioning other users — this avoids the need to reset passwords after switching authentication methods.
 
1. Log in as Tenant Admin.
2. Navigate to **Administration → Security → Single Sign-On** .
3. Follow the SSO configuration steps in [Identity & Access Management](#) .
4. Test SSO login before enabling it for all users.
5. Decide on authentication mode: **SSO Only** or **SSO + Local** .
 
> If SSO is not required, skip to Step 1.4 and use local authentication.
 
#### Step 1.4 — Set Password Policy
 
1. Navigate to **Administration → Security → Password Policy** .
2. Review and adjust the default password policy to match your organisation's security requirements.
3. Save.
 
---
 
### Phase 2: Users, Roles & Teams
 
These steps are performed by the **Tenant Admin** and establish the human structure of your operation.
 
#### Step 2.1 — Create Skills
 
Define skills before creating users, so they are available to assign immediately during user setup.
 
1. Navigate to **Administration → Routing → Skills → Add Skill** .
2. Create all skills your routing will reference — languages, product lines, technical tiers, and any other agent capabilities relevant to your routing design.
 
#### Step 2.2 — Create Agent Users
 
1. Navigate to **Administration → Users → Add User** (or **Import Users** for bulk provisioning).
2. Create all agent accounts. Assign the **Agent** role to each.
3. Assign relevant skills and proficiency levels to each agent.
4. Do not assign agents to teams yet — teams are created in the next step.
 
#### Step 2.3 — Create Supervisor Users
 
1. Create user accounts for all supervisors. Assign the **Supervisor** role to each.
2. Optionally assign skills to supervisors if they will handle escalated interactions directly.
 
#### Step 2.4 — Create Other Users
 
Create accounts for Quality Managers, Evaluators, and Reporting Users as required. Assign the appropriate role to each.
 
#### Step 2.5 — Create Teams
 
1. Navigate to **Administration → Teams → Add Team** .
2. Create each team. Assign the relevant supervisor(s) and add agent members.
3. Repeat for all teams in your organisation.
 
---
 
### Phase 3: Operational Configuration
 
These steps define how your operation runs — when it is open, how interactions are handled, and what agents do during and after each interaction.
 
#### Step 3.1 — Create Business Calendars
 
1. Navigate to **Administration → Business Calendars → Add Calendar** .
2. Create one calendar per operational region or time zone.
3. Set working hours, holidays, and any known special schedules.
4. Test each calendar using the built-in calendar tester before proceeding.
 
See [Business Calendars](#) for full configuration details.
 
#### Step 3.2 — Create Queues
 
1. Navigate to **Administration → Queues → Add Queue** .
2. Create each queue your operation requires.
3. For each queue:
 
- Assign the teams that will serve it.
- Set capacity and wrap-up settings.
- Link the appropriate business calendar.
- Leave overflow settings for now — configure these after routing rules are in place.
4. Save all queues before moving to routing configuration.
 
See [Agent Teams & Queues](#) for full configuration details.
 
#### Step 3.3 — Configure Routing Rules
 
1. Navigate to **Administration → Routing → Routing Rules → Add Rule** .
2. Create routing rules that direct incoming interactions to the correct queues.
3. Work through each channel and interaction type your operation handles.
4. Verify rule evaluation order using the drag handle on the rules list.
5. Ensure a default queue is set for any interactions that do not match a specific rule.
 
See [Channels & Routing Configuration](#) for full configuration details.
 
#### Step 3.4 — Configure Overflow Settings
 
With queues and routing rules in place, return to each queue and configure overflow behaviour:
 
1. Navigate to **Administration → Queues → [Queue] → Overflow Settings** .
2. Set wait time thresholds and define the fallback action for each threshold.
3. Confirm out-of-hours overflow is linked to the correct calendar behaviour.
 
#### Step 3.5 — Build Forms
 
1. Navigate to **Administration → Forms → Form Builder → New Form** .
2. Create forms for each stage of your agent workflow — verification, case intake, after-contact work, and escalation, as required.
3. Link each form to the relevant queues under **Administration → Queues → [Queue] → Forms** .
 
See [Forms & Form Builder](#) for full configuration details.
 
---
 
### Phase 4: Channel Configuration
 
Channel configuration connects external communication services to CelestaCX. The steps in this phase depend on which channels your operation will use.
 | Channel | Where to Configure |
| --- | --- |
| Voice | Voice & Video |
| WhatsApp | WhatsApp |
| Facebook & Instagram | Facebook & Instagram |
| Email | Email |
| Webchat / Customer Widget | Customer Widget |
| SMS | SMS (Twilio & SMPP) |
| Other digital channels | Channels & Integrations | 
Work through each channel your operation requires. For each:
 
1. Follow the channel-specific setup guide in the [Channels & Integrations](#) section.
2. Verify the channel connector is active and receiving test interactions.
3. Confirm routing rules exist to direct interactions from this channel to the correct queue.
4. Send a test interaction through the channel and confirm it reaches the expected queue and agent.
 
---
 
### Phase 5: Pre-Launch Verification
 
Before going live, work through this checklist to confirm the platform is correctly configured end to end.
 
#### Access & Authentication
 
- All users can log in successfully (SSO or local).
- Each user sees only the features and data appropriate to their role.
- Supervisor accounts can view and control their assigned teams.
- Platform Admin access is restricted to authorised personnel only.
 
#### Routing & Queues
 
- Each active channel has at least one routing rule directing interactions to a queue.
- A default queue exists to catch any unmatched interactions.
- All queues are linked to the correct business calendar.
- Queue capacity and wrap-up settings match operational requirements.
- Overflow settings are configured on all queues.
 
#### Calendars
 
- All calendars are set to the correct time zone.
- Working hours are accurate.
- Holidays for the current and next 12 months are entered.
- Each calendar has been tested using the calendar tester.
 
#### Channels
 
- All channel connectors are active and showing as connected.
- Test interactions have been successfully sent and received on each channel.
- Out-of-hours responses are configured and tested.
 
#### Forms
 
- All required forms are built and set to Active.
- Forms are linked to the correct queues.
- Conditional logic has been tested for all field conditions.
- Required fields are confirmed as necessary and not excessive.
 
#### Agent Experience
 
- Agents can log in to the Agent Desk.
- Test interactions reach the correct agents.
- Forms appear correctly in the interaction panel.
- Wrap-up behaviour is working as expected.
 
---
 
### Common Ongoing Administration Tasks
 
Once the platform is live, the following are the most frequently performed administration tasks:
 | Task | Where |
| --- | --- |
| Add or deactivate a user | Administration → Users |
| Reset a user's password | Administration → Users → [User] → Reset Password |
| Add an agent to a team | Administration → Teams → [Team] → Members |
| Update queue capacity or wrap-up time | Administration → Queues → [Queue] → Edit |
| Add a holiday to a calendar | Administration → Business Calendars → [Calendar] → Holidays |
| Update a routing rule | Administration → Routing → Routing Rules → [Rule] → Edit |
| Add a new form field | Administration → Forms → Form Builder → [Form] → Edit |
| View the audit log | Platform Administration → Audit Log |
| Create a new tenant (Platform Admin only) | Platform Administration → Tenants → New Tenant | 
---
 
### Getting Help
 
If you encounter issues during configuration or go-live, the following resources are available:
 
- **Troubleshooting Guide** — Common configuration problems and resolutions. See [Troubleshooting Guide](#) .
- **Logging & Monitoring** — Platform logs for diagnosing connector and routing issues. See [Logging & Monitoring](#) .
- **Your Implementation Partner** — OctaveBytes support is available for configuration assistance, go-live support, and post-launch reviews.
 
---
 
*Configuration & Administration is now complete. Continue to* [*Channels & Integrations*](#) *to configure the external channels that will deliver interactions into CelestaCX.*
