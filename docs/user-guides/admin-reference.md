# Unified Admin Reference Guide
#### Overview
 
This guide is the complete reference for CelestaCX administrators — the people responsible for configuring and maintaining the platform once it is deployed. It covers the full administrative surface of the Unified Admin interface, from initial tenant setup and user management through to routing configuration, channel onboarding, agent desk customisation, reporting, and day-to-day system operations.
 
Unified Admin is the primary web interface for all platform configuration. Some advanced tasks — particularly at the infrastructure level — require direct access to Kubernetes or the IAM system, and are cross-referenced to the [Deployment & Infrastructure](#) section where relevant.
 
**This guide is structured by administrative domain.** Use the section index below to jump directly to what you need, or work through it in sequence if you are configuring a new tenant from scratch.
 
---
 
#### Who Should Use This Guide
 | I am… | Most Relevant Sections |
| --- | --- |
| A reseller setting up a new customer tenant | Initial Setup → Identity & Security → Routing → Channels |
| A customer admin managing day-to-day operations | Users & Teams → Agent Desk → Outbound & Campaigns |
| An IT admin responsible for security and compliance | Identity & Security → System Operations |
| A supervisor with limited admin access | Agent Desk Configuration → Routing Configuration (read) |
| An admin enabling AI features | AI Configuration |
| An admin managing reporting infrastructure | Reporting & Analytics | 
---
 
#### Section Index
 
1. [Initial Setup](#)
2. [Identity & Access Management](#)
3. [Routing Configuration](#)
4. [Agent Desk Configuration](#)
5. [Digital Channel Configuration](#)
6. [Customer Widget](#)
7. [Voice Configuration](#)
8. [Outbound & Campaigns](#)
9. [AI Configuration](#)
10. [Reporting & Analytics](#)
11. [System Operations](#)
 
---
 
#### 1. Initial Setup
 
These are the first tasks to complete when configuring a new CelestaCX tenant. Complete them in the order listed to avoid dependency issues — for example, channels cannot be configured until users and routing foundations are in place.
 | Task | What It Covers |
| --- | --- |
| Channel and Connector Setup | High-level steps for onboarding a new channel and registering its connector in Unified Admin |
| User Management | Creating, editing, and deactivating users; assigning roles and adding users to teams |
| Adding Languages and Timezones | Configure the supported languages and timezone settings used across the platform and agent desk |
| Infrastructure Sizing Guidelines | Recommended resource sizing for different deployment scales — refer to this before expanding an existing deployment | 
> **Before you begin:** Ensure the platform is fully deployed and the IAM realm has been imported. See the [Deployment & Infrastructure](#) section if the platform is not yet running.
 
---
 
#### 2. Identity & Access Management
 
CelestaCX uses Keycloak as its identity provider. IAM configuration controls who can log in, what roles they hold, how authentication works, and how secrets are managed across the platform.
 | Task | What It Covers |
| --- | --- |
| IAM (Keycloak) Configuration | Import realm files, create system users, assign roles (Agent, Supervisor, Admin), and configure team membership |
| Two-Factor Authentication (2FA) — Overview | Business overview of 2FA — when to enable it and how it affects the login experience |
| 2FA Email Configuration | Step-by-step setup for OTP delivery via email, including tenant settings API configuration |
| 2FA Service Technical Reference | OTP flow internals, token lifetime, max attempts, cooldown logic, and advanced parameters |
| Encryption at Rest | Configure encryption for data stored in MongoDB, file storage, and backups |
| Configuring User Inactivity Logout | Automatically log out users who have been inactive for a configurable timeout period |
| Vault Configuration | Set up HashiCorp Vault for centralised secrets management across platform services |
| Vault Dynamic Credentials — ActiveMQ | Issue short-lived, auto-rotating credentials for ActiveMQ via Vault |
| Vault Dynamic Credentials — MongoDB | Issue short-lived, auto-rotating credentials for MongoDB via Vault |
| Vault Dynamic Credentials — Redis | Issue short-lived, auto-rotating credentials for Redis via Vault | 
#### Role Summary
 | Role | Assigned In | Can Access |
| --- | --- | --- |
| Agent | Keycloak + Unified Admin (team) | Agent Desk only |
| Supervisor | Keycloak + Unified Admin (team) | Agent Desk + Supervisor dashboards + limited Unified Admin |
| Admin | Keycloak | Full Unified Admin |
| Quality Manager | Keycloak | QM module + reports |
| Evaluator | Keycloak | QM evaluation workspace only |
| Routing Manager | Keycloak | Routing configuration in Unified Admin | 
> **Important:** Agents and Supervisors must be assigned to a team in Unified Admin in addition to having their role set in Keycloak. Role changes in Keycloak require the affected user to re-login before taking effect.
 
---
 
#### 3. Routing Configuration
 
Routing is the engine that connects customer conversations to the right agent. This section covers the full routing model — from the foundational Media Routing Domain structure through to precision queue configuration.
 
#### Routing Foundation
 
**Media Routing Domains (MRDs)** are the starting point for all routing configuration. An MRD defines a logical grouping of a particular media type — for example, all chat interactions, or all voice interactions. Queues are created within MRDs, and agents are made eligible for queues based on their MRD assignments.
 
You must define your MRDs before creating queues. Queues must exist before agents can be mapped to them.
 | Task | What It Covers |
| --- | --- |
| Media Routing Domains (MRD) Overview | How MRDs logically separate media types and channel groups; creating and naming MRDs |
| Routing Attributes and Precision Queues | Define agent skills and attributes (e.g. language, product expertise) used for attribute-based routing |
| Agent and Queue Mapping | How agents are assigned to queues and how those assignments affect which conversations they receive | 
#### Routing Strategies
 | Task | What It Covers |
| --- | --- |
| Precision Routing | Configure multi-attribute routing rules that match customers to agents based on skill combinations |
| Priority Routing | Set priority levels across queues so higher-priority interactions are routed first when agents handle multiple queue types |
| Queue Priority | Configure per-queue priority weights to control ordering within a set of queues |
| Queue Wait Time and Position | How estimated wait time and queue position are calculated and optionally communicated to the customer |
| Routing Strategy: Pull Mode | Configure pull-based routing — agents see a list of waiting interactions and choose which to handle, rather than being pushed conversations automatically | 
#### Customer Interaction Profiles
 
Customer Interaction Profiles allow you to create named customer profiles that drive personalised routing and priority decisions. For example, a VIP customer profile can be configured to route to a dedicated premium queue or a specific agent group.
 
---
 
#### 4. Agent Desk Configuration
 
These settings control what agents see and can do in the Agent Desk interface — from message formatting and file sharing through to how agent states are tracked and what wrap-up options are available.
 | Task | What It Covers |
| --- | --- |
| Configure Agent Desk Settings | Global Agent Desk settings — message formatting options, file sharing permissions, notification behaviour, and UI preferences |
| Configuring Reason Codes | Define the Not Ready and logout reason codes agents see when stepping away — these appear in adherence reports |
| Configuring Wrap-up Forms | Create wrap-up forms and assign them to specific channels or queues; configure wrap-up timers |
| Form Builder User Guide | Use the drag-and-drop Form Builder to design wrap-up forms, survey forms, and custom interaction forms without writing code |
| Auto-Sync State Logic | How agent state is automatically synchronised between CelestaCX and Cisco UCCE/UCCX in integrated deployments |
| Customer and Agent SLA Implementation | Technical details of how customer-facing and agent-side SLA thresholds are calculated, tracked, and flagged | 
#### Reason Code Best Practices
 
Reason codes directly affect the quality of your adherence and availability reporting. Keep them specific enough to be meaningful but not so granular that agents lose time choosing. A typical set:
 | Category | Examples |
| --- | --- |
| Not Ready — Scheduled | Break, Lunch, Meeting, Training |
| Not Ready — Unscheduled | Technical Issue, Bathroom, Coaching Session |
| Logout | End of Shift, System Restart | 
---
 
#### 5. Digital Channel Configuration
 
Each digital channel requires its own connector configuration. Channels are connected to CelestaCX via a connector service that handles authentication, message translation, and webhook management with the upstream provider.
 
#### Channel Configuration Reference
 | Channel | Guide | Notes |
| --- | --- | --- |
| WhatsApp (Meta Cloud API) | WhatsApp Cloud API Configuration | Requires a Meta Business Account and approved WhatsApp Business number |
| WhatsApp (360dialog) | 360-Connector Configuration Guide | For deployments using 360dialog as the WhatsApp BSP |
| Facebook Messenger | Facebook Channel Configuration Guide | Requires a Facebook Page and Meta app with Messenger permissions |
| Instagram | Instagram Connector Configuration & Deployment | Requires a connected Instagram Business account via Meta |
| Email | Email Channel Configuration (IMAP/SMTP) | Works with any standard IMAP/SMTP email provider |
| Viber | Viber Connector Configuration & Deployment | Requires a Viber Business account |
| SMS (Twilio) | Twilio SMS/MMS Connector Configuration | Requires a Twilio account with an active phone number |
| SMS (SMPP) | SMPP Configuration Guide | For direct SMPP connections to SMS gateways |
| Twitter / X | Twitter Configuration Guide | See limitations guide before configuring — API restrictions apply |
| YouTube | Google Play Store / YouTube Configuration | See limitations guide — comment moderation only, not full messaging |
| LinkedIn | LinkedIn Configuration Guide | Requires LinkedIn account onboarding first | 
#### Channel Limitations
 
Some channels have known constraints that affect what CelestaCX can do with them. Before configuring Twitter/X, YouTube, or LinkedIn channels, review the corresponding limitations guide for that channel. These are referenced in the [Channels & Integrations](#) section.
 
---
 
#### 6. Customer Widget
 
The Customer Widget is the embeddable web chat interface that appears on your customer's website or web application. It is the primary digital touchpoint for web-based customer interactions.
 | Task | What It Covers |
| --- | --- |
| Configuring the Customer Widget | Customise the widget's appearance, behaviour, greeting messages, and pre-chat form; generate the embed code for deployment |
| Customer Widget Features & Capabilities | Full feature reference — supported media types, language options, offline behaviour, mobile responsiveness, and bot handoff | 
#### Deployment
 
Once configured, the widget is embedded in your customer's website by adding a small JavaScript snippet to their page. The snippet is generated by Unified Admin and is unique to your tenant and channel configuration. No server-side code is required on the customer's end.
 
---
 
#### 7. Voice Configuration
 
Voice configuration in CelestaCX covers the EFSwitch (FreeSWITCH-based) voice server, WebRTC browser-based calling, call recording, and AI-powered real-time transcription.
 | Task | What It Covers |
| --- | --- |
| Accessing CX Voice Components | How to access and navigate the EFSwitch voice server administration interface |
| WebRTC Channel Configuration | Configure browser-based voice and video calling via WebRTC — no softphone client required |
| Media Server Configuration | Set up the media server for call recording storage, processing, and retrieval |
| Azure Transcription Setup | Configure Microsoft Azure Cognitive Services for real-time speech-to-text during voice calls |
| Google Transcription Setup | Configure Google Speech-to-Text for real-time speech-to-text during voice calls |
| Voice Channel Limitations | Known constraints in voice channel configuration and recommended workarounds | 
> **Transcription prerequisite:** Real-time transcription requires an active subscription to either Azure Cognitive Services or Google Speech-to-Text, plus the corresponding API credentials configured in the media server settings.
 
---
 
#### 8. Outbound & Campaigns
 
CelestaCX supports outbound dialling campaigns for proactive customer outreach. Campaigns are managed through Unified Admin and executed via the voice channel.
 | Task | What It Covers |
| --- | --- |
| Managing Outbound Campaigns | Create, schedule, configure, start, pause, and stop outbound dialling campaigns; manage contact lists and dispositions |
| Do Not Contact (DNC) Lists | Upload and manage DNC lists to ensure compliance with local regulations; configure automatic DNC checking before dialling | 
#### Campaign Types
 | Type | Description |
| --- | --- |
| Preview | Agent sees the customer record before the call is dialled and chooses when to initiate |
| Progressive | System dials automatically when an agent becomes available — one call per agent at a time |
| Predictive | System dials multiple numbers simultaneously and connects answered calls to available agents — highest efficiency, requires careful management | 
---
 
#### 9. AI Configuration
 
CelestaCX's AI features are configured by the administrator and made available to agents, supervisors, and the QM module based on the configuration applied.
 | Task | What It Covers |
| --- | --- |
| AI-Powered Customer Experience | Enable and configure the full AI feature set — Agent Co-Pilot (response suggestions, knowledge surfacing, sentiment), Supervisor AI Assistance, and AI-driven quality management auto-scoring | 
#### AI Features Overview
 | Feature | Who Benefits | What It Does |
| --- | --- | --- |
| Agent Co-Pilot | Agents | Suggests responses, surfaces knowledge articles, summarises conversation history, flags sentiment changes |
| Supervisor AI Assistance | Supervisors | Surfaces real-time context, sentiment signals, and coaching prompts for active conversations |
| AI-Driven QM Scoring | Quality Managers | Automatically scores 100% of closed interactions using the configured evaluation form | 
AI features require configuration of the underlying AI service connection (API keys and endpoints). Contact your CelestaCX partner for details on the AI service integration for your deployment.
 
---
 
#### 10. Reporting & Analytics
 
CelestaCX uses Apache Superset as its reporting and analytics layer. Initial configuration of Superset is performed by the administrator; ongoing report management is accessible to supervisors and quality managers through their respective interfaces.
 | Task | What It Covers |
| --- | --- |
| Reports Configuration (Superset) | Connect Superset to the CelestaCX data platform; import the standard report bundle; configure database connections |
| Superset Reports (Import & SSL) | Import custom or updated report definitions; configure SSL for secure Superset connections |
| Superset Alerts & Reports Enablement | Enable scheduled report delivery by email and configure alert thresholds |
| Deleting Reports from Superset | Safely remove custom or duplicate reports from the Superset instance |
| Monitoring Tenant License Consumption | Monitor how many licensed agent seats and interactions are being consumed per tenant; enforce limits where needed | 
> **Reports & Analytics** is covered in full depth in its own dedicated section of this documentation. See [Reports & Analytics](#) for field-level reference material on all report types, metrics definitions, and dashboard guides.
 
---
 
#### 11. System Operations
 
These are the operational and maintenance tasks that arise over the lifetime of a running CelestaCX deployment — migrations, certificate renewals, credential rotations, and log access.
 | Task | What It Covers |
| --- | --- |
| Archival Configuration | Configure interaction archival — set retention policies, storage targets, and triggers for moving older interactions to long-term storage |
| Migrate IAM Users | Move users between Keycloak instances or realms while preserving role assignments and group memberships |
| Migrate Keycloak Groups to CX Teams | Convert legacy Keycloak group-based team structures to the CelestaCX Teams model |
| Extending RKE2 SSL Certificate Expiry | Renew or extend SSL certificates on the RKE2 Kubernetes cluster before they expire |
| Change Default ActiveMQ Passwords | Rotate the default ActiveMQ credentials via Kubernetes ConfigMap — must be done before going live |
| Accessing Kubernetes Logs | How to access pod-level and component-level logs in the RKE2 cluster for troubleshooting |
| MongoDB Slow Query Logs | Enable and read MongoDB slow query logging to diagnose performance issues |
| Redis Slowlogs | Use the Redis SLOWLOG command to identify and investigate slow Redis commands | 
#### Operational Checklist for New Deployments
 
Before handing a new tenant over to the customer admin, confirm the following:
 ```
☐ Default ActiveMQ passwords changed
☐ IAM realm imported and admin user password set
☐ All users created, roles assigned, and team membership confirmed
☐ MRDs and queues created
☐ At least one channel configured and tested end-to-end
☐ Agent Desk settings reviewed and wrap-up forms assigned
☐ SSL certificates valid and expiry dates noted
☐ Vault configured (if secrets management is in scope)
☐ Superset reports imported and accessible to supervisors
☐ Archival policy configured
☐ License consumption monitoring enabled
``` 
---
 
#### Quick Reference: Unified Admin Navigation
 | I need to… | Go to |
| --- | --- |
| Create or edit a user | Users → User Management |
| Add a user to a team | Teams → select team → Add Member |
| Create a queue | Routing → Queues → New Queue |
| Add an agent to a queue | Routing → Agent-Queue Mapping |
| Configure a channel | Channels → select channel type → Configure |
| Set up wrap-up codes | Agent Desk → Wrap-up Forms |
| Configure reason codes | Agent Desk → Reason Codes |
| Set up a campaign | Outbound → Campaigns → New Campaign |
| Manage DNC lists | Outbound → DNC Lists |
| Access Superset reports | Reports → Open Superset |
| Monitor license usage | System → License Consumption |
| Check Keycloak | Navigate to https://<your-domain>/auth |
