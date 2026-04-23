# SMS (Twilio & SMPP)

Here is the **SMS (Twilio & SMPP)** page:

---

## SMS (Twilio & SMPP)

CelestaCX supports two-way SMS messaging through two integration methods: **Twilio** (a cloud-based SMS provider) and **SMPP** (Short Message Peer-to-Peer, a direct protocol connection to an SMS aggregator or mobile network operator). This page covers both methods, how to choose between them, and how to configure each in CelestaCX.

---

### How SMS Works in CelestaCX

Regardless of the connection method, SMS interactions follow the same path through CelestaCX once they are received:

```
Customer sends SMS
to your number
        │
        ▼
SMS Provider (Twilio or SMPP aggregator)
        │
  Webhook / SMPP delivery
        │
        ▼
CelestaCX Channel Connector
        │
        ▼
Routing Engine → Queue → Agent Desk
```

Agents respond from the Agent Desk. Replies are delivered back through the same provider to the customer's mobile number.

---

### Twilio vs. SMPP — Choosing the Right Method

| Consideration | Twilio | SMPP |
| --- | --- | --- |
| Setup complexity | Low — cloud-based, API-driven, minimal infrastructure | Higher — requires SMPP server configuration and network connectivity |
| Infrastructure required | None — fully managed by Twilio | SMPP relay or aggregator server required |
| Number provisioning | Self-service via Twilio console | Via your SMPP aggregator or MNO |
| Global coverage | Extensive — 180+ countries | Depends on aggregator agreements |
| Throughput | Moderate — suitable for most contact centre volumes | High — better for very large send volumes |
| Cost model | Per-message pricing via Twilio | Typically negotiated rate with aggregator |
| Alphanumeric sender IDs | Supported in eligible countries | Supported — depends on aggregator |
| Short codes | Supported | Supported |
| Long codes | Supported | Supported |
| Best for | Most deployments — straightforward setup, good global reach | High-volume operations, existing SMPP infrastructure, specific MNO relationships |

For most CelestaCX deployments, **Twilio is the recommended starting point** due to its ease of setup, self-service number provisioning, and comprehensive API. SMPP is better suited to operations with existing SMS infrastructure, very high throughput requirements, or specific regional aggregator relationships.

---

### SMS Number Types

Before configuring either connection method, understand the number type you will use:

| Number Type | Description | Best For |
| --- | --- | --- |
| Long Code | Standard 10-digit (or international equivalent) phone number | Two-way conversational SMS, lower volumes |
| Short Code | 5–6 digit number — requires carrier approval (US/Canada) | High-volume campaigns, transactional alerts |
| Alphanumeric Sender ID | A name instead of a number (e.g. "AcmeSupport") — outbound only, not available in all countries | Branded notifications, one-way alerts |
| Toll-Free Number | A toll-free number enabled for SMS (US/Canada) | Two-way SMS, moderate volumes, consumer-friendly |

> **Important:** Two-way SMS (inbound and outbound) requires a long code, short code, or toll-free number. Alphanumeric sender IDs are outbound only — customers cannot reply to them. Ensure your number type matches your use case before provisioning.

---

### SMS Compliance

SMS messaging is subject to significant legal and regulatory requirements that vary by country. Before going live with any SMS channel, ensure your operation complies with the relevant regulations for each country you will be messaging.

Key compliance areas:

| Requirement | Description |
| --- | --- |
| Opt-in consent | Customers must have explicitly consented to receive SMS messages from your organisation. The nature of the required consent varies by country and message type. |
| Opt-out handling | Customers must be able to opt out by replying STOP (or local equivalent). Opt-out requests must be honoured immediately. CelestaCX handles inbound STOP messages automatically — see Opt-Out Handling below. |
| Sender identification | Messages must clearly identify the sending organisation. |
| Time restrictions | Many jurisdictions restrict the hours during which marketing SMS can be sent. |
| Do Not Contact lists | Opted-out numbers must be maintained on a suppression list and excluded from future sends. |

> CelestaCX provides the technical capability to send and receive SMS — compliance with applicable regulations is the responsibility of your organisation. Engage your legal team before launching SMS communications.

---

### Configuring SMS via Twilio

#### Prerequisites

Before configuring the Twilio connector in CelestaCX:

- A **Twilio account** — sign up at [http://twilio.com](http://twilio.com)
- A **Twilio phone number** with SMS capability provisioned in your Twilio console
- Your Twilio **Account SID** and **Auth Token** from the Twilio console dashboard
- If using a Messaging Service (recommended for production): a **Messaging Service SID**

#### Setting Up Twilio

1. Log in to your Twilio console at [http://console.twilio.com](http://console.twilio.com) .
2. Provision an SMS-capable phone number under **Phone Numbers → Manage → Buy a Number**.
3. Note your **Account SID** and **Auth Token** from the console dashboard.
4. Optionally, create a **Messaging Service** under **Messaging → Services → Create Messaging Service**. Messaging Services provide additional features including number pooling, sticky sender (routing replies back through the same number), and compliance tools. For production deployments, using a Messaging Service is recommended over a single number.

#### Configuring the Twilio Connector in CelestaCX

1. Navigate to **Administration → Channels → Add Channel → SMS → Twilio**.
2. Enter a display name for this channel (e.g. "SMS — Acme Support UK").
3. Enter the following credentials:

| Field | Description |
| --- | --- |
| Account SID | From your Twilio console dashboard |
| Auth Token | From your Twilio console dashboard |
| Phone Number or Messaging Service SID | The Twilio phone number (E.164 format, e.g. +441234567890) or Messaging Service SID (MGXXXXXXXX) |
| Webhook Verify Token | A string you define for webhook request validation |

1. Copy the **CelestaCX Webhook URL** from the configuration page.
2. In the Twilio console, configure the webhook URL for your phone number or Messaging Service:

  - For a single number: **Phone Numbers → Manage → Active Numbers → [Number] → Messaging → A Message Comes In → Webhook** — paste the CelestaCX Webhook URL.
  - For a Messaging Service: **Messaging → Services → [Service] → Integration → Incoming Messages → Send a Webhook** — paste the CelestaCX Webhook URL.
3. Set the HTTP method to **POST**.
4. Click **Test Connection** in CelestaCX to verify credentials and webhook delivery.
5. Create a routing rule for SMS interactions: **Administration → Routing → Routing Rules → Add Rule** — set the trigger to this SMS channel.
6. Save and set the channel status to **Active**.

---

### Configuring SMS via SMPP

#### Prerequisites

Before configuring the SMPP connector in CelestaCX:

- An active account with an **SMPP aggregator** or direct connection to a mobile network operator
- SMPP connection credentials from your aggregator:

  - SMPP server hostname or IP address
  - SMPP server port
  - System ID (username)
  - Password
  - System Type (if required by your aggregator)
- A **source address** (sender number or alphanumeric ID) provisioned by your aggregator
- Network access from the CelestaCX deployment to the SMPP server on the required port (typically 2775 or 2776)

#### SMPP Connection Modes

CelestaCX supports the following SMPP bind modes:

| Bind Mode | Description |
| --- | --- |
| Transceiver (TRX) | Single connection for both sending and receiving messages. Recommended for most deployments. |
| Transmitter (TX) + Receiver (RX) | Separate connections for sending and receiving. Required by some aggregators. |

#### Configuring the SMPP Connector in CelestaCX

1. Navigate to **Administration → Channels → Add Channel → SMS → SMPP**.
2. Enter a display name for this channel (e.g. "SMS — SMPP Enterprise").
3. Enter the SMPP connection details:

| Field | Description |
| --- | --- |
| SMPP Host | Hostname or IP address of your aggregator's SMPP server |
| SMPP Port | Port number — typically 2775 (standard) or 2776 (TLS) |
| System ID | Your SMPP username provided by the aggregator |
| Password | Your SMPP password provided by the aggregator |
| System Type | Optional — provided by aggregator if required |
| Bind Mode | Transceiver, or Transmitter + Receiver |
| Source Address | The sender number or alphanumeric ID for outbound messages |
| Source Address TON | Type of Number for the source address (0 = Unknown, 1 = International, 5 = Alphanumeric) |
| Source Address NPI | Numbering Plan Indicator (typically 0 = Unknown or 1 = ISDN/E.164) |
| TLS Enabled | Toggle on if connecting via TLS (port 2776 or as specified by aggregator) |

1. Configure connection resilience settings:

| Field | Description | Default |
| --- | --- | --- |
| Enquire Link Interval | How frequently CelestaCX sends keepalive signals to maintain the SMPP connection (seconds) | 30 |
| Reconnect Interval | How long to wait before attempting to reconnect after a connection drop (seconds) | 10 |
| Max Reconnect Attempts | Number of reconnection attempts before raising an alert | 5 |

1. Click **Test Connection** to verify CelestaCX can establish an SMPP bind with your aggregator's server.
2. Create a routing rule for SMS interactions from this connector.
3. Save and set the channel status to **Active**.

---

### Opt-Out Handling

CelestaCX automatically handles inbound opt-out messages for SMS. When a customer replies with a recognised opt-out keyword, CelestaCX:

1. Records the opt-out against the customer's number in the platform.
2. Prevents further outbound SMS to that number from CelestaCX.
3. Sends an automatic confirmation reply (configurable) acknowledging the opt-out.
4. Logs the opt-out event in the interaction record.

#### Recognised Opt-Out Keywords

CelestaCX recognises the following opt-out keywords by default (case-insensitive):

`STOP`, `STOPALL`, `UNSUBSCRIBE`, `CANCEL`, `END`, `QUIT`

#### Recognised Opt-In Keywords

Customers can re-subscribe after opting out by sending:

`START`, `YES`, `UNSTOP`

#### Managing Opt-Out Lists

The opt-out list is accessible under **Administration → Channels → [Channel] → Opt-Out List**. From here, administrators can:

- View all opted-out numbers with opt-out timestamp and triggering keyword
- Manually add numbers to the opt-out list
- Manually remove numbers from the opt-out list (e.g. following a documented customer request to re-subscribe)
- Export the opt-out list as CSV for compliance reporting

> **Important:** Removing a number from the opt-out list should only be done following a documented, explicit opt-in from the customer. Sending SMS to opted-out numbers without renewed consent may violate applicable law.

---

### Character Limits & Encoding

Standard SMS messages are limited to 160 characters (GSM-7 encoding) or 70 characters (Unicode/UCS-2, used for non-Latin scripts such as Arabic, Chinese, or Cyrillic). CelestaCX handles encoding automatically based on message content.

Messages exceeding the single-message character limit are sent as **concatenated SMS** (multiple SMS parts joined together by the receiving handset). Each part counts as a separate message for billing purposes.

| Encoding | Single SMS Limit | Concatenated Part Limit |
| --- | --- | --- |
| GSM-7 (Latin characters) | 160 characters | 153 characters per part |
| Unicode / UCS-2 (non-Latin) | 70 characters | 67 characters per part |

The Agent Desk message composer displays a character count and part count indicator to help agents stay within efficient message lengths.

---

### Multiple SMS Numbers

CelestaCX supports multiple SMS numbers or SMPP connections within a single deployment. Each number or connection is configured as a separate channel connector with its own routing rules.

Common multi-number scenarios:

| Scenario | Approach |
| --- | --- |
| Separate numbers for different brands | One connector per number, each with its own routing rules |
| Separate numbers for different regions | One connector per number, routed to region-specific queues |
| High-volume: number pool via Twilio Messaging Service | Single Twilio connector using a Messaging Service with multiple pooled numbers |
| Mixed Twilio + SMPP | Separate connectors for each — no conflict, routed independently |

---

### Troubleshooting

#### Twilio

| Symptom | Likely Cause | Action |
| --- | --- | --- |
| Inbound SMS not appearing in CelestaCX | Webhook URL not configured in Twilio console | Verify webhook URL is set on the number or Messaging Service in Twilio console. |
| "Authentication Failed" error | Incorrect Account SID or Auth Token | Re-enter credentials from the Twilio console dashboard. |
| Outbound messages not delivered | Number not SMS-enabled or Twilio account suspended | Check number capabilities in Twilio console. Verify account status. |
| STOP replies not being honoured | Twilio opt-out handling conflict | Ensure Twilio's built-in opt-out management is disabled if CelestaCX is handling opt-outs — duplicate handling can cause conflicts. |

#### SMPP

| Symptom | Likely Cause | Action |
| --- | --- | --- |
| SMPP bind failing | Incorrect credentials or network connectivity issue | Verify System ID, password, host, and port. Confirm network access from CelestaCX to SMPP server on the configured port. |
| Connection drops frequently | Enquire Link interval too long for aggregator's timeout | Reduce Enquire Link interval to 20 seconds or as recommended by your aggregator. |
| Inbound messages not arriving | Receiver bind not established | If using Transmitter + Receiver mode, confirm both binds are active in connector status. |
| Outbound messages queued but not sent | Transmitter bind dropped | Check connector status for bind state. Reconnection will be attempted automatically — verify after reconnect interval. |
| Messages delivered but with wrong sender ID | Source address or TON/NPI misconfigured | Verify Source Address, TON, and NPI values with your aggregator. |

For persistent issues, check connector logs under **Platform Administration → Logs → Channel Connectors** or refer to the [Troubleshooting Guide](#).

---

*What's next: Proceed to*[*Email*](#)*to configure the email channel.*
