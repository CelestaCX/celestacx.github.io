Here is the **Digital Channels Overview** page:
 
---
 
## Digital Channels Overview
 
CelestaCX supports a broad range of digital messaging channels, allowing your agents to handle customer conversations from multiple platforms within a single interface. This page provides a consolidated overview of all supported digital channels, a comparison of their capabilities, and guidance to help you decide which channels to prioritise.
 
---
 
### Supported Digital Channels
 | Channel | Type | Direction | Rich  Media | File  Sharing | Bots  Supported | Typical  Use  Case |
| --- | --- | --- | --- | --- | --- | --- |
| WhatsApp | Messaging | Inbound  &  Outbound | Yes | Yes | Yes | High-volume  consumer  messaging,  notifications,  support |
| Facebook  Messenger | Messaging | Inbound  &  Outbound | Yes | Yes | Yes | Consumer  brand  support,  social  engagement |
| Instagram  Direct | Messaging | Inbound  &  Outbound | Images,  video | Yes | Yes | Brand  engagement,  influencer-adjacent  support |
| LinkedIn | Messaging | Inbound  &  Outbound | Limited | No | No | B2B  support,  professional  services |
| Twitter  /  X | Messaging  &  Mentions | Inbound  &  Outbound | Images | No | Limited | Public  brand  presence,  DM  support |
| Telegram | Messaging | Inbound  &  Outbound | Yes | Yes | Yes | Privacy-conscious  users,  technical  communities |
| Viber | Messaging | Inbound  &  Outbound | Yes | Yes | Yes | Regional  markets  (Eastern  Europe,  Southeast  Asia) |
| SMS | Messaging | Inbound  &  Outbound | No  (MMS  limited) | No | Limited | Transactional  alerts,  broad  reach,  low-data  users |
| Email | Messaging | Inbound  &  Outbound | HTML | Yes | Limited | Formal  support,  detailed  case  handling |
| YouTube | Comments | Inbound  &  Outbound | No | No | No | Public  comment  moderation,  brand  reputation |
| Google  Play  Store | Reviews | Inbound  &  Outbound | No | No | No | App  review  management,  public  response |
| Customer  Widget | Webchat | Inbound  &  Outbound | Yes | Yes | Yes | Website  visitors,  identified  user  sessions | 
---
 
### Channel Capability Detail
 
#### Messaging Direction
 
All primary messaging channels support both inbound (customer-initiated) and outbound (agent or system-initiated) messaging. YouTube and Google Play Store are inbound-only in terms of customer content — agents respond publicly to comments and reviews.
 
#### Rich Media Support
 
Rich media refers to the ability to send and receive content beyond plain text — images, video, audio, documents, location pins, and interactive elements such as buttons and quick replies.
 
WhatsApp, Facebook Messenger, Telegram, Viber, and the Customer Widget offer the most complete rich media support. SMS is text-only (MMS support is provider-dependent and not universally available). LinkedIn and Twitter/X have partial rich media support depending on the message type.
 
#### File Sharing
 
File sharing allows agents and customers to exchange attachments — PDFs, images, spreadsheets, and similar documents. File sharing is supported on WhatsApp, Facebook Messenger, Instagram, Telegram, Viber, Email, and the Customer Widget. Received files are accessible to agents in the interaction panel and stored with the interaction record.
 
#### Bot Integration
 
CelestaCX supports bot handoff on channels that allow automated pre-qualification, self-service, and triage before escalating to a human agent. Bot integration is most mature on WhatsApp, Facebook Messenger, Telegram, Viber, and the Customer Widget. For channels with limited or no bot support, interactions arrive directly in the routing queue without pre-qualification.
 
---
 
### Channel Comparison by Use Case
 
#### High-Volume Consumer Support
 
**Best channels:** WhatsApp, Facebook Messenger, Customer Widget
 
WhatsApp is the dominant consumer messaging platform in most markets outside North America. Facebook Messenger has broad reach across age groups. The Customer Widget serves customers who arrive via your website. Together these three channels cover the majority of digital-first consumer support demand.
 
#### Formal & Complex Case Handling
 
**Best channels:** Email, Customer Widget (for identified sessions)
 
Email remains the preferred channel for interactions that require detailed documentation, attachments, or formal records — contracts, complaints, regulatory matters. The Customer Widget can serve a similar purpose when the customer is authenticated and their session is tied to a known account.
 
#### B2B & Professional Services
 
**Best channels:** Email, LinkedIn, Voice
 
Business customers often prefer email for its formality and audit trail. LinkedIn is increasingly used for professional enquiries, particularly in industries where relationships are managed through that platform.
 
#### Transactional Notifications & Alerts
 
**Best channels:** SMS, WhatsApp (via message templates)
 
SMS reaches virtually any mobile device without requiring a smartphone or app installation. WhatsApp Business message templates serve a similar notification purpose with higher open rates, but require the customer to have WhatsApp and to have opted in.
 
#### Regional & Market-Specific Channels
 
**Best channels:** Viber (Eastern Europe, Southeast Asia), Telegram (privacy-conscious markets, technical communities), WhatsApp (most of the world outside North America and East Asia)
 
Channel prevalence varies significantly by geography. Before committing to a channel mix, confirm which platforms your specific customer base actually uses.
 
#### Social & Public Engagement
 
**Best channels:** Twitter/X, Facebook (comments), Instagram, YouTube, Google Play Store
 
These channels involve publicly visible interactions — comments, mentions, and reviews. Handling them through CelestaCX allows your team to manage public brand presence from the same interface as private support conversations.
 
---
 
### Message Templates & Outbound Messaging
 
Some channels — particularly WhatsApp — restrict what businesses can send to customers outside of an active conversation window. These restrictions exist because the channel provider wants to prevent spam and ensure customers have opted in to receive business messages.
 | Channel | Outbound  Restriction | How  to  Handle |
| --- | --- | --- |
| WhatsApp | Cannot  initiate  conversations  with  free-form  text.  Must  use  pre-approved  message  templates  outside  the  24-hour  customer  service  window. | Submit  templates  for  approval  via  your  WhatsApp  Business  API  provider  before  use. |
| Facebook  Messenger | Cannot  send  promotional  messages  outside  the  standard  messaging  window. | Use  within  the  24-hour  window  following  customer  contact,  or  use  sponsored  messaging  (Facebook  policy). |
| Instagram | Similar  window  restrictions  to  Facebook  Messenger. | Respond  within  the  customer-initiated  window. |
| SMS | No  window  restriction,  but  opt-out  compliance  (e.g.  STOP  commands)  is  mandatory. | Ensure  opt-out  handling  is  configured  on  your  SMS  provider  account. |
| Email | No  platform  window  restriction,  but  subject  to  anti-spam  regulations  (GDPR,  CAN-SPAM,  etc.). | Maintain  compliant  mailing  lists  and  include  unsubscribe  options. |
| Telegram,  Viber,  Customer  Widget | No  outbound  restrictions  beyond  normal  usage  policies. | No  additional  action  required. | **Important:** Compliance with channel provider policies and applicable communications law (GDPR, TCPA, CAN-SPAM, and equivalents) is the responsibility of your organisation. CelestaCX provides the technical capability — your legal and compliance team should review outbound messaging practices before go-live. 
---
 
### Conversation Windows
 
Several channels operate on a **conversation window** model — a period of time after the customer's last message during which the business can reply freely. Outside this window, restrictions apply.
 | Channel | Conversation  Window |
| --- | --- |
| WhatsApp | 24  hours  from  the  customer's  last  message |
| Facebook  Messenger | 24  hours  from  the  customer's  last  message |
| Instagram  Direct | 24  hours  from  the  customer's  last  message |
| Twitter/X  DMs | No  formal  window  —  standard  DM  rules  apply |
| Telegram | No  window  —  open  messaging |
| Viber | No  window  —  open  messaging |
| SMS | No  window  —  standard  messaging  rules  apply |
| Email | No  window  —  standard  email  rules  apply |
| Customer  Widget | Session-based  —  no  window  after  session  ends | 
CelestaCX displays the remaining conversation window time in the Agent Desk interaction panel for channels where this applies, so agents can see at a glance whether they are within a free-reply window or need to use a template.
 
---
 
### Channel Status & Monitoring
 
The connection status of all configured channel connectors is visible under **Administration → Channels** . Each channel displays:
 
- **Status:** Connected, Disconnected, or Error
- **Last activity:** Timestamp of the most recent inbound interaction
- **Error details:** If status is Error, a summary of the issue and recommended action
 
Connector health is also surfaced in the [Logging & Monitoring](#) dashboard for platform administrators.
 
---
 
### Adding a New Channel
 
Channels can be added to a live deployment at any time without disrupting existing configurations. To add a new channel:
 
1. Ensure you have the required provider account and API credentials ready.
2. Navigate to **Administration → Channels → Add Channel** .
3. Select the channel type and follow the channel-specific setup guide.
4. Test the channel end to end before routing live traffic through it.
 
Adding a channel does not automatically create routing rules — ensure routing rules directing interactions from the new channel to the appropriate queue are created before going live.
 
---
 
*What's next: Proceed to the specific channel page for each channel you are configuring, starting with* [*WhatsApp*](#) *— the most widely deployed digital channel in CelestaCX implementations.*