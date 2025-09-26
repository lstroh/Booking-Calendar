Got it ✅ — let’s break down the **“Option for clients to upload photos when booking”** into clear, testable requirements and success criteria you can later use as a checklist when reviewing the plugin implementation.

---

## **Requirements Breakdown**

### 1. Upload Functionality

* [ ] Clients can upload photos directly in the booking form.
* [ ] Support for **multiple file uploads** (not just one).
* [ ] Allowed file formats configurable (e.g., JPG, PNG, HEIC, PDF if needed).
* [ ] File size limit configurable (e.g., 10MB per file, total 50MB).

Based on the detailed architectural analysis of the plugin's core components, **Upload Functionality** as described is **not implemented** within the reviewed files.

On a scale of 1–10, I would mark the implementation of this feature as a **1 out of 10**.

This rating is based on the complete absence of any file upload-specific form field types, associated configuration settings (such as file size limits), or server-side processing logic necessary for handling file uploads.

### Analysis Against Acceptance Criteria

The functionality required for file uploads relies on three missing architectural elements identified in the source material:

| Acceptance Criterion | Implementation Status | Supporting Evidence |
| :--- | :--- | :--- |
| Clients can upload photos directly in the booking form. | **Not Implemented** | The plugin uses `core/form_parser.php` to define and render forms. This parser supports defined field types (e.g., text, select). **No evidence** exists for a dedicated form field shortcode (like `[file]` or `[upload]`) that would allow clients to select files. |
| Support for multiple file uploads (not just one). | **Not Implemented** | Dependent on the existence of a file upload field, which is not evidenced. Furthermore, the parser's mechanism is limited: the recognized shortcode types are **hardcoded** into the regular expression, making it difficult for developers to add a new, complex field type like "multi-file upload" without modifying core files. |
| Allowed file formats configurable. | **Not Implemented** | Configuration for file type constraints (e.g., JPG, PDF) would typically be found in the General Settings, likely defined within the `WPBC_Settings_API_General` class. The analysis of the plugin's settings systems and database options does not reference any fields related to file type or file size configuration. |
| File size limit configurable (e.g., 10MB per file, total 50MB). | **Not Implemented** | Configuration settings for imposing limits (like file size or total size) are not mentioned in the source material detailing the plugin's configuration systems. Such configuration is a prerequisite for a usable upload feature. |

While the plugin's data model is capable of storing custom booking data (such as a file path or URL) in a serialized array in the `booking_options` column, the necessary front-end form fields and server-side processing for receiving and securely storing the actual file content on the server are not present in the files analyzed.

The implementation of the **Upload Functionality** feature would require modification and extension across five key architectural layers of the Booking Calendar plugin: the Configuration Layer, the Form Parsing Layer, the Dynamic Processing (AJAX) Layer, the Data Persistence Layer, and the Administrative Display Layer.

Here is a high-level overview of the required implementation, utilizing the existing plugin architecture:

### 1. Configuration Layer: Defining Limits and Types

Configuration settings for allowed file formats and size limits must be defined in the plugin's General Settings area.

*   **File:** `core/admin/api-settings.php`.
*   **Action:** New fields for configuring file formats (e.g., `jpg, png, pdf`) and size limits (e.g., `10MB`) would be added to the `$this->fields` array within the `WPBC_Settings_API_General::init_settings_fields()` method.
*   **Persistence:** These settings would be saved to the database, likely as individual options in the `wp_options` table, leveraging the existing custom settings framework.

### 2. Form Parsing and Rendering Layer

The plugin must recognize a new file upload field in the administrator's form configuration string and render the appropriate HTML input.

*   **File:** `core/form_parser.php` (for parsing) and core UI files (for rendering).
*   **Action (Limitation):** This step presents a primary architectural challenge. The function `wpbc_parse_form()` in `core/form_parser.php` uses complex **regular expressions** to identify and parse field types (like `[text]` or `[select]`). To support file uploads, a new field shortcode (e.g., `[upload* your-files]`) must be recognized. Since the list of supported field types is **hardcoded** into the regex, adding a new complex field type requires core file modification or the introduction of a new filter to safely extend the parser.
*   **Rendering:** Once parsed, the rendering logic would use helper functions from the UI library (`core/any/admin-bs-ui.php`) to output the `<input type="file">` HTML element with the necessary attributes (`multiple`, required attributes, etc.).

### 3. Dynamic Processing Layer (AJAX and Security)

File uploads are asynchronous and must be handled by a secure AJAX endpoint to process the incoming file data before the booking record is finalized.

*   **File:** `core/lib/wpbc-ajax.php`.
*   **Action:** A dedicated AJAX action (e.g., `wpbc_ajax_upload_files`) would be required to handle the file processing requests from the client. This action would be safely registered using the **`wpbc_ajax_action_list` filter**, which is the designated extension point for custom AJAX endpoints.
*   **Security:** The new AJAX handler must strictly implement **nonce verification** using functions like `wpbc_check_nonce_in_admin_panel()` to prevent Cross-Site Request Forgery (CSRF), a necessary security measure for all sensitive administrative actions.
*   **Server-Side Validation:** The handler must read the file size and type limits from the settings (Step 1) and validate the uploaded files against these constraints.

### 4. Data Persistence Layer

The file paths or URLs must be stored alongside the booking record in the database.

*   **File:** `core/wpbc-core.php` and `core/wpbc-dev-api.php`.
*   **Action:** The file URLs/paths resulting from the successful upload must be stored as **booking meta options**. The custom function **`wpbc_save_booking_meta_option($booking_id, $option_arr)`** would be used.
*   **Storage Location:** This data is saved as a single, **serialized array** within the **`booking_options` column** of the plugin's custom database table (`{$wpdb->prefix}booking`).

### 5. Administrative Viewing Layer

The administrator must be able to retrieve and view the custom data (file links) in the dashboard.

*   **File:** `core/wpbc-dev-api.php` and the UI rendering files (`core/wpbc_functions.php`).
*   **Retrieval:** The Developer API function **`wpbc_api_get_booking_by_id()`** is used to fetch the booking details, and it includes logic to **unserialize** the stored `booking_options` array, making the file URLs accessible to the dashboard display routines.
*   **Display:** The administrative Booking Listing or detail page would use functions for structured UI display (e.g., meta boxes from `core/wpbc_functions.php`) to render clickable links to the uploaded files.


### 2. Storage & Access

* [ ] Photos are stored securely in WordPress (e.g., media library or dedicated folder).
* [ ] Files are linked to the specific booking record in the database.
* [ ] Tradesperson (admin or assigned staff) can view/download photos from the booking details screen.
* [ ] Access permissions respected — only tradesperson/admin, not other clients.

Based on the analysis from the sources and our previous discussion, the **Upload Functionality** required to implement this feature is **not present** in the core files reviewed. Consequently, the subsequent **Storage & Access** requirements for photos are also not implemented.

In our previous discussion, the Upload Functionality feature was rated **1 out of 10** because there was no evidence of a dedicated file upload form field type, configuration for file size/format limits, or server-side logic for handling the file upload process itself.

Since the core capability to upload files is missing, the downstream requirements for storage and access cannot be fulfilled by the analyzed plugin architecture.

Here is a breakdown of the specific requirements against the plugin's architecture:

### Implementation Status: Not Implemented (Score: 1/10)

| Requirement | Architectural Status | Evidence from Sources |
| :--- | :--- | :--- |
| **Photos are stored securely in WordPress (e.g., media library or dedicated folder).** | **Not Implemented.** | The files lack the necessary server-side PHP logic (usually involving `$_FILES` and `wp_handle_upload` or similar file processing) required to ingest and store physical files securely on the server. |
| **Files are linked to the specific booking record in the database.** | **Architecturally Possible, but Functionally Disabled.** | The plugin architecture *can* store paths or URLs (the link) to files. Custom form field values, known as **booking meta options**, are stored as a serialized array in the **`booking_options`** column of the custom `{$wpdb->prefix}booking` database table. If the upload feature were implemented, the resulting file path would be stored here. However, since no upload occurs, there is no path to store. |
| **Tradesperson (admin or assigned staff) can view/download photos from the booking details screen.** | **Not Implemented.** | Viewing photos relies on retrieving the file link from the database. The Developer API function `wpbc_api_get_booking_by_id()` can retrieve booking data and **unserialize** the `booking_options` array. If a file path were stored, this function would retrieve it; however, the UI logic for displaying that link as a secure viewer or download button on the admin screen is not evidenced, nor is the data path present. |
| **Access permissions respected — only tradesperson/admin, not other clients.** | **Not Implemented.** | While the plugin supports admin panel permissions, the specific logic required to restrict access to the uploaded files (e.g., controlling a secure file download endpoint based on user roles) is not documented. This level of access control would require dedicated functionality not seen in the files reviewed. |

In summary, while the plugin's data model is robust enough to store file paths as serialized metadata, the necessary **Upload Functionality** to place the file on the server and store that path in the database is not implemented.

This feature requires extending the plugin's architecture across three major domains: form validation, secure file handling via AJAX, and administrative retrieval with strict access control.

The high-level implementation for **Booking Storage & Access** leverages the plugin's custom form parser, robust AJAX controller (`core/lib/wpbc-ajax.php`), and custom metadata storage (`core/wpbc-core.php`).

### 1. Configuration and Form Layer Extension

Since the current `core/form_parser.php` file contains a **hardcoded list of field types** defined by complex regular expressions, the ability to recognize a new upload field (e.g., `[file your-photos]`) would require modifying the core parser or implementing a filter within it to allow for new shortcode definitions.

*   **Settings Integration:** New fields would be added to the General Settings (via the **`WPBC_Settings_API_General`** class in `core/admin/api-settings.php`) to define the **allowed file formats** and **file size limits**. These settings would be saved as individual options using the **`'separate'` strategy** in the `wp_options` table.

### 2. Secure File Upload and Persistence via AJAX

File uploads cannot be handled by the typical booking submission, requiring a dedicated, secure asynchronous process.

*   **AJAX Endpoint:** A new AJAX action (e.g., `wpbc_ajax_process_upload`) must be registered using the **`wpbc_ajax_action_list` filter** within the central AJAX router (`core/lib/wpbc-ajax.php`).
*   **Security:** This new AJAX handler must strictly enforce **nonce verification** using functions like `wpbc_check_nonce_in_admin_panel()` to prevent CSRF, a critical security measure applied to all sensitive admin-facing actions.
*   **File Storage:** The server-side AJAX handler processes the incoming file, moves it to a secure, dedicated folder (or the Media Library), and generates a unique, unguessable file path or URL.
*   **Database Linking (Meta Storage):** The file path/URL is linked to the booking record using the core metadata function **`wpbc_save_booking_meta_option($booking_id, $option_arr)`**. This metadata is stored as a **single, serialized array** in the **`booking_options` column** of the custom `{$wpdb->prefix}booking` database table.

### 3. Administrative Access and Viewing

The stored file links must be displayed in the booking details screen and linked to a secure download mechanism.

*   **Data Retrieval:** When the administrator opens the booking record, the function **`wpbc_api_get_booking_by_id()`** (`core/wpbc-dev-api.php`) retrieves the serialized data and **unserializes** the `booking_options` column into a readable array.
*   **UI Display:** The booking details screen (likely constructed using helper functions for meta boxes, such as **`wpbc_open_meta_box_section`** from `core/wpbc_functions.php`) would be extended to check for and render the file paths as secure, clickable links.

### 4. Security and Access Permissions

Access must be tightly restricted to prevent unauthorized viewing by clients or unauthorized staff.

*   **Secure Download Handler:** A second, specialized endpoint (e.g., `wpbc_ajax_secure_download`) is required to serve the actual file content. This endpoint must use the file path stored in the `booking_options` column, rather than exposing the direct file URL.
*   **Permission Check:** Before streaming the file content, the handler must check the current user's role and capability (e.g., using `current_user_can()` logic). It must verify that the user is an **administrator** or a staff member with **Admin Panel Permissions** to view booking details. If the check fails, the request must be aborted, and a 403 Forbidden error returned.

### 3. Consent & Compliance

* [ ] Clear **consent checkbox** before upload (e.g., “I consent to my photos being stored and used for service purposes”).
* [ ] Checkbox must be required before upload is submitted.
* [ ] Consent record stored with booking data (timestamp + wording).
* [ ] GDPR-compliant — clients can request deletion of their photos later.

This feature, **Consent & Compliance**, is implemented in terms of the underlying architectural support for mandatory fields and data persistence, but it cannot fully meet the compliance requirements concerning **photo deletion** because the core **Upload Functionality** is not implemented (as established in previous analysis).

I would mark the implementation of the core consent mechanism as an **8 out of 10**. This reflects the high certainty that a required consent field can be displayed and stored, but recognizes the failure to address the full compliance workflow for the non-existent photos.

---

### Implementation Breakdown

| Acceptance Criterion | Status & Architectural Support |
| :--- | :--- |
| **Clear consent checkbox & Checkbox must be required.** | **Functionally Implemented** |
| **Form Field:** The plugin uses a customizable, shortcode-like syntax for administrators to define front-end booking forms. The core parser, `core/form_parser.php`, is responsible for translating this syntax into a structured PHP array used for rendering the form. While a dedicated "consent" field is not explicitly mentioned, the parser recognizes field shortcodes (e.g., `[text* your-name]`) and supports necessary attributes. The core feature of providing a selectable option (like a checkbox) and defining it as **required** (often through an asterisk, e.g., `[checkbox*]`) is strongly supported by the form architecture. |
| **Consent record stored with booking data (timestamp + wording).** | **Fully Implemented** |
| **Data Persistence:** The plugin's data handling system is robust and fully supports saving custom form field values, which would include the required consent checkbox state and its wording. Custom data, known as **booking meta options**, is stored as a single, complex **serialized array** in the **`booking_options`** column of the custom `{$wpdb->prefix}booking` database table. The Developer API retrieves this data and **unserializes** it into a readable array for the administrator. This architecture can easily record the state of the consent at the time of submission. |
| **GDPR-compliant — clients can request deletion of their photos later.** | **Not Implemented (Due to Missing Dependency)** |
| **Photo Deletion:** This criterion **cannot be met** because the fundamental prerequisite, **Upload Functionality**, is missing from the plugin's core architecture. The system does not implement the necessary form field types, server-side processing, or storage of file paths required to manage physical files. **General Deletion:** The plugin does include architectural features for deleting bookings entirely via AJAX actions (`wpbc_ajax_DELETE_APPROVE`), and its deactivation logic respects an option to delete data (`booking_is_delete_if_deactive`). However, there is no evidence of a specific, client-initiated, GDPR-compliant workflow for managing *only* the photo data (which does not exist). |

### 4. User Experience

* [ ] Drag-and-drop upload support.
* [ ] Preview thumbnails before final submission.
* [ ] Option to remove/edit uploads before confirming booking.
* [ ] Mobile-friendly upload (camera integration on phones).

Based on the architectural analysis provided in the sources and our previous conversation regarding the **Upload Functionality**, the features related to advanced upload User Experience (UX) are **not implemented**.

In our conversation regarding **Upload Functionality**, the implementation score was rated **1 out of 10** because the core architectural components necessary for any file handling (a dedicated file upload form field type, configuration for file limits, and server-side processing logic) were confirmed as missing. Since the plugin does not support basic file uploading, it cannot support the advanced UX features tied to uploads (drag-and-drop, previews, or mobile camera integration).

Therefore, I would mark the implementation of this feature set as a **1 out of 10**.

### Analysis Against UX Acceptance Criteria

The following features rely entirely on a foundational file upload component, which is absent from the core files analyzed:

| UX Acceptance Criterion | Implementation Status | Justification Based on Architecture |
| :--- | :--- | :--- |
| **Drag-and-drop upload support.** | **Not Implemented.** | This advanced feature requires dedicated client-side JavaScript for file handling and HTML5 API integration, and a dedicated AJAX endpoint to receive the uploaded data. The plugin lacks the necessary file upload field type definition (in the `core/form_parser.php` engine) and the specialized AJAX handler to process file binary data. |
| **Preview thumbnails before final submission.** | **Not Implemented.** | This requires client-side JavaScript to read the file data (e.g., using `FileReader` API) and render thumbnails, which is only relevant if a file input field exists. Since a file field is not supported by the form parser and no associated JavaScript libraries for this functionality are identified among the enqueued assets, this is not present. |
| **Option to remove/edit uploads before confirming booking.** | **Not Implemented.** | This requires a functional upload feature and corresponding AJAX endpoints to delete or replace temporary files. The plugin's AJAX infrastructure (`core/lib/wpbc-ajax.php`) is robust for booking status changes (approve, delete), but contains no evidence of temporary file management actions. |
| **Mobile-friendly upload (camera integration on phones).** | **Not Implemented.** | This is dependent on the core file input field being implemented, often achieved through specific HTML attributes (e.g., `capture` attribute) on the `<input type="file">` tag. The lack of a file input field definition prevents this integration. |

### Note on Mobile-Friendly UI (General)

While the advanced upload features are missing, the plugin generally incorporates modern UI practices:

*   The administrative UI utilizes a **Flexbox** layout for responsiveness, as seen in the **Timeline V2** feature, which adapts via `@media` queries for smaller screens.
*   The plugin uses a sophisticated **JavaScript controller** (`core/wpbc-js.php`) and **data bridge** (`core/wpbc-js-vars.php`) that manage assets and configurations reliably, which is foundational for any modern, mobile-friendly interface.

The implementation of advanced **User Experience** (UX) features for file uploads—such as drag-and-drop, real-time previews, and pre-submission editing—requires significant architectural extensions, primarily focusing on the client-side presentation and a secure, dedicated AJAX pipeline, since the core upload capability is currently missing.

This high-level overview details how this feature set would be built, leveraging the plugin's existing custom frameworks for assets, dynamic interfaces, and data persistence.

### 1. Architectural Setup and Configuration Layer

The prerequisite rules for file size and format must be defined and stored, utilizing the plugin's established settings framework.

*   **File:** `core/admin/api-settings.php`
*   **Action:** New fields defining **Maximum File Size** and **Allowed MIME Types** would be added to the `$this->fields` array within the `WPBC_Settings_API_General` class. These configurations are saved as options in the database using the `'separate'` strategy. These rules will be enforced by the server-side AJAX handler.

### 2. Form Field Definition and Client-Side Asset Loading

A new, custom field type is needed to render the drag-and-drop zone, and a dedicated script must be loaded to manage the dynamic UX.

*   **Form Field (UX Container):** A new field shortcode (e.g., `[upload-files]`) must be added to the complex regular expression found in `core/form_parser.php`. This shortcode, when parsed, renders the specialized HTML container for the drag-and-drop interface.
*   **Dynamic Asset Loading:** A dedicated JavaScript file (e.g., `upload-handler.js`) must be registered and enqueued by the master controller in **`core/wpbc-js.php`**. This script would utilize the appropriate JavaScript APIs for handling drag-and-drop events and accessing mobile camera integration APIs.

### 3. Client-Side UX Logic (Previews and Editing)

The core UX features (previews, removal) are managed entirely client-side until final booking confirmation.

*   **Real-Time Previews:** The **`upload-handler.js`** script receives the files dropped/selected by the client. It uses browser APIs (like `FileReader`) to display immediate, localized **thumbnail previews** for photos, validating the size and type locally against the limits passed from the backend via the **`_wpbc` global object** (managed by `core/wpbc-js-vars.php`).
*   **Removal/Editing:** The client-side script maintains a list of temporary file references (e.g., temporary IDs returned by the server after partial upload). The "Remove" option simply deletes the file entry from this client-side list and sends a corresponding AJAX request to the server to delete the temporary file.

### 4. Secure AJAX Upload Pipeline (Temporary Storage)

File upload processing must be asynchronous, secure, and separated from the final booking submission.

*   **Endpoint:** A new AJAX action (e.g., `wpbc_ajax_upload_temp`) must be registered using the **`wpbc_ajax_action_list` filter** in the central AJAX router, **`core/lib/wpbc-ajax.php`**.
*   **Security:** The handler must strictly enforce **nonce verification** using `wpbc_check_nonce_in_admin_panel()` and then execute server-side validation against the configured file limits (Step 1).
*   **Temporary Storage:** On successful validation, the file is saved to a secure, temporary location on the server, and a temporary unique ID is returned to the client for tracking (Step 3).

### 5. Final Submission and Data Persistence

The final, permanent storage and linking of files occur only when the client submits the booking form.

*   **Booking Finalization:** The temporary file IDs/URLs are submitted along with the booking form. The server-side booking creation logic (likely residing in `core/lib/wpbc-booking-new.php` or its refactored counterpart) is extended to move the files from temporary storage to their **permanent secure WordPress location**.
*   **Database Linkage:** The permanent file URLs/paths are then collected and stored as a **serialized array** in the **`booking_options` column** of the custom booking database table (`{$wpdb->prefix}booking`). This is achieved using the core function **`wpbc_save_booking_meta_option()`**, linking the file securely to the specific booking record.

### 5. Notifications & Integration

* [ ] Tradesperson notified (email/SMS) when booking includes photos.
* [ ] Photos included as attachment or secure link in email notifications (optional, configurable).
* [ ] API/Webhook ready if integration with other tools (e.g., CRM, quoting).

Based on the architectural analysis of the plugin's notification system, email workflow, and integration mechanisms, the **Notifications & Integration** feature set is **partially implemented** but is significantly constrained by the lack of native file upload support and reliance on companion plugins for some advanced integration methods.

I would rate the implementation of this feature set as a **6 out of 10**.

### Implementation Breakdown

#### 1. Tradesperson notified (email/SMS) when booking includes photos.

The notification system is fully implemented for **email**, but the criterion regarding photo-specific triggers and SMS functionality is not confirmed.

*   **Tradesperson Notification (Email):** **Fully Implemented.** When a new booking is submitted, the system sends a notification to the administrator(s) defined in the settings. The email template, managed by the `WPBC_Emails_API_NewAdmin` class (defined in `core/admin/page-email-new-admin.php`), is designed to streamline the administrative workflow.
*   **Photo Trigger Logic:** **Unconfirmed.** While the system sends an email for a *new booking*, there is no evidence that the notification is specifically triggered *because* photos were included. The email template can be customized using a Shortcode Replacement Engine, which substitutes dynamic values with booking-specific data. If the photo status or links were available as shortcodes, they could be included in the email.
*   **SMS Notification:** **Not Implemented (No evidence).** The sources extensively detail the **Email API** framework, but there is no mention of a native SMS feature or dedicated SMS API integration.

#### 2. Photos included as attachment or secure link in email notifications (optional, configurable).

This is **Not Implemented**, as it relies on the core **Upload Functionality** which is absent (rated 1/10 in prior analysis).

*   **File Attachment/Link:** Since the plugin's architecture does not support file uploads, storage, or file path persistence for client-submitted photos (as established in previous analysis), there are no files or links to attach or embed in the notification emails.
*   **Email Customization:** The Email API uses the `wpbc_wp_mail()` wrapper for sending. Although this wrapper enhances deliverability (by fixing the Sender header via `phpmailer_init`), it does not inherently provide attachment functionality without a file path.

#### 3. API/Webhook ready if integration with other tools (e.g., CRM, quoting).

The integration mechanism is **implemented** through extensive custom hooks and a dedicated Developer API.

*   **Developer API:** **Fully Implemented.** The file `core/wpbc-dev-api.php` provides an official abstraction layer with a stable set of functions for third-party developers. This API enables programmatic interaction, such as creating or editing bookings (`wpbc_api_booking_add_new`).
*   **Event-Driven Hooks (Webhooks Foundation):** **Fully Implemented.** The plugin utilizes a custom internal hook system (`add_bk_action`, `apply_bk_filter`) that runs parallel to standard WordPress hooks. This system is used for workflow events. Crucially, the AJAX handler (`core/lib/wpbc-ajax.php`) triggers specific action hooks upon status changes:
    *   `wpbc_booking_approved`
    *   `wpbc_booking_delete`
    *   `wpbc_track_new_booking`
    These actions are the intended extension points for triggering side effects like integration with a CRM or external notifications.
*   **External Integration (Cron/Sync):** The system supports background automation via a custom **pseudo-cron** system (`WPBC_Cron`), which is used to trigger imports from external calendars like Google Calendar. This infrastructure could potentially be used for scheduled export or webhook checks.
*   **Import/Export Limitations:** Advanced synchronization with external tools via **.ics feeds** (which could be used for external data integration) is **delegated** via the `wpbm_ics_import_start` hook and **requires the "Booking Manager" companion plugin**.

The implementation of the **Notifications & Integration** feature set requires leveraging the plugin's highly developed **Email API** and **Developer API**, while assuming the prerequisite **Upload Functionality** is in place, meaning file links are stored as booking metadata.

Here is a high-level overview of how these capabilities would be implemented using the existing architectural components:

### 1. Configuration: Enabling Photo Notifications

The administrative settings must be extended to allow the tradesperson (administrator) to configure how and when notifications related to photos are sent.

*   **File:** `core/admin/api-settings.php` (for configuration) and specific email template files (e.g., `core/admin/page-email-new-admin.php`).
*   **Action:** Add a setting field, likely within the **General Settings** defined by the `WPBC_Settings_API_General` class, to toggle whether the email notifications should include photo links or attachments. This option would be saved using the plugin's standard `'separate'` saving strategy.
*   **Template Customization:** Ensure the **New Booking (Admin)** template, managed by the `WPBC_Emails_API_NewAdmin` class, includes a conditional shortcode that renders the photo links.

### 2. Notifications: Dynamic Shortcode and Email Attachment Logic

The core email API must be enhanced to dynamically retrieve the stored file paths and include them safely in the notification content.

*   **Data Retrieval:** The email sending function (e.g., `wpbc_send_email_new_admin()`) retrieves the booking data. The file paths, stored as **booking meta options** in the `booking_options` column, must be extracted by the appropriate helper functions (e.g., calling `wpbc_api_get_booking_by_id()` which **unserializes** the metadata).
*   **Dynamic Shortcode Generation:** The retrieval logic must hook into the **`wpbc_replace_params_for_booking`** filter. This filter, used by the sending logic before dispatch, is the mechanism to process a new shortcode (e.g., `[photo_links]`). The function hooked here will generate a secure list of links based on the file paths extracted from the meta options.
*   **Secure Links/Attachments:** The final sending process uses the wrapper function **`wpbc_wp_mail()`**. This function, which handles formatting and deliverability, would need to be enhanced to:
    *   **Attachments:** If attachments are configured, the physical file paths would be passed to the underlying `wp_mail()` function for inclusion.
    *   **Secure Links:** If links are configured, they must point to a secure, permission-checked download endpoint that verifies the recipient's role (admin/staff) before serving the file, ensuring **access permissions are respected**.

### 3. Integration: API and Webhook Readiness

The plugin's existing Developer API and custom hook system provide immediate readiness for integration with external tools (CRM, quoting), provided the custom data is accessible.

*   **Event Triggering:** Reliance on the plugin's established internal hook system (using **`make_bk_action`**) is crucial. External tools can listen to core actions that fire upon successful booking insertion or status change, such as **`wpbc_track_new_booking`** or **`wpbc_booking_approved`**.
*   **Data Access for Webhooks:** The Developer API (`core/wpbc-dev-api.php`) already offers **`wpbc_api_get_booking_by_id()`**. Since this function retrieves the booking data and **unserializes** the entire **`booking_options`** array, the custom file URLs/paths stored within that array are immediately accessible to any external webhook or API query listening to the workflow events.
*   **Scheduling Automation (If needed):** If the integration requires periodic data export (instead of real-time webhooks), the public methods of the custom **`WPBC_Cron`** pseudo-cron system (`core/lib/wpbc-cron.php`) can be used to schedule custom tasks that query booking data and push it to the external service.

---

## **Success Criteria**

✅ Clients can upload multiple photos during booking.
✅ Only supported formats & sizes are accepted (error messages shown if invalid).
✅ Uploads are tied to the correct booking in WordPress.
✅ Consent checkbox must be ticked before form submission is allowed.
✅ Admin/tradesperson can view/download photos from booking details.
✅ GDPR compliance: consent stored + ability to delete photos on request.
✅ Photos display previews in booking form before submission.
✅ Works smoothly on desktop and mobile.
✅ No other clients can access the uploaded files.
✅ Notifications (email/SMS) correctly include photo references or links.

---

Would you like me to also **estimate development effort & dependencies** (e.g., media handling, GDPR logging, mobile support) so you can plan this feature against the others in your backlog?
