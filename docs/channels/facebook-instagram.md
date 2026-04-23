# Facebook & Instagram
Here is the **Facebook & Instagram** page:
 
---
 
## Facebook & Instagram
 
CelestaCX supports both Facebook Messenger and Instagram Direct Messages through a single Meta integration. Since both platforms are owned by Meta and share a common API infrastructure, they are configured together and managed through the same Meta Business Manager account. This page covers how to connect both channels, configure permissions, and manage the differences between them.
 
---
 
### How Facebook & Instagram Work in CelestaCX
 
CelestaCX connects to Facebook Messenger and Instagram Direct via the **Meta Graph API** and **Webhooks** . When a customer sends a message to your Facebook Page or Instagram Business account, Meta delivers it to CelestaCX via webhook. Agents respond from the Agent Desk and replies are delivered back through the respective platform.
 ```
Customer sends message
on Facebook / Instagram
 │
 ▼
Meta Graph API
 │
 Webhook delivery
 │
 ▼
CelestaCX Channel Connector
 │
 ▼
Routing Engine → Queue → Agent Desk
``` 
Both channels appear as separate connectors in CelestaCX but are authenticated through the same Meta App and Business Manager account.
 
---
 
### Supported Features
 
#### Facebook Messenger
 | Feature | Supported |
| --- | --- |
| Inbound text messages | Yes |
| Outbound text messages | Yes |
| Inbound images, video, audio, files | Yes |
| Outbound images, video, audio, files | Yes |
| Quick reply buttons | Yes |
| Persistent menu | Yes (configured in Meta) |
| Message read receipts | Yes |
| Typing indicators | Yes |
| Ice Breakers (FAQ prompts) | Yes (configured in Meta) |
| Bot handoff protocol | Yes |
| Referral source tracking (ads, page CTAs) | Yes | 
#### Instagram Direct
 | Feature | Supported |
| --- | --- |
| Inbound text messages | Yes |
| Outbound text messages | Yes |
| Inbound images and video | Yes |
| Outbound images | Yes |
| Story mentions (inbound) | Yes — appear as interactions |
| Story replies (inbound) | Yes — appear as interactions |
| Quick reply buttons | Yes |
| Message read receipts | Yes |
| Typing indicators | Yes |
| Voice messages (inbound) | Displayed as audio attachment |
| Reels comments | Not supported — use YouTube/social monitoring | 
---
 
### Prerequisites
 
#### Facebook Messenger
 
Before configuring Facebook Messenger in CelestaCX:
 
- Your organisation must have a **Facebook Page** (not a personal profile)
- The Facebook Page must be connected to a **Meta Business Manager account**
- You must have **Page admin access** or be assigned the appropriate role in Business Manager
- A **Meta App** must be created in the Meta Developer Portal with Messenger permissions enabled
 
#### Instagram Direct
 
Before configuring Instagram Direct in CelestaCX:
 
- Your Instagram account must be an **Instagram Business account** (not a personal or Creator account)
- The Instagram Business account must be **connected to a Facebook Page** in Meta Business Manager
- The same Meta App used for Messenger can be used for Instagram — Instagram permissions must be added to it
- You must have admin access to both the Instagram account and the linked Facebook Page
 
> **Note:** Instagram Direct Messages cannot be accessed via the API without a linked Facebook Page. If your Instagram account is not yet connected to a Facebook Page, do this in Instagram Settings → Account → Switch to Professional Account, then link the Page in Meta Business Manager.
 
---
 
### Setting Up Your Meta App
 
Both Facebook Messenger and Instagram Direct require a Meta App configured in the **Meta Developer Portal** ( [http://developers.facebook.com](http://developers.facebook.com) ). If your organisation already has a Meta App for WhatsApp, you can add Messenger and Instagram permissions to the same app rather than creating a new one.
 
#### Creating or Configuring the Meta App
 
1. Log in to the Meta Developer Portal.
2. Create a new app or open your existing app.
3. Select **Business** as the app type if creating new.
4. Add the following products to your app:
 
- **Messenger** (for Facebook Messenger)
- **Instagram Graph API** (for Instagram Direct)
5. Under **Messenger → Settings** , connect your Facebook Page and generate a **Page Access Token** .
6. Under **Instagram → Settings** , connect your Instagram Business account.
7. Configure the webhook:
 
- Set the **Callback URL** to the CelestaCX webhook URL (obtained from the channel configuration page in CelestaCX).
- Set a **Verify Token** — a string you define, used to verify the webhook handshake.
- Subscribe to the following webhook fields:
 
- For Messenger: `messages` , `messaging_postbacks` , `messaging_referrals`
- For Instagram: `messages` , `messaging_postbacks` , `story_mentions`
8. Submit your app for **Advanced Access** for the `pages_messaging` permission (required for production use beyond the 24-hour test window). This review is conducted by Meta and typically takes 5–10 business days.
 
---
 
### Configuring Facebook Messenger in CelestaCX
 
1. Navigate to **Administration → Channels → Add Channel → Facebook Messenger** .
2. Enter a display name for this channel (e.g. "Facebook — Acme Support Page").
3. Enter the following credentials:
 | Field | Description |
| --- | --- |
| Page ID | Your Facebook Page's numeric ID — found in Facebook Page Settings → About |
| Page Access Token | Generated in Meta Developer Portal → Messenger → Settings |
| App Secret | Found in Meta Developer Portal → App Settings → Basic |
| Webhook Verify Token | The string you defined when configuring the webhook in Meta | 
1. Paste the **CelestaCX Webhook URL** into the Meta Developer Portal as the callback URL (if not already done during Meta App setup).
2. Click **Test Connection** to verify the credentials and webhook.
3. Create a routing rule linking this Facebook Page's interactions to the appropriate queue: **Administration → Routing → Routing Rules → Add Rule** — set the trigger to this Facebook Messenger channel.
4. Save and set the channel status to **Active** .
 
---
 
### Configuring Instagram Direct in CelestaCX
 
1. Navigate to **Administration → Channels → Add Channel → Instagram Direct** .
2. Enter a display name for this channel (e.g. "Instagram — Acme Brand Account").
3. Enter the following credentials:
 | Field | Description |
| --- | --- |
| Instagram Account ID | Your Instagram Business account's numeric ID — found via the Graph API or Meta Business Manager |
| Page Access Token | The access token for the Facebook Page linked to your Instagram account |
| App Secret | Found in Meta Developer Portal → App Settings → Basic |
| Webhook Verify Token | The string you defined when configuring the webhook in Meta | 
1. Click **Test Connection** to verify the credentials and webhook.
2. Create a routing rule for Instagram Direct interactions.
3. Save and set the channel status to **Active** .
 
> **Tip:** The Page Access Token for Instagram Direct is the token for the *Facebook Page* linked to the Instagram account — not an Instagram-specific token. This is a common source of configuration errors.
 
---
 
### Managing Multiple Pages or Accounts
 
CelestaCX supports multiple Facebook Pages and Instagram accounts within a single deployment. Each Page or account is configured as a separate channel connector with its own routing rules.
 
Common multi-account scenarios:
 | Scenario | Approach |
| --- | --- |
| Multiple brands on separate Facebook Pages | One connector per Page, each with its own routing rules and queues |
| Separate Instagram accounts for different regions | One connector per Instagram account |
| Same team handles both Facebook and Instagram | Configure both connectors to route to the same queue — agents see channel type in the interaction panel |
| Separate teams for social vs. support | Route Facebook and Instagram to different queues served by different teams | 
---
 
### Conversation Windows
 
Both Facebook Messenger and Instagram Direct operate on a **24-hour conversation window** — the period following the customer's last message during which agents can reply freely with any content.
 
Outside the 24-hour window, Meta restricts what businesses can send. CelestaCX handles this as follows:
 | State | Agent Capability |
| --- | --- |
| Within 24-hour window | Full free-text and media replies available |
| Outside 24-hour window — Messenger | Message Tags can be used for specific non-promotional use cases (e.g. confirmed event updates, post-purchase updates). Sending outside these cases risks policy violation. |
| Outside 24-hour window — Instagram | Replies are not permitted outside the window via the API. The interaction panel will indicate the window has closed. | 
CelestaCX displays the remaining window time in the Agent Desk interaction panel. When the window closes, the free-text composer is replaced with a notification indicating the window has expired.
 
> **Important:** Meta Message Tags on Messenger are not a general-purpose workaround for the messaging window. Using them for promotional or unrelated content violates Meta's policies and can result in app restrictions.
 
---
 
### Handling Comments vs. Direct Messages
 
A common question in Facebook and Instagram deployments is how public comments — on posts, ads, or stories — are handled versus direct messages.
 | Interaction Type | Handled in CelestaCX |
| --- | --- |
| Facebook Page direct messages (Messenger) | Yes — routed as standard interactions |
| Instagram Direct messages | Yes — routed as standard interactions |
| Facebook post comments | Not natively — requires a separate social listening integration |
| Instagram post comments | Not natively — requires a separate social listening integration |
| Instagram story mentions | Yes — appear as inbound interactions |
| Instagram story replies | Yes — appear as inbound interactions | 
If managing public Facebook or Instagram comments at scale is a requirement, discuss social listening integration options with your implementation partner.
 
---
 
### Story Mentions & Story Replies on Instagram
 
When a customer mentions your Instagram Business account in their story, or replies to one of your stories, CelestaCX receives these as inbound interactions. They appear in the Agent Desk with a story context indicator showing the relevant story content (subject to Meta's 24-hour story availability window).
 
Agents can reply to story mentions and story replies directly from the Agent Desk. Replies are sent as Instagram Direct messages to the customer.
 
---
 
### Rich Media Handling
 
#### Facebook Messenger
 | Media Type | Inbound | Outbound | Notes |
| --- | --- | --- | --- |
| Images (JPEG, PNG, GIF) | Yes | Yes | GIF plays as animation |
| Video (MP4) | Yes | Yes | Max 25 MB |
| Audio (MP3, OGG) | Yes | Yes | Max 25 MB |
| Files (PDF, DOCX, etc.) | Yes | Yes | Max 25 MB | 
#### Instagram Direct
 | Media Type | Inbound | Outbound | Notes |
| --- | --- | --- | --- |
| Images (JPEG, PNG) | Yes | Yes | Max 8 MB |
| Video (MP4) | Yes | No | Outbound video not supported via API |
| Voice messages | Yes (audio attachment) | No | |
| Animated GIFs | Yes | No | | 
---
 
### Referral Tracking on Facebook Messenger
 
When customers initiate a Messenger conversation via a Facebook ad, a page CTA button, or a referral link, CelestaCX captures the referral source and makes it available as an interaction attribute. This allows routing rules to direct ad-sourced conversations to specific queues — for example, routing conversations from a specific campaign to a dedicated sales queue.
 
Referral data is visible to agents in the interaction panel and is stored with the interaction record for reporting purposes.
 
---
 
### Troubleshooting
 | Symptom | Likely Cause | Action |
| --- | --- | --- |
| Inbound messages not appearing | Webhook not verified or subscribed | Confirm webhook URL and verify token in Meta Developer Portal. Check webhook subscriptions include messages . |
| "Invalid OAuth access token" error | Page Access Token expired or revoked | Regenerate the Page Access Token in Meta Developer Portal and update the connector. |
| Instagram connector shows Connected but no messages arrive | Instagram account not linked to Facebook Page | Verify the Instagram Business account is linked to the correct Facebook Page in Meta Business Manager. |
| Story mentions not appearing | story_mentions webhook field not subscribed | Add story_mentions to webhook subscriptions in Meta Developer Portal. |
| Messages sending but not delivered to customer | Customer has restricted or ignored the Page | No action possible — Meta does not expose block status to businesses. |
| App not approved for Advanced Access | App still in Development mode | Complete Meta's App Review process for pages_messaging Advanced Access before going live. | 
For persistent issues, check connector logs under **Platform Administration → Logs → Channel Connectors** or refer to the [Troubleshooting Guide](#) .
 
---
 
*What's next: Proceed to* [*LinkedIn*](#) *to configure LinkedIn messaging for professional and B2B support use cases.*
