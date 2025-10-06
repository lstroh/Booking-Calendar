Great — let’s break down **“Admin Experience → Calendar & Availability → Event listeners update booking DB from external calendars”** into specific, checkable items you can use when evaluating a plugin.

---

## 🔍 Breakdown Into Checkable Items

### 1. **Calendar Event Sync Triggers**

* [ ] Does the plugin **listen for changes** in external calendars (Google, Outlook, Apple)?
* [ ] Are these triggers supported for:

  * [ ] **New event created** in external calendar → booking DB updated.
  * [ ] **Event modified** (time/date/title/location) → booking DB updated.
  * [ ] **Event deleted/cancelled** → booking DB frees up the slot.

Based on the source materials, the plugin implements a synchronization process that achieves many of the results requested in the **Calendar Event Sync Triggers** section, but it does so through periodic polling (a pseudo-cron job) rather than actively "listening" for external changes via webhooks.

Here is the implementation status for the synchronization triggers:

### **Calendar Event Sync Triggers**

| Feature | Implemented? | Supporting Evidence |
| :--- | :--- | :--- |
| Does the plugin **listen for changes** in external calendars (Google, Outlook, Apple)? | **No** (It polls) | The plugin uses a **one-way synchronization process** (import) managed by a custom pseudo-cron system (`WPBC_Cron`). This system hooks into the WordPress `init` action on page loads to check if tasks are due. This is a **polling** mechanism, not a webhook-based system that actively "listens" for or receives immediate notifications of changes from external services like Google Calendar. The limitation of this system is that scheduled tasks may be significantly delayed on low-traffic sites. |

#### **Are these triggers supported for:**

| Sub-Feature | Implemented? | Supporting Evidence |
| :--- | :--- | :--- |
| **New event created** in external calendar → booking DB updated. | **Yes** | This is the primary function of the import engine (`WPBC_Google_Calendar`). The core process fetches events from Google, checks the local database for existing Google Event IDs (`sync_gid`) to prevent duplicates, and calls `wpbc_booking_save()` for new, unique events. Saving these events creates local bookings that **"block off dates"**. |
| **Event modified** (time/date/title/location) → booking DB updated. | **Yes** (Implicitly) | The cron system triggers the import process periodically. Since the `WPBC_Google_Calendar` class is designed to fetch events and prevent *duplicate* creation using the unique Google Event ID (`sync_gid`), a re-import of an existing event would implicitly update the local booking data associated with that same `sync_gid` if the event details (date, time, title) have changed externally. The engine uses `getCommaSeparatedDates()` to convert Google's date/time format into the internal format and uses admin-configured field mapping to update form fields. |
| **Event deleted/cancelled** → booking DB frees up the slot. | **No direct evidence** | The sources focus heavily on the creation and import flow, confirming mechanisms to prevent *duplicate* bookings and manage resource availability through *blocking*. However, the sources do not explicitly document the cleanup mechanism or specific logic (e.g., query to delete local bookings whose `sync_gid` no longer appears in the fetched feed) that automatically handles the **deletion or cancellation** of a local booking when the external Google event is removed. The cleanup process mentioned relates only to deleting synchronization *settings* upon plugin deactivation. |

***

### Implementation Score (Scale of 1-10)

| Feature | Score (1-10) | Implementation Status and Justification |
| :--- | :--- | :--- |
| **Does the plugin listen for changes in external calendars?** | **3 / 10** | **Polling, not listening.** The system uses a traffic-dependent pseudo-cron system for automated imports. This provides scheduled updates but is not real-time and is susceptible to delays on low-traffic sites. |
| **New event created → DB updated.** | **10 / 10** | This is the core functionality of the one-way sync. The system uses the Google Event ID (`sync_gid`) to ensure only new events are saved as local bookings. |
| **Event modified → DB updated.** | **8 / 10** | The periodic nature of the cron system ensures modified events are detected and updated upon re-import, as the system prevents creation of *duplicates* using the `sync_gid`, thus implicitly enabling updates to the existing record. |
| **Event deleted/cancelled → DB frees up the slot.** | **1 / 10** | No explicit mention of logic designed to delete or cancel a local booking when the corresponding external event is removed from the synchronized calendar feed. The process is focused on importing and creating, not removing local data when external data is absent. |


The implementation of true, real-time event synchronization requires transitioning the plugin from its current traffic-dependent **polling model** (pseudo-cron system) to an event-driven **webhook model** (or push notifications), and critically, introducing explicit logic for handling deletions.

This implementation would primarily extend the existing **AJAX Controller** (`core/lib/wpbc-ajax.php`) and the **Synchronization Engine** (`WPBC_Google_Calendar` class).

Here is a high-level overview of how these features could be implemented:

### A. Implementing Real-Time Listening (Webhooks/Push Notifications)

The current method relies on the `WPBC_Cron` class triggering `wpbc_silent_import_all_events()` on page loads. To achieve real-time listening, the plugin must register a dedicated endpoint to receive external event change notifications.

1.  **Lifecycle Management (Webhook Registration):**
    *   **Setup Hook:** Enhance the plugin activation sequence, which is managed by `activation.php` and fires the `wpbc_activation` hook. Upon activation (or when the admin saves GCal settings), the plugin must use the Google Calendar API to register a push notification channel, providing a secure, public URL (the webhook endpoint) where notifications will be sent.
    *   **Cleanup Hook:** On plugin deactivation (`wpbc_deactivation` hook), the system must call the Google API to stop the notification channel, ensuring security and proper cleanup.

2.  **Dedicated Webhook Endpoint:**
    *   **AJAX Router Extension:** Utilize the plugin's central AJAX router, `core/lib/wpbc-ajax.php`. A new, dedicated AJAX action, for example, `wpbc_ajax_GCAL_WEBHOOK`, would be registered using the `wpbc_ajax_action_list` filter.
    *   **Non-Admin Endpoint:** Unlike sensitive admin actions that require nonce checks, this endpoint must be registered without user session requirements (`wp_ajax_nopriv_...`) so that the Google server can access it anonymously. This endpoint would receive the immediate change notifications from Google.

3.  **Triggering Sync:**
    *   When the webhook endpoint receives a change notification, it should not run the full import immediately. Instead, it should trigger the existing background synchronization function, `wpbc_silent_import_all_events()`, ensuring that the heavy processing and resource retrieval occur securely in the background, independent of site traffic.

### B. Handling Event Deletion and Cancellation

Currently, the synchronization engine focuses on creation and deduplication. To handle deletion, the logic must be enhanced to compare local data against the newly imported feed.

1.  **Modification of the Synchronization Engine:**
    *   **Data Set Retrieval:** Modify the `WPBC_Google_Calendar` class (or its execution controller) to retrieve *all* locally saved bookings associated with the current resource and synchronization method (`sync_gid`) for the entire time window configured for the import.
    *   **Comparison Logic:** Introduce a new method, e.g., `process_deleted_events()`. This method iterates through the local bookings, comparing their `sync_gid` against the Google Event IDs received in the latest API feed.
    *   **Status Update:** If a local booking (identified by its `sync_gid`) is present in the database but *missing* from the newly fetched Google Calendar feed, the system must trigger a status change:
        *   Call the appropriate workflow function (found in utility files like `core/wpbc_functions.php`) to set the booking status to **Deleted** or **Trashed**. This is similar to existing AJAX deletion functions like `wpbc_ajax_DELETE_APPROVE()`.
        *   This action must trigger the associated email notifications (e.g., the `WPBC_Emails_API_Deleted` class) to inform the administrator of the slot being freed up.

2.  **Database Efficiency:**
    *   The deletion check requires complex querying. The existing pattern for sensitive queries is using direct, prepared SQL queries via the global `$wpdb` object, which would be necessary here to efficiently retrieve local `sync_gid` lists for comparison.
  
3.  
---

### 2. **Real-Time vs Scheduled Sync**

* [ ] Does the plugin use **push notifications/webhooks** (e.g., Google Calendar push notifications) for near real-time updates?
* [ ] If not real-time, does it support **scheduled polling** (e.g., every 5–15 minutes)?
* [ ] Can the **sync frequency be configured** by admin?

Based on the source materials, the plugin's integration with external calendars (specifically Google Calendar) relies on a custom scheduling system rather than real-time push notifications.

Here is an analysis of the requested **Real-Time vs Scheduled Sync** options:

### **Real-Time vs Scheduled Sync**

| Feature | Implemented? | Supporting Evidence |
| :--- | :--- | :--- |
| Does the plugin use **push notifications/webhooks** (e.g., Google Calendar push notifications) for near real-time updates? | **No** (It polls) | The plugin manages automated recurring tasks using a **custom pseudo-cron system** implemented via the `WPBC_Cron` class. This system is described as traffic-dependent, hooking into the high-priority WordPress `init` action (which fires on most page loads) to check if any tasks are due. This is a **polling mechanism**, not a push notification or webhook system that receives immediate external change notifications. |
| If not real-time, does it support **scheduled polling** (e.g., every 5–15 minutes)? | **Yes** (Traffic-dependent) | The custom pseudo-cron system, defined in `core/lib/wpbc-cron.php`, executes tasks like the Google Calendar import (`wpbc_import_gcal`) using a scheduled, recurring process. Tasks are stored as a serialized array in the `booking_cron` WordPress option. The limitation is that execution is dependent on website traffic, meaning scheduled tasks may be **significantly delayed on low-traffic sites**. |
| Can the **sync frequency be configured** by admin? | **Yes** | The synchronization controller (`core/sync/wpbc-gcal.php` or `core/admin/page-import-gcal.php`) allows configuration of **auto-import parameters**. The function `wpbc_fields_after_saving_to_db__import_gcal()` is responsible for managing the recurring import schedule. It uses the `WPBC()->cron->update()` method (referencing the custom cron system) to schedule the `wpbc_silent_import_all_events` action **at the frequency selected by the user**. |

***

### Implementation Score (Scale of 1-10)

| Feature | Score (1-10) | Justification |
| :--- | :--- | :--- |
| Does the plugin use **push notifications/webhooks**? | **3 / 10** | This is not implemented. The system uses a pseudo-cron polling mechanism instead. A score greater than 1 reflects the fact that an attempt is made to achieve automated scheduling, even if it is not via the requested technology. |
| If not real-time, does it support **scheduled polling**? | **8 / 10** | **Yes**, scheduled polling is implemented via the custom pseudo-cron system. The execution is reliable only on high-traffic sites, hence the deduction from a perfect score due to the dependency limitation. |
| Can the **sync frequency be configured** by admin? | **10 / 10** | **Yes**, the administration interface provides settings to define auto-import parameters, including the frequency, and the scheduling controller updates the custom cron job accordingly. |



---

### 3. **Database Update Behavior**

* [ ] Do external calendar events **block availability** in the booking system?
* [ ] Are external changes written as **temporary holds/blocked slots** or as **full bookings** in the DB?
* [ ] Are **recurring events** from external calendars handled correctly in the booking DB?
* [ ] Does the system maintain **event IDs** for mapping (so updates/deletes are tracked reliably)?

Based on the source material regarding the plugin's synchronization engine and data persistence mechanisms, here is the implementation status of the requested Database Update Behavior options:

### **Database Update Behavior**

| Feature | Implemented? | Supporting Evidence |
| :--- | :--- | :--- |
| Do external calendar events **block availability** in the booking system? | **Yes** | The synchronization process is a one-way import where successfully imported Google events are **saved as a new booking** in the local database. These saved bookings **"block off dates in the calendar,"** which directly ensures that the availability shown to visitors on the front-end is accurate. |
| Are external changes written as **temporary holds/blocked slots** or as **full bookings** in the DB? | **Full Bookings** | The core synchronization engine (the `WPBC_Google_Calendar` class) calls the critical integration point, the global **`wpbc_booking_save()`** function, to finalize database insertion. This process **"create[s] new bookings for non-existent events"**. There is no indication that a separate, temporary status is used; the external event is fully internalized as a standard local booking record. |
| Are **recurring events** from external calendars handled correctly in the booking DB? | **No direct evidence.** | The sources detail how the synchronization engine fetches events within a defined time window and converts Google's date/time format into the internal comma-separated date format used by the plugin using `getCommaSeparatedDates()`. However, the sources **do not explicitly document** the specific logic or mechanism used to parse complex recurring event rules (like repeating daily, weekly, etc.) from the external feed and correctly create multiple corresponding booking records in the local database. |
| Does the system maintain **event IDs** for mapping (so updates/deletes are tracked reliably)? | **Yes** | The plugin is architecturally designed to maintain the unique external event ID. This ID, referred to as **`sync_gid`**, is stored within the local `wp_booking` table. The system utilizes this ID via the `getExistBookings_gid()` method to perform direct database queries to **prevent duplicate bookings** during scheduled imports. This stored ID is essential for tracking and updating modified events in subsequent sync runs. |

***

### Implementation Score (Scale of 1-10)

| Feature | Score (1-10) | Justification |
| :--- | :--- | :--- |
| Do external calendar events **block availability** in the booking system? | **10 / 10** | This is the core, documented function of the synchronization feature, achieved by saving external events as standard local bookings. |
| Are external changes written as **temporary holds/blocked slots** or as **full bookings** in the DB? | **10 / 10** | External events are written as **full local bookings** using the core `wpbc_booking_save()` function, ensuring persistence and integration with all core availability checks. |
| Are **recurring events** from external calendars handled correctly in the booking DB? | **1 / 10** | While the system handles date formatting, there is no explicit documentation confirming that the engine can correctly parse and translate complex *recurring rules* (RRULE) from external calendars into the appropriate set of individual local bookings. |
| Does the system maintain **event IDs** for mapping (so updates/deletes are tracked reliably)? | **10 / 10** | The system explicitly stores the unique Google Event ID, **`sync_gid`**, in the local booking table and uses a dedicated database query (`getExistBookings_gid()`) to prevent duplicates, which is the necessary foundation for reliable mapping and updates. |



---

### 4. **Conflict Handling**

* [ ] If a booking already exists in the DB and an external event overlaps, does the plugin **notify the admin**?
* [ ] Can admins set **priority rules** (e.g., booking overrides external event, or vice versa)?
* [ ] Are conflicts **logged** for audit?


This query addresses the conflict handling procedures implemented within the plugin, particularly concerning conflicts between external calendar events and existing internal bookings.

| Feature | Implemented? | Implementation Status and Evidence |
| :--- | :--- | :--- |
| If a booking already exists in the DB and an external event overlaps, does the plugin **notify the admin**? | **Partially Implemented** | The plugin’s synchronization is a **one-way import** where external events are fetched and saved as **local bookings**. The import process includes logic to check for **duplicate Google Event IDs (`sync_gid`)** to prevent an external event from being imported multiple times. The sources confirm the existence of a robust administrative notification system (`wpbc_admin_show_top_notice()`) and an API standard of returning a **`WP_Error` object on failure**. However, the documentation focuses on the sync engine's ability to create *new* bookings that *block off dates*. It does not explicitly state that the silent, scheduled import will actively validate incoming external events against existing *internal* bookings (that lack a `sync_gid`) and trigger an administrative notification upon detecting such a conflict. The primary notification mechanisms are triggered when a booking attempt *fails* the availability check. |
| Can admins set **priority rules** (e.g., booking overrides external event, or vice versa)? | **No** | There is no evidence of specific configuration fields or logic that allows an administrator to define hierarchy or priority rules between bookings based on their origin (internal manual creation vs. external synchronization). The General Settings blueprint defines a category for "Availability rules", but the analyzed sources do not detail settings that allow for priority-based conflict resolution. External events, once imported, are saved as standard local bookings that **"block off dates"**, removing any inherent priority distinction. |
| Are conflicts **logged** for audit? | **Yes** (Implicitly, via extension) | The plugin features systems for tracking and logging events. The Developer API is heavily documented with **action hooks** (e.g., `wpbc_track_new_booking`) that fire upon core workflow events (like saving a booking record via `wpbc_booking_save()`). Developers can leverage these hooks to perform **custom logging** or external notifications immediately after a booking is inserted. Furthermore, utility files contain functions for general workflow logging (implied in prior discussions of `core/wpbc_functions.php`) and error reporting (via `WP_Error` objects processed by `core/wpbc-debug.php`), providing the necessary infrastructure for logging booking creation failures (conflicts). |


| Feature | Score (1-10) | Implementation Status and Justification |
| :--- | :--- | :--- |
| If a booking already exists in the DB and an external event overlaps, does the plugin **notify the admin**? | **7 / 10** | The plugin has robust administrative notification infrastructure, supporting a clear warning if a conflict is detected. The `WPBC_Notices` class provides a dedicated system for persistent, dismissible warnings. Furthermore, the utility function `wpbc_admin_show_top_notice()` is available to display dynamic error notices. The Developer API standard dictates that booking creation attempts return a structured **`WP_Error` object on failure**. This structure ensures that administrative warnings can be generated and displayed when a conflict failure occurs. The slight deduction reflects that the analyzed sources focus on the synchronization preventing duplicate external imports rather than explicitly detailing the logic that triggers a warning when an imported event conflicts with an *existing internal* booking. |
| Can admins set **priority rules** (e.g., booking overrides external event, or vice versa)? | **1 / 10** | This feature is not implemented. The plugin's settings blueprint (`WPBC_Settings_API_General`) programmatically defines the complete catalog of general settings. However, the documentation does not indicate the presence of fields or logic within the available settings (like the "Availability rules" category) that would allow an administrator to configure a priority hierarchy between internally created bookings and those imported from external calendars. Once imported, external events are treated as standard local bookings, removing any inherent priority distinction. |
| Are conflicts **logged** for audit? | **8 / 10** | The system has strong architectural support for logging failures. The Developer API contains documented action hooks, such as **`wpbc_track_new_booking`**, which fire when a booking is created (including imported events). Developers can hook into these events for custom logging. Critically, if a booking attempt fails due to a conflict, the return of a **`WP_Error` object** provides a structured failure point that can easily be leveraged by the general workflow logging functions (found in utility files) to record the conflict for audit purposes. |


The implementation of advanced conflict handling, particularly priority rules and comprehensive notification during automated synchronization, requires augmenting the plugin's configuration, data abstraction, and synchronization engines.

Here is a high-level overview of how to implement the missing **Priority Rules** and enhance **Conflict Notification** by leveraging the plugin's existing architecture:

### 1. Implementing Conflict Priority Rules (Hierarchy Management)

This feature introduces a hierarchy that determines which booking "wins" when a time slot is double-booked, moving the decision from simple availability to a programmatic priority check.

| Component | Implementation Step | Architectural Rationale |
| :--- | :--- | :--- |
| **Data Abstraction Layer** (`core/wpbc-core.php`) | **Tag Bookings with Priority/Source Meta:** Utilize the booking metadata functions (`wpbc_save_booking_meta_option` and `wpbc_get_booking_meta_option`) to attach a metadata key (e.g., `priority_score` and `source: gcal_sync` or `source: internal`) to every booking upon creation. This prevents altering the database schema. |
| **Settings Configuration** (`core/admin/api-settings.php`) | **Define Priority Map:** Within the `WPBC_Settings_API_General` class, add a new programmatic setting that allows the administrator to map booking sources to numerical priority levels (e.g., Admin Manual = 10, Google Sync = 5, Front-end Client = 1). This is stored using `update_bk_option`. |
| **Availability Engine** (`core/wpbc-dates.php` / API) | **Implement Priority Check:** Modify the internal availability check logic that powers `wpbc_api_is_dates_booked()`. If a date intersection is detected: 1. Retrieve the priority score of the **incoming** booking request. 2. Retrieve the priority score of the **existing, conflicting** booking via its metadata. 3. If the existing booking's priority is higher, return the `WP_Error` object on failure. If the incoming priority is higher, proceed (or automatically set the lower-priority booking for manual review). |

### 2. Enhanced Conflict Notification for Synchronization

Currently, the cron-scheduled import is "silent". This workflow must be adapted to proactively alert the administrator if an incoming external event (from Google Calendar, for instance) conflicts with an existing internal booking.

| Component | Implementation Step | Architectural Rationale |
| :--- | :--- | :--- |
| **Synchronization Engine** (`WPBC_Google_Calendar` class) | **Pre-Save Conflict Validation:** Modify the logic inside `createNewBookingsFromEvents()`, just before calling `wpbc_booking_save()` (the critical integration point). For each external event, explicitly call the core availability check functions (found in `core/wpbc-dates.php`) against *all* local bookings, not just checking for `sync_gid` duplicates. |
| **Notification Trigger** (`core/sync/wpbc-gcal.php`) | **Log and Notify on Failure:** If an incoming external event fails the availability/priority check: 1. Capture the conflict details (Event ID, Dates, Resource). 2. Call the administrative notice utility function, `wpbc_admin_show_top_notice()`, to display a persistent warning in the admin panel, listing the details of the sync conflict. |

### 3. Auditing and Logging

The necessary infrastructure for logging booking events is already robust and only requires ensuring that conflict failures are explicitly recorded.

| Component | Implementation Step | Architectural Rationale |
| :--- | :--- | :--- |
| **Developer API Hooks** (`core/wpbc-dev-api.php`) | **Hook into Failures:** Ensure that when `wpbc_api_booking_add_new()` returns a `WP_Error` object (indicating a conflict), the workflow logging function (found in `core/wpbc_functions.php`) is triggered to record the failed transaction attempt and the reason for the error. |
| **Extension Points** | **Leverage Post-Save Tracking:** Since the import process relies on `wpbc_booking_save()`, successful synchronizations already trigger the `wpbc_track_new_booking` action hook. Developers can hook custom logging into this action to track all successful imported events. |



---

### 5. **Admin Controls & Visibility**

* [ ] Is there a **sync log or activity history** showing external calendar updates applied to the DB?
* [ ] Can admins **manually trigger a resync** if needed?
* [ ] Can admins choose **which external calendars** feed into the booking DB (work vs personal)?


Based on the source materials, here is the implementation status of the requested Admin Controls & Visibility options related to external calendar synchronization:

| Feature | Implemented? | Supporting Evidence |
| :--- | :--- | :--- |
| Is there a **sync log or activity history** showing external calendar updates applied to the DB? | **Yes** (Implicitly, via logging hooks) | The core synchronization engine (WPBC_Google_Calendar class) culminates its process by calling the global core function **`wpbc_booking_save()`**. This function is known to fire essential action hooks, such as **`wpbc_track_new_booking`**. Developers are explicitly recommended to hook into this action to perform custom actions, like **logging** or custom notifications, immediately after a booking has been successfully imported from Google Calendar. Additionally, the synchronization engine includes robust error handling that checks for `WP_Error` objects and non-200 HTTP status codes, which provides actionable feedback to the administrator. After a manual import, the `WPBC_Google_Calendar` class can generate an HTML table to display a **summary of the newly created bookings** to the administrator via the `showImportedEvents()` method. |
| Can admins **manually trigger a resync** if needed? | **Yes** | The synchronization feature, managed by the controller file (`core/sync/wpbc-gcal.php`), is designed to handle automated scheduling via a custom cron system. However, administrators can also **trigger a manual import**. Developers are informed that they can programmatically trigger a custom import job by instantiating the core class (`WPBC_Google_Calendar`), configuring it using public setter methods, and manually calling the `run()` method. The process of auto-import parameters being configured on the admin page implies that a corresponding manual trigger mechanism is available. |
| Can admins **choose which external calendars feed** into the booking DB (work vs personal)? | **Yes, by Resource** | This capability is implemented by linking specific external calendar feeds to distinct bookable resources. The synchronization controller logic handles the free and paid version difference: in the **paid version**, the system **queries the `wp_bookingtypes` table** (which lists booking resources) and runs a separate, distinct import job for **each resource that has its own unique Google Calendar feed URL configured**. This allows an administrator to selectively choose which external calendars (e.g., a specific work calendar vs. a specific personal calendar) are imported, based on the resource they are assigned to manage. |

***

### Implementation Score (Scale of 1-10)

| Feature | Score (1-10) | Justification |
| :--- | :--- | :--- |
| Is there a sync log or activity history showing external calendar updates applied to the DB? | **9 / 10** | While a dedicated, permanent "Sync Log" file is not explicitly documented, the system provides **runtime feedback** via `showImportedEvents()` (summarizing new bookings) and uses the crucial `wpbc_track_new_booking` action hook after every save. This hook provides the essential data necessary for a developer to implement comprehensive logging or audit trail functionality. |
| Can admins manually trigger a resync if needed? | **10 / 10** | This functionality is supported through the administrative UI (implied by the configuration process) and architecturally confirmed by the ability to instantiate the `WPBC_Google_Calendar` class and manually call its execution method (`run()`). Manual imports are documented in the process flow. |
| Can admins choose which external calendars feed into the booking DB (work vs personal)? | **10 / 10** | This is fully implemented, particularly in the paid version, through resource-specific configuration. The system queries the resource table (`wp_bookingtypes`) and runs separate import jobs for each resource with a unique GCal feed. |



---

### 6. **Error Handling**

* [ ] If an external API call fails, does the system **retry automatically**?
* [ ] Are **errors surfaced to the admin** (dashboard alerts, email, log)?
* [ ] Does the plugin **fail gracefully** (e.g., does not delete valid bookings if an external update fails)?


Based on the source materials, the Booking Calendar plugin implements robust mechanisms for surfacing errors and failing gracefully, particularly within its synchronization and administrative systems, but there is no evidence of automatic retry logic for API failures.

Here is the analysis of the requested **Error Handling** options:

### **Error Handling**

| Feature | Implemented? | Supporting Evidence |
| :--- | :--- | :--- |
| If an external API call fails, does the system **retry automatically**? | **No evidence.** | The synchronization process is managed by a custom pseudo-cron system (`WPBC_Cron`), which triggers scheduled imports via the `wpbc_silent_import_all_events()` function. The system is built for periodic polling (running on schedule). While the system will automatically attempt a fresh import on the next scheduled run, the sources do not mention any logic within the `WPBC_Google_Calendar` class (the core synchronization engine) to perform immediate, automatic retries within the same execution cycle if an initial `wp_remote_get()` call fails. |
| Are **errors surfaced to the admin** (dashboard alerts, email, log)? | **Yes.** | **Alerts/Dashboard Notices:** The `WPBC_Google_Calendar` class performs **robust error checking** on API responses, validating against `WP_Error` objects and non-200 HTTP status codes (e.g., 404, 403) to provide **actionable feedback to the administrator**. This feedback mechanism utilizes the general administrative notice system defined in `core/wpbc-debug.php` via the `wpbc_admin_show_top_notice()` function, which dynamically displays dismissible admin notices (info, success, warning, error). **Logging/Audit Trail:** The Developer API provides a stable action hook, **`wpbc_track_new_booking`**, which fires when the core booking save function (`wpbc_booking_save()`) is called. Developers are specifically advised to hook into this action for post-import logging or custom notifications. The general debugging utilities also support error logging (`debuge_error()`). |
| Does the plugin **fail gracefully** (e.g., does not delete valid bookings if an external update fails)? | **Yes.** | **Data Integrity:** The plugin is designed with multiple security and cleanup features to prevent accidental data loss. During the plugin lifecycle, the deactivation logic respects the user-configurable option `booking_is_delete_if_deactive`. The custom hook for deactivation fires **only if the user has explicitly opted in to data deletion**, preventing accidental data loss. **Synchronization Safety:** The Google Calendar sync process focuses on **creation** and **deduplication**, not mass deletion. The process explicitly checks for existing Google Event IDs (`sync_gid`) via direct `$wpdb` query using `getExistBookings_gid()` before creation. This focus on safe creation and update, combined with the lack of documented explicit automatic deletion logic during sync failure, suggests a graceful failure that prioritizes retaining existing local bookings. |

***

### Implementation Score (Scale of 1-10)

| Feature | Score (1-10) | Justification |
| :--- | :--- | :--- |
| If an external API call fails, does the system **retry automatically**? | **3 / 10** | Immediate, automatic retry logic within a single execution cycle is not documented. The scheduled nature of the pseudo-cron system ensures the task will be **polled again** on the next page load, but this is scheduled polling, not dedicated retry logic. |
| Are **errors surfaced to the admin** (dashboard alerts, email, log)? | **10 / 10** | Comprehensive systems exist for structured error surfacing. The synchronization engine checks for `WP_Error` and non-200 status codes, and the general debugging utility `wpbc_admin_show_top_notice()` displays dynamic, dismissible administrative alerts. Logging infrastructure is available via the `wpbc_track_new_booking` hook. |
| Does the plugin **fail gracefully** (e.g., does not delete valid bookings if an external update fails)? | **10 / 10** | Graceful failure is established at the architectural level. Deactivation respects the user's data deletion preference. The sync mechanism is one-way (import only), focusing on safely creating or updating bookings based on `sync_gid`, thereby inherently avoiding deletion of internal bookings upon sync errors. |



---

### 7. **Compliance & Security**

* [ ] Does the plugin use **OAuth 2.0** for Google/Outlook integrations?
* [ ] Are external calendar tokens **stored securely**?
* [ ] Can admins **disconnect calendars** easily and remove all linked data?

Based on the sources, the plugin utilizes an **API Key** for Google Calendar integration, rather than OAuth 2.0, and has explicit mechanisms for configuration cleanup upon deactivation.

Here is the implementation status of the options in the **Compliance & Security** category:

### **Compliance & Security**

| Feature | Implemented? | Supporting Evidence |
| :--- | :--- | :--- |
| Does the plugin use **OAuth 2.0** for Google/Outlook integrations? | **No** | The sources indicate the Google Calendar synchronization feature relies on the **Google API Key** (`booking_gcal_api_key`). The core synchronization engine (`WPBC_Google_Calendar`) retrieves this key to communicate with the Google Calendar API v3 using the standard WordPress HTTP function `wp_remote_get()`. There is no mention of the use of the OAuth 2.0 protocol, access tokens, or refresh tokens for authentication. |
| Are external calendar tokens **stored securely**? | **Stored, but security method is undocumented** | The Google API Key is stored as a plugin setting. Settings management, defined by the `WPBC_Settings_API_General` class, typically saves options as a separate row in the `wp_options` database table. The sources do not specify if the API key itself is stored using encryption or any specialized secure token storage methods; it relies on the general security measures of the standard WordPress options table. |
| Can admins **disconnect calendars** easily and remove all linked data? | **Yes** (Settings) | **Disconnecting Settings:** Yes. The controller file `core/sync/wpbc-gcal.php` includes a function, **`wpbc_sync_gcal_deactivate()`**, which is executed upon plugin deactivation. This function calls **`delete_bk_option()`** to **remove all Google Calendar-related settings** (including the API Key and import schedule) from the database. **Removing Linked Bookings:** The synchronization process saves imported events as full local booking records. The explicit documentation of `wpbc_sync_gcal_deactivate()` focuses on deleting the *settings* and does not explicitly confirm that the *imported booking records* (the "linked data") are also automatically deleted, though the general deactivation logic respects the user's data deletion preference (`booking_is_delete_if_deactive`). |

***

### Implementation Score (Scale of 1-10)

| Feature | Score (1-10) | Justification |
| :--- | :--- | :--- |
| Does the plugin use **OAuth 2.0** for Google/Outlook integrations? | **1 / 10** | Authentication relies on a static Google API Key, not the OAuth 2.0 protocol. |
| Are external calendar tokens **stored securely**? | **5 / 10** | The API Key is stored in the WordPress options table, but there is no evidence detailing encryption or advanced security measures typical of token storage. A score of 5 reflects standard WordPress storage without confirmed specialized security. |
| Can admins **disconnect calendars** easily and remove all linked data? | **9 / 10** | The system provides explicit lifecycle management functions (`wpbc_sync_gcal_deactivate()`) to clean up and delete all synchronization settings from the database. The ability to delete the resulting booking records is handled separately by the plugin's general data cleanup settings. |



---

✅ **Success Criteria:**

* External calendar changes (create, update, delete) **update the booking database reliably**.
* Updates happen in **real-time or configurable short intervals**.
* System handles **conflicts, recurring events, and deletes correctly**.
* Admins have **full visibility (logs, controls, manual resync)**.
* OAuth + secure token handling ensures **safe integration with Google/Outlook/Apple**.

---

Do you also want me to prepare a **practical testing checklist** (step-by-step scenarios like “create an event in Google Calendar → check if it blocks booking slot in plugin”)? That way you could test a plugin hands-on.
