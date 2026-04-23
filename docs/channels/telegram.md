# Telegram
CelestaCX supports Telegram messaging through Telegram's Bot API, enabling agents to handle customer conversations initiated through a Telegram bot from the Agent Desk. Telegram is particularly popular in privacy-conscious markets, technical communities, and regions where it has become a primary consumer messaging platform — including Eastern Europe, the Middle East, and Southeast Asia.
 
---
 
### How Telegram Works in CelestaCX
 
CelestaCX connects to Telegram via the **Telegram Bot API** . A Telegram bot acts as the customer-facing endpoint — customers search for or follow a link to your bot, start a conversation, and messages are delivered to CelestaCX via webhook. Agents respond from the Agent Desk and replies are sent back to the customer through the bot.
 ```
Customer opens Telegram bot
and sends a message
 │
 ▼
Telegram Bot API
 │
 Webhook delivery
 │
 ▼
CelestaCX Channel Connector
 │
 ▼
Routing Engine → Queue → Agent Desk
``` 
Each Telegram channel in CelestaCX corresponds to one Telegram bot. If you want to present multiple distinct bots to customers — for example, a support bot and a sales bot — each requires a separate connector.
 
---
 
### Supported Features
 | Feature | Supported |
| --- | --- |
| Inbound text messages | Yes |
| Outbound text messages | Yes |
| Inbound images, video, audio, documents | Yes |
| Outbound images, video, audio, documents | Yes |
| Inbound voice messages | Yes — displayed as audio attachment |
| Inbound stickers | Yes — displayed as image |
| Inbound location sharing | Yes — displayed as coordinates |
| Inline keyboard buttons | Yes |
| Custom keyboard (reply keyboard) | Yes |
| Bot commands (e.g. /start, /help) | Yes |
| Bot handoff | Yes |
| Group chats | No — one-to-one conversations only |
| Channels (broadcast) | No |
| Message read receipts | No |
| Typing indicators | Yes — CelestaCX sends typing action while agent is composing | 
---
 
### Prerequisites
 
Telegram bot setup is straightforward compared to Meta and LinkedIn — no business account verification or API access approval is required. All you need is a Telegram account to create and manage bots.
 
Before configuring Telegram in CelestaCX:
 
- A Telegram account (personal or organisational) to create the bot
- A bot created via **BotFather** (Telegram's official bot management bot)
- The bot's **API token** , obtained from BotFather during creation
 
---
 
### Creating a Telegram Bot
 
All Telegram bots are created and managed through **BotFather** — a special Telegram bot that handles bot registration.
 
1. Open Telegram and search for **@BotFather** .
2. Start a conversation with BotFather and send the command `/newbot` .
3. Follow the prompts:
 
- Enter a **display name** for your bot (shown to users in Telegram — e.g. "Acme Support").
- Enter a **username** for your bot (must end in `bot` — e.g. `AcmeSupportBot` ). This is the unique identifier customers use to find your bot.
4. BotFather will respond with your bot's **API token** — a string in the format `123456789:ABCdefGhIJKlmNoPQRsTUVwxyZ` . Copy this immediately and store it securely.
5. Optionally configure additional bot properties via BotFather:
 
- `/setdescription` — text shown to users before they start the bot
- `/setabouttext` — text shown on the bot's profile page
- `/setuserpic` — bot profile picture
- `/setcommands` — define command shortcuts visible to users (e.g. `/start` , `/help` , `/agent` )
 
> **Security note:** Your bot's API token grants full control over the bot. Treat it like a password — do not share it publicly or commit it to version control. If a token is compromised, regenerate it immediately via BotFather using `/revoke` .
 
---
 
### Configuring Telegram in CelestaCX
 
1. Navigate to **Administration → Channels → Add Channel → Telegram** .
2. Enter a display name for this channel (e.g. "Telegram — Acme Support Bot").
3. Enter the following:
 | Field | Description |
| --- | --- |
| Bot Token | The API token provided by BotFather when the bot was created |
| Webhook Verify Token | A string you define, used as an additional security layer when verifying incoming webhook requests | 
1. Copy the **CelestaCX Webhook URL** displayed on the configuration page.
2. CelestaCX will automatically register this webhook URL with Telegram when you save the configuration — you do not need to register it manually in BotFather or the Telegram API.
3. Click **Test Connection** . CelestaCX will send a test request to the Telegram API to confirm the bot token is valid and the webhook is active.
4. Create a routing rule for Telegram interactions: **Administration → Routing → Routing Rules → Add Rule** — set the trigger to this Telegram channel.
5. Save and set the channel status to **Active** .
 
> **Note:** Telegram only allows one active webhook per bot. If a webhook was previously registered for this bot (e.g. from a different integration), CelestaCX will overwrite it during setup. Ensure no other system is actively using this bot before connecting it to CelestaCX.
 
---
 
### Bot Commands
 
Telegram bots support **commands** — predefined shortcuts prefixed with `/` that customers can send to trigger specific actions. Commands are visible to customers as a menu in the Telegram interface when they tap the command button.
 
#### Common Commands for Support Bots
 | Command | Suggested Use |
| --- | --- |
| /start | Welcome message and introduction to the bot |
| /help | List of available commands and how to get assistance |
| /agent | Request to be connected to a human agent |
| /status | Check the status of an existing case or request |
| /hours | Display business hours | 
Commands are registered with BotFather using `/setcommands` and handled in CelestaCX via routing rules or bot logic. When a customer sends a command, it arrives in CelestaCX as a standard inbound interaction with the command text as the message content.
 
If you are using a bot for pre-qualification before handoff to a human agent, commands can be used to initiate specific bot flows.
 
---
 
### Inline Keyboards & Custom Keyboards
 
CelestaCX supports two types of interactive button elements in Telegram:
 
#### Inline Keyboards
 
Buttons that appear directly below a specific message. Each button has a label and a callback data value that is returned to CelestaCX when tapped. Inline keyboard buttons remain visible in the conversation history after being tapped.
 
**Use case:** Presenting structured choices to the customer — for example, selecting a topic category, confirming an action, or navigating a self-service menu.
 
#### Custom Keyboards (Reply Keyboards)
 
Buttons that replace the customer's standard Telegram keyboard. Tapping a button sends the button's label as a text message. Custom keyboards are dismissed when the conversation ends or the keyboard is removed.
 
**Use case:** Guiding customers through structured flows where responses should be constrained to a predefined set of options.
 
Both keyboard types are configurable through the CelestaCX message template builder under **Administration → Channels → [Channel] → Message Templates** .
 
---
 
### Conversation Behaviour
 
Telegram does not impose conversation window restrictions like Meta's platforms. Agents can send messages to customers at any time through the bot, as long as the customer has started a conversation with the bot at least once (Telegram requires user-initiated contact before a bot can message a user).
 | Capability | Supported |
| --- | --- |
| Responding to customer-initiated messages | Yes |
| Sending follow-up messages after a conversation has ended | Yes — no window restriction |
| Proactive outbound to users who have never messaged the bot | No — Telegram requires user-initiated contact |
| Blocking detection | No — Telegram does not notify bots when a user blocks them | 
---
 
### Rich Media Handling
 | Media Type | Inbound | Outbound | Max Size | Notes |
| --- | --- | --- | --- | --- |
| Images (JPEG, PNG) | Yes | Yes | 10 MB | Compressed by Telegram above 10 MB |
| Documents (any format) | Yes | Yes | 50 MB | Sent uncompressed |
| Video (MP4) | Yes | Yes | 50 MB | |
| Audio (MP3, M4A) | Yes | Yes | 50 MB | Displayed as audio player |
| Voice messages (OGG) | Yes | No | — | Inbound only — displayed as audio |
| Stickers (WebP, TGS) | Yes | No | — | Displayed as image |
| Animated GIFs | Yes | Yes | 50 MB | Sent as document to preserve animation |
| Location | Yes | No | — | Displayed as coordinates with map link |
| Contact cards | Yes | No | — | Displayed as contact details | 
All received media is stored with the interaction record for the duration of your data retention period.
 
---
 
### Bot Handoff
 
CelestaCX supports bot-to-human handoff on Telegram. When a bot is handling a pre-qualification flow and determines that human assistance is needed — either because the customer requested it or because the bot cannot resolve the query — the conversation is passed to CelestaCX's routing engine and assigned to a human agent.
 
The handoff process:
 
1. Bot completes its pre-qualification flow and signals handoff to CelestaCX.
2. CelestaCX creates a routable interaction with the context collected by the bot (e.g. customer intent, language, selected topic).
3. The routing engine applies rules based on this context and routes the interaction to the appropriate queue.
4. An agent accepts the interaction and the conversation continues seamlessly — the customer sees no interruption in the Telegram conversation thread.
 
During bot handling, the interaction is not visible in agent queues. Once handoff is triggered, it enters the queue normally.
 
---
 
### Multiple Telegram Bots
 
Each Telegram bot is a separate channel connector in CelestaCX. Multiple bots can be configured to serve different purposes — for example, a customer support bot and a sales enquiry bot — each routing to different queues.
 
To add a second Telegram bot:
 
1. Create a new bot via BotFather (or use an existing bot with a separate token).
2. Navigate to **Administration → Channels → Add Channel → Telegram** .
3. Follow the same configuration steps with the new bot's token.
4. Create routing rules specific to this bot.
 
---
 
### Troubleshooting
 | Symptom | Likely Cause | Action |
| --- | --- | --- |
| Inbound messages not appearing in CelestaCX | Webhook registration failed or bot token incorrect | Verify bot token is correct. Navigate to Administration → Channels → [Channel] → Reregister Webhook to force webhook re-registration. |
| "Unauthorised" error on Test Connection | Bot token is invalid or has been revoked | Regenerate the bot token in BotFather using /revoke and update the connector. |
| Messages arriving but replies not sending | Bot has been blocked by the customer | No action possible — Telegram does not expose block status to bots. |
| Webhook conflict error | Another system has registered a webhook for this bot | Ensure no other platform or integration is using the same bot token. Delete the existing webhook via the Telegram API if necessary: https://api.telegram.org/bot[TOKEN]/deleteWebhook |
| Bot not responding to /start command | /start command not handled in routing or bot logic | Ensure a routing rule or bot flow handles the /start command payload. |
| Large file attachments not received | File exceeds Telegram's 50 MB bot API limit | Files above 50 MB cannot be received via the bot API. Customers must use alternative channels for large files. | 
---
 
*What's next: Proceed to* [*Viber*](#) *to configure Viber Business messaging.*
