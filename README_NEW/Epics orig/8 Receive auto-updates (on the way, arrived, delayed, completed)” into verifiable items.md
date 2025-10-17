Perfect — let’s break down **“Receive auto-updates (on the way, arrived, delayed, completed)”** into **verifiable items** so you can check if the plugin really implements it.

---

## **Requirements Breakdown**

### 1. Trigger Events

* [ ] Updates can be triggered when the tradesperson changes the **job/booking status** in the admin (e.g., “On the way”).
* [ ] Status options available at minimum: **On the way, Arrived, Delayed, Completed**.
* [ ] Admin can customise/add additional statuses if needed.


### 2. Client Notifications

* [ ] Clients receive **real-time notifications** when the status changes.
* [ ] Supported channels:

  * [ ] Email (default, required).
  * [ ] SMS (if Twilio/SendGrid integrated).
  * [ ] WhatsApp (if WhatsApp Business API integrated).
* [ ] Notifications include:

  * Service booked.
  * Job date/time.
  * New status (e.g., “Your tradesperson is on the way”).


### 3. Admin Controls

* [ ] Admin can **enable/disable** auto-updates.
* [ ] Admin can **customise message templates** per status (with placeholders like {client_name}, {status}, {time}).
* [ ] Admin can **choose which channels** each status is sent to (email only, SMS + email, etc.).
* [ ] Option to trigger updates **manually** (send “On the way” message without changing full job status).


### 4. Conversation History

* [ ] All auto-updates appear in the **client’s conversation history** (alongside two-way messaging if available).
* [ ] Admin can see which auto-updates were sent, with **timestamp and delivery status**.

### 5. Reliability & Compliance

* [ ] If delivery fails (SMS/email bounce), system logs the failure.
* [ ] Auto-updates are classified as **operational notifications** (not marketing).
* [ ] GDPR: Clients cannot opt out of these if they have an active booking, but opt-out applies after job completion.


## **Success Criteria Checklist**

✅ Status options (On the way, Arrived, Delayed, Completed) are available and trigger notifications.
✅ Clients receive notifications in real time when status changes.
✅ Notifications are customisable by admin (templates + channels).
✅ Auto-updates appear in conversation history and logs.
✅ Delivery failures are logged.
✅ Compliance: operational updates only, GDPR-safe.

---

👉 Would you like me to **also map these auto-updates into the earlier “two-way messaging hub” feature**, so you can check if the plugin merges them into the same conversation thread (instead of sending them separately)? That way we can see if the plugin is siloed or unified in messaging.
