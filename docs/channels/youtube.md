# YouTube & Google Play Store
Here is the **YouTube & Google Play Store** page:
 
---
 
## YouTube & Google Play Store
 
CelestaCX supports monitoring and responding to YouTube comments and Google Play Store reviews from the Agent Desk. These channels differ from direct messaging channels — interactions are publicly visible, responses are public, and the primary use case is brand reputation management and public engagement rather than private customer support.
 
---
 
### How These Channels Work in CelestaCX
 
Both YouTube and Google Play Store are connected via the **Google APIs** . CelestaCX polls for new comments and reviews at regular intervals, converts them into interactions, and routes them to the appropriate queue. Agents respond from the Agent Desk and replies are published publicly on the respective platform.
 ```
Customer posts a YouTube comment
or Google Play Store review
 │
 ▼
Google API (YouTube Data API / Google Play Developer API)
 │
 Polled by CelestaCX
 │
 ▼
CelestaCX Channel Connector
 │
 ▼
Routing Engine → Queue → Agent Desk
``` 
Because these channels are publicly visible, responses posted by agents through CelestaCX are immediately visible to all users of the respective platform. Agents should be aware that their replies are public and represent your organisation's official position.
 
---
 
### YouTube
 
#### Supported Features
 | Feature | Supported |
| --- | --- |
| Monitoring video comments | Yes |
| Responding to video comments | Yes |
| Monitoring comment replies (threads) | Yes |
| Responding to comment replies | Yes |
| Liking a comment | No |
| Hiding or holding comments for review | No — via CelestaCX |
| Deleting comments | No — via CelestaCX |
| Community posts | No |
| YouTube Live chat | No |
| Private messages | No — YouTube does not support business DMs | 
#### Prerequisites
 
Before configuring YouTube in CelestaCX:
 
- A **Google account** with ownership or management access to the YouTube channel you want to monitor
- A **Google Cloud project** with the **YouTube Data API v3** enabled
- OAuth 2.0 credentials created in the Google Cloud Console
- The YouTube channel must have comments enabled — channels with comments disabled will not generate interactions
 
#### Setting Up Google Cloud Credentials
 
1. Log in to the **Google Cloud Console** at [console.cloud.google.com](http://console.cloud.google.com) .
2. Create a new project or use an existing one.
3. Navigate to **APIs & Services → Library** and enable the **YouTube Data API v3** .
4. Navigate to **APIs & Services → Credentials → Create Credentials → OAuth 2.0 Client ID** .
5. Set the application type to **Web Application** .
6. Add the CelestaCX OAuth redirect URL to the **Authorised Redirect URIs** (provided on the YouTube channel configuration page in CelestaCX).
7. Note the **Client ID** and **Client Secret** .
 
#### Configuring YouTube in CelestaCX
 
1. Navigate to **Administration → Channels → Add Channel → YouTube** .
2. Enter a display name for this channel (e.g. "YouTube — Acme Official Channel").
3. Enter the **Client ID** and **Client Secret** from your Google Cloud project.
4. Click **Authorise with Google** . You will be redirected to Google to grant CelestaCX permission to access your YouTube channel's comments on behalf of your organisation. Sign in with the Google account that has management access to the YouTube channel.
5. Once authorised, select the **YouTube channel** to monitor from the dropdown (if the authorised account manages multiple channels).
6. Configure monitoring settings:
 | Setting | Description | Default |
| --- | --- | --- |
| Polling Interval | How frequently CelestaCX checks for new comments (seconds) | 120 |
| Monitor All Videos | Monitor comments on all videos on the channel | Enabled |
| Specific Video IDs | If Monitor All Videos is disabled, enter the IDs of specific videos to monitor | — |
| Minimum Comment Length | Ignore comments shorter than this character count — useful for filtering out emoji-only or spam comments | 5 |
| Create Interactions for Replies | Whether to create interactions for replies to existing comments, not just top-level comments | Enabled | 
1. Create a routing rule for YouTube interactions: **Administration → Routing → Routing Rules → Add Rule** — set the trigger to this YouTube channel.
2. Save and set the channel status to **Active** .
 
#### How YouTube Interactions Appear in the Agent Desk
 
YouTube interactions display the following in the interaction panel:
 
- The comment text
- The commenter's YouTube display name and profile picture (if public)
- The video title and a link to the specific video
- The timestamp of the comment
- Any existing replies to the comment (thread view)
- A reply composer for posting a public response
 
Agents cannot edit or delete comments through CelestaCX. Moderation actions (hiding, deleting, or marking as spam) must be performed directly in YouTube Studio.
 
#### Routing YouTube Comments
 
YouTube comment volumes can vary enormously depending on the nature of your content and audience. Consider the following when designing routing:
 | Scenario | Recommendation |
| --- | --- |
| Low volume — occasional comments on product or support videos | Route to existing social or support queue, blended with other channels |
| High volume — active brand channel with many comments | Dedicated YouTube queue served by a social media or community team |
| Mixed content — support queries and general comments | Use subject keyword routing to separate support-related comments from general engagement |
| Viral or high-attention periods | Configure overflow rules to manage volume spikes — consider temporarily increasing queue capacity | 
---
 
### Google Play Store
 
#### Supported Features
 | Feature | Supported |
| --- | --- |
| Monitoring app reviews | Yes |
| Responding to app reviews | Yes |
| Editing a previous response | Yes |
| Deleting a response | No — via CelestaCX |
| Monitoring ratings without text | No — rating-only reviews without text are not converted to interactions |
| Multiple apps | Yes — one connector per app |
| iOS App Store | No — Apple does not provide a public API for review responses | 
#### Prerequisites
 
Before configuring Google Play Store in CelestaCX:
 
- A **Google Play Developer account** with access to the app you want to monitor
- Access to the **Google Play Developer API** — this must be enabled for your Google Play Developer account
- A **Google Cloud project** linked to your Google Play Developer account with the **Google Play Developer API** enabled
- A **Service Account** with the appropriate permissions to access review data
 
#### Setting Up Google Play API Access
 
Google Play Store uses **Service Account** authentication rather than OAuth 2.0 user authorisation. This is because review access is managed at the developer account level rather than tied to a specific user.
 
1. Log in to the **Google Cloud Console** .
2. Create a new project or use an existing one linked to your Google Play Developer account.
3. Navigate to **APIs & Services → Library** and enable the **Google Play Developer API** .
4. Navigate to **IAM & Admin → Service Accounts → Create Service Account** .
5. Give the service account a name (e.g. "CelestaCX Play Store Integration").
6. Grant the service account the **Editor** role at the project level.
7. Under the service account, navigate to **Keys → Add Key → Create New Key → JSON** . Download the JSON key file — this contains the credentials CelestaCX needs.
8. In the **Google Play Console** , navigate to **Setup → API Access** .
9. Link the Google Cloud project to your Google Play Developer account.
10. Grant the service account access to your Play Developer account with at least **Reply to Reviews** permission.
 
#### Configuring Google Play Store in CelestaCX
 
1. Navigate to **Administration → Channels → Add Channel → Google Play Store** .
2. Enter a display name for this channel (e.g. "Google Play — Acme App").
3. Upload the **Service Account JSON key file** downloaded from the Google Cloud Console.
4. Enter the **Package Name** of the app you want to monitor (e.g. `com.acme.app` ).
5. Configure monitoring settings:
 | Setting | Description | Default |
| --- | --- | --- |
| Polling Interval | How frequently CelestaCX checks for new reviews (seconds) | 300 |
| Minimum Star Rating | Only create interactions for reviews at or below this star rating — useful for focusing on negative reviews | 3 |
| Include 4–5 Star Reviews | Toggle to also monitor positive reviews for acknowledgement and engagement | Disabled |
| Language Filter | Only create interactions for reviews in specific languages — leave blank to monitor all languages | All languages | 
1. Create a routing rule for Google Play Store interactions.
2. Save and set the channel status to **Active** .
 
#### How Google Play Interactions Appear in the Agent Desk
 
Google Play Store interactions display the following in the interaction panel:
 
- The review text
- The reviewer's display name (Google may anonymise some reviewer names)
- The star rating (1–5)
- The app version the reviewer was using
- The review timestamp
- Any existing response previously posted (if a response has already been given)
- A reply composer for posting or editing a public response
 
#### Responding to Reviews
 
When an agent submits a response through CelestaCX, it is published immediately as the official developer response on the Google Play Store listing. The response is publicly visible to all users viewing the app.
 
Key points about Google Play responses:
 
- Each review can have only one developer response. If a response already exists, submitting a new one via CelestaCX will **replace** the previous response.
- Responses cannot be deleted through CelestaCX — deletion must be done directly in the Google Play Console.
- Google notifies the reviewer by email when a response is posted, which may prompt further engagement.
- Response length is limited to 350 characters.
 
#### Routing Google Play Reviews
 | Scenario | Recommendation |
| --- | --- |
| Primarily negative review management | Set Minimum Star Rating to 3 to focus on 1–3 star reviews requiring response |
| Full review engagement including positive | Enable Include 4–5 Star Reviews and route all ratings to a community or CX team |
| Multi-language app with global reviews | Use Language Filter to route reviews by language to appropriately skilled agents |
| Multiple apps | Create one connector per app, each with its own routing rules | 
---
 
### Managing Multiple YouTube Channels or Apps
 
#### Multiple YouTube Channels
 
If your organisation manages multiple YouTube channels, each channel requires a separate connector. Each connector uses its own OAuth authorisation — the authorising Google account must have management access to the specific YouTube channel.
 
#### Multiple Google Play Apps
 
If your organisation publishes multiple apps on Google Play, each app requires a separate connector in CelestaCX. The same Service Account JSON key file can be used across multiple connectors provided the service account has been granted access to all relevant apps in the Google Play Console.
 
---
 
### Public Response Best Practices
 
Because YouTube comments and Google Play reviews are publicly visible, responses carry additional weight compared to private messaging channels. The following practices apply to both channels:
 
- **Respond promptly to negative feedback.** Public unanswered complaints are visible to prospective customers. Aim to respond to 1–3 star reviews within 24 hours.
- **Keep responses professional and constructive.** Avoid defensive or dismissive language. Acknowledge the customer's experience and offer a path to resolution.
- **Move sensitive conversations to private channels.** For complaints requiring personal data or detailed investigation, acknowledge publicly and invite the customer to contact you via email or direct message for resolution.
- **Acknowledge positive feedback.** Brief, genuine responses to positive reviews signal that your organisation is actively engaged with customers.
- **Do not reproduce customer complaints verbatim** in your response — this amplifies the negative content for readers who have not scrolled to read the review.
- **Establish response templates** for common review types to ensure consistency and reduce agent effort. Templates can be configured under **Administration → Channels → [Channel] → Message Templates** .
 
---
 
### Troubleshooting
 
#### YouTube
 | Symptom | Likely Cause | Action |
| --- | --- | --- |
| Comments not appearing as interactions | OAuth token expired or YouTube Data API quota exceeded | Reauthorise the channel via Administration → Channels → [Channel] → Reauthorise. Check Google Cloud Console for API quota usage. |
| Replies not posting | Authorised account does not have comment management permission on the channel | Verify the Google account used for authorisation has appropriate YouTube channel management access. |
| Only some videos being monitored | Monitor All Videos disabled and video IDs not listed | Enable Monitor All Videos or add the relevant video IDs under channel settings. |
| High volume of spam or short comments creating interactions | Minimum Comment Length too low | Increase Minimum Comment Length under channel settings to filter out short or low-quality comments. | 
#### Google Play Store
 | Symptom | Likely Cause | Action |
| --- | --- | --- |
| Reviews not appearing as interactions | Service account not linked to Play Developer account or insufficient permissions | Verify service account is granted Reply to Reviews permission in Google Play Console → Setup → API Access. |
| "Permission Denied" error | Google Play Developer API not enabled or service account key invalid | Confirm the Google Play Developer API is enabled in the Google Cloud project and the JSON key file is valid and not expired. |
| Responses not posting to Play Store | Response exceeds 350 character limit | Check response length in the Agent Desk composer. Shorten response to under 350 characters. |
| Previous response overwritten unexpectedly | Agent submitted a new response to a review that already had a response | Remind agents that submitting a response replaces any existing developer response. Review interaction history before responding. |
| Reviews in unexpected languages appearing | Language Filter not configured | Set Language Filter to specific languages under channel settings if language-based filtering is required. | 
For persistent issues, check connector logs under **Platform Administration → Logs → Channel Connectors** or refer to the [Troubleshooting Guide](#) .
 
---
 
*What's next: Proceed to* [*Voice & Video*](#) *to configure voice calling, IVR integration, and video calling.*
