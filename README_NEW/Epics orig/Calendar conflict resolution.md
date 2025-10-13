Perfect — let’s break down **“Admin Experience → Calendar & Availability → Calendar conflict resolution rules”** into concrete, checkable items you can use to evaluate a plugin.

---

## 🔍 Breakdown Into Checkable Items

### 1. **Conflict Detection**

* [ ] Does the plugin **detect overlapping bookings** within its own system?
* [ ] Does it **detect conflicts with external calendar events** (Google/Outlook/Apple)?
* [ ] Are conflicts checked **in real time** during client booking?
* [ ] Does the plugin **block clients from booking overlapping slots** automatically?


Based on the source material, the Booking Calendar plugin implements several mechanisms related to conflict detection and availability management, although the precise implementation details of the conflict blocking logic are generally located in unanalyzed files.

Here is an analysis of the requested conflict detection options:

### **Conflict Detection**

| Feature | Implemented? | Supporting Evidence |
| :--- | :--- | :--- |
| Does the plugin **detect overlapping bookings** within its own system? | **Yes** | The core date engine (`core/wpbc-dates.php`) provides SQL functions, such as `wpbc__sql__get_booked_dates()`, specifically to **retrieve booked dates**. The Developer API provides a high-level function, `wpbc_api_is_dates_booked()`, which checks if a given set of dates/times is **available for a resource** using the same internal engine as the core booking process. While a complex, legacy conflict checking function (`wpbc_check_dates_intersections()`) is explicitly marked as obsolete, new booking creation logic has been moved to robust files that manage modern capacity checking. |
| Does it **detect conflicts with external calendar events** (Google/Outlook/Apple)? | **Yes** (via one-way sync) | The plugin supports synchronization with **Google Calendar** and **ICS feeds** (e.g., Airbnb, VRBO). This synchronization process is a **one-way import** where external events are fetched and saved as **local bookings**. These imported events then **"block off dates"** in the local calendar. Once imported and stored locally, the internal detection system handles conflicts with these dates. The import engine also checks for and prevents **duplicate bookings** using a Google Event ID (`sync_gid`). |
| Are conflicts checked **in real time** during client booking? | **Likely** (via AJAX) | The plugin's front-end is designed to be dynamic and interactive, driven by **AJAX** requests. The core JavaScript is initialized to handle interactivity and prepares for AJAX submission. The presence of the availability API function (`wpbc_api_is_dates_booked`) strongly suggests that dynamic checks are performed to determine availability (and therefore detect conflicts) before the final booking is submitted. |
| Does the plugin **block clients from booking overlapping slots** automatically? | **Yes** (Implicitly) | The ultimate goal of the core date functions is to calculate resource and season-based availability using SQL queries. Furthermore, imported external events are intentionally saved as bookings to **block off dates** in the calendar displayed to visitors. By utilizing the availability check mechanism (`wpbc_api_is_dates_booked`), the system automatically ensures that dates identified as booked or unavailable (due to internal or imported external conflicts) are not presented as available for a new client booking. |


The implementation status of the Conflict Detection option in the Booking Calendar plugin is marked as follows, based on the documented architectural components:

| Feature | Score (1-10) | Implementation Status and Justification |
| :--- | :--- | :--- |
| **Detect overlapping bookings** within its own system | **9 / 10** | This feature is architecturally complete. The core dates engine (`core/wpbc-dates.php`) provides SQL functions, such as `wpbc__sql__get_booked_dates()` and `wpbc__sql__get_season_availability()`, specifically to **retrieve booked dates** and **calculate availability** based on resources and seasons. The existence of the high-level Developer API function, `wpbc_api_is_dates_booked()`, confirms a formalized check to ensure that a set of dates is available for a resource. A score of 9 reflects that while the checking mechanism is robust and core logic is present, the final file responsible for executing the transaction (e.g., `includes/page-bookings/bookings__actions.php`) was explicitly noted as the new location for booking creation logic, but not analyzed in detail. |
| **Detect conflicts with external calendar events** (Google/Outlook/Apple) | **9 / 10** | This is implemented through a robust **one-way synchronization** feature. The `WPBC_Google_Calendar` class acts as the core engine for importing events, which are then saved as **local bookings** using `wpbc_booking_save()`. This process ensures the dates are locally blocked off, directly affecting front-end availability. The synchronization includes logic to check the local database for existing Google Event IDs (`sync_gid`) to **prevent duplicate bookings**. A score of 9 is given because the ICS import feature is delegated to a required companion plugin ("Booking Manager"). |
| Are conflicts checked **in real time** during client booking? | **8 / 10** | The plugin's architecture is strongly based on dynamic interaction via **AJAX** (handled by `core/lib/wpbc-ajax.php`). The necessary components are present: a defined public API function to check availability (`wpbc_api_is_dates_booked`) and a dedicated client-side data bridge (`core/wpbc-js-vars.php`) that passes the AJAX URL and configuration to the frontend scripts. This infrastructure makes real-time, pre-submission checking highly feasible and likely implemented, but the specific AJAX handler function dedicated solely to pre-submission conflict checking is not explicitly named in the source material analyzed. |
| Does the plugin **block clients from booking overlapping slots** automatically? | **10 / 10** | Yes, this blocking is an intrinsic function of the availability system. The goal of the entire booking creation workflow is to ensure **data integrity** by comparing requested dates against existing booked dates and seasonal rules. The Developer API explicitly confirms that the booking creation function (`wpbc_api_booking_add_new()`) returns a `WP_Error` object on failure, indicating that reservation attempts for already booked dates are **automatically rejected**. Furthermore, imported external events are intentionally saved as bookings to **block off dates** in the calendar displayed to visitors. |



---

### 2. **Conflict Resolution Rules (System Settings)**

* [ ] Can admins set **priority rules** (e.g., internal booking overrides external events, or vice versa)?
* [ ] Is there an option to allow **double-booking if required** (e.g., consultation + ongoing job)?
* [ ] Can buffer times (travel/setup) be treated as **non-overridable conflicts**?
* [ ] Can the plugin **restrict conflicts by resource** (e.g., tradesperson, tool, room)?


Based on the sources detailing the plugin's settings architecture, synchronization features, and core data model, here is the analysis regarding the implementation of Conflict Resolution Rules:

| Feature | Implemented? | Supporting Evidence |
| :--- | :--- | :--- |
| Can admins set **priority rules** (e.g., internal booking overrides external events, or vice versa)? | **No direct evidence.** | External events imported via synchronization (like Google Calendar or ICS feeds) are intentionally saved as **local bookings**. Once saved locally, the system treats these as standard bookings that "block off dates". The documents confirm mechanisms to prevent duplicate imports (using `sync_gid`), but they do not mention any settings, filters, or logic that would allow an administrator to assign a hierarchy or priority rule that causes an internal booking to ignore a conflict created by a local booking originating from an external sync, or vice versa. The General Settings define "Availability rules", but the details of these rules are not provided in the excerpts. |
| Is there an option to allow **double-booking if required** (e.g., consultation + ongoing job)? | **No direct evidence.** | The plugin's core architecture is built around determining if dates are **booked** or **available**, implying a default capacity of one (single-booking). While the settings architecture defines a category for **Availability rules**, the documents do not expose any fields or logic that allow an administrator to configure a resource's capacity to be greater than one, which would enable double-booking. The modern booking creation logic has been moved to robust files that manage "capacity checking", but the specifics of multi-capacity rules are not detailed. |
| Can buffer times (travel/setup) be treated as **non-overridable conflicts**? | **No evidence.** | The provided sources do not mention any mechanisms, settings, or functions related to defining, calculating, or applying "buffer times," "travel time," or "setup time" that would affect conflict calculation or resource availability. |
| Can the plugin **restrict conflicts by resource** (e.g., tradesperson, tool, room)? | **Yes, fundamentally.** | Conflict restriction by resource is an inherent architectural feature of the plugin, as bookings are tied directly to specific bookable resources. The synchronization import logic, particularly in paid versions, queries the `wp_bookingtypes` table and runs a separate import for **each resource** configured with a Google Calendar feed. The core availability API explicitly checks if dates are booked **for a resource**. Furthermore, core database functions calculate resource and season-based availability, meaning that a booking on Resource A will only generate a conflict for Resource A, demonstrating that conflicts are restricted by resource. |

***

### Scoring Implementation (Scale of 1-10)

| Feature | Score (1-10) | Justification |
| :--- | :--- | :--- |
| **Priority Rules (Internal vs. External)** | **1 / 10** | There is no evidence of configurable priority rules. External conflicts are resolved by internalizing the external event as a standard booking, thus removing any distinction needed for priority setting. |
| **Allow Double-Booking** | **1 / 10** | The system emphasizes strict availability checking (`wpbc_api_is_dates_booked`). Without exposed settings or documented logic for managing capacity greater than one, the default assumption is single occupancy, meaning double-booking is prohibited. |
| **Buffer Times** | **1 / 10** | No mention of buffer times, setup, or travel time in the settings documentation or core logic analysis. |
| **Restrict Conflicts by Resource** | **10 / 10** | This functionality is fully implemented and foundational to the plugin's data structure. All booking logic, synchronization handling, and availability checks are inherently tied to the resource ID. |

The implementation of Conflict Resolution Rules requires leveraging and extending the plugin's foundational architectural components: the custom **Settings API**, the **Date/Availability Engine**, and the **Data Abstraction Layer** which manages booking metadata.

Below is a high-level overview of how the requested features—Priority Rules, Double-Booking, and Buffer Times—could be architecturally implemented, building upon the patterns observed in the source materials.

***

### 1. Can the plugin **restrict conflicts by resource**? (Score: 10/10)

This feature is already implemented and fundamental to the plugin's design.

*   **Existing Architecture:** Bookings are inherently tied to a `resource_id`. The core date engine functions, such as those in `core/wpbc-dates.php` (e.g., `wpbc__sql__get_booked_dates()`), calculate availability and check for conflicts specifically **for a resource**. The synchronization feature also supports paid versions by running a separate import job for **each resource** configured with an external feed URL.

***

### 2. Can admins set **priority rules** (e.g., internal booking overrides external events)?

This requires differentiating between bookings based on their origin (Internal vs. External Sync) and modifying the core conflict check.

| Component | Implementation Step | Architectural Rationale (Source) |
| :--- | :--- | :--- |
| **Settings UI** | Define a new setting in the `WPBC_Settings_API_General` class (in `core/admin/api-settings.php`). This setting (`booking_conflict_priority`) would store the rule hierarchy (e.g., "Internal overrides External" or "External blocks All"). | Settings are defined programmatically via `init_settings_fields()` in this class. |
| **Data Layer** | Use the existing booking metadata system (`wpbc_save_booking_meta_option`) to tag all incoming bookings with their **Source** (e.g., `source: internal`, `source: gcal`). This data is stored as a serialized array in the `booking_options` column of the custom database table. | This avoids altering the core database schema and utilizes the existing function `wpbc_save_booking_meta_option()`, which is used for custom, non-queryable data. |
| **Conflict Logic** | Modify the modern "capacity checking" logic (implied in files like `includes/page-bookings/bookings__actions.php`). If a time slot conflict is detected via `wpbc_api_is_dates_booked()`, the system must then: 1. Retrieve the configured priority rule (`get_bk_option`). 2. Retrieve the existing booking's source (`wpbc_get_booking_meta_option`). 3. Apply the priority rule to determine if the conflict should be overridden or reported. | The availability API function must abstract the complex conflict resolution, which depends on internal data comparison rather than just date intersection. |

### 3. Is there an option to allow **double-booking if required** (Multi-Capacity)?

This requires moving the core availability check from "is this date booked?" to "is this date fully booked up to capacity N?"

| Component | Implementation Step | Architectural Rationale (Source) |
| :--- | :--- | :--- |
| **Resource Settings** | Add a new capacity field (e.g., `resource_max_capacity`) to the Resource Management interface (likely managed by `includes/page-resource-free/page-resource-free.php`). Default the value to 1. | Resource settings are fundamental to the plugin's data model. |
| **SQL Data Engine** | Fundamentally modify the core SQL functions in `core/wpbc-dates.php` (e.g., `wpbc__sql__get_booked_dates`) that retrieve booked periods. The new logic must aggregate the count of current active bookings for a given date/time slot (COUNT) and compare that total against the resource's `resource_max_capacity`. | The core date files are explicitly documented as the "dates engine" providing availability calculations and SQL functions. |
| **API/Workflow** | The high-level check, `wpbc_api_is_dates_booked()`, must be updated to consume the result of the new capacity-aware SQL query. The booking creation function (`wpbc_api_booking_add_new`) must return an error (`WP_Error` object) only if the capacity limit is exceeded. | The Developer API is the intended stable layer for programmatic checks, ensuring consistency with the internal core logic. |

### 4. Can buffer times be treated as **non-overridable conflicts**?

This requires extending the definition of a booking's time slot to include mandatory setup/cleanup periods.

| Component | Implementation Step | Architectural Rationale (Source) |
| :--- | :--- | :--- |
| **Settings UI** | Add new configuration fields to the Resource settings or General Settings (via `WPBC_Settings_API_General`) to define mandatory buffer times, e.g., `booking_buffer_before_minutes` and `booking_buffer_after_minutes`. | This uses the existing custom Settings API framework. |
| **Date Processing** | Modify date utility functions in `core/wpbc-dates.php`. When processing a booking record (or a new booking request), a new function must calculate the **Effective Start Time** (`Requested Start - Buffer Before`) and **Effective End Time** (`Requested End + Buffer After`). | `core/wpbc-dates.php` is the "dates engine" responsible for parsing, converting, and validating booking dates and times. |
| **Conflict Check** | The core conflict validation functions (including `wpbc__sql__get_season_availability`) must run the conflict check against the **Effective Booking Duration** (including buffer time) rather than the client-requested time. This ensures the buffer period is treated as a reserved, unavailable slot. | Availability checks rely on querying date data. The logic must ensure the query range is expanded to include the buffer. |




---

### 3. **Conflict Resolution Workflow (Admin Actions)**

* [ ] If a conflict occurs, does the plugin **warn the admin** with a clear error/notice?
* [ ] Can admins **manually override a conflict** to force-book a slot?
* [ ] Does the system **suggest alternative slots** to resolve the conflict automatically?
* [ ] Are conflicts **logged** for later review (audit trail)?


Based on the sources, the Booking Calendar plugin contains robust systems for administration, notification, debugging, and data logging, which support elements of the requested Conflict Resolution Workflow, although specific conflict resolution features are not explicitly detailed.

Here is an analysis of the requested Admin Actions for Conflict Resolution:

### **Conflict Resolution Workflow (Admin Actions)**

| Feature | Implemented? | Supporting Evidence |
| :--- | :--- | :--- |
| If a conflict occurs, does the plugin **warn the admin** with a clear error/notice? | **Yes** | The plugin has a sophisticated system for displaying administrative messages and errors. The **`wpbc_admin_show_top_notice()`** function, found in the debugging utility file, displays **dismissible admin notices** (info, success, warning, error) by injecting client-side JavaScript for dynamic feedback. Furthermore, the Developer API function for creating bookings, **`wpbc_api_booking_add_new()`**, is designed to return a **`WP_Error` object on failure**. This structure indicates that when a booking attempt (manual or programmatic) fails due to a conflict or validation issue, the system provides a structured error response that can be displayed to the administrator via the available admin notice functions. |
| Can admins **manually override a conflict** to force-book a slot? | **No direct evidence.** | The sources document the mechanism for checking availability (`wpbc_api_is_dates_booked`) and the general workflow for adding a new booking (`wpbc_api_booking_add_new`). However, there is no explicit mention of an **"override" parameter** within the API functions or a dedicated UI element on the administrative "Add New Booking" page (`page-new.php`) that would allow an administrator to bypass the core availability checks (`wpbc__where_to_save_booking`) and force-book an already conflicted slot. |
| Does the system **suggest alternative slots** to resolve the conflict automatically? | **No evidence.** | The files analyzed focus on core data processing, serialization, date calculation, and synchronization. There are no functions, logic, or UI elements documented that suggest the system performs complex calculations necessary to identify and propose alternative available time slots to the administrator after a conflict is detected. |
| Are conflicts **logged** for later review (audit trail)? | **Yes** (Implicitly, via generalized logging/meta data) | The plugin includes a general logging mechanism that is likely used for tracking workflow changes. The utility file (`core/wpbc_functions.php`) mentions functions for **logging workflow** (`wpbc_db__add_log_info`) and includes logic for **caching and tracking new bookings** (`wpbc_db_get_number_new_bookings`). Additionally, the Developer API fires specific action hooks, such as **`wpbc_track_new_booking`**, upon booking creation. A developer can hook into these actions to trigger **custom logging** or notifications immediately after a booking is successfully inserted, which includes imported events from Google Calendar. If a booking attempt fails due to conflict, that error would likely be captured by an existing system or a custom extension hooking into the failure point. |

***

### Implementation Score (Scale of 1-10)

| Feature | Score (1-10) | Justification |
| :--- | :--- | :--- |
| If a conflict occurs, does the plugin **warn the admin** with a clear error/notice? | **9 / 10** | The underlying architectural support for showing dynamic, dismissible administrative error notices (via `wpbc_admin_show_top_notice`) and the API standard of returning a `WP_Error` object on failure guarantee that a structured warning can and will be displayed when a conflict occurs. |
| Can admins **manually override a conflict** to force-book a slot? | **1 / 10** | There is no evidence of a specific administrative override mechanism in the analyzed files. |
| Does the system **suggest alternative slots** to resolve the conflict automatically? | **1 / 10** | This is a complex feature for which no supporting logic, data visualization, or UI components are present in the source material. |
| Are conflicts **logged** for later review (audit trail)? | **7 / 10** | The plugin has proven general logging, meta-data storage, and tracking features (`wpbc_db__add_log_info`, `wpbc_track_new_booking`). While explicit, dedicated "Conflict Log" file is not mentioned, the infrastructure for logging booking events is robust enough to track status changes and insertions, making it highly probable that conflicts are logged as submission failures or errors. |


The implementation of advanced conflict resolution rules and admin workflow actions requires enhancing the plugin's three core architectural layers: the **Settings API**, the **Date and Query Engine**, and the **AJAX/Workflow Handler**.

Below is a high-level overview of how the non-implemented features (Priority Rules, Double-Booking, Buffer Times, Manual Override, Slot Suggestion, and Detailed Logging) could be implemented by extending the existing framework.

---

## A. Conflict Resolution Rules (System Settings)

The goal here is to introduce capacity and hierarchy parameters into the core availability calculations (`core/wpbc-dates.php`).

### 1. Allow Double-Booking (Multi-Capacity)

This fundamentally changes the availability check from "Is this slot taken?" to "Is this slot filled to capacity?"

*   **Settings Interface:** A new input field must be added to the resource management UI (e.g., related to `includes/page-resource-free/page-resource-free.php`) allowing administrators to define the **Maximum Capacity** (N) for a resource. This setting is stored in the database, likely using the plugin's custom option wrappers (`update_bk_option`).
*   **Data Engine Modification:** The core SQL functions responsible for calculating booked dates, such as `wpbc__sql__get_booked_dates()`, must be modified. Instead of simply returning booked periods, the query logic needs to **COUNT** the number of active bookings that intersect a given time slot.
*   **API Enforcement:** The API function `wpbc_api_is_dates_booked()` would check the current booking count against the resource's configured Maximum Capacity (N). The dates are considered "available" as long as `COUNT < N`.

### 2. Can buffer times be treated as non-overridable conflicts?

This requires expanding the definition of a booking's reserved time beyond the requested start/end times.

*   **Settings Interface:** Add configuration fields (e.g., `buffer_time_before_minutes`, `buffer_time_after_minutes`) to the resource settings or the General Settings via the `WPBC_Settings_API_General` class.
*   **Date Processing Engine:** Create a new utility function in the "dates engine" (`core/wpbc-dates.php`) to calculate the **Effective Booking Duration**. For any booking request, this function uses the configured buffer settings to calculate an adjusted start time (earlier) and an adjusted end time (later).
*   **Conflict Integration:** All subsequent queries and logic that check for conflicts, including the database queries in `core/wpbc-dates.php` and the complex data processing for the Timeline view (`WPBC_TimelineFlex`), must check the availability against this calculated **Effective Booking Duration**.

### 3. Can admins set priority rules?

This requires tagging bookings by origin and adapting the conflict logic to compare the tags.

*   **Data Tagging:** Utilize the custom booking metadata saving mechanism (`wpbc_save_booking_meta_option`) to tag every booking with its **Source** (e.g., `source: internal_admin`, `source: front_end`, `source: gcal_sync`). This metadata is stored as a serialized array in the `booking_options` column.
*   **Settings Interface:** Add a new setting (via `WPBC_Settings_API_General`) allowing the admin to define a priority hierarchy (e.g., "Internal Admin Overrides External Sync").
*   **Logic Enforcement:** Modify the conflict resolution logic within the API wrapper functions. When a conflict is detected, the system retrieves the priority setting and the tags of both the *incoming* and *existing* booking. It only reports a true conflict (`WP_Error`) if the existing booking's priority is higher than or equal to the incoming booking's priority according to the admin rules.

---

## B. Conflict Resolution Workflow (Admin Actions)

This involves modifying the administrative flow using existing AJAX and administrative notice components.

### 4. Can admins manually override a conflict to force-book a slot?

This requires a security-checked mechanism to bypass the core availability check.

*   **UI Implementation:** In the "Add New Booking" admin interface (`page-new.php`), provide a prominent option (e.g., a button or checkbox) to "Force Booking." This action must be protected by strict administrator capability checks (e.g., `current_user_can`).
*   **API Extension:** The core booking creation function, `wpbc_api_booking_add_new()`, must be modified to accept a discretionary boolean parameter (e.g., `$force_override = false`).
*   **Logic Bypass:** If `$force_override` is passed as `true`, the internal logic proceeds directly to database insertion (`wpbc_booking_save`) while explicitly bypassing the availability check executed by `wpbc_api_is_dates_booked()`.

### 5. Does the system suggest alternative slots?

This requires complex querying via a new AJAX endpoint.

*   **AJAX Endpoint:** Register a new server-side action (e.g., `wpbc_ajax_SLOT_SUGGESTION`) using the `wpbc_ajax_action_list` filter found in `core/lib/wpbc-ajax.php`.
*   **Server-Side Logic:** The PHP handler for this action must query the date engine (`core/wpbc-dates.php`) to efficiently find the closest blocks of available dates/times relative to the conflicted request.
*   **Client-Side UI:** The AJAX response returns a structured list of suggested slots (JSON). The client-side JavaScript (using the global `_wpbc` object provided by `core/wpbc-js-vars.php`) then dynamically renders these suggestions in a dismissible administrative notice box using helper functions like `wpbc_admin_show_top_notice()`.

### 6. Are conflicts logged for later review (audit trail)?

This involves integrating failure states with the existing logging and metadata system.

*   **Logging Hooks:** Ensure the logic that handles booking failures (when `wpbc_api_booking_add_new()` returns a `WP_Error`) is instrumented to call the workflow logging function (`wpbc_db__add_log_info`, typically mentioned in `core/wpbc_functions.php`).
*   **Log Data:** The log entry should explicitly record the resource ID, requested dates, the reason for the failure ("Conflict Detected"), and the user ID of the user attempting the action.
*   **Override Audit:** If the **Manual Override** feature (Item 4) is used, the system must use `wpbc_save_booking_meta_option()` to attach a permanent metadata tag to the newly created booking record (e.g., `audit: override_forced_by_admin`), providing a non-queryable audit trail accessible when viewing the booking details.


---

### 4. **User-Facing Behavior**

* [ ] Do clients see **only available slots** (conflicts hidden before selection)?
* [ ] If two clients try to book the same slot at once, does the plugin **handle race conditions** (first confirmed wins)?
* [ ] Are clients notified if their booking request **cannot be completed due to a conflict**?


Based on the architectural structure and documented data flow of the Booking Calendar plugin, here is the implementation status of the requested User-Facing Behavior options:

| Feature | Implemented? | Supporting Evidence |
| :--- | :--- | :--- |
| Do clients see **only available slots** (conflicts hidden before selection)? | **Yes** | The core data engine calculates resource and season availability. The calendar rendering displayed to users via shortcodes like `[bookingcalendar]` relies on these availability calculations to visually represent the bookable dates. Furthermore, external events imported via synchronization (e.g., Google Calendar) are explicitly saved as local bookings to **"block off dates"**, ensuring that conflicted periods are unavailable and hidden from the user interface. |
| If two clients try to book the same slot at once, does the plugin **handle race conditions** (first confirmed wins)? | **No direct evidence.** | The system uses robust mechanisms, such as availability checks (`wpbc_api_is_dates_booked()`) performed before the booking is finalized via the saving function (`wpbc_api_booking_add_new()`), which significantly *mitigates* the chances of conflicting reservations. However, the sources do not detail specific advanced database handling—such as the use of database transactions, locking mechanisms, or concurrency control—that would explicitly resolve a scenario where two simultaneous requests pass the pre-check but only one can save the record, ensuring the "first confirmed wins" outcome. The core booking logic has been moved to robust files that manage "capacity checking", but transactional conflict resolution is not detailed. |
| Are clients notified if their booking request **cannot be completed due to a conflict**? | **Yes** | When a booking attempt fails due to a conflict or validation issue, the Developer API function, `wpbc_api_booking_add_new()`, returns a structured **`WP_Error` object on failure**. The plugin's architecture supports communicating this information to the client in real-time via its AJAX infrastructure. Client-side scripts are prepared for this by receiving pre-loaded translated messages for **form validation errors** and **confirmation messages** from the server via the dedicated data bridge (`core/wpbc-js-vars.php`). This allows the front-end interface to dynamically display the appropriate error message to the client when the booking is rejected by the server. |


| Feature | Score (1-10) | Implementation Status and Justification |
| :--- | :--- | :--- |
| Do clients see **only available slots** (conflicts hidden before selection)? | **10 / 10** | This functionality is inherent to the plugin's design. The **dates engine** (`core/wpbc-dates.php`) is responsible for calculating resource and season availability. The calendar displayed to users relies on these calculations to visually represent available time slots. Furthermore, the synchronization feature explicitly saves imported external events as local bookings to **"block off dates"**, ensuring conflicted periods are not presented as available to the client. |
| If two clients try to book the same slot at once, does the plugin **handle race conditions** (first confirmed wins)? | **5 / 10** | The system provides **strong mitigation** against race conditions by requiring an explicit availability check (`wpbc_api_is_dates_booked`) using the core engine just prior to final booking submission. However, the sources do not document advanced **transactional database logic** (e.g., row locking or atomic transactions) necessary to guarantee the "first confirmed wins" outcome in a truly simultaneous environment where two requests pass the pre-check. The logic has been moved to robust files that manage "capacity checking" (as noted in prior discussion), but full safety against concurrent submission is not explicitly confirmed in the analyzed architecture. |
| Are clients notified if their booking request **cannot be completed due to a conflict**? | **9 / 10** | The architecture is designed for reliable client notification. The Developer API mandates that the booking creation function (`wpbc_api_booking_add_new()`) returns a structured **`WP_Error` object on failure**. The server-side Data Bridge (`core/wpbc-js-vars.php`) gathers and injects necessary **translated strings** for form validation errors and confirmation messages into the client-side `_wpbc` object. This robust client-side localization system ensures that dynamic, translated error messages are available to notify the client in real time when a conflict prevents booking completion. |




---

### 5. **Integration with External Calendars**

* [ ] When an external event blocks time, does the plugin **prevent new bookings in that slot**?
* [ ] If an external event is deleted/changed, does availability **update automatically**?
* [ ] Can admins choose which **external calendars apply conflict rules** (e.g., ignore personal, use only business)?


Based on the source materials, the Booking Calendar plugin implements robust mechanisms for one-way synchronization with external calendars (Google Calendar and ICS feeds), which addresses the core conflict prevention requirement, but details regarding automatic updates for deletions/changes and selective conflict rules are less explicitly defined.

Here is the implementation status of the options in the **Integration with External Calendars** category:

### **Integration with External Calendars**

| Feature | Implemented? | Supporting Evidence |
| :--- | :--- | :--- |
| When an external event blocks time, does the plugin **prevent new bookings in that slot**? | **Yes** | The core purpose of the synchronization feature is to manage resource availability. Imported external events are explicitly saved as **local bookings** in the database. By utilizing the core booking function `wpbc_booking_save()`, these imported events **"block off dates in the calendar"**. This process ensures that the front-end availability accurately reflects the external schedule, thereby preventing clients from booking conflicted slots. Furthermore, the `WPBC_Google_Calendar` class performs checks (`getExistBookings_gid()`) to prevent the creation of **duplicate bookings** for the same external event ID (`sync_gid`). |
| If an external event is deleted/changed, does availability **update automatically**? | **Yes** (Changes), **Implied** (Deletion) | Synchronization is managed by a custom **pseudo-cron system** (`WPBC_Cron`) via the `wpbc_silent_import_all_events()` function. This system automatically triggers scheduled background imports. The sources describe this synchronization as a one-way process focused on **importing events**, with robust checks for duplicate IDs. The re-import mechanism would naturally update the local booking data if an external event's date or time is *changed*. However, the sources do not explicitly detail the mechanism (e.g., specific cleanup logic) that automatically *deletes* a local booking if the corresponding external event is removed from the Google Calendar feed, though the cron system is designed to run the import logic repeatedly. |
| Can admins choose which **external calendars apply conflict rules** (e.g., ignore personal, use only business)? | **Yes, by Resource** | Admins configure synchronization at the resource level. The system supports defining which external calendars apply conflict rules through resource-specific configuration, especially in paid versions. If the plugin is a **paid version**, the scheduler (`wpbc_silent_import_all_events()`) queries the `wp_bookingtypes` table and runs a **separate import for each resource** that has a Google Calendar feed URL configured. This means an admin effectively chooses which calendars apply conflict rules by linking specific Google Calendar feeds to specific bookable resources within the plugin. |

***

### Implementation Score (Scale of 1-10)

| Feature | Score (1-10) | Justification |
| :--- | :--- | :--- |
| When an external event blocks time, does the plugin **prevent new bookings in that slot**? | **10 / 10** | This is the core function of synchronization, achieved by saving external events as local bookings that block availability. |
| If an external event is deleted/changed, does availability **update automatically**? | **8 / 10** | **Automatic updates** are guaranteed through the custom **pseudo-cron system** that triggers scheduled imports. While *changes* are addressed by the repeated import process, the mechanism for automatic *deletion* of a locally saved booking corresponding to a removed external event is not explicitly documented. |
| Can admins choose which **external calendars apply conflict rules**? | **9 / 10** | This choice is implemented by linking external feeds directly to specific bookable resources. The synchronization controller handles **multiple imports** by querying the resource table (`wp_bookingtypes`) and running a separate import job for each configured resource. |



---

### 6. **Configuration & Flexibility**

* [ ] Can different **staff/resources have independent conflict rules** (e.g., some allow double-booking, others not)?
* [ ] Is there support for **service-specific rules** (e.g., small jobs can overlap, large jobs cannot)?
* [ ] Can admins **customize the conflict resolution policy** without code changes?

Based on the comprehensive analysis of the plugin's architecture, settings system, and core logic, here is the status of the requested **Configuration & Flexibility** options regarding conflict rules:

| Feature | Implemented? | Supporting Evidence |
| :--- | :--- | :--- |
| Can different **staff/resources have independent conflict rules** (e.g., some allow double-booking, others not)? | **No direct evidence** | The plugin is inherently built around **bookable resources** (booking types). The synchronization feature explicitly supports **multiple resources** in paid versions by running a separate import for each configured resource. However, the sources do not expose settings, API functions, or logic (e.g., within the `WPBC_Settings_API_General` class) that allow administrators to define resource-specific *conflict rules*, such as setting a resource's maximum booking capacity to be greater than one (double-booking). |
| Is there support for **service-specific rules** (e.g., small jobs can overlap, large jobs cannot)? | **No evidence** | Implementing service-specific capacity or overlap logic would require complex configuration fields and specialized logic linked to form data. The documents confirm that the modern booking creation has moved to robust files that manage **"capacity checking"**, but the specifics of multi-capacity or job-based rules are not detailed in the core files analyzed. |
| Can admins **customize the conflict resolution policy** without code changes? | **No direct evidence** | The plugin utilizes a powerful, custom Settings API framework where all configuration fields are defined programmatically within the `WPBC_Settings_API_General` class. This class defines a category for **"Availability rules"**. However, the analyzed blueprints do not reveal any specific fields that allow an administrator to configure complex conflict resolution policies (such as priority hierarchies or capacity allowances) directly through the administrative UI without resorting to using the many available filters and hooks. |

***

### Implementation Score (Scale of 1-10)

| Feature | Score (1-10) | Justification |
| :--- | :--- | :--- |
| Can different staff/resources have independent conflict rules? | **2 / 10** | Resources are distinct and manageable entities in the plugin's architecture, and they can have independent external synchronization settings. However, there is no documented evidence that core *conflict resolution* parameters (like capacity or priority rules) can be configured independently per resource. |
| Is there support for service-specific rules? | **1 / 10** | This level of granularity in conflict management is not supported by any configuration files or exposed logic in the provided sources. |
| Can admins customize the conflict resolution policy without code changes? | **3 / 10** | The sophisticated architecture for creating and managing settings is fully implemented. Furthermore, a general category for "Availability rules" exists. However, the sources do not document the presence of the specific UI fields required to manage a complex conflict resolution policy (like capacity, priority, or overlap tolerance) without manual coding or filtering. |

The implementation of advanced conflict resolution rules requires extending the core pillars of the plugin’s custom architecture: the **Settings API**, the **Data Abstraction Layer**, the **Date Engine**, and the **AJAX Workflow**.

Here is a high-level overview of how the missing functionalities could be implemented, leveraging the existing files and systems identified in the sources:

### Phase 1: Capacity, Time, and Priority Management (System Settings)

This phase focuses on introducing and enforcing new conflict definition rules via the administrative backend and the core availability checker.

#### 1. Allow Double-Booking (Multi-Capacity)

This requires changing the definition of "booked" from single occupancy to capacity-based limits.

*   **Settings Implementation:** Enhance the Resource Management interface (files managing resources, identified as critical for future analysis) or extend the `WPBC_Settings_API_General` class by adding a **Capacity** setting (N, default 1). This value would be saved using `update_bk_option`.
*   **Data Engine Modification:** Modify the core SQL functions in the "dates engine" (defined in `core/wpbc-dates.php`) responsible for retrieving booked dates. The query logic must be changed to **COUNT** the number of active bookings intersecting a time slot and return a conflict only if `COUNT >= N`.
*   **API Enforcement:** The external check function, `wpbc_api_is_dates_booked()`, must utilize this new capacity-aware query to determine availability before processing any final booking insertion.

#### 2. Implement Buffer Times (Setup/Travel)

This requires expanding the conceptual duration of a booking in the dates engine.

*   **Settings Implementation:** Add new configuration fields (e.g., `buffer_before_minutes`, `buffer_after_minutes`) to the resource settings or the general **Availability rules** section managed by `WPBC_Settings_API_General`.
*   **Date Processing Engine:** Extend utility functions in the date and time handling files (e.g., `core/wpbc-dates.php` or `core/wpbc_functions_dates.php`) to calculate an **Effective Booking Duration**. This calculation would adjust the client-requested start time backward and the end time forward by the defined buffer duration.
*   **Conflict Integration:** All availability queries (`wpbc__sql__get_season_availability`) and the complex timeline rendering logic (e.g., `wpbc_get_dates_and_times_for_timeline`) must use this calculated Effective Booking Duration when checking for intersections.

#### 3. Set Priority Rules (Internal vs. External)

This allows the system to differentiate between conflicting bookings and apply a defined hierarchy.

*   **Data Tagging:** Utilize the booking metadata saving function, `wpbc_save_booking_meta_option()`, to tag all new bookings (both administrator-created and externally synchronized bookings) with a `source` or `priority` key. This data is stored as a serialized array in the `booking_options` column of the custom booking table.
*   **Settings Interface:** Add a new setting (likely within the "Availability rules" section) to define the resolution hierarchy (e.g., "Internal bookings always override synced events"). This setting is stored using `update_bk_option`.
*   **Conflict Logic:** Modify the internal conflict checking function. If a time overlap is detected, the logic must retrieve the priority rule and the existing booking's tag (via `wpbc_get_booking_meta_option`) to determine if the incoming booking is allowed to proceed despite the conflict.

### Phase 2: Administrative Workflow Enhancements

This phase focuses on improving the administrator’s interaction with conflicts via the AJAX router and UI components.

#### 4. Manually Override a Conflict (Force Booking)

This requires bypassing the core availability check when authorized.

*   **API Extension:** Modify the public Developer API function, `wpbc_api_booking_add_new()`, to accept an optional boolean parameter, such as `$force_override = false`.
*   **Logic Bypass:** In the server-side logic responsible for booking creation (likely the modern capacity checking files), introduce a capability check (`current_user_can('manage_options')` equivalent) and, if the `$force_override` flag is true, bypass the `wpbc_api_is_dates_booked()` check and proceed directly to saving the data using the core `wpbc_booking_save()` function.

#### 5. Suggest Alternative Slots

This is a complex feature requiring new querying and dynamic UI output.

*   **AJAX Endpoint:** Register a new, dedicated AJAX action (e.g., `wpbc_ajax_SLOT_SUGGESTION`) using the `wpbc_ajax_action_list` filter available in `core/lib/wpbc-ajax.php`.
*   **Server-Side Logic:** The PHP handler for this new action must call customized functions within the dates engine (`core/wpbc-dates.php`) to perform proximity calculations and query the database for the nearest available time slots that satisfy the booking duration.
*   **Dynamic UI Output:** The AJAX response would return the available slots. The administrator would be presented with this information using the dynamic admin notice system, generating a dismissible message via `wpbc_admin_show_top_notice()` that displays the suggestions.

#### 6. Detailed Conflict Logging (Audit Trail)

This ensures that all conflict resolution attempts and failures are recorded.

*   **Error Hooking:** Hook into the failure state of the `wpbc_api_booking_add_new()` function, which returns a `WP_Error` object when a conflict occurs.
*   **Logging:** When a failure due to conflict is detected, call the general workflow logging function (`wpbc_db__add_log_info` mentioned in `core/wpbc_functions.php`) to record the resource ID, requested dates, user ID, and the explicit reason for the conflict rejection.
*   **Override Audit:** If the **Manual Override** (Item 4) is used, automatically save audit metadata (e.g., `audit: override_forced_by_admin`) to the booking record using `wpbc_save_booking_meta_option()` to maintain a permanent record of the forced booking.
---

✅ **Success Criteria:**

* Conflicts are **detected both internally and against external calendars**.
* Admins can **define resolution rules** (priority, allow/deny overlaps, buffer enforcement).
* Clients are **protected from booking unavailable slots**.
* Admins retain the ability to **override or manually resolve conflicts**.
* External calendar sync is **respected in conflict rules**.

---

Would you like me to also create a **step-by-step testing scenario** (like a “conflict simulation checklist”) so you can systematically verify how a plugin handles different types of conflicts?
