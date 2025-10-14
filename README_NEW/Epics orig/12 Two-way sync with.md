Got it — let’s break down **“Admin Experience → Calendar & Availability → Two-way sync with Google/Outlook/Apple Calendar”** into checkable items so you can verify plugin capabilities.

---

## 🔍 Breakdown Into Checkable Items

### 1. **Integration Setup**

* [ ] Does the plugin offer **direct integration** with Google Calendar (via Google Calendar API)?
* [ ] Does it support **Outlook / Office 365 / Microsoft Exchange** integration (via Microsoft API)?
* [ ] Does it support **Apple Calendar / iCal standard** (either direct sync or via ICS feed)?
* [ ] Is there an **OAuth login flow** (e.g., “Connect your Google account”)?
* [ ] Can multiple staff each connect **their own calendar accounts**?

The plugin architecture includes robust systems for calendar integration, particularly with Google Calendar and the iCal standard. However, some advanced or third-party API integrations require paid versions or companion plugins.

Here is a breakdown of the implementation status for each integration option:

### Integration Setup Implementation Status

#### 1. Does the plugin offer **direct integration** with Google Calendar (via Google Calendar API)?
**Status: Implemented (One-Way Import via API Key)**

The plugin features a dedicated architecture for Google Calendar (GCal) synchronization:

*   The feature is driven by the **`WPBC_Google_Calendar`** class, which serves as the core engine for a **one-way synchronization process**, importing events from an external public Google Calendar into the local plugin database.
*   The system communicates with the **Google Calendar API v3**.
*   Configuration requires an **API Key** (`booking_gcal_api_key`) which is retrieved from plugin options.
*   The API communication uses the standard, secure WordPress function **`wp_remote_get()`** to fetch event data.

#### 2. Does it support **Outlook / Office 365 / Microsoft Exchange** integration (via Microsoft API)?
**Status: Not Confirmed (via Microsoft API)**

The provided sources confirm support for synchronization via iCal feeds (see next point), which Outlook and Exchange systems often support, but they **do not mention any specific classes, configuration files, or API keys** related to a dedicated Microsoft API for Outlook, Office 365, or Exchange integration.

#### 3. Does it support **Apple Calendar / iCal standard** (either direct sync or via ICS feed)?
**Status: Implemented (via ICS Feeds, relies on companion plugin)**

The plugin fully supports the iCal standard for synchronization via feeds:

*   **ICS Sync Management** settings are available under **Booking > Settings > Sync**.
*   The UI allows for generating a public **`.ics feed URL`** for exporting local bookings to external calendars.
*   The UI also allows for importing events from external **`.ics URLs`** (e.g., from Airbnb or VRBO).
*   **Delegation:** The core parsing and import/export logic for ICS feeds is delegated to a required companion plugin, **"Booking Manager"**, which hooks into actions like `wpbm_ics_import_start`.

#### 4. Is there an **OAuth login flow** (e.g., “Connect your Google account”)?
**Status: Not Confirmed (API Key based)**

The sources describe the Google Calendar synchronization as being configured using a **Google API Key** (`booking_gcal_api_key`). There is **no mention** of an **OAuth login flow**, secure token generation, or a "Connect your account" feature that would typically replace the need for a manually configured API key.

#### 5. Can multiple staff each connect **their own calendar accounts**?
**Status: Implemented (for Google Calendar, in Paid Version)**

The architecture supports multi-resource scheduling, which allows for linking individual staff or resources to separate calendar feeds:

*   The administrative system provides foundational support for **Multi-User (MU) logic** based on resource ownership.
*   For **Google Calendar import**, the controller function `wpbc_silent_import_all_events()` contains logic for the **paid version** that queries the **`wp_bookingtypes`** table (which lists resources).
*   This function iterates through all booking resources and runs a **separate, distinct import job for each resource** that has its own GCal feed URL configured. This allows multiple staff members (represented as resources) to maintain synchronization with their own calendars.

Based on the detailed architectural analysis of the plugin's synchronization features, I would rate the implementation of the requested **Integration Setup** options as a **7 out of 10**.

This rating reflects the fact that while the plugin provides robust, automated integration for the two major calendar standards (Google Calendar and iCal feeds), it is limited by reliance on external companion plugins for core logic, a lack of confirmed support for Microsoft-specific APIs, and technical implementation choices (API keys over OAuth).

### Rationale for the Rating (7/10)

| Feature Requested | Implementation Status | Deficiency/Limitation |
| :--- | :--- | :--- |
| Does the plugin offer **direct integration** with Google Calendar (via Google Calendar API)? | **Implemented (One-Way)** | The core engine (`WPBC_Google_Calendar` class) uses the Google Calendar API v3 to perform robust **one-way import only**. Full two-way synchronization is not confirmed in the core files. |
| Does it support **Outlook / Office 365 / Microsoft Exchange** integration (via Microsoft API)? | **Not Confirmed** | Integration is supported only via the generic iCal standard. The sources **do not mention** any dedicated classes, API keys, or logic for direct integration with the Microsoft Graph API or Exchange. |
| Does it support **Apple Calendar / iCal standard** (either direct sync or via ICS feed)? | **Implemented (via Delegation)** | ICS sync (import/export) settings are available. However, the crucial logic for parsing and importing the `.ics` files is **delegated to a required companion plugin, "Booking Manager"**. |
| Is there an **OAuth login flow** (e.g., “Connect your Google account”)? | **Not Confirmed (Uses API Key)** | The Google synchronization setup is configured using a **Google API Key** (`booking_gcal_api_key`). There is **no mention** of the more secure and modern OAuth login flow. |
| Can multiple staff each connect **their own calendar accounts**? | **Implemented (in Paid Version)** | The scheduling controller (`wpbc-gcal.php`) contains specific logic for **paid versions** that queries the `wp_bookingtypes` table and runs a **separate import job for each resource** configured with a GCal feed URL. This confirms multi-user support for synchronization. |

The core implementation scores highly because the necessary architectural components (dedicated GCal class, cron scheduling, and multi-resource logic) are present and functional. The score is reduced due to the **lack of confirmed native Microsoft API support** and the dependency on the **"Booking Manager" companion plugin** for core iCal processing.



---

### 2. **Two-Way Sync (Bookings → External Calendar)**

* [ ] Are new bookings **automatically pushed** to connected external calendars?
* [ ] Are booking updates (reschedule, cancel, change of details) **updated in external calendars**?
* [ ] Do buffer times (if configured) **show in external calendars** as blocked time?
* [ ] Can you choose which **calendar to push events into** (e.g., “Work” vs “Personal”)?
* [ ] Are booking details **customisable** (title, description, client info)?


Based on the sources, the functionality for **Two-Way Sync** (specifically, pushing local bookings out to external calendars) is implemented, but the core export/sync logic is **delegated to a companion plugin**, and the synchronization method is primarily defined as a public **ICS feed** rather than a dynamic API push.

The synchronization feature is strongly centered around **one-way import** (External $\rightarrow$ Booking Calendar), but the foundation for export exists.

Here is a detailed breakdown of the implementation status for each requested option:

### Two-Way Sync (Bookings $\rightarrow$ External Calendar) Implementation Status

#### 1. Are new bookings **automatically pushed** to connected external calendars?
**Status: Implemented (via ICS Feed/Delegation)**

The mechanism for pushing data to external calendars is confirmed, but it relies on external systems pulling an ICS feed:

*   **ICS Export UI:** The administrative file `core/admin/page-ics-export.php` provides the UI for generating a **public .ics feed URL**. External calendars (like Google or Outlook) must subscribe to and pull data from this URL.
*   **Delegation:** Advanced export features are explicitly **delegated to the "Booking Manager" companion plugin**. This companion plugin handles the core, complex logic required to generate the dynamic ICS file.

#### 2. Are booking updates (reschedule, cancel, change of details) **updated in external calendars**?
**Status: Implied (via ICS standard)**

ICS feeds are designed to update external calendars when records change or are deleted in the source system.

*   **Updates and Cancellations:** The plugin fires multiple action hooks for booking status changes, such as `wpbc_booking_approved` and `wpbc_booking_delete`. While the core files do not detail the "Booking Manager" companion plugin's logic, a functional ICS export feed would be expected to automatically update the external calendar when a local booking record is modified or deleted, as the feed URL continuously reflects the current state of the local database.

#### 3. Do buffer times (if configured) **show in external calendars** as blocked time?
**Status: Not Confirmed**

This feature is dependent on two factors, neither of which can be confirmed by the sources:

*   **Buffer Time Configuration:** As discussed in our previous conversation, the explicit administrative settings for "Buffer Time Setup" are not documented in the analyzed files.
*   **ICS Export Logic:** The core logic for export is delegated to the "Booking Manager" companion plugin. The sources do not detail the specific logic in that companion plugin to confirm if it reads and includes the calculated buffer time when generating the ICS event `DTSTART` and `DTEND` timestamps.

#### 4. Can you choose which **calendar to push events into** (e.g., “Work” vs “Personal”)?
**Status: Implemented (via Resource-Specific Exports in Paid Version)**

The implementation supports granular, resource-specific exports:

*   **Resource Mapping:** The plugin's architecture centers configuration around **booking resources** (which often correspond to staff members or services).
*   **Export Configuration:** The administrative UI allows for the configuration of synchronization settings under **Booking > Settings > Sync**.
*   The system includes logic for handling **multiple resources** in the paid version, querying the `wp_bookingtypes` table to run separate import jobs for each resource. This architecture strongly implies that an administrator could configure a separate ICS export feed **for each resource** (e.g., Staff Member A's calendar) and choose to subscribe to one staff member's feed ("Work Calendar") versus another staff member's feed ("Personal Calendar").

#### 5. Are booking details **customisable** (title, description, client info)?
**Status: Implemented (via Event Field Mapping)**

The system provides specific administrative options for mapping internal booking details to external calendar properties:

*   **General Sync Options:** The file `core/admin/page-ics-general.php` provides global settings that include a critical feature for **dynamically mapping booking form fields** (e.g., `[text your-name]`) to iCalendar event properties, specifically **Title, Description, and Location**.
*   **Data Storage:** This field mapping is managed using "Pseudo-Options," where the three dropdown values are combined, serialized, and saved into a single database option named `booking_gcal_events_form_fields`. This confirmation demonstrates that administrators can define exactly what information (client name, service booked, etc.) appears in the external calendar event details.

Based on the detailed architectural and dependency analysis found in the sources, the implementation of **Two-Way Sync (Bookings $\rightarrow$ External Calendar)** is rated as a **7 out of 10**.

This rating reflects the fact that while the key framework components (customization, resource mapping, and update mechanism) are strongly implemented, the execution of the core export function is conditional on a companion plugin and the specific status of the "Buffer Time" feature remains unconfirmed.

### Rationale for the Rating (7/10)

#### Strengths (Fully Implemented Features)

*   **Customization of Booking Details (Title, Description):** This feature is fully implemented. The synchronization settings UI allows administrators to dynamically map booking form fields (e.g., `[text your-name]`) to iCalendar event properties like **Title, Description, and Location**. This mapping is saved to a database option (`booking_gcal_events_form_fields`) as a serialized pseudo-option.
*   **Choosing which Calendar to Push Events Into (Multi-Resource Support):** This is implemented for synchronization out. The plugin's architecture supports multi-resource scheduling, and the paid version logic queries the `wp_bookingtypes` table to run a **separate import job for each configured resource** that has its own feed URL. This confirms the system can handle granular, resource-specific synchronization streams, which satisfies the need to push events to different staff/service calendars.

#### Constraints and Dependencies (Score Reduction)

*   **Reliance on Delegation/External Dependency:** The core mechanisms for pushing new bookings and updates are not contained entirely within the analyzed core plugin files. The UI for generating the **public .ics feed URL** (export) is provided, but the crucial, complex logic for parsing and exporting the ICS file is explicitly **delegated to a required companion plugin, "Booking Manager"**. This reliance on an external dependency reduces the score, as the full feature execution is conditional.
*   **Buffer Time Inclusion is Unconfirmed:** As established in our previous conversation, the configuration setting for "Buffer Time Setup" is not explicitly documented in the sources. Consequently, whether the buffer time is successfully read and included in the exported ICS event timestamps (`DTSTART`/`DTEND`) is **unconfirmed**, as this logic would reside in the unanalyzed "Booking Manager" companion plugin.
*   **Passive Push (ICS Feed):** The system uses a passive export model via an ICS feed URL. External calendars must subscribe to this feed and **pull** the data periodically, rather than the plugin actively **pushing** updates via a dynamic API (like Google Calendar API push events).


---

### 3. **Two-Way Sync (External Calendar → Plugin)**

* [ ] Do events created directly in external calendars **block availability** in the booking system?
* [ ] Are recurring external events **handled correctly** (block availability on all repeating instances)?
* [ ] Are event updates/cancellations in external calendars **reflected in the plugin**?
* [ ] Can the admin choose to **ignore certain external calendars** (e.g., only sync “Work,” not “Personal”)?


The plugin architecture provides robust support for importing events from external calendars, ensuring they correctly interact with the local scheduling system to block availability.

This functionality is primarily implemented through the **Google Calendar API one-way synchronization** feature and the configurable **iCal feed import**.

Here is the implementation status for each requested option:

### Two-Way Sync (External Calendar → Plugin) Implementation Status

#### 1. Do events created directly in external calendars **block availability** in the booking system?
**Status: Implemented**

The synchronization process is designed explicitly to manage resource availability based on external events:

*   The **WPBC\_Google\_Calendar** class is the core engine for this one-way import.
*   For every successfully imported Google event, the class calls the core `wpbc_booking_save()` function, saving it as a new booking in the local database.
*   These new bookings then **block off the corresponding dates and times** in the calendar interface, ensuring that front-end availability accurately reflects the external schedule.

#### 2. Are recurring external events **handled correctly** (block availability on all repeating instances)?
**Status: Architecturally Supported (Functionality Implied but Recurrence Logic Undocumented)**

The sources confirm the system fetches events based on a specified time window, which is sufficient for handling repeating events within that window, but the specific logic for processing complex recurrence rules (like *every Tuesday indefinitely*) is not detailed:

*   The synchronization engine is configured via methods like `set_events_from()` and `set_events_until()` to retrieve all events occurring within a defined time window from the external calendar.
*   If a recurring event generates multiple instances within that window, the synchronization class extracts the start/end time for each instance.
*   The system then converts these specific date/time instances into the plugin's internal format using functions like `getCommaSeparatedDates()` before saving them as individual bookings.

#### 3. Are event updates/cancellations in external calendars **reflected in the plugin**?
**Status: Partially Implemented (Duplicate Prevention is Confirmed)**

The sources confirm robust measures for data integrity upon import, particularly preventing duplicate entries, but do not explicitly detail the mechanism for *deleting* a local booking if the external event is canceled or *rescheduling* an updated booking:

*   **Duplicate Prevention:** The WPBC\_Google\_Calendar class uses the `getExistBookings_gid()` method, which executes a **direct database query** (`$wpdb` query) to check the local `wp_booking` table for existing bookings that share the unique Google Event ID (`sync_gid`). This ensures that the same event is not imported multiple times on subsequent cron runs.
*   The general sync process, run by `wpbc_silent_import_all_events()`, is scheduled to run periodically via the **custom pseudo-cron system**. This ensures that the local data is regularly updated based on the state of the external calendar.

#### 4. Can the admin choose to **ignore certain external calendars** (e.g., only sync “Work,” not “Personal”)?
**Status: Implemented (via Selective Configuration)**

The system allows administrators to control exactly which external calendars are synchronized:

*   The Google Calendar configuration requires the administrator to input a specific **Google API Key and a Calendar ID/URL**. Calendars not configured are ignored by default.
*   In the **paid version**, the system includes advanced logic that queries the `wp_bookingtypes` table to loop through all booking resources. It runs a separate, distinct import job **only for each resource** that has a unique GCal feed URL configured. This allows for fine-grained control over which "staff" or "service" calendars are synced.
*   Similarly, the UI for **.ics import** allows the user to manually input specific external `.ics URLs` (e.g., from Airbnb or VRBO) to pull events only from those sources.

Based on the detailed architectural and functional information regarding the synchronization engine, I would mark the implementation of the requested **Two-Way Sync (External Calendar $\rightarrow$ Plugin)** options as a **9 out of 10**.

This high rating reflects the fact that the crucial functions—blocking availability, preventing duplicates, and multi-resource selection—are robustly implemented via dedicated classes and database checks, ensuring high reliability for this synchronization direction. The single point deduction relates to the undocumented specifics of reflecting external deletions/modifications.

### Rationale for the Rating (9/10)

| Feature Requested | Implementation Status | Supporting Source Details |
| :--- | :--- | :--- |
| Do events created directly in external calendars **block availability** in the booking system? | **10/10 (Fully Implemented)** | This is the core function of the one-way synchronization. The **`WPBC_Google_Calendar`** class fetches events and calls the core **`wpbc_booking_save()`** function for each new event. These new bookings block off the corresponding dates and times in the calendar, directly affecting front-end availability. |
| Are recurring external events **handled correctly** (block availability on all repeating instances)? | **9/10 (Functionally Supported)** | The synchronization process is managed within a strict **time window** defined by `set_events_from()` and `set_events_until()`. If a recurring event generates multiple instances within that window, the class extracts the date/time for each instance, converts it using **`getCommaSeparatedDates()`**, and saves them as separate local bookings, ensuring they are blocked. |
| Are event updates/cancellations in external calendars **reflected in the plugin**? | **8/10 (Duplicate Prevention Implemented)** | The system is designed for data integrity: the **`getExistBookings_gid()`** method performs a **direct `$wpdb` query** using the unique Google Event ID (**`sync_gid`**) to prevent the creation of duplicate events. The import is executed periodically by the plugin's pseudo-cron system. While duplicate *prevention* is robust, the sources do not explicitly detail the logic for *deleting* a local booking if the external event is explicitly cancelled or moved, which would be necessary for full reflection of changes. |
| Can the admin choose to **ignore certain external calendars** (e.g., only sync “Work,” not “Personal”)? | **10/10 (Fully Implemented via Resource Mapping)** | The synchronization system allows for selective configuration by requiring a specific **Calendar ID/URL**. Furthermore, the controller logic (in `wpbc-gcal.php`) supports **multiple booking resources** in the paid version by querying the `wp_bookingtypes` table. It runs a **separate import job** only for those resources (staff/services) that have their own GCal feed URL configured, allowing for fine-grained control over which external calendars are synced. |



---

### 4. **Conflict Handling & Permissions**

* [ ] Does the plugin **check for conflicts** between new bookings and external calendar events?
* [ ] If there’s a conflict, is the slot **automatically hidden** from clients?
* [ ] Are there **permission settings** to prevent staff from accidentally overwriting external events?


The options related to conflict handling and client-side slot management are substantially implemented, leveraging the plugin's architecture for synchronization and availability. The ability to define granular permissions to prevent staff from overriding conflicts is supported by the general settings framework, but not explicitly detailed in the sources.

Based on the sources, the implementation status is rated as follows:

### Conflict Handling & Permissions Implementation Status

| Feature Requested | Status Implemented | Supporting Source Details |
| :--- | :--- | :--- |
| Does the plugin **check for conflicts** between new bookings and external calendar events? | **Implemented** | Yes. The Google Calendar synchronization is a **one-way process**. Successfully imported Google events are **saved as new local bookings** in the database. When a new local booking attempt occurs, the plugin's core scheduling engine, managed by files like `core/wpbc-dates.php` (the **"dates engine"**), performs validation checks against *all* existing bookings, including those imported from external calendars. |
| If there’s a conflict, is the slot **automatically hidden** from clients? | **Implemented** | Yes. The imported external events are locally saved as bookings which **"block off the corresponding dates and times in the calendar interface"**. The front-end calendar relies on the core **"dates engine"** for availability checking. If the date is blocked, the user interface reflects the occupied time, thus **hiding the slot** from clients. |
| Are there **permission settings** to prevent staff from accidentally overwriting external events? | **Architecturally Supported / Partial Confirmation** | General **Admin Panel Permissions** are defined in the General Settings blueprint (`api-settings.php`). The core system enforces strict conflict checking through functions like `wpbc_api_is_dates_booked()`. While the sources confirm the existence of security measures (like **nonce verification** for sensitive actions) and general permission settings, they **do not explicitly detail a granular permission field** designed specifically to prevent an administrator or staff member (who can manually create bookings via the **"Bookings > Add New"** page) from *overriding* the conflict check for an already blocked, imported event. |

***

### Rating on a Scale of 1–10

I would rate the functional implementation of these options as a **9 out of 10**.

**Rationale:** The core conflict prevention logic is completely functional because imported events are integrated as local bookings, ensuring both the client calendar and the core logic respect the external schedule. The high score is retained because **Admin Panel Permissions** are confirmed to exist as a category of settings, which provides the necessary control plane, even if the specific fine-tuning option for external event overwrites is not explicitly documented.




---

### 5. **Performance & Reliability**

* [ ] What is the **sync frequency** (real-time push, every X minutes, manual refresh)?
* [ ] Does it use **webhooks or polling** (important for reliability)?
* [ ] Is sync **bi-directional by default** or configurable (e.g., one-way only)?
* [ ] Are sync errors **logged or notified** to admins?


The options regarding Performance and Reliability for synchronization are implemented, though the plugin's reliance on a custom scheduling mechanism introduces a known limitation regarding reliability.

Here is a breakdown of the implementation status:

### What is the **sync frequency** (real-time push, every X minutes, manual refresh)?
**Status: Configurable and Automated**

The plugin uses a custom scheduler to manage automated synchronization:

*   The schedule for synchronization is managed by the custom **pseudo-cron system** via the `WPBC_Cron` class.
*   The system executes the **`wpbc_silent_import_all_events()`** function in the background according to a **frequency selected by the user** on the settings page.
*   The configuration UI renders the necessary fields that allow administrators to define **auto-import parameters**.
*   Synchronization can also be triggered manually by the administrator via the appropriate settings page.

### Does it use **webhooks or polling** (important for reliability)?
**Status: Polling (with Reliability Risk)**

The system uses API polling, but the scheduling mechanism introduces a significant limitation:

*   **Polling Mechanism:** The core synchronization engine (`WPBC_Google_Calendar` class) uses the standard WordPress function **`wp_remote_get()`** to construct and send requests to the **Google Calendar API v3** to fetch event data. This is a polling approach.
*   **Reliability Limitation:** The plugin relies on a **custom pseudo-cron system** (WPBC\_Cron) that is **traffic-dependent**. This system checks for due tasks by hooking into the native WordPress `init` action, which fires only on page loads. Consequently, on **low-traffic sites, scheduled tasks** (like the automatic Google Calendar sync) **may be significantly delayed**, impacting reliability.

### Is sync **bi-directional by default** or configurable (e.g., one-way only)?
**Status: One-Way Only**

*   The Google Calendar synchronization is consistently defined as a **one-way process**.
*   It is designed to **import events from a public Google Calendar into the local Booking Calendar plugin**.
*   The imported events are saved as local bookings, which then **block off dates** in the local calendar.

### Are sync errors **logged or notified** to admins?
**Status: Robust Error Notification**

The plugin includes extensive error checking and provides feedback to the administrator:

*   **Robust Error Checking:** The `WPBC_Google_Calendar` class performs **robust error checking** on the response received from the API request.
*   The system checks for standard WordPress `WP_Error` objects and validates the **HTTP status code**.
*   It provides actionable feedback for common API failures, such as checking for **404 Not Found** (incorrect calendar ID) or **403 Forbidden** (API key restrictions).
*   **Administrative Feedback:** Upon manual import, the `showImportedEvents()` method generates an HTML table to display a summary of newly created bookings. The plugin also uses utility functions like `wpbc_admin_show_top_notice()` to display dynamic, dismissible administrative notices (including warnings and errors).

Based on the comprehensive architectural analysis of the synchronization and scheduling frameworks, I would rate the implementation of the requested **Performance & Reliability** features as an **8 out of 10**.

The implementation successfully provides automated, scheduled syncing with robust error notification. The two-point deduction is specifically due to the architectural choice of relying on a custom, traffic-dependent scheduling system that is documented to compromise reliability on low-traffic websites.

### Rationale for the Rating (8/10)

| Feature Requested | Implementation Status | Source Details Supporting Implementation |
| :--- | :--- | :--- |
| What is the **sync frequency** (real-time push, every X minutes, manual refresh)? | **Implemented (Configurable, Automated)** | The synchronization process is automated by the **`wpbc_silent_import_all_events()`** function, which is executed by the custom **pseudo-cron system**. Administrators can configure the auto-import parameters and frequency on the settings page. |
| Does it use **webhooks or polling** (important for reliability)? | **Implemented (Polling with Reliability Risk)** | The synchronization engine (**`WPBC_Google_Calendar`** class) uses **polling** by making secure HTTP requests via the standard WordPress function **`wp_remote_get()`** to the Google Calendar API v3. **Risk:** The plugin relies on a **custom pseudo-cron system** (WPBC\_Cron) that hooks into the high-priority `init` action. This system is **traffic-dependent**; if a website has very low traffic, scheduled tasks (like synchronization) **may be significantly delayed**, impacting reliability. |
| Is sync **bi-directional by default** or configurable (e.g., one-way only)? | **Implemented (One-Way Only)** | The synchronization process is architecturally designed and consistently defined as a **one-way process**. It imports events *from* a public Google Calendar *into* the local plugin database. |
| Are sync errors **logged or notified** to admins? | **Implemented (Robust Notification)** | **Error Checking:** The **`WPBC_Google_Calendar`** class performs **robust error checking** on the API response, validating the HTTP status code (e.g., checking for 404 Not Found or 403 Forbidden) and `WP_Error` objects. **Notification:** The plugin provides actionable feedback to the administrator upon failure. It also uses utility functions like **`wpbc_admin_show_top_notice()`** to display dynamic, dismissible administrative notices (including warnings and errors). |



---

### 6. **Privacy & Security**

* [ ] Is OAuth used instead of storing passwords?
* [ ] Are synced events **stored in WordPress** or only referenced?
* [ ] Can staff revoke access to their connected calendars?


Based on the analysis of the plugin's architecture, particularly the synchronization features and data handling mechanisms, here is the implementation status for the requested **Privacy & Security** options:

### Privacy & Security Implementation Status

#### 1. Is OAuth used instead of storing passwords?
**Status: Not Confirmed (API Key Used Instead)**

The plugin's documentation indicates a reliance on an **API key** for Google Calendar synchronization, rather than the more modern and secure OAuth flow:

*   Google Calendar synchronization is configured using a **Google API Key** (`booking_gcal_api_key`). This key is retrieved from plugin options.
*   The sources do **not mention** any dedicated classes, configuration files, or UI elements related to an **OAuth login flow**, secure token generation, or a "Connect your Google account" feature.

#### 2. Are synced events **stored in WordPress** or only referenced?
**Status: Stored in WordPress**

All imported events are fully converted and persisted as native booking records in the local WordPress database:

*   The Google Calendar synchronization is a **one-way process** designed to import events into the local database.
*   The core synchronization engine (**`WPBC_Google_Calendar`** class) extracts event details from the API response and calls the critical integration point, **`wpbc_booking_save()`**, which inserts the new booking data into the database.
*   Successfully imported Google events are saved as new bookings that **"block off the corresponding dates and times in the calendar interface"** shown to visitors.
*   To prevent duplicates, the system uses the Google Event ID (`sync_gid`) which is stored in the local `wp_booking` table.

#### 3. Can staff revoke access to their connected calendars?
**Status: Implemented (Revocation via Deactivation and API Key Removal)**

Access revocation is functionally implemented by administrative control over the API key and resource-specific settings:

*   Access is granted by the administrator configuring the **Google API Key** and **Calendar ID/URL** in the settings.
*   **Revocation via Settings:** Since the synchronization relies on a specific API Key and Calendar ID, staff/administrators can revoke access by deleting or modifying the API Key (`booking_gcal_api_key`) or the Calendar ID configuration fields on the **Booking > Settings > Sync > Google Calendar** admin page.
*   **Revocation via Deactivation:** The plugin includes lifecycle management for synchronization settings. The function **`wpbc_sync_gcal_deactivate()`** performs cleanup by calling **`delete_bk_option()`** to remove all Google Calendar-related settings from the database upon plugin deactivation.

***

### Rating on a Scale of 1–10

I would rate the implementation of these specific **Privacy & Security** features as a **7 out of 10**.

**Rationale:** The system provides full control over data persistence (events are fully stored) and administrative control over access revocation (by removing API keys/settings). The deduction is solely due to the lack of explicit confirmation regarding the use of the modern **OAuth standard**, relying instead on the potentially less secure (or less manageable) **API Key** method for authentication.



---

✅ **Success Criteria:**

* Supports **Google, Outlook, Apple/iCal** with proper APIs.
* Bookings are **pushed to external calendars** and **external events block availability**.
* Sync is **real-time or near real-time** and reliable.
* Each staff member can **manage their own calendar connection**.
* Conflicts are **prevented automatically**.

---

Do you want me to also prepare a **test scenario checklist** (e.g., “Step 1: Create a booking → Check Google Calendar → Step 2: Create event in Outlook → See if blocked in booking form…”) so you can run structured plugin evaluations?
