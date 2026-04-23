# Key Concepts & Terminology

### Key Concepts & Terminology

This page defines the core terms and concepts you'll encounter across CelestaCX and throughout this documentation. If you come across an unfamiliar term anywhere in this knowledge base, this is the place to look it up.

Terms are grouped by topic so that related concepts appear together.

---

### Conversations & Interactions

**Conversation** A Conversation is the top-level container for a customer interaction in CelestaCX. It groups together all the messages, activities, and participants involved in a single customer engagement — regardless of which channel the customer used. A Conversation can span multiple channels (e.g., a customer starts on WhatsApp and switches to voice) and persists across sessions.

**Channel Session** A Channel Session represents one active connection between a customer and CelestaCX on a specific channel. For example, when a customer sends a WhatsApp message, a Channel Session is created for that WhatsApp interaction. A single Conversation can have multiple Channel Sessions if the customer interacts across different channels.

**Room** A Room is a persistent store of all activities associated with a customer. Think of it as the customer's long-term interaction history — it survives beyond a single Conversation and accumulates over time across all interactions that customer has ever had with your contact center.

**Task** A Task is a unit of work assigned to an agent. When a Conversation is routed to an agent, a Task is created representing that agent's responsibility to handle it. Tasks have states — Reserved, Active, Closed — and are what the routing engine tracks and manages.

**Activity** An Activity is any individual event or action within a Conversation — a message sent, a file shared, a note added, a wrap-up applied. Activities make up the full timeline of a Conversation.

**Wrap-up** A Wrap-up is the categorization an agent applies at the end of a conversation to describe its outcome — for example, "Issue Resolved", "Escalated", or "Callback Requested". Wrap-up codes are configured by admins and help with reporting and quality analysis.

---

### Channels & Connectivity

**Channel** A Channel is a configured connection to a specific communication service. For example, a WhatsApp Channel is a set of configurations that connects CelestaCX to a specific WhatsApp Business Account. You can have multiple Channels of the same type — for instance, two separate WhatsApp numbers for different departments.

**Channel Type** A Channel Type is the category of communication channel — WhatsApp, Facebook, Email, Voice, Web Chat, etc. A Channel Type defines what kind of messages can be sent and received. Each Channel belongs to exactly one Channel Type.

**Channel Connector** A Channel Connector is the integration layer between CelestaCX and an external messaging platform. It translates messages from the external platform into CelestaCX's standard message format (CIM) and vice versa. Connectors exist for all supported channels and can also be built custom by developers.

**CIM (Customer Interaction Message)** CIM is CelestaCX's unified message format. All messages — regardless of the channel they came from — are converted into CIM format inside CelestaCX. This is what allows agents to handle messages from any channel in a consistent way. CIM supports text, media, buttons, carousels, forms, voice, and more.

**Customer Widget** The Customer Widget is a configurable web chat component that can be embedded on any website or web application. It allows customers to start a conversation with your contact center directly from your website. It supports text, media, and WebRTC-based voice and video calls.

---

### Routing & Queues

**MRD (Media Routing Domain)** An MRD — also called a Channel Category — is a grouping of channels by media type. For example, all chat-based channels (WhatsApp, Web Chat, Facebook) might belong to one MRD, while voice channels belong to another. MRDs control how many simultaneous tasks of that type an agent can handle at once.

**Queue** A Queue is a waiting area where incoming Conversations are held until an appropriate agent is available. Queues have routing rules, priority settings, and SLA timers. An agent can be assigned to one or more queues.

**Routing Attribute** A Routing Attribute is a skill or characteristic assigned to an agent — for example, "Spanish Speaker", "Technical Support Level 2", or "VIP Customer Handling". Routing attributes are used in Precision Routing to match conversations to the most suitable available agent.

**Precision Routing** Precision Routing is a routing strategy that matches incoming conversations to agents based on their specific routing attributes, rather than simply the next available agent. This ensures customers are connected to the agent best equipped to help them.

**Pull Mode** Pull Mode is a routing model where agents actively pick up conversations from a list rather than having them automatically pushed. Useful for non-urgent channels like email where agents manage their own workload.

**Push Mode** Push Mode is the standard routing model where the routing engine automatically assigns an incoming conversation to an available agent. The agent receives a notification and must accept the task within a configured timeout (RONA timer).

**RONA (Redirect on No Answer)** RONA is what happens when an agent receives a task in Push Mode but does not accept it within the configured time limit. The task is automatically returned to the queue and the agent's state is changed to Not Ready to prevent further tasks being routed to them.

**Agent State** An Agent State reflects an agent's current availability. The main states are **Ready** (available to receive tasks), **Not Ready** (unavailable, with an optional reason code), and **Logout**. Agents also have per-MRD states that control availability for specific channel types independently.

---

### Platform Components

**Agent Desk** The Agent Desk is the web-based interface where agents handle customer conversations. It shows incoming tasks, conversation history, customer profiles, wrap-up options, and real-time notifications across all channels.

**Unified Admin** The Unified Admin is the web-based administration panel where admins configure the platform — managing users, channels, queues, routing rules, wrap-up codes, business calendars, and system settings.

**Agent Manager** The Agent Manager is the core backend service that manages agent states, task routing, and real-time communication between the Agent Desk and the routing engine. Developers building custom Agent Desk implementations integrate directly with the Agent Manager via [http://Socket.io](http://Socket.io)  events and REST APIs.

**IAM / Keycloak** IAM (Identity and Access Management) is the CelestaCX component that handles user authentication and authorization. It is built on Keycloak, an open-source identity platform. IAM controls who can log in, what roles they have, and what they are permitted to do within the platform.

**API Gateway / APISIX** The API Gateway sits in front of all CelestaCX services and handles API authentication, authorization, rate limiting, and routing of external API requests. It is built on APISIX, an open-source API gateway.

**Data Platform** The Data Platform is the reporting infrastructure layer. It extracts data from CelestaCX's operational databases, transforms it into a structured reporting schema, and loads it into a reporting database that powers historical reports and dashboards.

**Conversation Studio** Conversation Studio is a visual flow builder for designing automated conversation flows — for example, IVR menus, bot escalation logic, and pre-conversation routing flows.

---

### Deployment & Infrastructure

**Kubernetes (K8s)** Kubernetes is the container orchestration platform on which CelestaCX is deployed. All CelestaCX services run as containers managed by Kubernetes. CelestaCX supports several Kubernetes distributions including RKE2, K3s, and K0s.

**Helm Chart** A Helm Chart is a packaged set of Kubernetes configuration files that defines how CelestaCX services are deployed. Admins use Helm to install, upgrade, and manage CelestaCX deployments.

**FQDN (Fully Qualified Domain Name)** The FQDN is the full domain name at which CelestaCX is accessible — for example, `celestacx.yourcompany.com`. It is configured during deployment and used by all services and users to reach the platform.

**High Availability (HA)** High Availability refers to a deployment configuration where CelestaCX continues to operate even if one or more infrastructure nodes fail. HA deployments use multiple nodes with a load balancer so that no single failure takes the system offline.

**Tenant** In a multi-tenant deployment, a Tenant is an isolated customer organization hosted on the same CelestaCX infrastructure. Each tenant has their own users, channels, data, and configuration — fully separated from other tenants — but shares the underlying infrastructure with them.

---

### Quality & Compliance

**Quality Management (QM)** Quality Management is the CelestaCX module for reviewing and scoring agent performance. Supervisors and quality managers use it to listen to call recordings, review conversation transcripts, apply evaluation forms, and track quality trends.

**Evaluation Form** An Evaluation Form is a structured scoring template used in Quality Management. It defines the criteria on which an agent's interaction is assessed — for example, greeting quality, problem resolution, tone, and compliance.

**PII (Personally Identifiable Information)** PII refers to any data that could identify a specific individual — names, phone numbers, email addresses, national IDs, etc. CelestaCX includes PII masking features to redact sensitive customer data from logs, screens, and reports where required by compliance policies.
