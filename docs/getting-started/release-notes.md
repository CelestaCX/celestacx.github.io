This page tracks all CelestaCX releases — what changed, when it was released, and how long it is supported. Use this page to understand what version you are running, what's new in the latest release, and whether your version is still receiving updates.
 
---
 
### Current Supported Versions
 | Version | Release  Date | Status | Support  Until |
| --- | --- | --- | --- |
| 5.0 | November  2025 | ✅  Current  —  Generally  Available | TBC |
| 4.10 | August  2025 | ✅  Active  Maintenance | August  2028 |
| 4.9 | 2025 | ⚠️  Maintenance  Only | Check  with  support |
| 4.8  and  earlier | — | ❌  End  of  Life | No  longer  supported | ℹ️ **Which version am I on?** Log in to the Unified Admin panel. The version number is displayed in the bottom left corner of the interface. 
---
 
### CelestaCX 5.0
 
**Release Date:** November 2025 **Status:** Generally Available **Type:** Major Release
 
#### What's New
 
**Multi-Tenancy Support** CelestaCX 5.0 introduces full multi-tenancy, enabling a single CelestaCX deployment to host multiple isolated customer organizations — each with their own users, channels, data, and configuration — on shared underlying infrastructure. This is the foundation for CCaaS (Contact Center as a Service) delivery models where partners manage multiple customers from one platform.
 
Key capabilities introduced with multi-tenancy:
 
- Tenant-level isolation of all data and configuration
- Partner admin role for managing multiple tenants
- Per-tenant channel and routing configuration
- Tenant onboarding guide for partners
 
**Platform Stability & Maintainability** A range of internal improvements to core platform components improving reliability under high load and simplifying long-term maintenance.
 
#### Upgrade Guide
 
Upgrading from CelestaCX 4.10 to 5.0? See the [Upgrade Guide →](#) in the Deployment & Infrastructure section.
 
#### Known Limitations
 
- Multi-tenancy is not supported with Cisco Contact Center integration in this release
- WFM (Workforce Management) multi-tenant support is planned for a future release
 
---
 
### CelestaCX 4.10
 
**Release Date:** August 2025 **Status:** Active Maintenance — supported until August 2028 **Type:** Major Release
 
#### What's New
 
**LinkedIn Integration** CelestaCX 4.10 adds LinkedIn as a supported social media channel, allowing contact centers to manage LinkedIn messages and social interactions alongside all other channels in the Agent Desk.
 
**Redesigned Form Builder** The Form Builder has been significantly redesigned with an improved user interface, making it easier to create and manage pre-conversation forms, data capture forms, and agent-facing forms.
 
**In-Conversation Forms** Agents can now send forms directly to customers within an active conversation, enabling structured data collection without leaving the conversation flow.
 
**Quality Management Enhancements**
 
- AI-driven quality assessments introduced
- Real-time performance alerts for low-scoring interactions
- Agent & Team Leaderboard report added
- Review Scheduler for automated evaluation workflows
 
**Reporting Additions** New reports added in 4.10 include:
 
- Repeated Caller Report
- Multichannel Session Detail
- Outbound Summary Report
- Queue-wise Stats Summary
- WebRTC Detail and Summary Reports
 
**Data Platform Improvements**
 
- Alembic schema migration support for automated database schema updates during upgrades
- WFM data pipeline introduced
- Cisco Agent & Team Sync pipeline added
 
**Voice & Dialer**
 
- CX Dialer benchmarking documentation added
- CX Voice Upgrade to 4.10 path documented and supported
 
#### Upgrade Guide
 
Upgrading to CelestaCX 4.10 from an earlier version? See the [Upgrade Guide →](#) in the Deployment & Infrastructure section.
 
#### Known Limitations
 
- Firefox and Microsoft Edge browser compatibility not yet fully tested on Agent Desk, Customer Widget, and Unified Admin — Google Chrome is recommended
- Cisco Unified Call Manager voice channel integration is present as a placeholder — not yet fully integrated in this release
 
---
 
### Version History
 | Version | Release  Date | Key  Highlights |
| --- | --- | --- |
| 5.0 | Nov  2025 | Multi-tenancy,  CCaaS  support |
| 4.10 | Aug  2025 | LinkedIn,  Form  Builder  redesign,  AI  QM |
| 4.9 | 2025 | Voice  enhancements,  reporting  updates |
| 4.8 | 2024 | Platform  performance  improvements |
| 4.7 | 2024 | WFM  integration,  additional  channel  support |
| 4.6 | 2024 | Quality  Management  module,  campaign  updates |
| 4.5 | 2023 | CX  Voice  native  dialer  introduced | ℹ️ Full release notes for versions prior to 4.10 are available on request from your CelestaCX account manager. 
---
 
### How to Read a Version Number
 
CelestaCX uses the following versioning convention:
 ```
5  .  0  .  1
    │     │     │
    │     │     └── Patch — bug fixes only, no new features
    │     └──────── Minor — new features, backward compatible
    └────────────── Major — significant new capabilities or
                            architectural changes
``` 
**Major releases** (e.g., 4.x → 5.0) may include architectural changes and require careful upgrade planning. Always read the upgrade guide before proceeding.
 
**Minor releases** (e.g., 4.9 → 4.10) introduce new features but are designed to be backward compatible with the same major version.
 
**Patch releases** (e.g., 4.10.0 → 4.10.1) contain bug fixes only and are safe to apply without feature impact.