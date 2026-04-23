# Viber
Here is the **Viber** page:
 
---
 
## Viber
 
CelestaCX supports Viber Business Messages, enabling agents to handle customer conversations initiated through Viber from the Agent Desk. Viber has strong adoption in Eastern Europe, the Middle East, and parts of Southeast Asia, making it an important channel for operations serving customers in those regions.
 
---
 
### How Viber Works in CelestaCX
 
CelestaCX connects to Viber via the **Viber REST API** . A Viber Business Account acts as the customer-facing endpoint — customers find your business via a Viber deep link, QR code, or by searching for your verified account name within Viber. Messages are delivered to CelestaCX via webhook. Agents respond from the Agent Desk and replies are sent back to the customer through Viber.
 ```
Customer opens Viber and
messages your Business Account
 │
 ▼
Viber REST API
 │
 Webhook delivery
 │
 ▼
CelestaCX Channel Connector
 │
 ▼
Routing Engine → Queue → Agent Desk
``` 
---
 
### Supported Features
 | Feature | Supported |
| --- | --- |
| Inbound text messages | Yes |
| Outbound text messages | Yes |
| Inbound images, video, audio, documents | Yes |
| Outbound images, video, audio, documents | Yes |
| Inbound stickers | Yes — displayed as image |
| Inbound location sharing | Yes — displayed as coordinates |
| Inbound contact sharing | Yes — displayed as contact details |
| Rich cards (image + text + button) | Yes |
| Keyboard buttons | Yes |
| Bot handoff | Yes |
| Message delivery receipts | Yes |
| Message seen receipts | Yes |
| Typing indicators | No |
| Group messaging | No — one-to-one only |
| Viber Communities | No | 
---
 
### Prerequisites
 
Before configuring Viber in CelestaCX, you need a **Viber Business Account** . There are two types of Viber business presence relevant to CelestaCX:
 | Account Type | Description | Suitable for CelestaCX |
| --- | --- | --- |
| Viber Bot | A bot account created via the Viber Developer Portal. No verification required. Lower trust level — shows as a bot to users. | Yes — for lower-volume or non-verified deployments |
| Viber Business Messages (Verified Account) | A verified business account with an official badge. Requires Viber approval. Higher trust and better deliverability. | Yes — recommended for production deployments | 
For production contact centre use, a **Viber Business Messages verified account** is strongly recommended. Customers are more likely to engage with a verified business account than an unverified bot.
 
#### Applying for a Viber Business Account
 
Verified Viber Business accounts are managed through **Viber's business partner network** . The application process involves:
 
1. Contacting a Viber Business partner or applying directly via [http://partners.viber.com](http://partners.viber.com)
2. Providing business details — company name, registration documents, use case description
3. Viber reviews and approves the account — this typically takes 1–3 weeks
4. Upon approval, Viber provides an **Authentication Token** for API access
 
For Viber Bot accounts (unverified), the token is obtained directly from the Viber Developer Portal at [http://developers.viber.com](http://developers.viber.com) without a review process.
 
---
 
### Configuring Viber in CelestaCX
 
1. Navigate to **Administration → Channels → Add Channel → Viber** .
2. Enter a display name for this channel (e.g. "Viber — Acme Support").
3. Enter the following:
 | Field | Description |
| --- | --- |
| Authentication Token | The token provided by Viber upon account approval (Business Account) or from the Viber Developer Portal (Bot account) |
| Account Name | Your Viber Business Account or bot name — must match exactly as registered with Viber |
| Webhook Verify Token | A string you define, used as an additional security layer when verifying incoming webhook requests | 
1. Copy the **CelestaCX Webhook URL** displayed on the configuration page.
2. CelestaCX will automatically register the webhook with Viber when the configuration is saved — manual webhook registration is not required.
3. Click **Test Connection** to verify the authentication token is valid and the webhook is active.
4. Create a routing rule for Viber interactions: **Administration → Routing → Routing Rules → Add Rule** — set the trigger to this Viber channel.
5. Save and set the channel status to **Active** .
 
> **Note:** Like Telegram, Viber only allows one active webhook per account. If a webhook was previously registered for this account, CelestaCX will overwrite it during setup. Ensure no other system is actively using this Viber account before connecting it to CelestaCX.
 
---
 
### Conversation Behaviour
 
Viber does not impose conversation window restrictions equivalent to Meta's 24-hour policy. However, there are important behavioural differences between Viber Bot accounts and Viber Business Message accounts:
 | Capability | Viber Bot | Viber Business Messages |
| --- | --- | --- |
| Responding to customer-initiated messages | Yes | Yes |
| Sending follow-up messages after a conversation | Yes — no window restriction | Yes — subject to Viber policy on unsolicited messaging |
| Proactive outbound to users who have never messaged | No | Yes — with user opt-in via Viber's outbound messaging program |
| Delivery and seen receipts | Yes | Yes |
| Official verified badge | No | Yes | 
For Viber Business Messages accounts, outbound-initiated messaging to opted-in customers is supported through Viber's outbound messaging program. This is configured separately with your Viber business partner and is distinct from the standard inbound/outbound response flow handled through CelestaCX.
 
---
 
### Rich Media Handling
 | Media Type | Inbound | Outbound | Max Size | Notes |
| --- | --- | --- | --- | --- |
| Images (JPEG, PNG) | Yes | Yes | 1 MB | Viber compresses images above this limit |
| GIFs | Yes | Yes | 1 MB | |
| Video (MP4) | Yes | Yes | 26 MB | |
| Audio (MP3, AAC, OGG) | Yes | Yes | 26 MB | |
| Documents (PDF, DOCX, etc.) | Yes | Yes | 26 MB | |
| Stickers | Yes | No | — | Inbound displayed as image |
| Location | Yes | No | — | Displayed as coordinates with map link |
| Contact cards | Yes | No | — | Displayed as contact details |
| Rich cards | No | Yes | — | Outbound only — image + text + button | 
---
 
### Rich Cards
 
Viber supports **rich cards** — structured messages combining an image, title, descriptive text, and one or more action buttons. Rich cards are useful for presenting structured information or guiding customers through a self-service flow.
 
Rich cards are sent outbound only — customers cannot send rich cards. They are configured as message templates in CelestaCX.
 
#### Rich Card Components
 | Component | Description | Required |
| --- | --- | --- |
| Image | A header image displayed at the top of the card | No |
| Title | Bold headline text | No |
| Description | Body text below the title | No |
| Action button | A tappable button with a label and an action (open URL, send message, call number) | Yes — at least one | 
#### Configuring Rich Card Templates
 
1. Navigate to **Administration → Channels → [Channel] → Message Templates → Add Template** .
2. Select **Rich Card** as the template type.
3. Upload or link an image, enter title and description text, and define one or more action buttons.
4. Save and activate the template.
 
Agents can send rich card templates from the Agent Desk message composer by selecting from the template library.
 
---
 
### Keyboard Buttons
 
Similar to Telegram's custom keyboards, Viber supports **keyboard buttons** — a custom input panel displayed below the conversation that presents predefined options to the customer. Tapping a button sends a predefined text message or triggers a specific action.
 
Keyboard buttons are useful for:
 
- Presenting menu options at the start of a conversation
- Guiding customers through structured flows
- Offering quick reply shortcuts during an active conversation
 
Keyboards are configured as part of message templates or bot flows. A keyboard can be sent alongside any outbound message and remains visible until replaced by a new keyboard or removed.
 
---
 
### Bot Handoff
 
CelestaCX supports bot-to-human handoff on Viber. The handoff process follows the same model as Telegram:
 
1. A bot handles initial customer contact and collects pre-qualification data.
2. When human assistance is needed, the bot signals handoff to CelestaCX.
3. CelestaCX creates a routable interaction with the context gathered by the bot.
4. The routing engine assigns the interaction to the appropriate queue.
5. An agent accepts and the conversation continues in the customer's Viber thread.
 
The customer experience is seamless — the conversation continues in the same Viber thread without requiring the customer to restart or switch channels.
 
For technical implementation details, see the [Bot Connector Developer Guide](#) .
 
---
 
### Delivery & Seen Receipts
 
Viber provides both **delivery receipts** (message delivered to the customer's device) and **seen receipts** (message opened and read by the customer). Both are surfaced in the Agent Desk interaction panel alongside each outbound message.
 | Receipt Type | Indicator in Agent Desk |
| --- | --- |
| Sent | Single tick |
| Delivered | Double tick |
| Seen | Double tick (highlighted) | 
Seen receipts are only available when the customer has not disabled read receipts in their Viber privacy settings. If a customer has disabled read receipts, delivery receipts are still available but seen receipts will not appear.
 
---
 
### Multiple Viber Accounts
 
If your organisation manages multiple Viber Business accounts — for example, separate accounts for different brands or regions — each account requires a separate channel connector in CelestaCX.
 
To add a second Viber account:
 
1. Obtain the authentication token for the additional account from Viber.
2. Navigate to **Administration → Channels → Add Channel → Viber** .
3. Follow the same configuration steps with the new account's token.
4. Create routing rules specific to this account.
 
---
 
### Viber Account Settings
 
Some Viber Business account settings are configured directly with Viber or via the Viber Developer Portal rather than in CelestaCX. These include:
 | Setting | Where Configured |
| --- | --- |
| Account display name | Viber Business Partner / Developer Portal |
| Account avatar / logo | Viber Business Partner / Developer Portal |
| Account background image | Viber Business Partner / Developer Portal |
| Welcome message shown to new subscribers | CelestaCX — Administration → Channels → [Channel] → Welcome Message |
| Out-of-hours automated reply | CelestaCX — configured via routing overflow settings |
| Verified badge | Granted by Viber upon business account approval | 
---
 
### Troubleshooting
 | Symptom | Likely Cause | Action |
| --- | --- | --- |
| Inbound messages not appearing in CelestaCX | Webhook registration failed or authentication token incorrect | Verify token is correct and active. Navigate to Administration → Channels → [Channel] → Reregister Webhook. |
| "Invalid token" error on Test Connection | Token has expired or been regenerated by Viber | Obtain a new token from the Viber Developer Portal or your Viber business partner and update the connector. |
| Messages sending but not delivered | Customer has blocked the account or unsubscribed from the bot | No action possible — Viber does not expose block or unsubscribe status. |
| Rich cards not displaying correctly | Image URL not publicly accessible or image exceeds size limit | Ensure the image is hosted on a publicly accessible URL and is under 1 MB. |
| Webhook conflict — events not arriving after reconfiguration | Previous webhook registration still active in another system | Reregister the webhook via Administration → Channels → [Channel] → Reregister Webhook to overwrite any existing registration. |
| Seen receipts not appearing | Customer has disabled read receipts in Viber privacy settings | No action possible — this is a customer-controlled privacy setting. | 
For persistent issues, check connector logs under **Platform Administration → Logs → Channel Connectors** or refer to the [Troubleshooting Guide](#) .
 
---
 
*What's next: Proceed to* [*SMS (Twilio & SMPP)*](#) *to configure SMS messaging.*
