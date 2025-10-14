Excellent — this one’s important because it touches both **data minimisation** and **calendar integration privacy** under GDPR.
Let’s break it down into **auditable items** you can check against your plugin’s implementation (for Google Calendar or Outlook).

---

### ✅ **Feature: Calendar Privacy – “Store personal calendar events as ‘busy’ only”**

**Category:** GDPR & UK Data Protection Act
**Priority:** Must-Have
**Effort:** 16h
**Risk:** High (GDPR compliance)
**Dependencies:** Google Calendar API / Microsoft Outlook API

---

### 🔍 **Breakdown for Plugin Audit**

#### 1. **Calendar Connection Scope**

| Item | Description                                                                                                                                                   | How to Check                                                       |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| 1.1  | The plugin requests **only minimal permissions** from Google or Outlook (e.g., `read-only busy/free` or `calendar.events.list` with `private` fields masked). | Review OAuth scopes in the integration setup or developer console. |
| 1.2  | The plugin does **not** store or sync event details (title, attendees, description) unless explicitly needed.                                                 | Check what fields are fetched from API responses.                  |

The source documents provide extensive detail regarding the architecture of the **Google Calendar (GCal) synchronization** feature, but they do not explicitly contain the security or scope details necessary to definitively answer all parts of your query.

Here is an analysis of whether the provided synchronization options are implemented:

### 1.1 Minimal Permissions / Scope

| Item | Status | Source Information and Details |
| :--- | :--- | :--- |
| **1.1** The plugin requests **only minimal permissions** from Google or Outlook (e.g., `read-only busy/free` or `calendar.events.list` with `private` fields masked). | **Inconclusive (Scope Not Explicitly Defined)** | The synchronization process is a **one-way import** designed to fetch events from a **public Google Calendar**. The configuration requires a **Google API Key**. The `WPBC_Google_Calendar` class uses the standard WordPress function `wp_remote_get()` to communicate with the **Google Calendar API v3**. While using a public feed and API key suggests a limited read-only scope, the architectural review **does not specify the required OAuth scopes** (e.g., `calendar.events.list`) or confirm if it operates purely in `read-only busy/free` mode. |

### 1.2 Minimal Data Storage

| Item | Status | Source Information and Details |
| :--- | :--- | :--- |
| **1.2** The plugin does **not** store or sync event details (title, attendees, description) unless explicitly needed. | **Partially Implemented (Title, Description, and Location *Are* Stored)** | The plugin **does store** event details derived from the Google Calendar feed. The core synchronization engine extracts event details such as the **title, description, and location** from the JSON data returned by the API. These fields are then mapped by the administrator's configuration (via the `booking_gcal_events_form_fields` option) to the local booking form fields. Every successfully imported Google event is saved as a **new booking** in the local database, which includes this mapped data. The purpose of importing is to **block off dates** and directly affect front-end availability. Therefore, the details (title, description, location) are stored locally as part of the booking record, fulfilling the requirement for the plugin's data model. The sources do not explicitly mention the storage of *attendees*. |



Based on the analysis of the Google Calendar synchronization feature described in the sources, the implementation status of the security scopes can be scored as follows:

| Item | Status Rating (1-10) | Justification Based on Source Material |
| :--- | :--- | :--- |
| **1.1** The plugin requests **only minimal permissions** from Google or Outlook (e.g., `read-only busy/free` or `calendar.events.list` with `private` fields masked). | **5 / 10** | **Inconclusive Implementation.** The synchronization is confirmed to be a **one-way import** from a **public Google Calendar**, which implies a certain degree of read-only, minimal permission. However, the sources **do not define the specific OAuth scopes** used or guarantee that private fields are masked. Furthermore, the engine extracts detailed fields like **title, description, and location**, which goes beyond a simple `read-only busy/free` check. The rating of 5 reflects the inherent read-only nature of the import, balanced against the lack of technical scope detail and the retrieval of non-minimal data fields. |
| **1.2** The plugin does **not** store or sync event details (title, attendees, description) unless explicitly needed. | **2 / 10** | **Feature Contradiction.** This requirement is largely contradicted by the documented architectural necessity of the feature. The synchronization process **explicitly stores** event details retrieved from the API. The engine extracts the **title, description, and location** from the Google feed and maps these details to local booking form fields using the `booking_gcal_events_form_fields` option. Every imported Google event is saved as a **new booking** in the local database via `wpbc_booking_save()`, which includes the stored title and description data. Therefore, the plugin **does sync and store** the requested event details (except for attendees, which are not mentioned in the sources). |

The implementation of the **Calendar Connection Scope** requirements—specifically ensuring **only minimal permissions (read-only busy/free)** and **not storing event details**—would require significant architectural changes focused primarily on data handling, replacing the current practice of saving imported events as full local bookings.

The following is a high-level overview of how this minimal-scope synchronization could be implemented, leveraging the existing architecture of the Booking Calendar plugin:

---

### High-Level Implementation Overview

The current Google Calendar (GCal) synchronization architecture is split between a controller (`core/sync/wpbc-gcal.php`) that manages scheduling and configuration, and a core engine (`WPBC_Google_Calendar` class) that handles API communication and database saving.

To meet the requirements of minimal permission and minimal data storage, the implementation would focus on modifying the core engine's API request and replacing its final database saving method.

### 1. Architectural Setup (The Controller Layer)

The scheduler and configuration systems would remain largely intact, but the UI must be simplified:

*   **Remove Data Mapping UI:** The administrator configuration interface (rendered by helper functions like `wpbc_gcal_settings_content_field_*()` in `wpbc-gcal.php`) would **eliminate fields** related to mapping event title, description, and location to local form fields (the current implementation uses the `booking_gcal_events_form_fields` option for this purpose). This physically prevents the admin from trying to import detailed data.
*   **Cron Management:** The automated scheduling remains functional. The function `wpbc_silent_import_all_events()` would still be triggered by the custom pseudo-Cron system (`WPBC_Cron` class) to start the background import process.
*   **Configuration Delivery:** The controller would continue to retrieve necessary settings (API Key, Calendar ID, resource ID, time window, etc.) using `get_bk_option()` and pass them to the core engine's setter methods (`setUrl()`, `setResource()`, etc.).

### 2. Core Engine Modification (Minimal Fetch and Data Transformation)

The core `WPBC_Google_Calendar` class would need substantial refactoring, which the sources note is typically risky due to the complexity of API authentication and date conversion.

*   **Minimal Scope API Request (Requirement 1.1):**
    *   The `WPBC_Google_Calendar` class uses `wp_remote_get()` to communicate with the GCal API v3. This request must be modified to use a Google Calendar API endpoint that specifically returns **busy/free time slots** rather than a list of detailed events (`calendar.events.list`), thus requesting the absolute minimal permission necessary for availability checking.
    *   The engine must be configured to fetch *only* the start and end timestamps for unavailable periods, ignoring fields like summary, description, and attendees that typically accompany full event details.
*   **Data Extraction:** The processing logic would bypass the current steps that extract details such as the event **title, description, and location**. It would focus solely on extracting the **start date/time** and **end date/time** for the blocking period.
*   **Date Conversion:** The engine would still rely on its internal helper methods, such as `getCommaSeparatedDates()`, to convert Google's date/time format into the internal format used by the plugin's date engine (`core/wpbc-dates.php`).

### 3. Data Storage Rerouting (Requirement 1.2)

To avoid storing event details (title, description), the implementation must bypass the normal booking creation pipeline:

*   **Bypass `wpbc_booking_save()`:** Currently, the engine calls the core function `wpbc_booking_save()` for every imported event. This step creates a **new booking** in the local database, which is where the unwanted event details (title, description) are stored. This call must be entirely removed.
*   **Direct Availability Meta Storage:** Instead of creating a booking, the system would perform a **direct database query** (using the `$wpdb` object) to save the busy/free dates directly as *resource unavailability metadata* (or equivalent data) associated with the target resource ID. This lightweight metadata would only record the necessary dates and times, ensuring no extraneous event details are kept locally.
*   **Integration with Availability Engine:** The new storage location must be integrated into the plugin’s availability calculation logic. The plugin relies on functions in `core/wpbc-dates.php` (such as `wpbc__sql__get_booked_dates()` or `wpbc__sql__get_season_availability()`) to compute front-end availability. The availability calculation queries must be updated to include a check against this new minimal data table/column to correctly mark the dates as blocked.

By replacing the `wpbc_booking_save()` call with a targeted update to an availability-only data structure, the plugin meets the requirement of minimal data storage while still ensuring that front-end availability is accurately maintained.






---

#### 2. **Privacy Mode Option**

| Item | Description                                                                                      | How to Check                                                                         |
| ---- | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------ |
| 2.1  | Users can **toggle** between “Show details” and “Busy only” when connecting a calendar.          | Look for a setting in plugin UI (e.g., “Privacy mode” or “Display only busy times”). |
| 2.2  | Default behaviour should be **“Busy only”** for compliance (minimise data).                      | Check plugin defaults or onboarding instructions.                                    |
| 2.3  | When “Busy only” is enabled, events are displayed as **blocked slots** with no identifying info. | Inspect the front-end booking calendar rendering.                                    |


Based on the detailed architectural review of the synchronization features (Google Calendar and ICS), the **Privacy Mode Option** as you have defined it is **not implemented** in the plugin. The existing architecture is designed primarily for detailed data import and storage, which directly contradicts the "Busy only" mode requirement.

Here is an analysis of each item based on the source documents and our previous discussion regarding data handling:

### Implementation Status: Not Implemented

| Item | Status | Justification Based on Source Information |
| :--- | :--- | :--- |
| **2.1** Users can **toggle** between “Show details” and “Busy only” when connecting a calendar. | **Not Implemented** | The source analysis, focusing on the configuration UI (`core/sync/wpbc-gcal.php` and settings systems), does **not** mention any specific setting, toggle, or UI element that allows an administrator to select between importing detailed event information or restricting the import to busy/free status only. The synchronization setup requires configuration of parameters like the Google API Key and resource ID. |
| **2.2** Default behaviour should be **“Busy only”** for compliance (minimise data). | **Not Implemented** | The default behavior of the synchronization feature is to perform a detailed import. The system fetches event details (including the **title, description, and location**) from the Google Calendar API or ICS feed. These details are configured to be mapped to local booking form fields (`booking_gcal_events_form_fields` option). Therefore, the system defaults to capturing and storing identifying data, not minimizing it. |
| **2.3** When “Busy only” is enabled, events are displayed as **blocked slots** with **no identifying info**. | **Contradicted by Architecture** | This function is directly contradicted by the core architecture of the existing synchronization process. The `WPBC_Google_Calendar` class saves every successfully imported external event as a **new booking** in the local database by calling the critical integration function `wpbc_booking_save()`. This process **stores** the event's identifying details (**title, description, and location**). Although the purpose of the imported booking is to **block off dates**, the identifying information is permanently stored as part of the resulting local booking record. |

In summary, the design intent of the current synchronization feature is data consumption and storage (Title, Description, Location) for the purpose of maintaining accurate availability. It lacks the specific UI controls and minimal data processing required to support a "Busy only" privacy mode.



Based on the architectural analysis of the synchronization features described in the sources, the **Privacy Mode Option** is **not implemented**. The existing system's design for data storage directly conflicts with the requirements for minimizing data and providing privacy controls.

Here is the rating for each item on a scale of 1 to 10:

| Item | Status Rating (1-10) | Justification Based on Source Material |
| :--- | :--- | :--- |
| **2.1** Users can **toggle** between “Show details” and “Busy only” when connecting a calendar. | **1 / 10** | **Not Implemented.** The documentation details the configuration UI functions (`wpbc_gcal_settings_content_field_*()`) but does not mention any setting, toggle, or control that allows an administrator to switch the synchronization scope between detailed imports and restricted busy/free information. |
| **2.2** Default behaviour should be **“Busy only”** for compliance (minimise data). | **1 / 10** | **Contradictory Default.** The default architectural behavior is the opposite of minimizing data. The synchronization engine is designed to **fetch and store** detailed event fields—such as the **title, description, and location**—which are mapped to local booking form fields (via the `booking_gcal_events_form_fields` option). |
| **2.3** When “Busy only” is enabled, events are displayed as **blocked slots** with **no identifying info**. | **1 / 10** | **Architecturally Contradicted.** While imported events successfully **block off dates** and affect front-end availability, the critical step in the process is calling the global function **`wpbc_booking_save()`**. This means every external event is saved as a complete **new booking** in the local database, permanently storing the event's identifying information (title, description, location). Thus, the requirement that the identifying information is *not stored* cannot be met by the current design. |

The low score of **1/10** across all items reflects that the core architecture for synchronization is fundamentally designed for data consumption and persistence, which conflicts directly with the goal of data minimization and scoped privacy control.


This implementation requires modifying the plugin's architecture at the Configuration, Synchronization Engine, and Data Persistence layers, directly contradicting the current design, which is centered on storing detailed event data as local booking records.

Drawing on the plugin's established modular patterns, here is a high-level overview of how the **Privacy Mode Option** would be implemented:

---

### High-Level Implementation Overview: "Busy Only" Mode

The goal is to move the synchronization from a **Data Persistence** model (saving full bookings) to an **Availability Signaling** model (saving only dates blocked).

### 1. Configuration and UI Layer (Requirements 2.1 & 2.2)

This addresses the ability to toggle the mode and set the default behavior.

*   **Settings Field Registration:** A new setting, for example, `booking_gcal_privacy_mode`, would be registered within the synchronization configuration system (`core/sync/wpbc-gcal.php`). The setting's default value would be set to `"Busy only"` to meet the compliance requirement.
*   **Admin UI Implementation:** UI helper functions (`wpbc_gcal_settings_content_field_*()` in `core/sync/wpbc-gcal.php`) would render a toggle or selection field to switch between "Show details" (legacy behavior) and "Busy only" (new privacy mode).
*   **Dynamic Field Hiding:** Client-side JavaScript (using jQuery, similar to the logic found in `api-settings.php` for dynamic UI interactivity) would hide the existing configuration fields that define data mapping (the fields related to the `booking_gcal_events_form_fields` option) when "Busy only" is active. This reinforces that identifying data (title, description, location) will not be used.

### 2. Synchronization Engine Layer (Requirement 2.3 – Minimal Fetch)

The core engine (`WPBC_Google_Calendar` class) must be modified to request and process only the minimum necessary data.

*   **Configuration Check:** The automated execution function, `wpbc_silent_import_all_events()`, which is triggered by the custom pseudo-cron system (`WPBC_Cron`), would retrieve the `booking_gcal_privacy_mode` setting.
*   **Minimal API Call:** If "Busy only" mode is detected:
    *   The `WPBC_Google_Calendar` class must construct its request (using `wp_remote_get()`) to the Google Calendar API v3 using a scope or API endpoint that requests only busy/free information, thus avoiding retrieval of event summaries, titles, or descriptions.
    *   The core processing logic within the class's `run()` method would be bypassed for all data fields except for the event start time and end time.
*   **Data Transformation:** The engine would still rely on its date handling utilities (like converting the time format using a method similar to `getCommaSeparatedDates()`) to prepare the date ranges for the internal availability engine.

### 3. Data Persistence Layer (Requirement 2.3 – Minimal Storage)

This is the most critical step, requiring the system to stop creating full booking records for imported events.

*   **Bypass `wpbc_booking_save()`:** When in "Busy only" mode, the `WPBC_Google_Calendar` class's final step (`createNewBookingsFromEvents()`) must **not** call the core database insertion function `wpbc_booking_save()`. This ensures that detailed data, such as the event title or description, is not serialized and stored in the database.
*   **Direct Availability Data Storage:** Instead of creating a booking:
    *   The system would save the fetched date ranges directly to a minimal database structure (e.g., a new custom table or a specific resource metadata entry) dedicated solely to external unavailability data. This data would only include the Resource ID and the blocked date/time range.
    *   This data structure would replace the need for the duplication check (`getExistBookings_gid()`), as the system would simply overwrite the previously imported unavailability data during each cron run.
*   **Availability Engine Integration:** The core date engine functions (`core/wpbc-dates.php`) responsible for querying availability (e.g., those producing SQL queries like `wpbc__sql__get_booked_dates()`) must be updated. They would need to query this new minimal availability data source *in addition* to querying the standard local booking tables to correctly mark dates as unavailable on the front-end calendar.


---

#### 3. **Data Storage & Processing**

| Item | Description                                                                                                  | How to Check                                                |
| ---- | ------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------- |
| 3.1  | Calendar event data is **not stored permanently** in plugin DB — only temporary availability data is cached. | Inspect DB schema or API logs for stored fields.            |
| 3.2  | If event details are stored (for scheduling logic), there is **clear user consent**.                         | Check consent flow or settings.                             |
| 3.3  | No event metadata (titles, attendees, descriptions) is sent to external analytics or third-party APIs.       | Check webhook, API, or analytics integration configuration. |

Based on the analysis of the Booking Calendar plugin's synchronization features and its core data handling mechanisms, the **Data Storage & Processing** options you listed are generally **not implemented** in the manner requested, and in some cases, are contradicted by the documented architectural behavior.

Here is a detailed breakdown of each item:

### 3.1 Calendar event data is not stored permanently in plugin DB — only temporary availability data is cached.

**Status: Contradicted (Data is Stored Permanently)**

The source documents explicitly detail that external calendar events (from Google Calendar or ICS feeds) are imported and **stored permanently** in the plugin's database as new booking records.

*   **Storage Mechanism:** Every successfully imported Google Calendar event is saved as a **new booking** in the local database via a call to the core function `wpbc_booking_save()`.
*   **Data Stored:** The synchronization engine extracts and stores event details such as the **title, description, and location**. These fields are mapped to local booking form fields using the `booking_gcal_events_form_fields` option.
*   **Effect:** The imported data directly affects availability by **blocking off dates** in the calendar displayed to visitors.
*   **Conclusion:** The event data is not treated as temporary availability cache; rather, it is converted into persistent local booking records, which store the identifying metadata permanently.

### 3.2 If event details are stored (for scheduling logic), there is clear user consent.

**Status: Not Implemented (Consent Flow Not Mentioned)**

The sources provide extensive detail on the plugin's architectural components, including:

*   **Settings System:** The abstract `WPBC_Settings_API` framework defines the structure for all configuration pages.
*   **Form Parsing:** The `core/form_parser.php` file handles the validation and interpretation of administrator-configured forms.
*   **Synchronization Setup:** The administrative UI (`wpbc-gcal.php`) allows configuration of parameters like the Google API Key, Calendar ID, and field mappings.

However, the sources **do not mention any specific feature, setting, or flow** related to obtaining **clear user consent** (or administrator consent regarding the end-user data stored in the Google Calendar feed) specifically for the storage of event details. The process relies on the administrator configuring the import job using the Google API Key and Calendar ID.

### 3.3 No event metadata (titles, attendees, descriptions) is sent to external analytics or third-party APIs.

**Status: Inconclusive (Internal Hooks Exist for Third-Party Integration)**

While the sources do not specify that the plugin itself sends imported event metadata to *external analytics platforms*, the architecture is designed with **extensibility hooks** that allow developers to integrate with third-party APIs immediately after a booking is created or modified.

*   **Import Process:** Event metadata (title, description, location) is successfully extracted and saved locally as a new booking.
*   **Post-Creation Hooks:** The synchronization process ends with a call to the global function `wpbc_booking_save()`. This function fires critical action hooks, such as **`wpbc_track_new_booking`**, and potentially others like `wpbc_booking_approved`.
*   **Extension Intent:** Developers are explicitly instructed that these hooks are the **safest extension points** to trigger custom functionality—such as **integrating with an external CRM or external calendar service**—immediately after a booking (including an imported one) is finalized.

Therefore, while the core plugin is not documented as having built-in analytics integration, the architecture provides robust and intentional **hooks** that permit event metadata to be relayed to external systems by companion plugins or custom code.

Based on the comprehensive architectural review of the synchronization features (Google Calendar and ICS) provided in the sources and our previous conversation, here is the rating for the **Data Storage & Processing** options:

| Item | Status Rating (1-10) | Justification Based on Source Material and Conversation History |
| :--- | :--- | :--- |
| **3.1** Calendar event data is **not stored permanently** in plugin DB — only temporary availability data is cached. | **1 / 10** | **Contradicted by Core Architecture.** The synchronization process actively converts external calendar events into **permanent booking records** in the local database by calling **`wpbc_booking_save()`**. These records explicitly store details like **title, description, and location**, which are mapped from the API feed using the `booking_gcal_events_form_fields` option. The data is designed for **persistence**, not temporary caching. |
| **3.2** If event details are stored (for scheduling logic), there is **clear user consent**. | **1 / 10** | **Consent Flow Not Implemented.** The sources detail the plugin's settings systems and synchronization configuration, but there is **no mention of any UI element, flow, or setting** designed to obtain clear administrator or user consent specifically for the storage of external event details (title, description, location). |
| **3.3** No event metadata (titles, attendees, descriptions) is sent to external analytics or third-party APIs. | **3 / 10** | **Inconclusive Implementation (High Extensibility Risk).** The core synchronization engine does not appear to send metadata to external analytics. However, the crucial integration point is the final saving step: **`wpbc_booking_save()`**. This function is known to fire critical action hooks like **`wpbc_track_new_booking`**. Developers are recommended to use this hook to trigger custom functionality (e.g., integrating with an external CRM or external calendar service). Because the metadata (title, description) is fully saved and available when these hooks fire, the architecture is designed to **permit and facilitate** sending this metadata to third-party APIs via extension, though the core plugin itself is not documented as doing so. The low rating reflects the lack of explicit denial coupled with the high architectural risk posed by the exposed metadata via hooks. |


The current synchronization feature relies on converting external calendar events into **permanent local booking records**. To implement the requested **Data Storage & Processing** requirements—namely minimal storage, clear consent, and protection from third-party data leakage—the synchronization architecture must undergo a fundamental shift from a **Data Persistence Model** to an **Availability Signaling Model**.

This implementation overview utilizes the plugin's existing architectural components (Settings API, Custom Cron, and Date Engine) while replacing the critical database insertion mechanism.

### A. Core Architectural Shift: From Permanent Booking to Temporary Cache (Requirement 3.1)

The most critical step is eliminating the creation of permanent booking records for imported events.

1.  **Minimal Data Retrieval:**
    *   The **`WPBC_Google_Calendar`** class must be configured to request **only minimal availability data** from the external calendar API (start and end timestamps for blocked time).
    *   The internal logic in the synchronization engine must eliminate extraction, parsing, and storage of detailed event metadata such as the **title, description, and location**.

2.  **Bypass Booking Creation Pipeline:**
    *   The synchronization engine must **remove the critical integration call** to the global function **`wpbc_booking_save()`**. This function is responsible for assembling the data and performing the final database insertion that currently creates the permanent local booking record.
    *   By bypassing `wpbc_booking_save()`, the system also avoids the inherent risks associated with obsolete date logic, such as that found in `core/lib/wpbc-booking-new.php`.

3.  **Dedicated Cache Storage:**
    *   Instead of creating a record in the main booking table (`{$wpdb->prefix}booking`), the system must perform a **direct database insertion** using the `$wpdb` object into a specialized, minimal data structure (e.g., a dedicated temporary cache table or a resource-specific meta field).
    *   This structure would only store the **Resource ID** and the **blocked date/time range**, effectively serving as a **temporary availability cache**.

4.  **Integration with Date Engine:**
    *   The plugin's core "dates engine" utility file, `core/wpbc-dates.php`, contains SQL functions like `wpbc__sql__get_booked_dates()` and `wpbc__sql__get_season_availability()`. These functions must be updated to query **both** the permanent local booking records **and** the new temporary synchronization cache when calculating front-end availability.

### B. Implementing Clear User Consent (Requirement 3.2)

To allow the administrator to decide whether detailed data is required (e.g., for internal scheduling notes), a clear consent mechanism must be integrated into the administrative UI.

1.  **Settings UI Integration:**
    *   A consent mechanism (e.g., a clearly labeled checkbox or toggle) must be registered within the plugin's custom **Settings API framework** on the Google Calendar settings page (`core/sync/wpbc-gcal.php`).
    *   If the administrator enables this setting, a mandatory confirmation must be recorded in the database, potentially using the filterable option wrappers (e.g., `update_bk_option`).

2.  **Conditional Data Flow:**
    *   The presence of consent would trigger the original, "Show details" flow (allowing retrieval and storage of metadata) and subsequently execute the permanent booking saving mechanism (`wpbc_booking_save()`).
    *   The absence of consent would enforce the "Busy only" flow defined in Section A, ensuring only minimal availability data is processed.

### C. Preventing Leakage to Third-Party APIs (Requirement 3.3)

This requirement is intrinsically linked to Requirement 3.1. By eliminating the storage of identifying metadata in the core database, the risk of external leakage is minimized.

1.  **Anonymizing Hook Data:**
    *   The plugin’s architecture relies on a **custom action and filter hook system** (`add_bk_action`, `apply_bk_filter`) and the Developer API (`core/wpbc-dev-api.php`) for external integration.
    *   The crucial action hook **`wpbc_track_new_booking`**, which is fired when a booking is saved, is the main point used by developers for integrating with CRMs or external services.
    *   If the "Busy only" mode is active, the imported data bypasses the full `wpbc_booking_save()` routine, meaning this hook is either not triggered, or it is triggered with a highly restricted payload that contains **no event metadata** (title, description, attendees), thereby ensuring that no identifying information can be captured or sent to external analytics or third-party APIs via these integration points.
  
2.  

---

#### 4. **User Consent & Transparency**

| Item | Description                                                                                          | How to Check                                               |
| ---- | ---------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| 4.1  | During calendar connection, users see a **disclosure or consent prompt** (explaining what’s synced). | Inspect UI/UX or connection modal.                         |
| 4.2  | The plugin’s **Privacy Policy** mentions how personal calendar data is processed.                    | Review linked Privacy Policy or admin settings.            |
| 4.3  | Users can **disconnect** the calendar and revoke permissions easily.                                 | Test disconnect flow and check whether tokens are deleted. |


Based on the comprehensive architectural review of the synchronization features (Google Calendar and ICS) and the plugin's settings systems, the requirements for **User Consent & Transparency** are generally **not implemented** in the manner described.

The current architecture is optimized for importing and permanently storing external event details, which conflicts with requirements for data minimization and explicit consent.

Here is a detailed analysis of each item:

### 4.1 Disclosure or Consent Prompt

| Status | Justification |
| :--- | :--- |
| **Not Implemented** | The source documents describe the plugin's custom settings framework (`WPBC_Settings_API`) and the UI helper functions (`wpbc_gcal_settings_content_field_*()`) that render the Google Calendar synchronization configuration page. This configuration allows the administrator to input the Google API Key and set import parameters. However, the sources provide **no evidence** of a specific disclosure notice, banner, or consent prompt that informs the administrator that event details (such as title and description) will be permanently stored locally when connecting the calendar. |

### 4.2 Privacy Policy Mentions

| Status | Justification |
| :--- | :--- |
| **Not Implemented (Not Referenced in Sources)** | The sources focus on the technical architecture, settings, and feature implementation of the plugin. They detail the administrative UI construction, the system for persistent administrative notices (`WPBC_Notices` class), and data persistence. However, the sources **do not contain any reference** to a plugin-specific Privacy Policy document, text, or administrative setting that addresses how the personal calendar data (titles, descriptions, locations) imported from the external calendar feed is processed, stored, or managed. |

### 4.3 Users can disconnect the calendar and revoke permissions easily.

| Status | Justification |
| :--- | :--- |
| **Partially Implemented (Configuration Revocation is Clean, Data Revocation is Not)** | The synchronization architecture is designed with clear lifecycle cleanup: |
| | **Configuration Revocation:** The controller file (`wpbc-gcal.php`) defines the function `wpbc_sync_gcal_deactivate()`. This function executes upon plugin deactivation and specifically calls `delete_bk_option()` to **remove all Google Calendar configuration settings** (such as the API key and field mappings) from the database. This ensures a clean revocation of connection permissions. |
| | **Data Persistence Issue:** However, the source material explicitly notes that every imported Google event is saved as a **new local booking**, which includes permanent storage of event details (title, description, location). Disconnecting the calendar only stops future imports; it does **not** automatically delete the records already stored in the database. Thus, while the connection is revoked, the personal data synced during previous imports persists. |

***

## Scoring (1 – 10 Scale)

Based on the evidence, the transparency and consent requirements score low due to the lack of dedicated UI/documentation and the permanent storage mechanism employed by the current synchronization feature.

| Item | Status Rating (1-10) | Justification |
| :--- | :--- | :--- |
| **4.1** Disclosure or consent prompt | **1 / 10** | No prompt or disclosure mechanism is mentioned in the analysis of the admin UI or settings framework. |
| **4.2** Privacy Policy mentions data processing | **1 / 10** | The source documents contain no reference to a Privacy Policy or any dedicated administrative disclosure regarding the processing of imported calendar data. |
| **4.3** Users can disconnect and revoke permissions easily | **4 / 10** | Configuration settings (like the API key) are cleanly deleted upon deactivation. However, the architecture permanently saves identifying event details as local bookings, meaning revocation is **incomplete** regarding the removal of stored personal data. |



This implementation requires introducing explicit consent controls in the administrative interface and fundamentally altering the data persistence architecture to enable the easy and complete removal of imported event data upon calendar disconnection.

This overview focuses on implementing the required controls and the necessary changes to the data flow, leveraging the plugin's custom architectural patterns.

### 1. Configuration and Transparency Layer (4.1 & 4.2)

The administrative settings page for Google Calendar synchronization (`core/sync/wpbc-gcal.php`) must be modified to incorporate the necessary disclosure and consent prompts, utilizing the plugin's custom UI and Settings API structure.

| Requirement | Implementation Step | Architectural Component |
| :--- | :--- | :--- |
| **4.1 Disclosure/Consent Prompt** | **Implement a Mandatory Checkbox:** A prominent checkbox and corresponding warning text must be rendered using the existing UI helper functions (`wpbc_gcal_settings_content_field_*()`). This prompt must clearly state that connecting the calendar allows the plugin to read and store external event details (Title, Description, Location) unless "Busy Only" mode is active. | `core/sync/wpbc-gcal.php`, `WPBC_Settings_API` framework. |
| **4.1 Consent Gate:** | The system must prevent the administrator from saving the synchronization settings (API Key, Calendar ID) unless this consent checkbox is explicitly checked. This uses the validation flow managed by the Settings API framework. | `WPBC_Settings_API_General`, `validate_post()` methods. |
| **4.2 Privacy Policy** | **Documentation Requirement:** The implementation would include a development note ensuring the external-facing documentation references the data processing practices, including the storage of event metadata. | *External Documentation/Legal (Not in code sources).* |

### 2. Data Persistence Modification (Prerequisite for Revocation)

The core architectural requirement for easy revocation is that event data should **not** be stored as permanent, standard bookings (which currently occurs via `wpbc_booking_save()`). This must be addressed by shifting to a minimal data storage model.

1.  **Conditional Booking Bypass:** The `WPBC_Google_Calendar` class would check if the administrator has configured the import for detailed storage. If not, the `createNewBookingsFromEvents()` method would bypass the call to `wpbc_booking_save()`.
2.  **Availability Cache Storage:** Instead of creating a booking record, the system would insert only the resource ID and the blocked date/time range into a lightweight, resource-specific data structure (a dedicated custom table or user meta) that acts as a temporary cache. This prevents the event **title, description, and location** from being stored permanently.
3.  **Cron Dependency:** The custom pseudo-cron system (`WPBC_Cron`) would need to be scheduled frequently to refresh this temporary availability cache, as the data would expire or be overwritten by the next import.

### 3. Disconnect and Data Revocation (4.3)

Implementing easy disconnection requires not only deleting the configuration but also completely removing all imported data associated with that specific calendar feed.

| Requirement | Implementation Step | Architectural Component |
| :--- | :--- | :--- |
| **4.3 Disconnect UI** | **Add Revocation Button:** The settings page (`core/sync/wpbc-gcal.php`) needs a clear "Disconnect and Delete All Imported Data" button. | UI helper functions, `admin-bs-ui.php` for button styling. |
| **4.3 Revocation Action Hook:** | The button click would initiate an AJAX request to `admin-ajax.php`. The request is routed by `core/lib/wpbc-ajax.php` and would need a new action hook, registered via the `wpbc_ajax_action_list` filter. | `core/lib/wpbc-ajax.php`, `wpbc_ajax_action_list` filter. |
| **4.3 Data Deletion Logic** | The corresponding AJAX handler function must execute database commands (using the global `$wpdb` object): 1. **Delete Configuration:** Remove the API Key and Calendar ID using `delete_bk_option()`. 2. **Delete Records:** Execute a prepared SQL query to search the local booking table (`{$wpdb->prefix}booking`) for and permanently delete all booking records that contain the `sync_gid` (Google Event ID) associated with the disconnected calendar feed. | `$wpdb` object, `wpbc_api_booking_delete()` (or similar helper function). |



---

#### 5. **Compliance & API References**

| Item | Description                                                                                                                                  | How to Check                             |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| 5.1  | Google Calendar integration follows [Google OAuth verification guidelines](https://developers.google.com/workspace/marketplace/verify).      | Review Google Cloud project status.      |
| 5.2  | Outlook integration follows [Microsoft Graph privacy and permissions policy](https://learn.microsoft.com/en-us/graph/permissions-reference). | Review Azure App registration.           |
| 5.3  | The plugin provides instructions for **revoking data access** via Google/Microsoft dashboards.                                               | Check documentation or support articles. |

Based on the detailed architectural review of the synchronization features found in the sources, the implementation status of the **Compliance & API References** items is as follows:

### 5.1 Google OAuth Verification Guidelines

| Status | Justification Based on Source Material |
| :--- | :--- |
| **Inconclusive (Scope of Integration is Limited)** | The sources confirm the Google Calendar (GCal) feature uses an **API Key** (stored as `booking_gcal_api_key`) to communicate with the **Google Calendar API v3** via `wp_remote_get()`. The integration is a **one-way import**. Since the system relies on an API Key and interacts with what is specified as a **public Google Calendar**, the sources suggest a technical implementation designed for server-side read access rather than a full user-authenticated OAuth 2.0 flow. However, the architectural review **does not contain any information** regarding the plugin's submission status, scope definition, or compliance verification by Google Workspace or Google Cloud for OAuth verification guidelines. |

### 5.2 Outlook Integration Policy

| Status | Justification Based on Source Material |
| :--- | :--- |
| **Not Implemented (Outlook Integration Not Documented)** | The synchronization features documented in the sources are strictly limited to **Google Calendar** (API-based sync) and **ICS Feeds** (file-based sync for URLs like Airbnb or VRBO). There is **no mention** in the analyzed files of any integration, API calls, or configuration related to Outlook, Microsoft, Azure, or Microsoft Graph. |

### 5.3 Instructions for Revoking Data Access

| Status | Justification Based on Source Material |
| :--- | :--- |
| **Not Implemented (Internal Cleanup, External Instructions Missing)** | The synchronization controller (`core/sync/wpbc-gcal.php`) includes robust functions for **lifecycle management**. The function `wpbc_sync_gcal_deactivate()` is defined to execute upon plugin deactivation, and its role is to perform cleanup by calling `delete_bk_option()` to **remove all Google Calendar configuration settings** (including the API key) from the database. However, the architectural documents **do not provide any evidence** of administrative settings, links, or documentation text that instructs the administrator on how to manually revoke permissions or API Key access *externally* via their respective Google or Microsoft dashboards. |



Based on the detailed architectural review of the synchronization features available in the source documents, the **Compliance & API References** options are rated as follows:

| Item | Status Rating (1-10) | Justification Based on Source Material |
| :--- | :--- | :--- |
| **5.1** Google Calendar integration follows Google OAuth verification guidelines. | **3 / 10** | **Inconclusive/Undocumented Compliance.** The existence of the Google Calendar synchronization feature is confirmed, and it uses a **Google API Key** (`booking_gcal_api_key`) to communicate with the **Google Calendar API v3** via `wp_remote_get()` for a **one-way import** from a public calendar. However, the sources **do not contain any information** verifying that the plugin's API usage or configuration adheres to or has gone through the formal Google OAuth verification guidelines or processes [5.1]. The score reflects the technical existence of the feature without confirmation of the external compliance status. |
| **5.2** Outlook integration follows Microsoft Graph privacy and permissions policy. | **1 / 10** | **Not Implemented.** The documentation focuses exclusively on Google Calendar API synchronization and ICS feed synchronization (e.g., Airbnb, VRBO). There is **no mention** or evidence of any integration with Outlook, Microsoft Graph, or Azure [5.2]. |
| **5.3** The plugin provides instructions for **revoking data access** via Google/Microsoft dashboards. | **3 / 10** | **Incomplete Implementation.** The plugin implements robust **internal cleanup** logic: the function `wpbc_sync_gcal_deactivate()` executes upon deactivation, calling `delete_bk_option()` to remove all Google Calendar configuration settings (including the API Key) from the database. However, the sources **do not indicate** that the plugin provides any administrative documentation, links, or instructions to guide the administrator on how to manually revoke access or delete the project *externally* via their Google dashboard [5.3]. |


The implementation of the **Compliance & API References** requirements necessitates significant updates, particularly a shift from the current Google API Key model to a full OAuth implementation, and the creation of an entirely new synchronization feature for Outlook/Microsoft.

This high-level overview leverages the plugin's existing object-oriented architecture, custom settings framework, and data management utilities.

### 1. Refactoring Google Calendar for OAuth Compliance (Requirement 5.1)

The current GCal synchronization relies on an **API Key** for communication with the **Google Calendar API v3**. To meet modern **Google OAuth verification guidelines** [5.1], the system must transition to a user-authenticated flow.

*   **Configuration Modification (Controller):** The UI helper functions (`wpbc_gcal_settings_content_field_*()`) in the controller file (`core/sync/wpbc-gcal.php`) would be refactored to collect OAuth **Client ID** and **Client Secret** instead of the static API Key.
*   **Engine Update (WPBC\_Google\_Calendar):** The **`WPBC_Google_Calendar`** class would implement the necessary logic for the 3-legged OAuth flow. This involves directing the admin to the Google authorization URL, receiving the authorization code, and exchanging it for a persistent **Refresh Token** and a short-lived **Access Token**.
*   **Token Management:** The Refresh Token, essential for automated imports, would be stored securely in the database using the internal option wrappers (`update_bk_option`, provided by `core/wpbc-core.php`).
*   **Minimal Scoping:** Crucially, the OAuth request must specify the **minimal scope** necessary (e.g., read-only access to busy/free information) to reduce the risk surface and facilitate compliance verification. The engine would continue to use `wp_remote_get()` for communication with the API.

### 2. Implementing Outlook/Microsoft Integration (Requirement 5.2)

Since **Outlook integration is not currently documented** in the source material [5.2], this requires creating a new synchronization mechanism, built using the same Controller/API separation pattern as the existing GCal feature. This new system must adhere to **Microsoft Graph privacy and permissions policy** [5.2].

*   **New Architecture:** Define a new controller file (e.g., `core/sync/wpbc-ms-graph.php`) and a corresponding engine class (e.g., `WPBC_Microsoft_Calendar`).
*   **Authentication Flow:** The new engine would implement the OAuth flow required to authenticate with the **Microsoft Graph API**.
*   **Minimal Permissions:** The implementation must request **minimal permissions** from Microsoft Graph, limiting access to calendar data needed for availability checking (read-only busy/free data).
*   **Synchronization Logic:** The new engine class would handle Microsoft-specific data parsing, date transformation, deduplication (by saving a unique Microsoft Event ID to the booking record), and final database integration by calling the core **`wpbc_booking_save()`** function.
*   **Automated Sync:** The controller would hook the import job into the custom **`WPBC_Cron`** pseudo-cron system using `WPBC()->cron->update()`.

### 3. Revocation Instructions and Transparency (Requirement 5.3)

To ensure users can easily **disconnect the calendar and revoke permissions** [5.3], the administration UI and supporting documentation must be updated.

*   **Revocation Instructions in UI:**
    *   The administrative settings pages (for both Google Calendar and the new Microsoft integration) would be updated to include a clear, prominent section providing **step-by-step instructions** on how to log into the respective **Google Cloud Project** or **Azure App Registration** to manually revoke the application's API access [5.3].
    *   This content would be rendered using existing UI helper functions that utilize procedural helper functions (like those found in `core/any/admin-bs-ui.php`).
*   **Lifecycle Cleanup:** The existing `wpbc_sync_gcal_deactivate()` function already handles internal database cleanup of configuration settings (`delete_bk_option`). This process would be mirrored for the new Microsoft implementation.
*   **Developer API Documentation:** The `core/wpbc-dev-api.php` file, which serves as the **official abstraction layer** and documents plugin hooks, would be updated to clearly document the OAuth token formats and the database location of the tokens (stored as plugin options via `core/wpbc-core.php` wrappers) to aid external developers who may need to build custom revocation tools.
*   **Proactive Warnings:** The **`WPBC_Notices`** class could be utilized to display persistent, dismissible admin warnings guiding administrators who are still using the deprecated API Key method to switch to the compliant OAuth flow.

*   
---

Would you like me to turn this into a **checklist table (CSV format)** so you can directly tick off what’s implemented in the plugin?
