Excellent — this one’s important because it touches both **data minimisation** and **calendar integration privacy** under GDPR.
Let’s break it down into **auditable items** you can check against your plugin’s implementation (for Google Calendar or Outlook).

---

### ✅ **Feature: Calendar Privacy – “Store personal calendar events as ‘busy’ only”**

**Category:** GDPR & UK Data Protection Act
**Priority:** Must-Have
**Effort:** 16h
**Risk:** High (GDPR compliance)
**Dependencies:** Google Calendar API / Microsoft Outlook API

---

### 🔍 **Breakdown for Plugin Audit**

#### 1. **Calendar Connection Scope**

| Item | Description                                                                                                                                                   | How to Check                                                       |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| 1.1  | The plugin requests **only minimal permissions** from Google or Outlook (e.g., `read-only busy/free` or `calendar.events.list` with `private` fields masked). | Review OAuth scopes in the integration setup or developer console. |
| 1.2  | The plugin does **not** store or sync event details (title, attendees, description) unless explicitly needed.                                                 | Check what fields are fetched from API responses.                  |

#### 2. **Privacy Mode Option**

| Item | Description                                                                                      | How to Check                                                                         |
| ---- | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| 2.1  | Users can **toggle** between “Show details” and “Busy only” when connecting a calendar.          | Look for a setting in plugin UI (e.g., “Privacy mode” or “Display only busy times”). |
| 2.2  | Default behaviour should be **“Busy only”** for compliance (minimise data).                      | Check plugin defaults or onboarding instructions.                                    |
| 2.3  | When “Busy only” is enabled, events are displayed as **blocked slots** with no identifying info. | Inspect the front-end booking calendar rendering.                                    |



#### 3. **Data Storage & Processing**

| Item | Description                                                                                                  | How to Check                                                |
| ---- | ------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------- |
| 3.1  | Calendar event data is **not stored permanently** in plugin DB — only temporary availability data is cached. | Inspect DB schema or API logs for stored fields.            |
| 3.2  | If event details are stored (for scheduling logic), there is **clear user consent**.                         | Check consent flow or settings.                             |
| 3.3  | No event metadata (titles, attendees, descriptions) is sent to external analytics or third-party APIs.       | Check webhook, API, or analytics integration configuration. |



#### 4. **User Consent & Transparency**

| Item | Description                                                                                          | How to Check                                               |
| ---- | ---------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| 4.1  | During calendar connection, users see a **disclosure or consent prompt** (explaining what’s synced). | Inspect UI/UX or connection modal.                         |
| 4.2  | The plugin’s **Privacy Policy** mentions how personal calendar data is processed.                    | Review linked Privacy Policy or admin settings.            |
| 4.3  | Users can **disconnect** the calendar and revoke permissions easily.                                 | Test disconnect flow and check whether tokens are deleted. |



#### 5. **Compliance & API References**

| Item | Description                                                                                                                                  | How to Check                             |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| 5.1  | Google Calendar integration follows [Google OAuth verification guidelines](https://developers.google.com/workspace/marketplace/verify).      | Review Google Cloud project status.      |
| 5.2  | Outlook integration follows [Microsoft Graph privacy and permissions policy](https://learn.microsoft.com/en-us/graph/permissions-reference). | Review Azure App registration.           |
| 5.3  | The plugin provides instructions for **revoking data access** via Google/Microsoft dashboards.                                               | Check documentation or support articles. |



Would you like me to turn this into a **checklist table (CSV format)** so you can directly tick off what’s implemented in the plugin?
