Got it — let’s break down **“Cancellation and reschedule options for clients”** into concrete **checklist items** so you can verify plugin implementation.

---

## **Requirements Breakdown**

### 1. Client-Side Functionality

* [ ] Client can **cancel a booking** directly from their confirmation email or client portal.
* [ ] Client can **reschedule a booking** by selecting a new available date/time (with validation against admin’s availability).
* [ ] Cancellation/reschedule links are **secure and unique** to each booking (no shared links).
* [ ] When rescheduling, the system shows **real-time availability** (not just free-text request).
* [ ] Clients receive an **updated confirmation** after reschedule/cancellation.




### 2. Admin-Side Controls

* [ ] Admin can **enable/disable** cancellation and rescheduling (global setting).
* [ ] Admin can configure **cut-off rules** (e.g., cannot cancel within 24h of the job).
* [ ] Admin can set **reschedule rules** (e.g., only allow rescheduling up to 30 days ahead).
* [ ] Admin receives **notifications** of client-initiated cancellations/reschedules.
* [ ] Admin can override or lock bookings (e.g., jobs requiring prep time).




### 3. Notifications & Logs

* [ ] Cancellation/reschedule triggers an **automatic notification** to both client and admin.
* [ ] Updated booking details replace old reminders (so client doesn’t still get reminders for cancelled jobs).
* [ ] System keeps an **audit log** of cancellations/reschedules (who, when, reason if captured).



### 4. Compliance & UX

* [ ] GDPR-compliant handling of any cancellation “reason” text (if collected).
* [ ] Cancellation/reschedule options are **clearly visible** but not intrusive (e.g., link in confirmation email + client portal).
* [ ] If payment is involved:

  * [ ] Refunds/credits are triggered automatically or flagged for admin review.
  * [ ] Policy messaging (e.g., “Non-refundable within 24h”) is shown before finalising cancellation.




## **Success Criteria Checklist**

✅ Clients can cancel or reschedule bookings via portal/email.
✅ Links are secure and unique per booking.
✅ System checks admin’s availability before rescheduling.
✅ Cut-off and reschedule rules can be configured.
✅ Notifications go out to both client and admin.
✅ Old reminders are cancelled after changes.
✅ Audit log tracks all cancellations/reschedules.
✅ Refunds/credits follow policy rules (if payments integrated).

---

Would you like me to **also draft a “basic vs advanced” split** (MVP vs later enhancements) for this feature? That way you can test whether the plugin implements just the essentials or also supports the advanced rules like refund triggers and audit logs.
