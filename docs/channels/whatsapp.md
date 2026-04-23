Here is the **WhatsApp** page:
 
---
 
## WhatsApp
 
WhatsApp is the most widely used messaging channel in CelestaCX deployments. With over two billion active users globally, it is the default digital support channel for consumer-facing operations in most markets outside North America and East Asia. This page covers how to connect WhatsApp Business to CelestaCX, configure message templates, manage opt-ins, and handle rich media.
 
---
 
### How WhatsApp Works in CelestaCX
 
CelestaCX connects to WhatsApp through the **WhatsApp Business API** — the enterprise-grade access tier provided by Meta for businesses handling significant message volumes. This is distinct from the WhatsApp Business App, which is a mobile application designed for very small businesses and does not support integration with contact centre platforms.
 
To use WhatsApp in CelestaCX you need:
 
- A **Meta Business Account** (verified)
- A **WhatsApp Business Account (WABA)** linked to your Meta Business Account
- A **phone number** dedicated to WhatsApp Business use (cannot be an existing personal or WhatsApp Business App number without first migrating it)
- Access through a **WhatsApp Business Solution Provider (BSP)** or direct API access via Meta
 
CelestaCX acts as the platform that receives, routes, and responds to WhatsApp messages — the BSP or direct API connection is the bridge between CelestaCX and WhatsApp's infrastructure.
 
---
 
### Supported WhatsApp Features
 | Feature | Supported  in  CelestaCX |
| --- | --- |
| Inbound  text  messages | Yes |
| Outbound  text  messages | Yes |
| Inbound  images,  video,  audio,  documents | Yes |
| Outbound  images,  video,  audio,  documents | Yes |
| Location  sharing  (inbound) | Yes |
| Message  templates  (outbound,  outside  window) | Yes |
| Quick  reply  buttons | Yes |
| List  messages | Yes |
| Stickers  (inbound) | Displayed  as  attachment |
| WhatsApp  Flows | Roadmap |
| Reaction  messages | Displayed  as  event,  not  routable |
| End-to-end  encryption | Maintained  —  CelestaCX  does  not  decrypt  in  transit | 
---
 
### Prerequisites
 
Before configuring WhatsApp in CelestaCX, complete the following outside the platform:
 
#### 1. Verify Your Meta Business Account
 
Your organisation's Meta Business Account must be verified by Meta. Verification confirms your business identity and is required before a WhatsApp Business Account can be created or a phone number registered for API access.
 
Verification is done via **Meta Business Manager → Business Settings → Security Centre** . The process typically takes 1–5 business days and may require business registration documents.
 
#### 2. Create a WhatsApp Business Account (WABA)
 
A WABA is the Meta entity that holds your WhatsApp phone numbers and message templates. It is linked to your Meta Business Account.
 
If you are connecting through a BSP, the BSP may create the WABA on your behalf or guide you through the process. If you are using direct Meta API access, create the WABA via **Meta Business Manager → Accounts → WhatsApp Accounts** .
 
#### 3. Register a Phone Number
 
The phone number used for WhatsApp must be:
 
- Capable of receiving SMS or voice calls for verification
- Not currently registered to any WhatsApp account (personal or Business App)
- Dedicated exclusively to WhatsApp Business API use
 
Numbers that are currently registered to WhatsApp must be migrated before they can be used with the API. Migration removes the number from its existing WhatsApp account — this cannot be undone without re-registration.
 
#### 4. Obtain API Credentials
 
Depending on your access method:
 | Access  Method | Credentials  Required |
| --- | --- |
| Via  BSP | BSP-provided  API  endpoint,  access  token,  and  phone  number  ID |
| Direct  Meta  API | Meta  System  User  access  token,  Phone  Number  ID,  WABA  ID | 
---
 
### Configuring WhatsApp in CelestaCX
 
1. Navigate to **Administration → Channels → Add Channel → WhatsApp** .
2. Enter a **display name** for this WhatsApp channel (used internally — agents see this in the interaction panel).
3. Select your connection method: **BSP** or **Direct Meta API** .
4. Enter the required credentials:
 
**For BSP connections:**
 | Field | Description |
| --- | --- |
| API  Endpoint | The  BSP-provided  webhook  or  API  URL |
| Access  Token | Authentication  token  provided  by  your  BSP |
| Phone  Number  ID | The  identifier  for  the  registered  WhatsApp  number |
| Webhook  Verify  Token | A  token  you  define  —  used  to  verify  webhook  authenticity | 
**For Direct Meta API connections:**
 | Field | Description |
| --- | --- |
| Phone  Number  ID | From  Meta  Business  Manager  →  WhatsApp  Accounts  →  Phone  Numbers |
| WABA  ID | WhatsApp  Business  Account  ID  from  Meta  Business  Manager |
| System  User  Access  Token | Permanent  access  token  from  a  Meta  System  User  with  WhatsApp  permissions |
| Webhook  Verify  Token | A  token  you  define  —  used  to  verify  webhook  authenticity | 
1. Copy the **CelestaCX Webhook URL** displayed on the configuration page.
2. Register this webhook URL in Meta Business Manager (or via your BSP) as the endpoint to receive incoming WhatsApp messages.
3. Enter the Webhook Verify Token in both CelestaCX and Meta to complete webhook verification.
4. Click **Test Connection** . A successful test confirms CelestaCX can communicate with the WhatsApp API.
5. Link the channel to a **routing rule** — navigate to **Administration → Routing → Routing Rules** and create a rule that routes interactions arriving on this WhatsApp number to the appropriate queue.
6. Save and set the channel status to **Active** .
 
---
 
### Message Templates
 
WhatsApp requires businesses to use **pre-approved message templates** when initiating conversations or sending messages outside the 24-hour customer service window. Templates must be submitted to Meta for approval before use.
 
#### Template Categories
 
Meta classifies message templates into three categories:
 | Category | Use  Case | Typical  Approval  Time |
| --- | --- | --- |
| Utility | Transaction  confirmations,  account  alerts,  order  updates | 24–48  hours |
| Authentication | OTP  codes,  verification  messages | 24–48  hours |
| Marketing | Promotional  offers,  announcements,  re-engagement | 48–72  hours  (stricter  review) | 
#### Creating and Submitting Templates
 
Templates are created and submitted for approval in **Meta Business Manager → WhatsApp Accounts → Message Templates** , not in CelestaCX. Once approved by Meta, templates become available in CelestaCX automatically (subject to a sync interval of up to 15 minutes).
 
Templates can include:
 
- Static text with variable placeholders (e.g. `Hello {{1}}, your case {{2}} has been updated.` )
- Header elements: text, image, video, or document
- Footer text
- Action buttons: quick reply buttons or call-to-action buttons (URL or phone number)
 
#### Using Templates in CelestaCX
 
Agents can send approved templates from the Agent Desk when:
 
- Initiating a new outbound WhatsApp conversation
- Responding to a customer outside the 24-hour window
 
To send a template:
 
1. In the Agent Desk interaction panel, click **Send Template** .
2. Select the approved template from the dropdown.
3. Fill in any variable placeholders.
4. Review the message preview and send.
 
Agents cannot send free-form text outside the conversation window — the template selector is the only option available in that state.
 
---
 
### Opt-In Management
 
Meta requires that customers have explicitly opted in to receive WhatsApp Business messages before a business initiates contact. CelestaCX does not manage opt-in collection — this must be handled by your organisation through your own channels (website forms, SMS, email, in-store consent, etc.).
 
Your organisation is responsible for:
 
- Collecting and recording opt-in consent
- Providing a clear opt-out mechanism
- Honouring opt-out requests promptly
- Maintaining opt-in records for compliance purposes
 
CelestaCX can store a customer's WhatsApp opt-in status as a CRM field if your CRM connector is configured — but the collection and legal basis for consent remains your responsibility.
 **Important:** Sending unsolicited WhatsApp messages to customers who have not opted in violates Meta's Business Messaging Policy and can result in your WhatsApp Business Account being restricted or banned. This risk cannot be mitigated by technical configuration alone — it requires operational discipline and appropriate legal review. 
---
 
### Rich Media Handling
 
CelestaCX supports sending and receiving rich media on WhatsApp. The following file types and size limits apply:
 | Media  Type | Inbound | Outbound | Max  Size |
| --- | --- | --- | --- |
| Images  (JPEG,  PNG) | Yes | Yes | 5  MB |
| Video  (MP4,  3GPP) | Yes | Yes | 16  MB |
| Audio  (MP3,  OGG,  AAC) | Yes | Yes | 16  MB |
| Documents  (PDF,  DOCX,  XLSX,  etc.) | Yes | Yes | 100  MB |
| Stickers  (WebP) | Yes  (shown  as  attachment) | No | — |
| Location | Yes  (shown  as  coordinates) | No | — | 
Received media files are stored with the interaction record and are accessible to agents in the interaction panel. Files are available for the duration of your platform data retention period.
 
When sending media outbound, agents can upload files directly from the Agent Desk message composer or select from a pre-loaded media library if one has been configured.
 
---
 
### Handling Multiple WhatsApp Numbers
 
A single CelestaCX deployment can support multiple WhatsApp numbers — for example, separate numbers for different brands, regions, or product lines. Each number is configured as a separate channel connector and can be linked to different routing rules and queues.
 
To add a second WhatsApp number:
 
1. Navigate to **Administration → Channels → Add Channel → WhatsApp** .
2. Follow the same configuration steps with the credentials for the additional number.
3. Create routing rules specific to this number.
 
There is no platform limit on the number of WhatsApp connectors. Practical limits are determined by your WhatsApp Business Account configuration and BSP agreement.
 
---
 
### WhatsApp Channel Status & Monitoring
 
The status of your WhatsApp connector is visible under **Administration → Channels → [Channel Name]** . Key status indicators include:
 | Indicator | Meaning |
| --- | --- |
| Connected | Webhook  is  active  and  receiving  messages  normally |
| Disconnected | Webhook  is  not  reachable  —  check  network  configuration  and  Meta  webhook  settings |
| Token  Expired | Access  token  has  expired  —  generate  a  new  token  and  update  the  connector  configuration |
| Quality  Rating | Meta  assigns  a  quality  rating  (High,  Medium,  Low)  based  on  customer  feedback  on  your  messages.  Low  ratings  can  restrict  template  sending. |
| Message  Limit | Meta  enforces  per-day  message  initiation  limits  based  on  quality  rating  and  account  tier.  Current  limit  is  shown  here. | 
Quality rating and message limit information is sourced from the Meta API and refreshed every 15 minutes.
 
---
 
### Troubleshooting
 | Symptom | Likely  Cause | Action |
| --- | --- | --- |
| Inbound  messages  not  appearing  in  CelestaCX | Webhook  not  registered  or  incorrect  URL | Verify  webhook  URL  in  Meta  Business  Manager.  Re-run  Test  Connection. |
| "Token  Invalid"  error | Access  token  expired  or  revoked | Generate  a  new  System  User  token  in  Meta  Business  Manager  and  update  the  connector. |
| Templates  not  appearing  in  Agent  Desk | Templates  not  yet  synced  after  approval | Wait  up  to  15  minutes  after  Meta  approval.  Trigger  a  manual  sync  under  Administration  →  Channels  →  [Channel]  →  Sync  Templates. |
| Messages  sending  but  not  delivered | Customer  has  blocked  the  number,  or  number  quality  rating  is  Low | Check  quality  rating  in  channel  status.  Review  recent  template  content  for  policy  compliance. |
| Outbound  messages  rejected  with  policy  error | Template  content  violates  Meta  policy | Review  template  against  Meta's  Business  Messaging  Policy.  Edit  and  resubmit  for  approval. | 
For persistent issues not resolved by the above, see the [Troubleshooting Guide](#) or check connector logs under **Platform Administration → Logs → Channel Connectors** .
 
---
 
*What's next: Proceed to* [*Facebook & Instagram*](#) *to configure Meta's social messaging channels.*