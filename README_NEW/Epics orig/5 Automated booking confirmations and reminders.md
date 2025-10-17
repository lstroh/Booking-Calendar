Perfect — let’s break down **“Automated booking confirmations and reminders”** into concrete items you can **verify in the plugin**.

---

## **Requirements Breakdown**

### 1. Trigger Points

* [ ] **Booking confirmation** is automatically sent immediately after a booking is made (via form or admin).
* [ ] **Reminders** are sent before the booking, at configurable intervals (e.g., 24h before, 1h before).
* [ ] **Cancellation/reschedule confirmation** is sent automatically if booking status changes.



### 2. Message Content

* [ ] Messages include key details:

  * Client name
  * Service booked
  * Date & time
  * Location (if applicable)
  * Any special instructions
* [ ] Messages are **customisable via admin panel** (templates or WYSIWYG editor).
* [ ] Templates support **shortcodes/variables** (e.g., {client_name}, {booking_date}).



### 3. Delivery Channels

* [ ] Email confirmation is sent by default.
* [ ] SMS/WhatsApp reminders are supported if API integration is enabled (Twilio/SendGrid/WhatsApp Business).
* [ ] Clients can opt to receive multiple channels (email + SMS).


### 4. Admin Controls

* [ ] Admin can **turn confirmations/reminders on/off** globally.
* [ ] Admin can set **reminder timing rules** (e.g., only send if booking made > X hours in advance).
* [ ] Admin can preview templates before activating.
* [ ] Failed delivery logs are available (e.g., bounced email, failed SMS).



### 5. Client Experience

* [ ] Client receives confirmation within a minute of booking.
* [ ] Reminders arrive reliably at the set times.
* [ ] If booking is rescheduled, the system cancels old reminders and generates new ones.
* [ ] If booking is cancelled, no reminders are sent.





### 6. Compliance

* [ ] All notifications respect **GDPR opt-out for marketing** but allow **operational messages** (like confirmations).
* [ ] SMS/email provider handles **data retention** securely.


## **Success Criteria Checklist**

✅ Confirmation email/SMS is sent automatically after booking.
✅ Reminder messages are sent at admin-defined intervals.
✅ Message content is accurate, with placeholders filled in correctly.
✅ Templates are customisable in the admin dashboard.
✅ Notifications are sent only via enabled channels (email/SMS/etc.).
✅ If booking is updated, notifications adjust accordingly.
✅ System prevents duplicate/conflicting notifications.
✅ Delivery failures are logged for troubleshooting.
✅ Operational notifications remain GDPR-compliant.

---

Do you want me to also **add recommended default reminder timings** (e.g., 24h before + 1h before) so you have a baseline to check against?

