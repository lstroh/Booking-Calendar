Great — let’s break down **“Parallel booking support (e.g., consultation during long job)”** into **requirements** and **success criteria** so you’ll have a checklist to confirm implementation in the plugin.

---

## **Requirements Breakdown**

### 1. Core Functionality

* [ ] System allows **more than one booking to occur at overlapping times** if business rules permit.
* [ ] Admin can configure **which services/resources allow parallel booking** (e.g., “consultation” can overlap with “main job”, but two main jobs cannot).
* [ ] Booking form supports selecting a **secondary/parallel service** linked to the primary booking.
* [ ] Each booking (primary and parallel) is stored as a **separate record** but linked together under a job ID or group.


### 2. Scheduling Rules

* [ ] Admin can define **max number of parallel bookings** per resource/tradesperson.
* [ ] Parallel bookings **cannot conflict** if they require the same exclusive resource (e.g., one tradesperson).
* [ ] If parallel service uses a different resource (e.g., admin staff for consultation), system checks that person’s availability separately.
* [ ] Booking duration logic supports overlap — e.g., a 1-hour consultation inside an 8-hour main job.





### 3. User Experience

* [ ] Client-facing form makes it clear when a **parallel service** is being booked (e.g., “Book consultation during your installation”).
* [ ] Confirmation screen and emails list **both the main booking and the parallel session** with their times.
* [ ] Calendar view shows overlapping bookings visually (e.g., stacked events, side-by-side).






### 4. Payment & Invoicing

* [ ] Payment can be handled either:

  * As a **single combined invoice** for main + parallel service, or
  * As **separate line items** configurable by admin.
* [ ] Total cost clearly reflects parallel services (no double-charging).


### 5. Admin Controls

* [ ] Admin can see linked parallel bookings in dashboard/job view.
* [ ] Admin can reschedule/cancel a parallel booking without affecting the main booking (and vice versa).
* [ ] Reports and exports treat parallel bookings as distinct but linked entities.




### 6. Notifications

* [ ] Notifications (email/SMS) include both main and parallel bookings.
* [ ] Rescheduling prompts system to **check conflicts** with parallel bookings.
* [ ] Clients and staff receive clear details on both bookings.



## **Success Criteria**

✅ Admin can configure which services support parallel booking.
✅ Client can add a parallel service during or after booking the main job.
✅ Parallel bookings are stored as linked but distinct records.
✅ Scheduling rules prevent conflicts when services share resources.
✅ Calendar view displays overlapping bookings clearly.
✅ Confirmation and reminder messages list both main and parallel bookings.
✅ Payment/invoices show both services accurately.
✅ Admin can manage parallel bookings independently (reschedule, cancel, report).
✅ Works for different scenarios:

* Consultation during job.
* Overlapping jobs by different staff.
* Add-on services (e.g., inspection, delivery).

---

Would you like me to also **map possible technical approaches** (e.g., custom booking table with a `parent_job_id` and `booking_type` column, or extending WP post relationships) so you have developer-facing guidance too?

