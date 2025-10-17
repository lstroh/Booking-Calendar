Perfect — this one focuses on **user consent and withdrawal** under GDPR Articles 7 & 21, which are often missed in plugins that send marketing or automated emails/texts.
Here’s a full breakdown you can use to **audit whether the plugin meets GDPR opt-out requirements**.

---

## ✅ **Feature: GDPR & UK Act – “Clear opt-out options for marketing communications”**

**Category:** GDPR & UK Data Protection Act
**Priority:** Performance Driver
**Effort:** 12h
**Risk:** Medium
**Requirement:** Requires GDPR compliance

---

### 🔍 **Breakdown for Plugin Audit**

#### 1. **Consent Capture for Marketing**

| Item | Description                                                                                                         | How to Check                                                |
| ---- | ------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| 1.1  | Marketing communications (emails, SMS, push notifications) are **sent only to users who have explicitly opted in**. | Check registration or booking forms for consent checkboxes. |
| 1.2  | The **purpose of marketing consent** is clearly described (e.g., “Receive offers and service updates”).             | Review wording on forms and privacy notice.                 |
| 1.3  | Consent is **recorded with timestamp and method** (for audit purposes).                                             | Look for consent logs in the database or CRM integration.   |


#### 2. **Opt-Out Visibility**

| Item | Description                                                                                            | How to Check                                                                        |
| ---- | ------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------- |
| 2.1  | Every marketing email includes a **visible “Unsubscribe” link**.                                       | Inspect email templates or automation platform (e.g., SendGrid, Mailchimp, Twilio). |
| 2.2  | SMS or WhatsApp marketing messages include an **opt-out keyword** (e.g., “Reply STOP to unsubscribe”). | Review message templates in plugin or SMS gateway.                                  |
| 2.3  | Push notifications include a **disable toggle** in user preferences or app settings.                   | Check plugin dashboard or mobile app settings.                                      |


#### 3. **Opt-Out Functionality**

| Item | Description                                                                                                  | How to Check                                                         |
| ---- | ------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------- |
| 3.1  | Opting out updates user’s preference immediately — **no further marketing messages** after that.             | Review unsubscribe logic or test with a dummy account.               |
| 3.2  | Opt-out works **independently of transactional communications** (e.g., booking confirmations still allowed). | Verify messaging type separation in settings.                        |
| 3.3  | The system allows **partial opt-outs** (e.g., choose to receive only certain message types).                 | Look for marketing preference categories (optional but recommended). |


#### 4. **Record Keeping & Compliance**

| Item | Description                                                                   | How to Check                                   |
| ---- | ----------------------------------------------------------------------------- | ---------------------------------------------- |
| 4.1  | Opt-in and opt-out actions are **logged** with timestamp and user ID.         | Inspect audit trail or consent database table. |
| 4.2  | Plugin provides a **data export** of consent history for GDPR requests.       | Check admin settings or privacy tools.         |
| 4.3  | Consent records are stored securely and accessible only to authorised admins. | Review database permissions or documentation.  |



#### 5. **Privacy & Transparency**

| Item | Description                                                                                                                              | How to Check                                              |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| 5.1  | Privacy Policy clearly explains users’ right to opt out of marketing.                                                                    | Review plugin documentation or privacy templates.         |
| 5.2  | Users can access their marketing preferences easily from their account or booking page.                                                  | Check customer portal UI.                                 |
| 5.3  | The plugin integrates with **third-party APIs** (SendGrid, Twilio, WhatsApp) that support compliance (unsubscribe links, STOP keywords). | Verify integration settings and compliance configuration. |

Would you like me to format this as a **CSV checklist** (so you can mark “Implemented / Not Implemented / Needs Review” for each item)?
