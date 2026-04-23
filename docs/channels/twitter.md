Here is the **Twitter / X** page:
 
---
 
## Twitter / X
 
CelestaCX supports Twitter/X Direct Messages, enabling agents to handle private customer conversations initiated through Twitter/X from the Agent Desk. Twitter/X is primarily used for public-facing brand presence and customer service in industries where customers expect to be able to reach businesses publicly — telecommunications, utilities, airlines, retail, and financial services.
 
---
 
### How Twitter/X Works in CelestaCX
 
CelestaCX connects to Twitter/X via the **Twitter/X API v2** and the **Account Activity API** . When a customer sends a Direct Message to your Twitter/X account, the event is delivered to CelestaCX via webhook. Agents respond from the Agent Desk and replies are delivered back through Twitter/X as Direct Messages.
 ```
Customer sends Twitter/X
Direct Message
        │
        ▼
Twitter/X API v2 / Account Activity API
        │
  Webhook delivery
        │
        ▼
CelestaCX Channel Connector
        │
        ▼
Routing Engine → Queue → Agent Desk
``` 
Public mentions — tweets that tag your account — are surfaced as notifications in CelestaCX but are not routed as full interactions in the same way as Direct Messages. See the section on Mention Monitoring below for details.
 
---
 
### Supported Features
 | Feature | Supported |
| --- | --- |
| Inbound  Direct  Messages | Yes |
| Outbound  Direct  Messages | Yes |
| Text  messages | Yes |
| Inbound  image  attachments | Yes |
| Outbound  image  attachments | Yes |
| GIFs  (inbound) | Yes |
| Video  (inbound) | Yes |
| Public  mention  monitoring | Yes  —  notification  only |
| Replying  to  public  mentions | No  —  via  CelestaCX |
| Typing  indicators | No |
| Read  receipts | No |
| Bot  handoff | Limited |
| Quick  reply  buttons | Yes  —  via  Direct  Message  CTA | 
---
 
### Prerequisites
 
Before configuring Twitter/X in CelestaCX:
 
- Your organisation must have a **Twitter/X account** used as your official brand or support handle
- The account must be a standard Twitter/X account — no special account type is required for basic API access
- You must apply for a **Twitter/X Developer Account** at [http://developer.twitter.com](http://developer.twitter.com)
- A **Twitter/X Developer App** must be created within the Developer Portal
- Access to the **Account Activity API** is required — this is available on the **Basic** tier or above of Twitter/X's API access plans
 
#### Twitter/X API Access Tiers
 
Twitter/X restructured its API access in 2023. CelestaCX requires access at the **Basic tier or above** to use the Account Activity API for Direct Message webhooks. The Free tier does not include the Account Activity API and is not sufficient for contact centre use.
 | API  Tier | Account  Activity  API | Suitable  for  CelestaCX |
| --- | --- | --- |
| Free | No | No |
| Basic | Yes | Yes  —  minimum  required |
| Pro | Yes | Yes |
| Enterprise | Yes | Yes | 
API tier costs and terms are set by Twitter/X and are subject to change. Review current pricing at [http://developer.twitter.com](http://developer.twitter.com) before committing to a plan.
 
---
 
### Setting Up Your Twitter/X Developer App
 
1. Log in to the **Twitter/X Developer Portal** at [http://developer.twitter.com](http://developer.twitter.com) with the account that owns or has admin access to your brand Twitter/X account.
2. Create a new **Project** and **App** within the Developer Portal, or use an existing App if one has already been created.
3. Under **App Settings → Keys and Tokens** , generate the following:
 
- **API Key** (also called Consumer Key)
- **API Key Secret** (also called Consumer Secret)
- **Access Token** (for the specific Twitter/X account CelestaCX will manage)
- **Access Token Secret**
4. Ensure the App has **Read and Write** permissions and **Direct Messages** permission enabled under **App Permissions** .
5. Under **Dev Environments** , create a new environment for the Account Activity API and note the **Environment Name** .
 
---
 
### Configuring Twitter/X in CelestaCX
 
1. Navigate to **Administration → Channels → Add Channel → Twitter/X** .
2. Enter a display name for this channel (e.g. "Twitter/X — @AcmeSupport").
3. Enter the following credentials:
 | Field | Description |
| --- | --- |
| API  Key | From  Twitter/X  Developer  Portal  →  App  →  Keys  and  Tokens |
| API  Key  Secret | From  Twitter/X  Developer  Portal  →  App  →  Keys  and  Tokens |
| Access  Token | Generated  for  the  specific  Twitter/X  account  in  the  Developer  Portal |
| Access  Token  Secret | Generated  alongside  the  Access  Token |
| Environment  Name | The  Account  Activity  API  environment  name  from  the  Developer  Portal |
| Webhook  Verify  Token | A  string  you  define,  used  to  verify  the  webhook  handshake | 
1. Copy the **CelestaCX Webhook URL** and register it in the Twitter/X Developer Portal under **Dev Environments → Account Activity API → [Environment] → Add Webhook URL** .
2. Twitter/X will send a CRC (Challenge Response Check) to the webhook URL to verify it. CelestaCX handles the CRC response automatically — the verification should complete within a few seconds of registering the URL.
3. Subscribe to the **Direct Message events** for your account under the Account Activity API environment.
4. Click **Test Connection** in CelestaCX to confirm the integration is working.
5. Create a routing rule for Twitter/X interactions: **Administration → Routing → Routing Rules → Add Rule** — set the trigger to this Twitter/X channel.
6. Save and set the channel status to **Active** .
 
---
 
### Mention Monitoring
 
CelestaCX surfaces public mentions — tweets that tag your account — as lightweight notifications rather than full routable interactions. This gives your team visibility of public activity without flooding the interaction queue.
 
#### How Mention Monitoring Works
 
When a tweet mentioning your account is detected:
 
- A notification appears in the Agent Desk notification panel
- The tweet text, author, and link are displayed
- Agents can choose to respond by opening a Direct Message conversation with the author
- Public replies to tweets are not sent through CelestaCX — agents who need to reply publicly must do so directly in Twitter/X
 
#### Routing Mentions to Queues
 
For operations where public mention monitoring is a core workflow — for example, a social media team responding to brand mentions — mentions can be configured to create routable interactions rather than notifications only. This is configured under **Administration → Channels → [Channel] → Mention Handling** :
 | Setting | Behaviour |
| --- | --- |
| Notifications  only  (default) | Mentions  appear  as  notifications  in  the  Agent  Desk.  No  queue  routing. |
| Route  as  interactions | Mentions  create  routable  interactions  that  enter  the  queue  and  are  assigned  to  agents.  Agents  respond  via  Direct  Message  —  public  replies  are  not  sent  through  CelestaCX. | **Note:** Routing mentions as interactions will generate a queue interaction for every public tweet that mentions your account, including automated mentions, retweets, and spam. Review mention volume before enabling this setting and ensure appropriate filtering rules are in place. 
---
 
### Conversation Behaviour
 
Twitter/X does not impose a formal conversation window equivalent to Meta's 24-hour policy. Agents can send Direct Messages to customers at any time, provided the customer has previously sent a message to your account or follows your account (Twitter/X DM settings permitting).
 | Capability | Supported |
| --- | --- |
| Responding  to  customer-initiated  DMs | Yes |
| Initiating  DMs  to  customers  who  follow  your  account | Yes  —  if  the  customer's  DM  settings  allow  it |
| Initiating  DMs  to  customers  who  do  not  follow  your  account | No  —  Twitter/X  DM  restrictions  apply |
| Sending  public  replies  via  CelestaCX | No | 
---
 
### Rich Media Handling
 | Media  Type | Inbound | Outbound | Notes |
| --- | --- | --- | --- |
| Images  (JPEG,  PNG,  GIF) | Yes | Yes | Max  5  MB  for  images;  GIF  max  15  MB |
| Video  (MP4) | Yes | No | Inbound  video  displayed  as  attachment |
| Audio | No | No | Not  supported  via  DM  API |
| Documents | No | No | Not  supported  via  DM  API | 
Received media is stored with the interaction record and accessible to agents in the interaction panel.
 
---
 
### Quick Reply Buttons in Direct Messages
 
Twitter/X supports **Quick Reply buttons** in Direct Messages — predefined response options presented to the customer as tappable buttons. These are useful for self-service triage, routing customers to the right team, or collecting structured input before connecting them to an agent.
 
Quick Reply buttons are configured as part of outbound Direct Message templates in CelestaCX. Each button has a label (displayed to the customer) and a metadata value (received by CelestaCX when the customer taps the button). The metadata value can be used as a routing attribute to direct the conversation to a specific queue.
 
To configure Quick Reply templates:
 
1. Navigate to **Administration → Channels → [Channel] → Message Templates → Add Template** .
2. Select **Direct Message with Quick Replies** as the template type.
3. Add the message text and define up to five Quick Reply options with labels and metadata values.
4. Save and activate the template.
 
Agents can send Quick Reply templates from the Agent Desk message composer. Quick Reply buttons expire after 24 hours if not tapped by the customer.
 
---
 
### Managing Multiple Twitter/X Accounts
 
If your organisation manages multiple Twitter/X accounts — for example, a main brand account and a dedicated support handle — each account requires a separate channel connector in CelestaCX.
 
Each connector requires its own set of API credentials (Access Token and Access Token Secret are account-specific) and its own Account Activity API webhook registration.
 
To add a second Twitter/X account:
 
1. Navigate to **Administration → Channels → Add Channel → Twitter/X** .
2. Follow the same configuration steps with the credentials for the additional account.
3. Ensure the Account Activity API environment has capacity for the additional webhook subscription (environment subscription limits apply based on your API tier).
 
---
 
### Rate Limits
 
Twitter/X imposes API rate limits that affect how many messages can be sent and received within a given time window. CelestaCX handles rate limit management automatically and queues outbound messages when limits are approached.
 
Key limits to be aware of at the Basic tier:
 | Operation | Limit |
| --- | --- |
| Direct  Messages  sent  per  day | 1,000  per  account |
| Account  Activity  API  subscriptions | 1  per  environment  (Basic  tier) |
| Read  operations  (mention  monitoring) | 10,000  per  month | 
If your operation requires higher volumes, upgrade to the Pro or Enterprise API tier. Current limits are published at [**developer.twitter.com/en/docs/twitter-api/rate-limits**](http://developer.twitter.com/en/docs/twitter-api/rate-limits) .
 **Note:** Twitter/X API limits and pricing have changed frequently since 2023. Always verify current limits directly with Twitter/X before planning your deployment around specific volume assumptions. 
---
 
### Troubleshooting
 | Symptom | Likely  Cause | Action |
| --- | --- | --- |
| Inbound  DMs  not  appearing  in  CelestaCX | Webhook  not  registered  or  CRC  verification  failed | Re-register  webhook  URL  in  Developer  Portal.  CelestaCX  will  re-attempt  CRC  verification  automatically. |
| "401  Unauthorised"  error | Access  Token  or  Access  Token  Secret  incorrect  or  expired | Regenerate  Access  Token  and  Secret  in  Developer  Portal  and  update  the  connector. |
| Webhook  registered  but  no  events  arriving | Account  Activity  API  subscription  not  created | Confirm  the  account  is  subscribed  to  the  environment  under  Developer  Portal  →  Dev  Environments  →  [Environment]  →  Subscriptions. |
| Mentions  not  appearing | App  does  not  have  sufficient  read  permissions | Verify  App  Permissions  include  Read  access.  Regenerate  tokens  after  changing  permissions. |
| Rate  limit  errors  appearing  in  logs | Daily  DM  send  limit  reached | Review  outbound  message  volume.  Upgrade  API  tier  if  sustained  high  volumes  are  required. |
| CRC  challenge  failing | CelestaCX  webhook  URL  not  publicly  reachable | Verify  network  configuration  and  ensure  the  webhook  URL  is  accessible  from  the  public  internet. | 
---
 
*What's next: Proceed to* [*Telegram*](#) *to configure Telegram bot-based messaging.*
 
---