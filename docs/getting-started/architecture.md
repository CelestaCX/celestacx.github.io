# Platform Architecture Overview
CelestaCX is a microservices-based platform deployed on Kubernetes. Rather than a single monolithic application, it is composed of a set of specialized services that each handle a specific responsibility — routing, messaging, identity, reporting, and so on. These services communicate with each other through APIs and message queues.
 
This page gives you a high-level understanding of those components and how they relate to each other — essential reading before you begin deployment, configuration, or integration work.
 
---
 
### Architecture Layers
 
CelestaCX's architecture is organized into five logical layers. Understanding these layers helps you reason about how data flows through the system and where each component sits.
 
---
 
### Layer 1 — Client Interfaces
 
These are the web-based applications that humans interact with directly.
 
#### Agent Desk
 
The primary interface for agents and supervisors. Agents use it to receive and handle customer conversations across all channels. Supervisors use it for real-time monitoring, silent monitoring, barge-in, and team management. The Agent Desk communicates with the Agent Manager via [http://Socket.io](http://Socket.io) for real-time events and REST APIs for data operations.
 
#### Unified Admin
 
The administration interface for platform configuration. Admins use it to manage users, channels, queues, routing rules, wrap-up codes, business calendars, and system-wide settings. It communicates with Core Platform Services via REST APIs through the API Gateway.
 
#### Customer Widget
 
A configurable web component that can be embedded on any website to allow customers to initiate conversations. Supports text chat, media sharing, and WebRTC-based voice and video calls. The Customer Widget connects to the appropriate Channel Connector to bring the conversation into CelestaCX.
 
---
 
### Layer 2 — API Gateway
 
All traffic between client interfaces and core platform services passes through the API Gateway.
 
#### APISIX
 
CelestaCX uses APISIX as its API Gateway. It sits in front of all internal services and is responsible for:
 
- **Authentication** — verifying that requests carry valid tokens issued by IAM
- **Authorization** — enforcing which users and roles can access which APIs
- **Rate limiting** — protecting services from excessive request volumes
- **Routing** — directing requests to the correct internal service
- **SSL termination** — handling HTTPS at the gateway level
 
All external API calls — from the Agent Desk, Unified Admin, Customer Widget, and any third-party integrations — pass through APISIX before reaching internal services.
 
---
 
### Layer 3 — Core Platform Services
 
These are the backend services that power CelestaCX's contact center functionality.
 
#### Agent Manager
 
The central coordination service for all agent and conversation activity. It manages agent states, handles real-time event communication with the Agent Desk via [http://Socket.io](http://Socket.io) , coordinates task assignment with the Routing Engine, and maintains the state of active conversations. It is the primary integration point for custom Agent Desk implementations.
 
#### Routing Engine
 
The intelligent routing service responsible for matching incoming customer conversations to the right agent or queue. It evaluates agent availability, skills (routing attributes), queue priorities, SLA timers, and business hours rules to make routing decisions in real time.
 
#### CIM (Customer Interaction Manager)
 
The messaging backbone of CelestaCX. CIM receives messages from all channel connectors, normalizes them into a unified message format, stores conversation history, and distributes messages to the appropriate agents. It handles all message types — text, media, buttons, carousels, forms, voice, and system events.
 
#### IAM — Identity & Access Management (Keycloak)
 
The authentication and authorization service for all CelestaCX users. Built on Keycloak, IAM manages user accounts, roles, groups, and permissions. Every login to Agent Desk, Unified Admin, or any CelestaCX service is authenticated through IAM. It issues JWT tokens that the API Gateway uses to authorize requests.
 
#### Channel Connectors
 
Each supported communication channel — WhatsApp, Facebook, Instagram, Email, Voice, etc. — has a dedicated connector service. Channel Connectors translate messages between the external platform's format and CelestaCX's CIM format, handling both inbound (customer to CelestaCX) and outbound (CelestaCX to customer) communication.
 
#### Conversation Studio
 
A flow builder service that enables automated conversation flows — IVR menus, bot escalation paths, and pre-conversation routing logic. Flows are built visually and executed by the platform when a conversation is initiated or a specific trigger condition is met.
 
#### Bot Connectors
 
Integration services that connect CelestaCX to NLU (Natural Language Understanding) bot platforms such as Rasa and Teneo. When a customer message arrives, the Bot Connector can forward it to the configured bot, receive the bot's response, and route it back through CIM — escalating to a live agent when the bot reaches its limits.
 
---
 
### Layer 4 — Data & Storage
 
CelestaCX uses several specialized databases, each chosen for what it does best.
 | Database | Type | What it stores |
| --- | --- | --- |
| MongoDB | Document database | Conversations, messages, customer profiles, channel sessions, configuration data |
| PostgreSQL | Relational database | Reporting data, QM evaluations, campaign data, structured analytics |
| Redis | In-memory cache | Agent state, session data, real-time presence information, temporary caches |
| ActiveMQ | Message broker | Asynchronous inter-service communication, event queues between platform components | 
---
 
### Layer 5 — Reporting & Observability
 
These services handle data analytics, monitoring, and log management.
 
#### Data Platform (ETL Pipelines)
 
A set of Extract-Transform-Load pipelines that continuously move data from the operational databases (MongoDB) into the reporting database (PostgreSQL). This separation means reporting queries never impact the performance of live contact center operations.
 
#### Apache Superset
 
The business intelligence and reporting interface used for historical reports and dashboards. Superset connects to the PostgreSQL reporting database and powers all the out-of-the-box CelestaCX reports available to supervisors and managers.
 
#### Grafana
 
The real-time monitoring and alerting dashboard used by IT and operations teams. Grafana connects to platform metrics and displays system health, performance indicators, and infrastructure alerts.
 
#### OpenSearch
 
The log aggregation and search platform. All CelestaCX service logs are collected and indexed in OpenSearch, allowing operations teams to search, filter, and analyse logs for troubleshooting and auditing purposes.
 
---
 
### How a Conversation Flows Through the System
 
To make the architecture concrete, here is what happens when a customer sends a WhatsApp message to your contact center:
 ```
1. Customer sends a WhatsApp message
 │
 ▼
2. WhatsApp Channel Connector receives the message
 and converts it to CIM format
 │
 ▼
3. CIM stores the message and creates or updates
 the Conversation record in MongoDB
 │
 ▼
4. Routing Engine evaluates available agents,
 queue rules, and SLA settings
 │
 ▼
5. Routing Engine assigns a Task to the best
 available agent
 │
 ▼
6. Agent Manager notifies the Agent Desk
 via Socket.io — agent sees the incoming request
 │
 ▼
7. Agent accepts the Task — conversation becomes Active
 │
 ▼
8. Agent and customer exchange messages through
 CIM in real time
 │
 ▼
9. Agent closes the conversation and applies a Wrap-up
 │
 ▼
10. Data Platform ETL picks up the closed conversation
 and writes it to PostgreSQL for reporting
``` 
---
 
### Key Integration Points
 
For teams building custom integrations or extensions, the key integration points in the architecture are:
 | Integration Point | Protocol | Primary Use Case |
| --- | --- | --- |
| Agent Manager | http://Socket.io + REST | Build a custom Agent Desk |
| CIM | REST + http://Socket.io | Send and receive messages programmatically |
| Channel Connector API | REST | Build a custom channel connector |
| Bot Connector API | REST | Integrate a custom NLU bot |
| Routing Engine API | REST | Programmatic queue and routing management |
| IAM (Keycloak) | OAuth 2.0 / OpenID Connect | Single sign-on and token-based auth |
| API Gateway (APISIX) | HTTPS | All external API access | 
> 👉 For full API documentation and integration guides, see the [Developer Hub →](#)
 
---
 
### Deployment Topology
 
CelestaCX is deployed on Kubernetes and supports three main deployment topologies depending on your infrastructure requirements:
 
**Single Node** — All services run on a single Kubernetes node. Suitable for development, POC, or small deployments. Not recommended for production.
 
**Multi-Node** — Services are distributed across multiple Kubernetes nodes. Suitable for production deployments with moderate load.
 
**High Availability (HA)** — Multiple control plane nodes with an external load balancer and replicated storage. Recommended for production deployments requiring zero-downtime and failover resilience.
 
> 👉 For sizing guidance and deployment instructions, see [Deployment & Infrastructure →](#)
