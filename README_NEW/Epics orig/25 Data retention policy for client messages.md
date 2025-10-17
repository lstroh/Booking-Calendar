Excellent — this one covers **data retention and deletion** for client communications (messages, chat logs, etc.), which is a **core GDPR Article 5(1)(e)** requirement: *personal data must not be kept longer than necessary*.

Here’s your full breakdown to audit whether the plugin is compliant and correctly implements a **data retention policy for client messages** 👇

---

## ✅ **Feature: GDPR & UK Act – “Data retention policy for client messages”**

**Category:** GDPR & UK Data Protection Act
**Priority:** Must-Have
**Effort:** 8 h
**Risk:** High
**Requirement:** Requires GDPR compliance

---

### 🔍 **Breakdown for Plugin Audit**

#### 1. **Retention Policy Definition**

| Item | Description                                                                                   | How to Check                                          |
| ---- | --------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| 1.1  | Plugin defines how long client messages are stored (e.g., 12 months, 24 months).              | Check plugin settings or documentation.               |
| 1.2  | Retention period is **justified by business need** (e.g., dispute resolution or audit).       | Review privacy policy or data handling documentation. |
| 1.3  | Policy includes both **sent** and **received** messages (email, SMS, in-app, WhatsApp, etc.). | Confirm supported message channels.                   |


#### 2. **Automated Retention Enforcement**

| Item | Description                                                                                        | How to Check                                   |
| ---- | -------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| 2.1  | Plugin automatically **deletes or anonymises** client messages after the defined retention period. | Look for a scheduled cleanup or cron job.      |
| 2.2  | System supports **different retention periods** per message type (operational vs marketing).       | Check admin or developer options.              |
| 2.3  | Deleted data is **irretrievable** — no shadow copies or backups holding unexpired personal data.   | Confirm via database structure or plugin docs. |



#### 3. **Manual Deletion Controls**

| Item | Description                                                                        | How to Check                        |
| ---- | ---------------------------------------------------------------------------------- | ----------------------------------- |
| 3.1  | Admins can **manually delete client message history** per user or booking.         | Check message management UI.        |
| 3.2  | Plugin provides **bulk deletion** or “Delete all messages older than X days.”      | Inspect admin tools or settings.    |
| 3.3  | Deletion triggers also remove **attachments, photos, or logs** linked to messages. | Review how deletion cascades in DB. |

#### 4. **User Rights & Transparency**

| Item | Description                                                                 | How to Check                               |
| ---- | --------------------------------------------------------------------------- | ------------------------------------------ |
| 4.1  | Privacy notice clearly states **how long messages are kept** and why.       | Review privacy notice or legal templates.  |
| 4.2  | Users can **request deletion** of their message history (Right to Erasure). | Check for GDPR request tools in plugin.    |
| 4.3  | Users are notified if retention changes (e.g., update in policy).           | Review communications policy or changelog. |


#### 5. **Compliance & Security**

| Item | Description                                                               | How to Check                          |
| ---- | ------------------------------------------------------------------------- | ------------------------------------- |
| 5.1  | Message data stored in encrypted format at rest (DB or API storage).      | Verify encryption configuration.      |
| 5.2  | Only authorised admins can access message logs (access control enforced). | Review role permissions.              |
| 5.3  | Plugin logs **retention actions** (what was deleted, when, by whom).      | Check for audit trail or log entries. |


Would you like me to convert this into a **CSV checklist** (columns: *Item | Description | How to Check | Status*) so you can use it for systematic plugin verification like the previous ones?
