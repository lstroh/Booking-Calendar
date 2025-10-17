Perfect — let’s break down **“Ability to split a job into multiple bookings”** into testable requirements and success criteria so you’ll know exactly what to look for in the plugin implementation.

---

## **Requirements Breakdown**

### 1. Job Definition

* [ ] Client (or admin) can define a job that requires **more time than one booking slot allows** (e.g., 8 hours when slots are 4 hours each).
* [ ] Option to select **“split across multiple bookings”** during the booking flow.
* [ ] Job retains a single **Job ID** even when split into multiple booking entries.



### 2. Scheduling Logic

* [ ] System suggests the **next available slots** automatically (e.g., two consecutive days or times).
* [ ] Client can manually select different days/times for split sessions.
* [ ] Minimum/maximum time per split segment configurable (e.g., cannot split into less than 2 hours).
* [ ] Ensure system prevents **overlap or double-booking** of resources/tradesperson.



### 3. Data Structure & Storage

* [ ] Each sub-booking is stored as a **linked record** under the parent job.
* [ ] Booking records include reference to parent job ID.
* [ ] Admin can view both:

  * Full job overview (total hours, all sessions).
  * Individual booking details.



### 4. User Experience

* [ ] Booking form clearly explains when a job is being split (e.g., “Your 8-hour job will be booked as 2 sessions of 4 hours each”).
* [ ] Calendar view shows split bookings linked together visually (e.g., grouped or color-coded).
* [ ] Confirmation email/SMS lists **all scheduled segments** with dates/times.




### 5. Payment & Invoicing

* [ ] Payment handled **once per job**, not separately for each booking, unless configured otherwise.
* [ ] Invoices/quotes reflect **total job cost** and note the split booking schedule.



### 6. Notifications & Rescheduling

* [ ] Notifications (email/SMS) include all scheduled split bookings.
* [ ] If one booking is rescheduled, system prompts whether to adjust other linked bookings.
* [ ] Admin can cancel or edit individual sessions without losing parent job record.


## **Success Criteria**

✅ Client/admin can book a job longer than available slots.
✅ System supports splitting into multiple bookings automatically or manually.
✅ All sub-bookings remain linked to a parent job ID.
✅ Double-booking of tradesperson is prevented.
✅ Clients see all their split bookings in confirmation and reminders.
✅ Payment is processed once for the entire job (unless configured otherwise).
✅ Admin calendar shows linked bookings clearly.
✅ Rescheduling/cancellation workflows support split bookings correctly.
✅ Works smoothly for both short (e.g., 2 sessions) and long (e.g., 5+ sessions) jobs.

---

Do you want me to also add **technical implementation notes** (like whether this should extend WordPress’s `post_parent` relationship or use custom booking tables), so it’s easier to guide a developer?
