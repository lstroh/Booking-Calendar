Great — let’s break down **“Right to opt out of marketing but still receive operational updates”** into clear **checklist items** you can use to verify plugin implementation.

---

## **Requirements Breakdown**

### 1. Client Opt-Out Options

* [ ] Clients have a **clear way to opt out of marketing communications** (e.g., checkbox in booking form, profile settings, or unsubscribe link in emails).
* [ ] Opt-out applies **only to marketing messages** (promotions, newsletters, referral campaigns).
* [ ] Opt-out does **not affect operational updates** (booking confirmations, reminders, status updates).



### 2. Message Categorisation

* [ ] Plugin distinguishes between **operational messages** and **marketing messages** in configuration.
* [ ] Operational = confirmations, reminders, cancellations, auto-updates.
* [ ] Marketing = newsletters, loyalty/referral promotions, special offers.
* [ ] Each message template is tagged as **“operational” or “marketing.”**


### 3. Admin Controls

* [ ] Admin can configure marketing campaigns separately from operational notifications.
* [ ] Admin can **see client consent status** (opted in/out of marketing).
* [ ] Marketing tools (Mailchimp, SendGrid, etc.) integrate with **subscription preferences** automatically.
* [ ] Opt-out does not delete client data but updates **consent flag in DB**.


### 4. Compliance & Transparency

* [ ] Unsubscribe links in all marketing emails are **clear and functional**.
* [ ] Consent status and changes are **logged with timestamp** (audit trail).
* [ ] Privacy notice (in booking form or client portal) explains the difference between operational vs marketing messages.
* [ ] If SMS/WhatsApp marketing is used, plugin supports **STOP/UNSUBSCRIBE commands**.


## **Success Criteria Checklist**

✅ Clients can opt out of marketing without losing booking confirmations/reminders.
✅ Plugin separates marketing vs operational templates.
✅ Admin can check client consent status in dashboard/CRM.
✅ Marketing integrations respect opt-out automatically.
✅ Opt-out preferences logged with timestamp.
✅ Unsubscribe links/buttons work in every marketing message.
✅ Privacy notice explains operational vs marketing categories.

---

👉 Do you want me to **also draft test cases** (e.g., “Opt out → still get booking confirmation, not promo email”) so you can run them against the plugin to validate this feature?
