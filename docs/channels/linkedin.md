# LinkedIn

Here is the **LinkedIn** page:

---

## LinkedIn

CelestaCX supports LinkedIn messaging, enabling agents to handle LinkedIn Direct Messages from the Agent Desk alongside all other channels. LinkedIn is primarily used in B2B and professional services contexts, where customers and prospects engage through LinkedIn rather than consumer messaging platforms.

---

### How LinkedIn Works in CelestaCX

CelestaCX connects to LinkedIn via the **LinkedIn Marketing Developer Platform API**. When a customer sends a Direct Message to your LinkedIn Company Page, LinkedIn delivers it to CelestaCX via webhook. Agents respond from the Agent Desk and replies are delivered back through LinkedIn.

```
Customer sends LinkedIn
Direct Message to Company Page
        │
        ▼
LinkedIn Marketing Developer Platform API
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
| Inbound Direct Messages | Yes |
| Outbound Direct Messages | Yes |
| Text messages | Yes |
| Inbound image attachments | Yes |
| Outbound image attachments | Yes |
| File attachments (PDF, DOCX) | Inbound only |
| Message read receipts | Yes |
| Typing indicators | No |
| Bot handoff | No |
| Public post comments | No |
| LinkedIn InMail | No |
| Connection requests | No |

> **Note:** LinkedIn's API for messaging is more restrictive than Meta's platforms. Only Direct Messages sent to your Company Page are supported. InMail, connection requests, and public post comments are not accessible via the API.

---

### Prerequisites

Before configuring LinkedIn in CelestaCX:

- Your organisation must have a **LinkedIn Company Page**
- You must be a **Page Super Admin** on the Company Page
- Your organisation must apply for and be granted access to the **LinkedIn Marketing Developer Platform**
- A **LinkedIn Developer App** must be created and approved for the required API products

#### LinkedIn API Access Requirements

LinkedIn's messaging API is not publicly open — access requires approval through the LinkedIn Marketing Developer Platform program. The approval process involves:

1. Creating a LinkedIn Developer App at [**developer.linkedin.com**](http://developer.linkedin.com)
2. Applying for the **Marketing Developer Platform** product on your app
3. Applying specifically for the **Messaging API** access within that program
4. Providing LinkedIn with details about your use case and expected message volumes
5. Waiting for LinkedIn's review — this typically takes 2–4 weeks and is not guaranteed

> **Important:** LinkedIn API access approval is controlled entirely by LinkedIn and is not within the control of CelestaCX or your implementation partner. Begin the application process well in advance of your planned go-live date. If access is not granted, LinkedIn messaging cannot be enabled in CelestaCX.

---

### Setting Up Your LinkedIn Developer App

Once your Marketing Developer Platform access is approved:

1. Log in to [**developer.linkedin.com**](http://developer.linkedin.com) with a LinkedIn account that has Super Admin access to your Company Page.
2. Open your approved Developer App.
3. Under **Products**, confirm **Marketing Developer Platform** and **Messaging API** are active.
4. Under **Auth**, note your **Client ID** and generate a **Client Secret**.
5. Add the CelestaCX OAuth redirect URL to the **Authorised Redirect URLs** list:

  - This URL is provided on the LinkedIn channel configuration page in CelestaCX (Administration → Channels → Add Channel → LinkedIn).
6. Under **Settings**, note your **App ID**.

---

### Configuring LinkedIn in CelestaCX

1. Navigate to **Administration → Channels → Add Channel → LinkedIn**.
2. Enter a display name for this channel (e.g. "LinkedIn — Acme Company Page").
3. Enter the following credentials:

| Field | Description |
| --- | --- |
| Client ID | From your LinkedIn Developer App → Auth tab |
| Client Secret | Generated in your LinkedIn Developer App → Auth tab |
| Company Page ID | Your LinkedIn Company Page's numeric ID — found in the Page URL ( linkedin.com/company/[ID] ) |
| Webhook Verify Token | A string you define, used to verify the webhook handshake |

1. Click **Authorise with LinkedIn**. You will be redirected to LinkedIn to grant CelestaCX permission to access your Company Page's messages on behalf of your organisation. Complete the OAuth authorisation flow.
2. Once authorised, copy the **CelestaCX Webhook URL** and register it in your LinkedIn Developer App under **Webhooks**.
3. Subscribe to the `messageCreate` webhook event.
4. Click **Test Connection** in CelestaCX to verify the setup.
5. Create a routing rule for LinkedIn interactions: **Administration → Routing → Routing Rules → Add Rule** — set the trigger to this LinkedIn channel.
6. Save and set the channel status to **Active**.

---

### OAuth Token Management

LinkedIn uses OAuth 2.0 for authentication. The access token granted during setup has a limited validity period and must be refreshed periodically. CelestaCX handles token refresh automatically where LinkedIn's API supports it.

However, in some cases — particularly if the authorising admin's LinkedIn account loses Super Admin access to the Company Page, or if the Developer App credentials change — the token may become invalid and require manual reauthorisation.

| Symptom | Cause | Action |
| --- | --- | --- |
| Channel status shows "Token Expired" | OAuth token has expired and could not be refreshed | Re-authorise via Administration → Channels → [Channel] → Reauthorise |
| Channel status shows "Authorisation Revoked" | Admin revoked app access in LinkedIn settings | Re-authorise the app and verify the admin account still has Page Super Admin access |
| Messages arriving but replies not sending | Insufficient permissions granted during OAuth | Reauthorise ensuring all requested permissions are accepted |

> **Best practice:** The LinkedIn account used to authorise CelestaCX should be a dedicated service account with Super Admin access to the Company Page — not a personal LinkedIn account. This prevents the integration from breaking if an individual employee's access changes.

---

### Conversation Behaviour

Unlike WhatsApp and Facebook Messenger, LinkedIn does not impose a formal conversation window on business messaging. Agents can send Direct Messages to customers at any time, provided the customer has initiated contact first.

LinkedIn does not currently support outbound-initiated conversations via the API — CelestaCX can only respond to messages that the customer has sent first. Proactive outreach via LinkedIn messaging is not supported through the platform integration.

| Capability | Supported |
| --- | --- |
| Responding to customer-initiated messages | Yes |
| Proactive outbound message initiation | No |
| Following up after a conversation has gone quiet | Yes — no window restriction |
| Sending messages to any LinkedIn user | No — only to users who have messaged first |

---

### Rich Media Handling

| Media Type | Inbound | Outbound | Notes |
| --- | --- | --- | --- |
| Images (JPEG, PNG) | Yes | Yes | Max 5 MB outbound |
| Documents (PDF, DOCX) | Yes | No | Inbound displayed as attachment |
| Video | No | No | Not supported via Messaging API |
| Audio | No | No | Not supported via Messaging API |
| GIFs | No | No | Not supported via Messaging API |

Received attachments are stored with the interaction record and accessible to agents in the interaction panel.

---

### Routing Considerations for LinkedIn

LinkedIn interactions tend to differ in character from consumer support channels. Consider the following when designing routing for LinkedIn:

| Consideration | Recommendation |
| --- | --- |
| Message tone and complexity | LinkedIn messages are often more formal and may require more considered responses. Avoid very high concurrent capacity limits for agents handling LinkedIn. |
| Response time expectations | B2B customers on LinkedIn typically expect responses within a few hours rather than minutes. SLA targets and queue overflow settings should reflect this. |
| Agent skill requirements | Consider routing LinkedIn to agents with strong written communication skills and familiarity with your B2B products or services. |
| Volume | LinkedIn message volumes are typically lower than WhatsApp or email. LinkedIn may not need a dedicated team — consider blending it with email queues handled by the same agents. |

---

### Multiple Company Pages

If your organisation manages multiple LinkedIn Company Pages, each page requires a separate channel connector in CelestaCX. Each connector has its own OAuth authorisation, credentials, and routing rules.

To add a second LinkedIn Company Page:

1. Navigate to **Administration → Channels → Add Channel → LinkedIn**.
2. Follow the same configuration steps with the credentials for the additional Company Page.
3. Ensure the LinkedIn Developer App has been granted access for the additional Page.

---

### Troubleshooting

| Symptom | Likely Cause | Action |
| --- | --- | --- |
| Inbound messages not appearing | Webhook not registered or messageCreate event not subscribed | Verify webhook URL and event subscription in LinkedIn Developer App. |
| "Invalid Client" error | Incorrect Client ID or Client Secret | Re-enter credentials from LinkedIn Developer App → Auth tab. |
| Authorisation fails during OAuth flow | Authorising account does not have Super Admin access to the Company Page | Confirm the LinkedIn account used has Super Admin role on the Company Page. |
| Token expired and auto-refresh failing | Refresh token has also expired | Reauthorise the channel manually via Administration → Channels → [Channel] → Reauthorise. |
| Replies not delivered to customer | Customer has withdrawn connection or restricted messages | No action possible — LinkedIn does not expose restriction status to businesses. |
| App access application taking longer than expected | LinkedIn review backlog | Contact LinkedIn Developer Support for status updates. No workaround is available while under review. |

---

*What's next: Proceed to*[*Twitter / X*](#)*to configure Twitter/X Direct Messages and public mention handling.*
