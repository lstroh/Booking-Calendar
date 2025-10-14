Excellent — this is a **core GDPR requirement** and one that’s often overlooked in WordPress-based systems handling client and booking data.

Let’s break down:

> **GDPR & UK Act → Data Rights → Right to export or delete booking, loyalty, referral, and messaging history**
> Type: *Must-Have* | Effort: *12h* | Priority: *High*
> Dependencies: *Custom rewards engine* or *third-party loyalty API*

Below is a **complete verification checklist** so you can test whether the plugin supports full compliance for user data access, export, and deletion.

---

## ⚖️ GDPR & UK Act — Data Rights (Export & Deletion)

**Goal:** Allow users to **access, export, and delete** their personal data (bookings, loyalty/rewards, referrals, and messages) in a transparent, secure, and auditable way.

---

### **1. User Data Identification**

| #   | Check Item               | Description                                                                                                   | Implemented? |
| --- | ------------------------ | ------------------------------------------------------------------------------------------------------------- | ------------ |
| 1.1 | Unified data mapping     | Plugin maintains a clear mapping of all personal data types (bookings, messages, photos, rewards, referrals). | ☐            |
| 1.2 | Linked to user account   | Each record is linked to a user ID or identifiable email, enabling lookup for export/deletion.                | ☐            |
| 1.3 | Anonymous guest handling | For guest bookings, system still allows lookup via email or booking reference.                                | ☐            |

Based on the detailed architectural analysis provided in the sources, here is a breakdown of whether the User Data Identification items are implemented or explicitly supported by the documented components and data structures of the plugin:

| # | Check Item | Supported by Sources? | Analysis of Implementation |
| :--- | :--- | :--- | :--- |
| **1.1** | **Unified data mapping** | **No** | The sources detail the existence of core data (bookings, dates), form data (stored as a complex, custom-serialized string in `booking_options`), and email content. However, there is **no mention** of a "unified data mapping" system that clearly catalogs all personal data types (such as bookings, messages, photos, rewards, or referrals) [143–234]. The plugin does map form fields (e.g., `[text your-name]`) to iCalendar event properties for synchronization. |
| **1.2** | **Linked to user account** | **Partially** | Every booking has a unique `booking_id`. The logic for **deletion** (`wpbc_ajax_DELETE_APPROVE()`) and **export** (via .ics feed) exists. Submitted bookings capture the visitor's email address via the form data to enable notifications and replies. The plugin uses the WordPress User API for auto-filling form fields for logged-in users. However, the sources **do not explicitly confirm** that all booking records are systematically linked to a WordPress **User ID** (as opposed to just storing the data submitted in the form) specifically "for export/deletion lookup" in a data privacy context [143–234]. |
| **1.3** | **Anonymous guest handling** | **Yes** | Anonymous or guest bookings are identifiable and retrievable via unique references and captured form data: |
| | | | 1. **Booking Reference/ID:** The Developer API includes `wpbc_api_get_booking_by_id()`, allowing programmatic retrieval of a booking using its unique identifier. The Timeline feature also supports filtering via a `booking_hash` parameter for a Customer-Specific View. |
| | | | 2. **Identifiable Email:** For guests, their email address is captured and stored as part of the **serialized form data** and is used to send confirmation emails and set the Reply-To header for the administrator. The system also uses external identifiers like the Google Event ID (`sync_gid`) to prevent duplicates during import. |

### Further Context on Data Retrieval and Storage

The architecture of the plugin impacts how data lookup is performed:

*   **Data Storage:** Custom booking data (form fields, which include guest name and email) are stored as a **single serialized array** in the `booking_options` column of the custom `{$wpdb->prefix}booking` table, rather than in separate database columns or using the standard WordPress metadata API. While this is a custom method for storing per-booking meta options, it means retrieving specific user data fields (like an email) often requires **unserializing** the `booking_options` column after querying the main booking table.
*   **Data Retrieval:** The data retrieval engine (`core/admin/wpbc-sql.php`) constructs complex, dynamic SQL queries and processes raw results to parse the serialized form data into a structured array for display in the admin UI.
*   **Deletion:** Booking deletion logic is handled dynamically via the AJAX controller (`core/lib/wpbc-ajax.php`), which includes functions for permanent deletion (`wpbc_ajax_DELETE_APPROVE()`).


Based on the detailed architectural information available in the sources, here is the assessment of the implementation status of each User Data Identification option on a scale of 1 to 10 (where 1 is not implemented and 10 is fully implemented):

| # | Check Item | Score (1-10) | Implementation Analysis Based on Sources |
| :--- | :--- | :--- | :--- |
| **1.1** | **Unified data mapping** | **1/10** | This feature is not implemented. The sources **do not mention** a unified data mapping or cataloging system for all personal data types (photos, rewards, referrals) [143–234]. Custom data is stored as a **single serialized array** within the `booking_options` column of the custom database table, which is an inefficient and non-queryable method that breaks database normalization. |
| **1.2** | **Linked to user account** | **6/10** | This feature is **partially implemented**. While every booking record has a unique `booking_id`, and functionality for deletion (`wpbc_ajax_DELETE_APPROVE()`) and programmatic retrieval (`wpbc_api_get_booking_by_id()`) exists, the sources **do not confirm** that every booking record is systematically linked to a WordPress **User ID** (UID) via a dedicated, queryable database column. Instead, the data relied upon for lookup (like the visitor's email) is captured and stored within the complex, custom-serialized form data. |
| **1.3** | **Anonymous guest handling** | **9/10** | This feature is highly implemented. Anonymous or guest bookings are fully identifiable and retrievable via: 1) The unique **Booking Reference/ID** (which enables programmatic retrieval via `wpbc_api_get_booking_by_id()` and UI filtering via a `booking_hash` for the Timeline Customer-Specific View), and 2) The **identifiable email address** and name stored in the serialized form data. The system is capable of performing critical actions like deletion using these identifiers via the AJAX controller (`core/lib/wpbc-ajax.php`). |



The implementation of a robust **User Data Identification** system would require leveraging the plugin's foundational architectural components, particularly the custom hook system, the data abstraction layer, and the reliance on direct database interaction for core business logic.

The high-level implementation overview would focus on two major architectural changes: **Database Denormalization for PII** and the creation of a **Dedicated Data Management Controller**.

---

### Phase 1: Database and Data Abstraction Layer Changes

The current architecture stores custom form data, including user name and email, as a single **serialized array** in the `booking_options` column of the custom `{$wpdb->prefix}booking` table. This is **not queryable** and breaks database normalization.

To support efficient lookup for export and deletion (Features 1.1 and 1.2), the system must treat core Personal Identifiable Information (PII) as primary data fields:

#### 1. Introduce Dedicated PII Columns (Addressing 1.1 & 1.2)

1.  **Schema Modification:** Update the `{$wpdb->prefix}booking` database table during plugin activation (`wpbc_activation` hook from the `WPBC_Install` class) to include new, dedicated, non-serialized columns for critical lookup identifiers:
    *   `visitor_email` (Extracted directly from the submitted form data).
    *   `wordpress_user_id` (To explicitly link the booking to a registered user ID).
2.  **Data Persistence Update:** Modify the core function responsible for saving new bookings (likely located in newer files like `includes/page-bookings/bookings__actions.php` or through the `wpbc_booking_save()` global function) to perform the following steps immediately after processing the booking:
    *   **Extract Data:** Use the Form Parser (`core/form_parser.php`) and related logic to reliably extract the visitor's email address and name from the submitted form array.
    *   **User Check:** Check if the user is logged in using the WordPress User API (a pattern already used for auto-filling form fields). If `is_user_logged_in()` is true, retrieve the `current_user_id()`.
    *   **Direct Saving:** Use a **direct prepared `$wpdb` query** (a pattern utilized by critical AJAX handlers and the GCal sync engine) to securely save the extracted `visitor_email` and, if available, the `wordpress_user_id` into the new columns in the `{$wpdb->prefix}booking` table.

### Phase 2: Building the User Data Management Controller

A new administrative component would be needed to expose the search and action functionality required for user data identification, relying on the plugin's existing modular design patterns:

#### 2. Create a Data Management UI (Addressing 1.2 & 1.3)

1.  **Menu Registration:** Use the custom menu system handled by `core/wpbc.php` to register a new, dedicated admin page (e.g., "Data Tools") under the main "Bookings" menu entry. The core structure would be managed by the **WPBC\_Admin\_Menus** class.
2.  **UI Scaffolding:** On this new admin page, utilize the procedural helper functions from `core/any/admin-bs-ui.php` to generate standardized, Bootstrap-style search inputs for filtering by **Booking ID**, **Email Address**, or **WordPress User ID**.
3.  **Data Engine Integration:**
    *   Implement a new function in the data engine (`core/admin/wpbc-sql.php` pattern) that constructs a search query based on the admin's input.
    *   The function would use **sanitized $_REQUEST parameters** (via `wpbc_check_request_paramters()`) to build a WHERE clause that searches the new `visitor_email` and `wordpress_user_id` columns efficiently using **prepared SQL queries**.
4.  **Action Handlers:** The UI would include buttons (Export/Delete) that trigger dynamic, authenticated requests:
    *   **AJAX Interface:** All sensitive actions must be routed through the AJAX controller (`core/lib/wpbc-ajax.php`). The request payload must include a security **nonce** that is verified using `wpbc_check_nonce_in_admin_panel()` to prevent CSRF attacks.
    *   **Deletion:** The AJAX handler would execute the permanent deletion logic by calling the existing workflow function, such as `wpbc_ajax_DELETE_APPROVE()`, which triggers the `wpbc_booking_delete` lifecycle action.
    *   **Export:** The export action would retrieve the full booking data array using the Developer API function, **`wpbc_api_get_booking_by_id()`**, which correctly *unserializes* the form data into a readable array for export processing.

By implementing these two phases, the plugin utilizes its existing architecture (custom SQL, AJAX security, and API abstraction) to efficiently support PII management and address the previous implementation gap regarding queryable data storage.



---

### **2. Data Export Functionality**

| #   | Check Item                 | Description                                                                                                 | Implemented? |
| --- | -------------------------- | ----------------------------------------------------------------------------------------------------------- | ------------ |
| 2.1 | Export request trigger     | Users (or admin on behalf of user) can initiate a **data export request** via dashboard or settings.        | ☐            |
| 2.2 | Supported formats          | Export is generated in a **structured, machine-readable format** (e.g., JSON or CSV).                       | ☐            |
| 2.3 | Scope of export            | Includes all relevant personal data: bookings, job photos, loyalty points, referrals, and messaging logs.   | ☐            |
| 2.4 | Third-party data inclusion | Includes data from integrations (e.g., Twilio messages, loyalty API logs) if stored locally.                | ☐            |
| 2.5 | Email delivery             | Export file is securely delivered (e.g., temporary download link, password-protected zip, or emailed link). | ☐            |
| 2.6 | Audit trail                | System logs who initiated the export and when (for compliance proof).                                       | ☐            |

The architectural analysis and accompanying file reviews reveal that the Data Export functionality is partially implemented, but primarily focused on synchronization feeds and often requires a companion plugin for advanced features.

Here is the breakdown of the implementation status for each check item:

| # | Check Item | Implemented? | Analysis of Implementation |
| :--- | :--- | :--- | :--- |
| **2.1** | **Export request trigger** | $\checkmark$ | **Implemented** via public .ics feeds. The UI for generating a **public .ics feed URL** is provided on the `core/admin/page-ics-export.php` settings page. This feed allows bookings to be exported to external calendars. However, the sources note that **Advanced export features are delegated to the "Booking Manager" companion plugin**. |
| **2.2** | **Supported formats** | $\checkmark$ | **Partially supported.** The plugin explicitly supports the **iCalendar (.ics)** format for synchronization export, which is a structured, machine-readable format. The Developer API function `wpbc_api_get_booking_by_id()` can retrieve booking data as a rich, unserialized PHP array, which is the raw data needed for custom JSON or CSV generation, but the sources **do not mention** built-in administrative tools to export data in **JSON or CSV** formats directly. |
| **2.3** | **Scope of export** | $\boxtimes$ | **Not fully implemented.** The export functionality primarily targets core **booking data** (dates and form fields). However, there is **no evidence** in the sources that the export scope includes other PII categories like **job photos, loyalty points, referrals, or messaging logs** [189–234]. |
| **2.4** | **Third-party data inclusion** | $\boxtimes$ | **Not explicitly supported.** The plugin saves Google Calendar event data locally as new bookings, so this data is implicitly included in the core booking export. However, the sources provide **no explicit mechanism** to query and include data from non-booking integrations (e.g., Twilio messages or loyalty API logs) in the export, especially since custom booking metadata is stored as an inefficient, non-queryable serialized array (`booking_options`). |
| **2.5** | **Email delivery** | $\boxtimes$ | **Not supported by core export.** The primary documented export method is generating a **public .ics feed URL**, which is generally accessible and not considered a "securely delivered" file (e.g., password-protected zip, temporary download link, or emailed link). The email utilities focus on transactional emails (confirmation, approval, moderation links). |
| **2.6** | **Audit trail** | $\boxtimes$ | **Not supported.** While the plugin manages its workflow via custom actions (e.g., `wpbc_booking_approved`, `wpbc_booking_delete`) and includes functions for internal logging, there is **no mention** of a feature that specifically logs or creates an **audit trail** detailing who initiated a data export request and when, for the purpose of compliance proof [189–234]. |



Based on the comprehensive architectural analysis of the plugin's components, particularly the dedicated synchronization and core functionality files, here is the implementation assessment for the **Data Export Functionality** on a scale of 1 to 10:

| # | Check Item | Score (1-10) | Implementation Analysis Based on Sources |
| :--- | :--- | :--- | :--- |
| **2.1** | **Export request trigger** | **7/10** | **Partially implemented.** The plugin provides a UI for generating a **public .ics feed URL** (`core/admin/page-ics-export.php`), which serves as an export trigger for external calendar applications. However, **advanced export features** are explicitly delegated to a required companion plugin, the "Booking Manager". This reliance on an external dependency prevents a full score, but the core mechanism for triggering external data feeds exists. |
| **2.2** | **Supported formats** | **4/10** | **Partially supported.** The plugin officially supports **iCalendar (.ics)** format for synchronization. While this is a structured, machine-readable format, the sources **do not mention** built-in functionality to export data in other common formats like **JSON or CSV**. The Developer API (`core/wpbc-dev-api.php`) does offer `wpbc_api_get_booking_by_id()` which retrieves data as a structured PHP array, enabling custom implementation of other formats, but this is not a built-in admin feature. |
| **2.3** | **Scope of export** | **2/10** | **Minimally supported.** The documented export focuses heavily on **core booking data** (dates and form fields) as needed for the .ics standard. The sources provide **no evidence** that the export mechanism includes or can easily retrieve **job photos, loyalty points, referrals, or dedicated messaging logs** [189–234]. Retrieving all related PII is inherently difficult because custom booking form data is stored inefficiently as a single **serialized array** in the `booking_options` column. |
| **2.4** | **Third-party data inclusion** | **2/10** | **Minimally supported.** Data from Google Calendar imports is saved locally as new booking records and would thus be included in a standard booking export. However, the sources **do not document** a generalized feature or architecture for systematically including other arbitrary third-party integration data (e.g., dedicated Twilio logs or Loyalty API data) in a unified export file [189–234]. |
| **2.5** | **Email delivery** | **1/10** | **Not supported.** The primary documented export method is generating a **public, perpetually accessible feed URL** (the .ics feed). There is no mention in the sources of providing secure file delivery methods such as temporary download links, password-protected ZIP archives, or emailed encrypted links, which are typical requirements for secure data exports. |
| **2.6** | **Audit trail** | **1/10** | **Not supported.** The plugin has robust systems for logging administrative actions related to status changes (e.g., `wpbc_db__add_log_info` in `core/wpbc_functions.php`) and uses custom hooks for triggering post-event actions (like `wpbc_track_new_booking`). However, the sources **do not mention** any function or setting dedicated to logging or creating an **audit trail** specifically for data export requests for compliance proof [189–234]. |


The implementation of a comprehensive Data Export feature requires overcoming the current architectural limitations, specifically the reliance on companion plugins for advanced features and the core flaw of storing PII in non-queryable serialized data fields.

Drawing on the plugin’s robust framework of custom APIs, AJAX security, and internal hooks, the implementation would proceed in four high-level phases: **Data Structuring**, **Admin Interface**, **Export Processing**, and **Delivery & Audit**.

---

### Phase 1: Data Structuring and PII Normalization

The goal of this phase is to ensure that personal data necessary for lookup (like email and name) is immediately queryable, moving away from the inefficient reliance on the serialized `booking_options` column.

1.  **Database Schema Update:** Modify the custom `{$wpdb->prefix}booking` database table to include new, dedicated, indexed columns for core PII, such as `visitor_email` and `wordpress_user_id` (if applicable). This update should be triggered during the plugin's activation lifecycle via the `wpbc_activation` hook.
2.  **PII Extraction on Save:** Modify the core booking insertion logic (functions associated with `wpbc_booking_save()`) to immediately extract the visitor's email address, name, and logged-in WordPress ID from the form submission array and store them directly in the new database columns.
3.  **Indexed Data Access:** Update the data engine (`core/admin/wpbc-sql.php`) to prioritize queries based on the new indexed columns when searching for a user's data, ensuring efficient lookup for export/deletion.

### Phase 2: Administrative Interface and Request Trigger (2.1)

This phase establishes the user interface and the secure trigger for initiating the export.

1.  **New Admin UI:** Create a dedicated "Data Tools" sub-tab or admin page using the custom menu framework managed by `WPBC_Admin_Menus`.
2.  **Search Input & Validation:** Use the procedural helper functions defined in `core/any/admin-bs-ui.php` to render a search input allowing the administrator to look up bookings by email, WordPress User ID, or Booking ID. Input parameters must be sanitized using `wpbc_check_request_paramters()`.
3.  **AJAX Endpoint:** Implement a new, dedicated AJAX endpoint (`wpbc_ajax_EXPORT_USER_DATA`) within the core AJAX controller (`core/lib/wpbc-ajax.php`). This endpoint handles the export request initiation.
4.  **Security:** Crucially, the AJAX endpoint must enforce security immediately using `wpbc_check_nonce_in_admin_panel()` to prevent Cross-Site Request Forgery (CSRF).

### Phase 3: Export Processing Engine (2.2, 2.3, 2.4)

A new, centralized engine is needed to gather all relevant data and convert it into the requested machine-readable format.

1.  **Data Retrieval and Unserialization:** The engine first uses the PII lookup fields to retrieve all relevant booking records. For each record, it calls the Developer API function `wpbc_api_get_booking_by_id()`, which conveniently retrieves the full booking data and reliably **unserializes** the `booking_options` column, making all custom form data (2.3) readable.
2.  **Third-Party Data Hooks (Scope Extensibility):** Implement a filter (e.g., `wpbc_data_export_add_fields`) that fires before final compilation (2.4). This filter allows companion plugins (e.g., for loyalty points or external messaging APIs) to securely attach their locally stored data to the main export array, ensuring a comprehensive data scope.
3.  **Format Generation:** Implement dedicated methods within the processing engine to convert the final, compiled PHP data array into **structured, machine-readable formats** such as JSON and CSV (2.2).

### Phase 4: Secure Delivery and Audit Trail (2.5, 2.6)

This final phase addresses security and compliance requirements by ensuring the file is delivered securely and the transaction is logged.

1.  **Secure File Storage:** The generated export file should be stored temporarily in a non-public, randomized directory within the uploads folder.
2.  **Secure Email Delivery:** Utilize the email functionality centralized in `core/wpbc-emails.php` to send a transactional email to the requesting administrator or user. The email content will contain a **time-limited, nonce-protected download link** to the stored file (2.5). The email should use the functions wrapped by `wpbc_wp_mail()` to ensure good deliverability via proper header setting.
3.  **Audit Logging:** Immediately upon completion of the export and dispatch of the delivery email, trigger a custom internal action hook (e.g., `make_bk_action('wpbc_data_export_logged')`). The function hooked to this action would record the **timestamp**, the **User ID** (or identifying email), and the **scope** of the export into the plugin's logging system, creating a non-repudiable audit trail for compliance (2.6).


---

### **3. Data Deletion Functionality**

| #   | Check Item             | Description                                                                                              | Implemented? |
| --- | ---------------------- | -------------------------------------------------------------------------------------------------------- | ------------ |
| 3.1 | Delete request trigger | Users can request deletion via account dashboard or data request form.                                   | ☐            |
| 3.2 | Selective deletion     | Admin can delete all data for a user OR specific categories (e.g., only messages, only referrals).       | ☐            |
| 3.3 | Safe anonymisation     | Instead of hard delete, data can be anonymised (keep booking stats, remove personal identifiers).        | ☐            |
| 3.4 | Dependent records      | Deletion cascades properly across dependent tables (e.g., deleting booking removes photos and messages). | ☐            |
| 3.5 | External API deletion  | If using loyalty or CRM APIs, deletion request triggers data deletion at those third-party systems too.  | ☐            |
| 3.6 | Retention compliance   | System enforces exceptions (e.g., accounting data may be kept for legal retention periods).              | ☐            |
| 3.7 | Confirmation message   | User receives confirmation when deletion is completed.                                                   | ☐            |


Based on the analysis of the plugin's architecture, particularly the AJAX handler, email system, and data management, here is a breakdown of whether the Data Deletion Functionality check items are implemented:

| # | Check Item | Implemented? | Analysis of Implementation |
| :--- | :--- | :--- | :--- |
| **3.1** | **Delete request trigger** | $\boxtimes$ | **Not supported for end-users.** The sources describe administrative controls for deletion (via the AJAX controller functions like `wpbc_ajax_DELETE_APPROVE()` and `wpbc_ajax_TRASH_RESTORE()`), and administrative emails for deletion/cancellation. However, there is **no mention** of a user-facing account dashboard or public data request form that an end-user could use to trigger deletion of their data. |
| **3.2** | **Selective deletion** | $\boxtimes$ | **Not supported.** Deletion actions are described for entire booking records via the AJAX controller (e.g., permanent deletion via `wpbc_ajax_DELETE_APPROVE()`). The architectural analysis **does not mention** the capability to delete specific categories of data (e.g., only messages, photos, or referrals) while keeping the main booking record intact [189–234]. |
| **3.3** | **Safe anonymisation** | $\boxtimes$ | **Not supported.** The defined deletion methods are primarily permanent **hard deletions** (`wpbc_ajax_DELETE_APPROVE()`) or moving the booking to the trash (`wpbc_ajax_TRASH_RESTORE()`). There is **no evidence** of a dedicated function or setting that automatically strips personal identifiers (anonymization) while retaining booking statistics or date usage data [189–234]. |
| **3.4** | **Dependent records** | $\checkmark$ | **Partially supported for core data.** The deletion process is orchestrated through core actions (`wpbc_booking_delete`) triggered by the AJAX handler. While the source code analysis does not explicitly list *all* dependent tables and data that are cleaned up, the functions managing booking data likely remove entries from custom tables like `{$wpdb->prefix}booking` and related `{$wpdb->prefix}bookingdates`. However, the sources **do not confirm** proper cascading deletion for non-core data types like photos, loyalty points, or external message logs. |
| **3.5** | **External API deletion** | $\boxtimes$ | **Not supported explicitly.** While the plugin supports numerous extension points, particularly action hooks (`wpbc_booking_delete`) fired by the AJAX handler during the deletion lifecycle, there is **no explicit function or documentation** indicating that the core plugin or its analyzed components automatically trigger deletion requests at third-party APIs (like CRM or loyalty systems) [189–234]. A developer would need to manually hook into the `wpbc_booking_delete` action to implement this feature. |
| **3.6** | **Retention compliance** | $\boxtimes$ | **Not supported.** The sources **do not mention** any built-in logic, filters, or settings related to enforcing legal or regulatory data retention exceptions (e.g., automatically excluding accounting data from deletion for X years) [189–234]. Deactivation cleanup is based solely on a user-configurable option (`booking_is_delete_if_deactive`) for permanent deletion. |
| **3.7** | **Confirmation message** | $\checkmark$ | **Implemented for deletion/cancellation.** The plugin's **Email API** includes a dedicated email template handled by the `core/admin/page-email-deleted.php` file and managed by the `WPBC_Emails_API_Deleted` class. This email is used when a booking is canceled or declined by an administrator, and the corresponding sending function (`wpbc_send_email_deleted()`) populates shortcodes like `[denyreason]` before dispatching the cancellation notice to the visitor. |



Based on the detailed architectural analysis of the plugin's core files and administrative components, here is the implementation assessment for the **Data Deletion Functionality** on a scale of 1 to 10:

| # | Check Item | Score (1-10) | Implementation Analysis Based on Sources |
| :--- | :--- | :--- | :--- |
| **3.1** | **Delete request trigger** | **1/10** | This is **not implemented** for end-users. Deletion functionality is strictly administrative, relying on AJAX triggers (e.g., `wpbc_ajax_DELETE_APPROVE()`) from the backend. The sources do not document a user-facing account dashboard or public form that allows a visitor to initiate data deletion [189–234]. |
| **3.2** | **Selective deletion** | **1/10** | **Not implemented.** The administrative actions documented handle the permanent deletion of an **entire booking record**. There is no mention of functionality that allows the administrator to delete specific categories of data (e.g., just messages or referrals) while retaining the core booking history [189–234]. |
| **3.3** | **Safe anonymisation** | **1/10** | **Not implemented.** The plugin focuses on two types of deletion: moving to trash (`wpbc_ajax_TRASH_RESTORE()`) or permanent deletion (`wpbc_ajax_DELETE_APPROVE()`). There is **no defined function** or architectural support for stripping PII while retaining statistical booking data [189–234]. |
| **3.4** | **Dependent records** | **6/10** | **Partially supported.** Core deletion logic, handled by the AJAX controller, triggers internal action hooks (e.g., `wpbc_booking_delete`). This ensures that the primary booking record is deleted, and it is implied that related core data (such as dates in the `{$wpdb->prefix}bookingdates` table) would be removed. However, the analysis **does not confirm** cascading deletion across all possible dependent tables for external or optional data types (e.g., photos or loyalty logs) [189–234]. |
| **3.5** | **External API deletion** | **2/10** | **Minimally supported (via extension).** The core plugin **does not** automatically trigger deletion requests to external third-party systems [189–234]. However, the AJAX handler fires action hooks (`wpbc_booking_delete`) which provide the necessary **extension point** for a developer to manually implement this feature in a companion plugin. |
| **3.6** | **Retention compliance** | **1/10** | **Not implemented.** The plugin's lifecycle management only respects a general cleanup preference (`booking_is_delete_if_deactive`) upon plugin deactivation. There is **no logic or filtering mechanism** mentioned to enforce mandatory legal or accounting retention periods that would exempt certain data from deletion [189–234]. |
| **3.7** | **Confirmation message** | **9/10** | **Fully implemented.** The plugin uses a dedicated component within the Email API (`core/admin/page-email-deleted.php`) and the `WPBC_Emails_API_Deleted` class to send a deletion or cancellation notice to the visitor. The sending function (`wpbc_send_email_deleted()`) ensures this confirmation is delivered, optionally including a reason for the action via the `[denyreason]` shortcode. |

### Overall Implementation Score: **3/10**

The core functionality to permanently destroy a booking record and send a notification email is present, justifying a score higher than 1. However, the complete absence of architectural support for end-user control (3.1), anonymization (3.3), selective deletion (3.2), and legal retention compliance (3.6) significantly lowers the overall implementation maturity for data privacy requirements.


The implementation of robust **Data Deletion Functionality** requires extending the plugin’s existing core components—specifically its AJAX handling for secure actions, its custom database schema, and its established Email API—to support user-initiated requests, selective removal, and safe anonymization.

Here is a high-level overview of the architectural changes required, drawing on the plugin's patterns of custom hooks, data abstraction, and secure administrative workflow:

### Phase 1: PII Normalization and Indexing

The current limitation is that core user data (like email) is buried within a non-queryable **serialized array** in the `booking_options` column. This architecture prevents efficient deletion lookup and selective data removal.

1.  **Database Schema Enhancement:** Introduce dedicated, indexed database columns (e.g., `visitor_email`, `wordpress_user_id`) to the custom booking table. This change must be orchestrated during the plugin activation process.
2.  **PII Extraction on Save:** Modify the logic responsible for finalizing a new booking record to extract and securely save the email address and logged-in WordPress user ID into these new queryable columns.

### Phase 2: Secure Deletion and Anonymization Logic

This phase introduces the necessary server-side capabilities for responding to deletion requests, building upon the existing administrative AJAX controller.

1.  **New AJAX Endpoints (3.3):** Implement new, dedicated AJAX functions within `core/lib/wpbc-ajax.php` to handle advanced data actions. These should run parallel to the existing permanent deletion function (`wpbc_ajax_DELETE_APPROVE()`):
    *   **`wpbc_ajax_ANONYMIZE_APPROVE()`:** This critical function performs **safe anonymization (3.3)**. Instead of physically removing the booking record, it updates the PII fields (both the new dedicated columns and the complex serialized `booking_options` string) to generic, non-identifiable values (e.g., setting the name to "Anonymous" and email to "anon@deleted.com"). This allows retention of necessary statistical data (3.6).
    *   **Security:** Every new endpoint must enforce **nonce verification** using functions like `wpbc_check_nonce_in_admin_panel()` to secure the action against CSRF.

2.  **Selective Deletion (3.2):** Enhance the deletion/anonymization routine to allow the administrator (via the new queryable PII fields) to select which categories of data (e.g., booking details vs. future message logs or photos) are removed or anonymized. This logic would rely on filtering data before execution and cleaning related tables (3.4) through the `wpbc_booking_delete` action hook.

### Phase 3: Workflow Trigger and External Integration

This phase establishes the mechanism for users to initiate the process and ensures that the core action cascades across the entire ecosystem.

1.  **User Request Trigger (3.1):** Although the plugin lacks a user dashboard, a mechanism must be created for users to initiate a request. This could be an administrative tool where the admin searches for a user by the newly normalized `visitor_email` and triggers the Anonymize or Delete action on their behalf. This leverages the new indexing system defined in Phase 1.
2.  **External API Deletion (3.5):** The core action hooks triggered during the booking lifecycle—specifically `wpbc_booking_delete` and the hooks fired by `wpbc_booking_save()` (like `wpbc_track_new_booking`)—must be reliably fired by the new deletion and anonymization functions. This provides a clear, documented **extension point** where companion plugins (for loyalty or CRM APIs) can hook their custom logic (via `add_bk_action`) to cascade the deletion request to third-party systems.
3.  **Confirmation Messaging (3.7):** Use the existing **Email API pattern** (e.g., using the `WPBC_Emails_API_Deleted` class) to generate and send an automated confirmation notice (3.7) to the user once the deletion or anonymization process is complete.


---

### **4. Admin Tools & Automation**

| #   | Check Item             | Description                                                                                                                   | Implemented? |
| --- | ---------------------- | ----------------------------------------------------------------------------------------------------------------------------- | ------------ |
| 4.1 | Admin data export tool | Admin can search for a user and generate export manually.                                                                     | ☐            |
| 4.2 | Admin deletion tool    | Admin can delete a specific user’s entire record via dashboard.                                                               | ☐            |
| 4.3 | WP core integration    | Plugin integrates with **WordPress Personal Data Export/Erase tools** (available under *Tools → Export/Erase Personal Data*). | ☐            |
| 4.4 | Workflow automation    | Admin can set automatic deletion for inactive users after X months.                                                           | ☐            |

Based on the detailed analysis of the plugin's core architecture, data management, and administrative tools, here is the implementation status of the Admin Tools & Automation options:

| # | Check Item | Implemented? | Analysis of Implementation |
| :--- | :--- | :--- | :--- |
| **4.1** | **Admin data export tool** | $\checkmark$ | **Partially implemented.** The plugin provides administrative tools for export, but limitations exist regarding format and scope. |
| | | | 1. **Export Capability:** The UI for generating a **public .ics feed URL** is provided on the `core/admin/page-ics-export.php` settings page, which exports booking data to external calendars. |
| | | | 2. **Delegation/Limitation:** **Advanced export features** are explicitly delegated to the "**Booking Manager**" companion plugin. The sources do not confirm an administrative tool that allows searching **by user** and generating a manual export in common formats like JSON or CSV, only the ICS feed. |
| **4.2** | **Admin deletion tool** | $\checkmark$ | **Implemented.** The administrator has full control over deleting booking records via the backend dashboard. |
| | | | 1. **Deletion Workflow:** Core booking lifecycle actions, including permanent deletion (`wpbc_ajax_DELETE_APPROVE()`) and moving to trash (`wpbc_ajax_TRASH_RESTORE()`), are handled dynamically via the **AJAX controller** (`core/lib/wpbc-ajax.php`). |
| | | | 2. **Trigger:** The admin can perform quick management actions, including approval and decline, directly from their notification email using shortcodes like `[click2approve]` and `[click2decline]`. |
| **4.3** | **WP core integration** | $\boxtimes$ | **Not supported.** The plugin implements its own custom systems for hooks (`add_bk_action`), settings (`WPBC_Settings_API`), data retrieval (`wpbc_get_bookings_sql`), and lifecycle management (`WPBC_Install`). There is **no mention** in the sources of any integration points or implementation related to the native WordPress Personal Data Export or Erase tools found under *Tools* [1–234]. |
| **4.4** | **Workflow automation** | $\boxtimes$ | **Not supported by core.** The plugin utilizes a custom pseudo-cron system (`WPBC_Cron`) for scheduled tasks, such as Google Calendar synchronization (`wpbc_import_gcal`). However, the sources **do not mention** any setting or feature that allows the administrator to configure the *automatic deletion* of inactive user data or records after a set duration (e.g., X months) [189–234]. Deactivation cleanup is based on a global option (`booking_is_delete_if_deactive`), not conditional date ranges. |

### Summary of Key Administrative Tools and Gaps

The plugin is architecturally designed to support comprehensive administrative management, particularly through its powerful data engine (`core/admin/wpbc-sql.php`) for querying booking lists and its secure AJAX controller for immediate actions.

However, the analysis highlights significant architectural gaps regarding standardized compliance and advanced data governance:

*   **Reliance on Custom Formats:** Export is limited to the **iCalendar (.ics)** format. The core booking logic relies on storing complex user data (PII) in a **serialized array** within the `booking_options` column, which prevents efficient search and manual export customization.
*   **Delegation of Features:** Key advanced features, including ICS Import and advanced Export, are explicitly **delegated to the required "Booking Manager" companion plugin**.
*   **Lack of Core Integration:** The absence of integration with the native WordPress Personal Data tools (4.3) is consistent with the plugin's philosophy of building **custom architectural frameworks** for menus, settings, and lifecycle management.


The scores below reflect the degree to which the implementation of each Admin Tool & Automation feature is supported and described in the provided architectural and code analysis sources, on a scale of 1 (not implemented) to 10 (fully implemented):

| # | Check Item | Score (1-10) | Implementation Analysis Based on Sources |
| :--- | :--- | :--- | :--- |
| **4.1** | **Admin data export tool** | **4/10** | **Partially supported.** The plugin provides a UI for generating an **iCalendar (.ics) feed URL** for export, and the admin interface allows data querying. However, the sources note that advanced export functionality is explicitly **delegated to a required companion plugin** ("Booking Manager"), and there is no mention of a built-in tool for manual export in common privacy formats like JSON or CSV. |
| **4.2** | **Admin deletion tool** | **9/10** | **Highly implemented.** The administrative capability to delete specific booking records is fully supported. Permanent deletion is handled by dedicated functions in the AJAX controller (`wpbc_ajax_DELETE_APPROVE()`). This functionality is central to the administrative workflow, and actions can even be triggered directly via quick action links in admin notification emails. |
| **4.3** | **WP core integration** | **1/10** | **Not implemented.** The plugin is architecturally built on custom frameworks for settings, menus, and hooks (e.g., using `WPBC_Settings_API` and `add_bk_action`). There is **no documentation or evidence** in the sources that the plugin integrates with the native WordPress Personal Data Export or Erase tools [1–234]. |
| **4.4** | **Workflow automation** | **1/10** | **Not implemented.** While the plugin features a **custom pseudo-cron system** (`WPBC_Cron`) used for scheduling tasks like Google Calendar synchronization, there is **no documented logic or setting** that allows the administrator to configure the automatic deletion or anonymization of user data based on inactivity or age after a specific duration (e.g., X months) [1–234]. |



The implementation of the remaining Admin Tools & Automation features requires extending the plugin's modular architecture, particularly by addressing the storage of user data, integrating with native WordPress functions, and leveraging the existing custom cron system.

The high-level implementation overview would proceed in three phases: **Data Normalization for Lookup**, **Data Export Tool & Formatting**, and **Workflow Automation**.

---

### Phase 1: Data Normalization and Admin Lookup Enhancement (Addressing 4.1)

The most significant architectural hurdle is the current method of storing Personal Identifiable Information (PII), such as the visitor's email address, inside a non-queryable **serialized array** within the `booking_options` column of the custom database table.

1.  **Database Schema Update:** Update the custom booking database table (`{$wpdb->prefix}booking`) during the plugin activation process (`wpbc_activation` hook) to include dedicated, indexed columns for key lookup fields, such as `visitor_email` and `wordpress_user_id`.
2.  **Data Persistence Update:** Modify the booking creation process to ensure that when a booking is saved, these PII fields are extracted from the submitted form data and stored directly in the new database columns, enabling efficient SQL lookups.
3.  **Admin Search Tool:** Utilize the existing administrative UI component library (`core/any/admin-bs-ui.php`) and menu framework (`WPBC_Admin_Menus`) to create a new "Admin Data Tool" page. This tool will allow administrators to efficiently search for a user's records using the new indexed `visitor_email` or `wordpress_user_id` columns, leveraging the dynamic SQL capabilities of the data engine (`core/admin/wpbc-sql.php`).

### Phase 2: Data Export Tool and WP Core Integration (Addressing 4.1 & 4.3)

This phase addresses the need for structured export formats (JSON/CSV) and compliance with the native WordPress data privacy tools.

1.  **AJAX Export Endpoint (Manual Tool):** Implement a new handler in the central AJAX controller (`core/lib/wpbc-ajax.php`) to execute the manual export request triggered by the Admin Data Tool (Phase 1).
    *   This endpoint must first verify security using **nonce verification** (`wpbc_check_nonce_in_admin_panel()`).
    *   It will retrieve all relevant booking records using the efficient PII lookups and then use the **Developer API function `wpbc_api_get_booking_by_id()`** for each record. This function is critical because it handles the necessary **unserialization** of the booking metadata, providing a complete, structured PHP array of the user's data.
    *   The compiled data array is then converted into a structured, machine-readable format (e.g., JSON or CSV) and securely delivered.
2.  **WordPress Core Integration (Privacy Tools) (4.3):**
    *   **Export:** Hook into the native WordPress filter for personal data exporters (`wp_privacy_personal_data_exporters`). The function hooked here will query the plugin's custom booking table using the User ID or email provided by WordPress and retrieve the booking data using the same Developer API/unserialization workflow described above, ensuring all custom form fields are included in the export.
    *   **Erase:** Hook into the native WordPress filter for personal data erasure (`wp_privacy_personal_data_erasers`). The hooked function will query booking IDs for the specified user and then trigger the plugin's existing administrative hard deletion workflow (functions associated with `wpbc_ajax_DELETE_APPROVE()`) for each record.

### Phase 3: Workflow Automation (Addressing 4.4)

This phase integrates automated data management (deletion/anonymization) using the plugin's custom background task system.

1.  **Retention Setting:** Define a new global setting (e.g., `booking_retention_period`) using the plugin’s custom **Settings API (`WPBC_Settings_API_General`)** to allow the administrator to configure the retention period (e.g., "Delete/Anonymize records after X months of inactivity").
2.  **Cron Task Definition:** Utilize the plugin's custom **pseudo-cron system** (`WPBC_Cron` class) by defining a new scheduled action (e.g., `wpbc_automated_data_cleanup`).
    *   The scheduling of this task is handled by the public CRUD methods on the global `WPBC()->cron` object.
    *   The custom cron system relies on site traffic to trigger execution (hooked to the `init` action).
3.  **Cleanup Execution:** The function hooked to the `wpbc_automated_data_cleanup` action will execute the following logic:
    *   Query the custom booking table for records older than the configured `booking_retention_period`.
    *   For qualifying records, trigger the defined deletion workflow (either **hard deletion** via the core delete actions or, ideally, a **safe anonymization** process that strips PII from the newly normalized columns and the existing `booking_options` serialized array).
    *   Trigger the appropriate lifecycle hooks (e.g., `wpbc_booking_delete`) for compliance and to allow companion plugins (3.5) to perform external cleanup tasks.
  
4.  
---

### **5. Security & Logging**

| #   | Check Item                  | Description                                                                                 | Implemented? |
| --- | --------------------------- | ------------------------------------------------------------------------------------------- | ------------ |
| 5.1 | Authenticated requests only | Only logged-in users (or verified via secure token/email link) can request export/deletion. | ☐            |
| 5.2 | Temporary data handling     | Export files are deleted from the server after short expiry (e.g., 48h).                    | ☐            |
| 5.3 | Logs                        | Logs kept for data requests (who, when, what action) — required by ICO (UK).                | ☐            |
| 5.4 | API audit                   | Integrations (loyalty, CRM) log deletion confirmations or failures.                         | ☐            |

Based on the architectural analysis provided in the sources and our previous discussion regarding data export and deletion, here is the assessment of the implementation status for the Security & Logging check items:

| # | Check Item | Implemented? | Analysis of Implementation |
| :--- | :--- | :--- | :--- |
| **5.1** | **Authenticated requests only** | **Partially supported.** | Core administrative actions like deletion (`wpbc_ajax_DELETE_APPROVE()`) and status changes are highly secure, mandating **nonce verification** using functions like `wpbc_check_nonce_in_admin_panel()` to ensure the request is authenticated and originates from a legitimate admin session. Furthermore, the admin notification email includes quick actions ([click2approve], [click2decline]) that rely on a **secure token/link** for instant status changes. However, the core **.ics export feed** is exposed via a **public URL** and is not tied to a secure token or login requirement. |
| **5.2** | **Temporary data handling** | $\boxtimes$ | **Not supported.** The primary documented export method is generating a **public, perpetually accessible .ics feed URL**. The sources do not describe any architectural mechanism for creating temporary files that expire, nor is there mention of a process to delete export files from the server after a short duration (like 48 hours) [143–234]. |
| **5.3** | **Logs** | $\checkmark$ | **Partially supported.** The plugin includes internal logging mechanisms for tracking workflow. The "toolbox" file (`core/wpbc_functions.php`) includes functions for adding workflow logs (`wpbc_db__add_log_info`). The Developer API also includes action hooks such as `wpbc_track_new_booking` and `wpbc_booking_approved`, which can be used to log when specific booking events occur. However, the sources **do not explicitly confirm** the existence of a dedicated system that logs administrative data *request* actions (who initiated an export/deletion request and when) for compliance purposes [189–234]. |
| **5.4** | **API audit** | $\boxtimes$ | **Not supported.** While the Developer API provides hooks such as `wpbc_booking_delete` and `wpbc_track_new_booking` that allow external integrations (CRM/Loyalty) to *trigger* their own actions, the sources **do not document** any built-in core functionality that actively receives and logs deletion confirmations or failures from those third-party APIs for audit purposes [189–234]. A developer would need to manually implement this logging in their third-party extension. |

### Summary of Security Architecture

The plugin implements robust security primarily around administrative functionality and server hardening:

*   **CSRF Prevention:** All sensitive AJAX actions, including deletion and status updates, are protected by **Nonce verification**.
*   **Directory Protection:** The use of "silent index" files (`index.php` containing `<?php // Silence is golden. ?>`) in key directories prevents unauthorized directory listing, hiding sensitive code.
*   **Data Validation:** The data engine (`core/admin/wpbc-sql.php`) includes `wpbc_check_request_paramters()` to sanitize request parameters before use in SQL queries, preventing SQL injection.


Based on the detailed architectural analysis of the plugin's code, security components, and administrative workflows, here is the assessment of the Data Deletion Functionality check items on a scale of 1 to 10:

| # | Check Item | Score (1-10) | Implementation Analysis Based on Sources |
| :--- | :--- | :--- | :--- |
| **5.1** | **Authenticated requests only** | **6/10** | **Partially supported.** The system implements robust security for administrative *actions*. All sensitive admin-facing functions, including booking management (approve, trash, delete) are protected by **nonce verification** using functions like `wpbc_check_nonce_in_admin_panel()` to prevent CSRF attacks. Quick management links in administrator emails (`[click2approve]`, `[click2decline]`) also rely on a secure token mechanism. However, the primary documented export mechanism is the provision of a **public .ics feed URL**, which is not an authenticated request. |
| **5.2** | **Temporary data handling** | **1/10** | **Not supported.** The primary method for data export is the generation of a **public, perpetually accessible .ics feed URL**. The sources do not describe any architectural mechanism for creating temporary files that expire after a set duration (e.g., 48 hours) or for automatically cleaning up export files from the server [143–234]. |
| **5.3** | **Logs** | **5/10** | **Partially supported.** The plugin possesses the core infrastructure for logging: the custom utility file (`core/wpbc_functions.php`) includes `wpbc_db__add_log_info` for general **workflow logs**, and critical administrative actions fire explicit **action hooks** (e.g., `wpbc_booking_approved`, `wpbc_booking_delete`). However, there is **no explicit confirmation** in the sources that the system utilizes this logging capability specifically to track and store an audit trail of *data export requests* or *user deletion requests* (who, when, what) for compliance reporting. |
| **5.4** | **API audit** | **2/10** | **Minimally supported (via extension).** The core plugin **does not** log external API deletion confirmations or failures. However, the plugin provides documented, event-driven action hooks (like `wpbc_booking_delete`). This hook is a clear **extension point** that allows companion plugins (for loyalty or CRM APIs) to trigger their own external deletion routines. Logging the result of that external interaction would need to be custom-implemented by the integrating developer. |



The implementation of **Security & Logging** (5.1-5.4) must integrate deeply with the plugin's existing custom architecture, specifically leveraging its robust AJAX controller, custom cron system, and transactional Email API patterns.

Here is a high-level overview of the implementation phases:

---

### Phase 1: Secure, Authenticated Request Infrastructure (Addressing 5.1)

The implementation must ensure that data access requests (export or deletion) from non-logged-in users are authenticated via a secure, one-time token, mirroring the security provided to administrative actions.

1.  **Request Trigger and Token Generation:**
    *   A user submits a data request form (or the admin initiates on their behalf).
    *   The system uses the plugin's existing **Email API pattern** to send a confirmation email containing a **unique, time-limited, security tokenized link**. This process is architecturally analogous to the `[click2approve]` and `[click2decline]` shortcodes used in administrative notification emails.
2.  **AJAX Validation and Execution:**
    *   The click on the tokenized link sends the request to the central **AJAX controller** (`core/lib/wpbc-ajax.php`).
    *   The request handler performs **token validation** (checking the token's authenticity and expiry).
    *   If the request originates from the admin dashboard, the handler executes the standard **nonce verification** (`wpbc_check_nonce_in_admin_panel()`) to prevent CSRF attacks, a mandatory security practice for all sensitive admin actions.

### Phase 2: Temporary Data Handling and Cleanup Automation (Addressing 5.2)

Export files must be stored securely and deleted automatically, as the current export method only supports the perpetually accessible .ics feed URL.

1.  **Secure File Storage:**
    *   The export processing logic generates the file (e.g., JSON or CSV, as previously planned) and stores it in a non-public, randomized subdirectory (e.g., outside the web root or with `.htaccess` protection).
    *   The server generates the **time-limited download link** based on the secure storage location and the initial authentication token.
2.  **Automated Deletion:**
    *   Immediately after the export file is created, the system must utilize the custom **pseudo-cron system** defined by the `WPBC_Cron` class (`core/lib/wpbc-cron.php`).
    *   A custom cleanup task (e.g., `wpbc_automated_export_cleanup`) is scheduled using the cron object's public methods (`WPBC()->cron->add()`) to run after a short expiry period (e.g., 48 hours).
    *   This task queries the temporary file directory and permanently deletes expired files. The scheduler relies on website traffic to trigger execution via the WordPress `init` hook.

### Phase 3: Comprehensive Audit Logging (Addressing 5.3 & 5.4)

The system must log all data requests and deletion outcomes, extending the plugin’s existing custom event and logging infrastructure.

1.  **Request Audit Trail (5.3):**
    *   A new, dedicated logging function must be implemented to capture the details of every authenticated PII request (export or deletion). This function should record: **Who** (Admin ID or User Email), **When** (Timestamp), **What Action** (Export/Delete), and the **Target PII Identifier** (e.g., email address).
    *   This logging function should be triggered by a **custom action hook** (e.g., `wpbc_data_request_logged`) immediately after the request is authorized and executed.
    *   The logs should be stored either in a dedicated custom database table or by leveraging the existing workflow logging utility (`wpbc_db__add_log_info` in `core/wpbc_functions.php`).

2.  **External API Deletion Audit (5.4):**
    *   Leverage the plugin's strong focus on **extension points** through its **custom hook system**.
    *   The core deletion logic already fires specific action hooks (e.g., `wpbc_booking_delete`).
    *   A new **filter** (e.g., `wpbc_audit_external_deletion_result`) should be introduced at the end of the deletion workflow. Developers of companion plugins (e.g., loyalty or CRM APIs) would hook into this filter to pass back status data (success/failure, timestamp) after they attempt external deletion.
    *   The core logging system (from 5.3) would listen to this filter and record the external audit confirmation or failure, ensuring a complete record for compliance purposes.
  
3.  
---

### **6. User Communication**

| #   | Check Item        | Description                                                            | Implemented? |
| --- | ----------------- | ---------------------------------------------------------------------- | ------------ |
| 6.1 | Instructions page | Users can easily find instructions for how to request export/deletion. | ☐            |
| 6.2 | Status tracking   | Users can see the status of their request (pending / completed).       | ☐            |
| 6.3 | Contact for DPO   | Privacy notice lists data controller contact (email or form).          | ☐            |

Based on the analysis of the plugin's architecture, administrative components, and email workflow, here is the implementation status of the User Communication items:

| # | Check Item | Implemented? | Analysis of Implementation |
| :--- | :--- | :--- | :--- |
| **6.1** | **Instructions page** | $\boxtimes$ | **Not supported.** The sources detail admin tools for deletion and synchronization, but there is **no mention** of a user-facing instructions page, account dashboard, or public data request form where a user could easily find or trigger data export or deletion requests [189–234]. |
| **6.2** | **Status tracking** | $\boxtimes$ | **Not supported for data requests.** The plugin has a robust **Email API** that manages confirmation and notification for **booking workflow status** changes (e.g., Pending, Approved, Deleted, Denied). However, because the core architecture does not document a system for initiating user data export/deletion requests (6.1), there is logically **no documented feature** for users to track the status of such a data request (pending or completed). |
| **6.3** | **Contact for DPO** | $\boxtimes$ | **Not explicitly supported.** While the plugin uses a custom **Settings API** framework where a privacy notice or contact information could be configured, the sources **do not contain** any explicit mention of settings fields or required content that mandate listing a data controller (DPO) contact email or form for data privacy compliance purposes. The plugin does, however, enable direct communication via the Admin Notification email template which features an `enable_replyto` option that sets the Reply-To header to the visitor’s address. |

The architectural analysis indicates a strong focus on **administrative workflow** (deletion tools, quick action links in admin emails, administrative notices), but a **general absence of end-user facing tools** for data management (export/deletion requests and status tracking) [189–234].



Based on the detailed architectural review of the plugin's components, particularly the administrative and communication systems, here is the implementation assessment for the **User Communication** options on a scale of 1 to 10:

| # | Check Item | Score (1-10) | Implementation Analysis Based on Sources |
| :--- | :--- | :--- | :--- |
| **6.1** | **Instructions page** | **1/10** | **Not implemented.** The sources detail extensive administrative controls for deletion and status management (via the AJAX controller and quick action emails), but they **do not mention** a user-facing account dashboard, public data request form, or dedicated instructions page for end-users to initiate export or deletion requests [189–234]. |
| **6.2** | **Status tracking** | **1/10** | **Not implemented.** The plugin uses a robust Email API to track and communicate **booking status** changes (Approved, Deleted, Denied). However, since there is **no documented mechanism** for a user to trigger a data export/deletion request (6.1), there is logically no mechanism for the user to track the status (pending/completed) of such a request. |
| **6.3** | **Contact for DPO** | **1/10** | **Not supported.** The plugin uses a custom Settings API (`WPBC_Settings_API_General`) to catalog all configuration fields. The analysis of these settings and the Email API focuses on transactional details and reply-to options. The sources **do not contain any explicit evidence** of a dedicated setting field, administrative requirement, or default content that requires listing a Data Controller Contact (DPO email or form) for privacy compliance purposes [189–234]. |


The implementation of the **User Communication** features (Instructions page, Status tracking, and DPO Contact) requires creating end-user interfaces and integrating them into the plugin's custom administrative and email systems.

The high-level implementation would focus on three phases: **Settings for Compliance**, **Public Interface Creation**, and **Status Messaging Workflow**.

---

### Phase 1: Compliance Settings and Data Preparation (Addressing 6.3)

This phase ensures that the necessary compliance information is configurable and that user data is structured for efficient lookup (a prerequisite for any deletion/export request).

1.  **Introduce DPO Contact Setting (6.3):**
    *   Leverage the plugin's **custom Settings API framework** by defining a new field (e.g., a text area or email field) for the Data Protection Officer (DPO) contact information.
    *   This field should be introduced either by extending a settings class or by using an available filter hook (like `apply_filters`) within the field definition method (`init_settings_fields()`).
    *   The data would be persisted using the plugin's dedicated option management wrappers, such as `update_bk_option`.
2.  **PII Normalization:** (As detailed in previous answers) Since the current architecture stores PII in an inefficient, **serialized array** in the `booking_options` column, the implementation must modify the custom booking table (`{$wpdb->prefix}booking`) to include dedicated, indexed columns (like `visitor_email`) to enable fast lookup for handling requests.

### Phase 2: Public Interface and Request Trigger (Addressing 6.1)

Since the plugin lacks a native user-facing dashboard, a public endpoint is required to serve the instruction page and capture the request.

1.  **Register New Shortcode:** Use the existing architectural pattern for exposing front-end functionality through shortcodes.
    *   Register a new shortcode (e.g., `[booking_data_request]`) via the `wpdev_booking` class initialization logic.
    *   The shortcode's rendering function would output the necessary **instructions, privacy policy link**, and a form allowing the user to submit their email address and request type (Export/Deletion).
2.  **AJAX Submission Endpoint:** Define a new action handler in the central **AJAX controller** (`core/lib/wpbc-ajax.php`).
    *   This non-authenticated endpoint (`wp_ajax_nopriv_...`) would accept the user's data request (email).
    *   The handler initiates the internal workflow: logging the request (Phase 3) and triggering the secure authentication process (Phase 1 of Security implementation overview: sending an email with a secure, time-limited token for verification).

### Phase 3: Status Messaging and Confirmation Workflow (Addressing 6.2 & 3.7)

This phase builds the necessary communications to keep the user informed and record the event for audit purposes.

1.  **Custom Email Template for Status:** Leverage the plugin's structured **Email API pattern**.
    *   Create a new concrete email class (e.g., `WPBC_Emails_API_DataStatus`) that extends the abstract `WPBC_Emails_API`.
    *   This class would define fields for three primary notification types: "Request Received," "Verification Required," and "Request Completed."
2.  **Workflow Integration and Logging (6.2):**
    *   **Initial Status:** Immediately after the request is submitted (Phase 2) and verification is requested, trigger the new email template's sending function (`wpbc_send_email_data_status()`) to confirm receipt and provide next steps.
    *   **Final Status:** When the administrator completes the action (Deletion or Export) using the appropriate administrative tool (e.g., executing permanent deletion via `wpbc_ajax_DELETE_APPROVE()`), ensure this action fires the core `wpbc_booking_delete` lifecycle hook.
    *   A function listening to this hook would then trigger the final **"Request Completed" email**, providing confirmation of the completed data deletion or a secure link to the exported data.
    *   All status changes and sending events should be recorded using the internal logging utilities, such as `wpbc_db__add_log_info`.

---

### ✅ **Success Levels**

| Level                      | Description                                                                                                                                         |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Basic**                  | Plugin integrates with WordPress’s built-in export/erase tools for bookings only.                                                                   |
| **Good**                   | Includes loyalty/referral/messaging data and handles anonymisation correctly.                                                                       |
| **Full Compliance (High)** | Full user self-service for export and deletion, covers third-party integrations, keeps audit logs, and supports selective or complete data purging. |

---

Would you like me to **convert this into a CSV checklist** (like the others) so you can track plugin implementation status (Yes / Partial / No)?
