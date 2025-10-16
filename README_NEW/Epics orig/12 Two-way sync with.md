Got it — let’s break down **“Admin Experience → Calendar & Availability → Two-way sync with Google/Outlook/Apple Calendar”** into checkable items so you can verify plugin capabilities.

---

## 🔍 Breakdown Into Checkable Items

### 1. **Integration Setup**

* [ ] Does the plugin offer **direct integration** with Google Calendar (via Google Calendar API)?
* [ ] Does it support **Outlook / Office 365 / Microsoft Exchange** integration (via Microsoft API)?
* [ ] Does it support **Apple Calendar / iCal standard** (either direct sync or via ICS feed)?
* [ ] Is there an **OAuth login flow** (e.g., “Connect your Google account”)?
* [ ] Can multiple staff each connect **their own calendar accounts**?


### 2. **Two-Way Sync (Bookings → External Calendar)**

* [ ] Are new bookings **automatically pushed** to connected external calendars?
* [ ] Are booking updates (reschedule, cancel, change of details) **updated in external calendars**?
* [ ] Do buffer times (if configured) **show in external calendars** as blocked time?
* [ ] Can you choose which **calendar to push events into** (e.g., “Work” vs “Personal”)?
* [ ] Are booking details **customisable** (title, description, client info)?



### 3. **Two-Way Sync (External Calendar → Plugin)**

* [ ] Do events created directly in external calendars **block availability** in the booking system?
* [ ] Are recurring external events **handled correctly** (block availability on all repeating instances)?
* [ ] Are event updates/cancellations in external calendars **reflected in the plugin**?
* [ ] Can the admin choose to **ignore certain external calendars** (e.g., only sync “Work,” not “Personal”)?



### 4. **Conflict Handling & Permissions**

* [ ] Does the plugin **check for conflicts** between new bookings and external calendar events?
* [ ] If there’s a conflict, is the slot **automatically hidden** from clients?
* [ ] Are there **permission settings** to prevent staff from accidentally overwriting external events?



### 5. **Performance & Reliability**

* [ ] What is the **sync frequency** (real-time push, every X minutes, manual refresh)?
* [ ] Does it use **webhooks or polling** (important for reliability)?
* [ ] Is sync **bi-directional by default** or configurable (e.g., one-way only)?
* [ ] Are sync errors **logged or notified** to admins?


### 6. **Privacy & Security**

* [ ] Is OAuth used instead of storing passwords?
* [ ] Are synced events **stored in WordPress** or only referenced?
* [ ] Can staff revoke access to their connected calendars?


✅ **Success Criteria:**

* Supports **Google, Outlook, Apple/iCal** with proper APIs.
* Bookings are **pushed to external calendars** and **external events block availability**.
* Sync is **real-time or near real-time** and reliable.
* Each staff member can **manage their own calendar connection**.
* Conflicts are **prevented automatically**.

---

Do you want me to also prepare a **test scenario checklist** (e.g., “Step 1: Create a booking → Check Google Calendar → Step 2: Create event in Outlook → See if blocked in booking form…”) so you can run structured plugin evaluations?
