# Email
Here is the **Email** page:
 
---
 
## Email
 
CelestaCX supports a full email channel, enabling agents to receive, manage, and respond to customer emails from the Agent Desk alongside all other channels. Email remains the preferred channel for formal, complex, or document-heavy interactions — complaints, contracts, regulatory enquiries, and detailed technical support cases. This page covers how to connect an email account to CelestaCX, configure routing, manage threading, and handle attachments.
 
---
 
### How Email Works in CelestaCX
 
CelestaCX connects to email accounts via **IMAP** (for receiving) and **SMTP** (for sending). Incoming emails are fetched from the mailbox at regular intervals, converted into CelestaCX interactions, and routed to the appropriate queue. Agents reply from the Agent Desk and outbound emails are sent via the configured SMTP server.
 ```
Customer sends email
to your support address
 │
 ▼
Email Server (IMAP mailbox)
 │
 Polled by CelestaCX
 │
 ▼
CelestaCX Email Connector
(converts to interaction, extracts metadata)
 │
 ▼
Routing Engine → Queue → Agent Desk
``` 
Replies from agents follow the reverse path — composed in the Agent Desk, sent via SMTP, and delivered to the customer's inbox as a standard email reply.
 
---
 
### Supported Features
 | Feature | Supported |
| --- | --- |
| Inbound email via IMAP | Yes |
| Outbound email via SMTP | Yes |
| HTML email composition | Yes |
| Plain text email composition | Yes |
| Inbound attachments | Yes |
| Outbound attachments | Yes |
| Email threading (conversation view) | Yes |
| CC and BCC on replies | Yes |
| Auto-acknowledgement replies | Yes |
| Email signatures | Yes — per agent and per queue |
| Routing by sender domain | Yes |
| Routing by subject line keywords | Yes |
| Routing by recipient address | Yes |
| Shared mailbox support | Yes |
| OAuth 2.0 authentication (Microsoft 365, Google Workspace) | Yes |
| Basic authentication (IMAP/SMTP username + password) | Yes — where supported by provider |
| Email forwarding mode | Yes — as alternative to direct IMAP connection |
| Spam filtering | Basic — relies on upstream email server filtering | 
---
 
### Prerequisites
 
Before configuring email in CelestaCX, you need:
 
- A dedicated **email address** for the support channel (e.g. [support@acme.com](#) or [helpdesk@acme.com](#) )
- IMAP and SMTP access enabled on the email account
- Authentication credentials — either OAuth 2.0 (recommended for Microsoft 365 and Google Workspace) or username and password
- For Microsoft 365: an Azure AD app registration with Mail.Read and Mail.Send permissions (if using OAuth)
- For Google Workspace: a Google Cloud project with Gmail API enabled and appropriate OAuth scopes (if using OAuth)
 
#### Dedicated Mailbox Recommendation
 
CelestaCX monitors the entire inbox of the connected mailbox. For this reason, always use a **dedicated mailbox** for each CelestaCX email channel — do not connect a shared or personal mailbox that receives non-support email. Unrelated emails arriving in the mailbox will be converted into interactions and routed to agents.
 
---
 
### Authentication Methods
 
CelestaCX supports two authentication methods for connecting to email accounts:
 
#### OAuth 2.0 (Recommended)
 
OAuth 2.0 is the modern, secure authentication standard used by Microsoft 365 and Google Workspace. It does not require storing a password in CelestaCX — instead, CelestaCX is granted permission to access the mailbox through an authorisation flow.
 
**Microsoft 365 / Exchange Online:**
 
1. Register an application in **Azure Active Directory → App Registrations → New Registration** .
2. Add the following API permissions under **Microsoft Graph** :
 
- `Mail.Read` — to read incoming emails via IMAP
- `Mail.Send` — to send emails via SMTP
- `offline_access` — to maintain access without repeated re-authorisation
3. Grant admin consent for the permissions.
4. Generate a **Client Secret** under **Certificates & Secrets** .
5. Note the **Application (Client) ID** , **Directory (Tenant) ID** , and **Client Secret** .
 
**Google Workspace / Gmail:**
 
1. Create a project in the **Google Cloud Console** .
2. Enable the **Gmail API** .
3. Create OAuth 2.0 credentials under **APIs & Services → Credentials → Create Credentials → OAuth 2.0 Client ID** .
4. Set the authorised redirect URI to the CelestaCX OAuth redirect URL (provided on the email channel configuration page).
5. Note the **Client ID** and **Client Secret** .
 
#### Basic Authentication (Username + Password)
 
Some email providers still support basic authentication using a username (email address) and password or app-specific password. Basic authentication is simpler to configure but less secure — use it only where OAuth 2.0 is not available.
 
> **Note:** Microsoft 365 has disabled basic authentication for most account types. If you are connecting a Microsoft 365 mailbox, OAuth 2.0 is required. Google Workspace similarly recommends OAuth 2.0 and may disable basic authentication for accounts with advanced security policies enabled.
 
---
 
### Configuring Email in CelestaCX
 
#### Step 1 — Add the Channel
 
1. Navigate to **Administration → Channels → Add Channel → Email** .
2. Enter a display name for this channel (e.g. "Email — Acme Support").
3. Enter the **from address** — the email address that will appear in the From field of all outbound emails from this channel (e.g. [support@acme.com](#) ).
4. Enter the **display name** that will appear alongside the from address in the customer's inbox (e.g. "Acme Customer Support").
 
#### Step 2 — Configure Inbound (IMAP)
 | Field | Description |
| --- | --- |
| IMAP Host | Your email server's IMAP hostname (e.g. imap.gmail.com , outlook.office365.com ) |
| IMAP Port | 993 (IMAP over TLS — recommended) or 143 (IMAP with STARTTLS) |
| TLS | Enabled (recommended) |
| Authentication Method | OAuth 2.0 or Basic |
| Username | The email address of the mailbox (for Basic auth) |
| Password / App Password | The account password or app-specific password (for Basic auth) |
| OAuth Credentials | Client ID, Client Secret, Tenant ID (Microsoft) or Client ID, Client Secret (Google) — for OAuth |
| Polling Interval | How frequently CelestaCX checks for new emails (default: 60 seconds) |
| Monitored Folder | The mailbox folder to monitor for incoming emails (default: Inbox) |
| Processed Folder | The folder emails are moved to after being converted to interactions (default: Processed) | 
#### Step 3 — Configure Outbound (SMTP)
 | Field | Description |
| --- | --- |
| SMTP Host | Your email server's SMTP hostname (e.g. smtp.gmail.com , smtp.office365.com ) |
| SMTP Port | 587 (SMTP with STARTTLS — recommended) or 465 (SMTP over TLS) |
| TLS / STARTTLS | Enabled (recommended) |
| Authentication Method | OAuth 2.0 or Basic (must match inbound authentication method) |
| Username | The email address used for SMTP authentication | 
#### Step 4 — Test the Connection
 
1. Click **Test Inbound Connection** — CelestaCX will attempt to connect to the IMAP server and list the monitored folder. A success message confirms the connection is working.
2. Click **Test Outbound Connection** — CelestaCX will send a test email to the configured from address via SMTP. Check the mailbox to confirm receipt.
3. If either test fails, review the error message and verify the credentials and server settings.
 
#### Step 5 — Create Routing Rules and Save
 
1. Create routing rules for this email channel: **Administration → Routing → Routing Rules → Add Rule** — set the trigger to this email channel and configure conditions based on sender domain, recipient address, or subject keywords as needed.
2. Save the channel configuration and set the status to **Active** .
 
---
 
### Email Routing Options
 
CelestaCX can route email interactions based on several attributes extracted from the incoming email:
 | Routing Attribute | Description | Example |
| --- | --- | --- |
| Recipient address | The To address the email was sent to | Route emails to billing@acme.com to the Billing queue |
| Sender domain | The domain of the customer's email address | Route emails from @enterprise-client.com to the Enterprise queue |
| Subject keywords | Words or phrases in the email subject line | Route emails containing "urgent" or "complaint" to a high-priority queue |
| Sender address | A specific customer email address | Route emails from a VIP contact to a dedicated queue |
| Attachment present | Whether the email contains an attachment | Route emails with attachments to a team equipped to handle documents | 
Multiple conditions can be combined in a single routing rule. Rules are evaluated in priority order as with all other channel types.
 
#### Catch-All Routing
 
Ensure a default routing rule exists to catch any emails that do not match a more specific rule. Without a catch-all rule, unmatched emails will not be routed and will remain in the connector's backlog.
 
---
 
### Email Threading
 
CelestaCX maintains email conversations as threaded interactions. When a customer replies to an agent's email, CelestaCX matches the reply to the original interaction using the email's **Message-ID** and **In-Reply-To** headers.
 
Threading behaviour:
 | Scenario | Behaviour |
| --- | --- |
| Customer replies to an agent email | Reply is appended to the existing interaction thread |
| Customer starts a new email (new subject, no reply headers) | A new interaction is created |
| Agent replies from the Agent Desk | Reply uses the original Message-ID for threading — customer's email client shows it as part of the same conversation |
| Customer emails a different address (new recipient) | A new interaction is created, even if content is related |
| Interaction is closed and customer emails again | A new interaction is created — the previous thread is referenced in interaction history | 
Threading relies on standard email headers. Some email clients or forwarding configurations strip these headers, which can cause replies to be treated as new interactions. If threading issues are occurring, check whether email forwarding is stripping headers — see Email Forwarding Mode below.
 
---
 
### Auto-Acknowledgement
 
CelestaCX can send an automatic acknowledgement email to customers when their email is received and converted into an interaction. Auto-acknowledgement reassures customers that their email has been received and sets expectations for response time.
 
To configure auto-acknowledgement:
 
1. Navigate to **Administration → Channels → [Channel] → Auto-Acknowledgement** .
2. Enable auto-acknowledgement and compose the acknowledgement email body.
3. Optionally include the interaction reference number in the acknowledgement using the `{{interaction_id}}` variable — this allows customers to quote their reference number in follow-up communications.
4. Set the From name and address for the acknowledgement (defaults to the channel's configured from address).
5. Save.
 
Auto-acknowledgement emails are sent immediately upon interaction creation, before the interaction is assigned to an agent.
 
---
 
### Email Signatures
 
CelestaCX supports email signatures at two levels:
 | Level | Description | Configured In |
| --- | --- | --- |
| Queue signature | A standard signature applied to all emails sent from a specific queue | Administration → Queues → [Queue] → Email Signature |
| Agent signature | A personal signature that agents can configure for their own replies | Agent Desk → Profile → Email Signature | 
When both a queue signature and an agent signature are configured, the agent signature takes precedence. Agents can preview and edit their signature in the Agent Desk message composer before sending.
 
Signatures support plain text and basic HTML formatting — logos and images can be included via HTML image tags referencing publicly accessible URLs.
 
---
 
### Attachments
 
#### Inbound Attachments
 
CelestaCX accepts all standard email attachment types. Received attachments are:
 
- Displayed in the interaction panel with filename, type, and size
- Accessible for download by the handling agent
- Stored with the interaction record for the duration of the data retention period
- Visible in interaction history after the interaction is closed
 
#### Inbound Attachment Limits
 | Limit | Value |
| --- | --- |
| Maximum attachment size per email | 25 MB (total across all attachments) |
| Maximum number of attachments per email | 20 | 
Emails with attachments exceeding these limits are still received as interactions, but oversized attachments are truncated and flagged in the interaction panel with a warning.
 
#### Outbound Attachments
 
Agents can attach files to outbound emails from the Agent Desk message composer. Attachments can be:
 
- Uploaded from the agent's local machine
- Selected from the CelestaCX media library (if configured)
 
Outbound attachment limits are governed by the receiving email server's limits, which CelestaCX cannot control. As a general guideline, keep outbound attachments under 10 MB to ensure reliable delivery across all email providers.
 
---
 
### Email Forwarding Mode
 
As an alternative to direct IMAP connection, CelestaCX supports **email forwarding mode** — where your email server forwards incoming messages to a CelestaCX-provided inbound address rather than CelestaCX polling your mailbox via IMAP.
 
#### Advantages of Forwarding Mode
 
- No IMAP credentials required in CelestaCX
- Works with email servers that do not support IMAP or have restricted IMAP access
- Lower polling overhead — interactions are created immediately on receipt rather than at the next poll interval
- Compatible with complex email routing setups where emails arrive via distribution lists or forwarding rules
 
#### Configuring Forwarding Mode
 
1. Navigate to **Administration → Channels → Add Channel → Email → Forwarding Mode** .
2. CelestaCX provides a unique **inbound email address** for the channel (e.g. `channel-abc123@inbound.celestacx.com` ).
3. In your email server or email provider, create a forwarding rule that forwards all emails arriving at your support address to the CelestaCX inbound address.
4. Configure outbound (SMTP) settings as normal — forwarding mode only affects inbound receipt.
5. Test by sending an email to your support address and confirming it appears as an interaction in CelestaCX.
 
> **Note:** Email forwarding mode may affect threading reliability if the forwarding configuration modifies email headers. Test threading behaviour thoroughly before going live with forwarding mode.
 
---
 
### CC and BCC Handling
 
#### Inbound CC
 
If a customer copies additional recipients on their email, the CC addresses are captured and displayed in the interaction panel. Agents can see who was copied but cannot directly manage the CC recipients' experience.
 
#### Agent CC and BCC on Replies
 
When composing a reply in the Agent Desk, agents can:
 
- Add CC recipients — addresses added to CC receive a copy of the reply
- Add BCC recipients — addresses added to BCC receive a copy of the reply invisibly
 
CC and BCC fields are available in the email reply composer. BCC is not visible to the customer or CC recipients.
 
---
 
### Spam & Unwanted Email Handling
 
CelestaCX does not include a built-in spam filter — it relies on upstream filtering by your email server or provider. To prevent spam emails from creating interactions in CelestaCX:
 
- Ensure spam filtering is enabled and tuned on your email server
- Configure your email server to move spam to a separate folder before CelestaCX polls the inbox
- Set the CelestaCX monitored folder to a folder that receives only legitimate customer email
 
If spam emails do arrive as interactions, agents can mark them as spam in the Agent Desk. Marked interactions are closed and the sender address is added to a local suppression list under **Administration → Channels → [Channel] → Suppressed Senders** . Future emails from suppressed senders are not converted into interactions.
 
---
 
### Troubleshooting
 | Symptom | Likely Cause | Action |
| --- | --- | --- |
| Inbound emails not appearing as interactions | IMAP connection failing or polling interval issue | Run Test Inbound Connection. Check IMAP credentials and server settings. Verify the monitored folder name matches exactly. |
| OAuth authentication failing (Microsoft 365) | Insufficient API permissions or admin consent not granted | Verify Mail.Read and Mail.Send permissions are granted with admin consent in Azure AD. |
| OAuth authentication failing (Google Workspace) | OAuth scope not approved or redirect URI mismatch | Verify Gmail API is enabled and the redirect URI in Google Cloud Console matches the CelestaCX OAuth redirect URL exactly. |
| Outbound emails not delivered | SMTP authentication failing or port blocked | Run Test Outbound Connection. Check SMTP credentials, port, and TLS settings. Confirm port 587 or 465 is not blocked by your network. |
| Email replies creating new interactions instead of threading | Reply headers stripped by forwarding or email client | Check whether email forwarding is stripping Message-ID and In-Reply-To headers. Consider switching to forwarding mode or reviewing forwarding rules. |
| Auto-acknowledgement not sending | SMTP connection issue or auto-acknowledgement disabled | Verify SMTP connection is working and auto-acknowledgement is enabled under Administration → Channels → [Channel] → Auto-Acknowledgement. |
| Attachments missing from received interactions | Attachment exceeds size limit | Check attachment size against the 25 MB limit. Oversized attachments are flagged in the interaction panel. |
| Emails from a specific sender not creating interactions | Sender is on the suppression list | Check Administration → Channels → [Channel] → Suppressed Senders and remove the sender if suppression was applied in error. | 
For persistent issues, check connector logs under **Platform Administration → Logs → Channel Connectors** or refer to the [Troubleshooting Guide](#) .
 
---
 
*What's next: Proceed to* [*YouTube & Google Play Store*](#) *to configure comment monitoring and response on those platforms.*
