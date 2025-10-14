Perfect — this is an essential compliance feature for UK- and EU-based platforms. Let’s break down:

> **GDPR & UK Act → Transparency → Privacy notice covering bookings, photos, and messages**
> Type: *Must-Have* | Effort: *20* | Priority: *High*
> Dependencies: Twilio / SendGrid / WhatsApp Business API

Below is a **detailed breakdown** you can use to verify if your plugin correctly implements transparency and privacy notice requirements.

---

## ⚖️ GDPR & UK Act — Privacy Notice for Bookings, Photos & Messages

**Goal:** Ensure users (clients and tradespeople) are informed about what data is collected, how it’s used, and their rights.

---

### **1. Privacy Notice Availability**

| #   | Check Item       | Description                                                                                                                           | Implemented? |
| --- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------- | ------------ |
| 1.1 | Notice exists    | Does the plugin provide or link to a **Privacy Policy / Notice** page?                                                                | ☐            |
| 1.2 | Notice placement | Is the notice clearly visible **before or at the time of data collection** (e.g., booking form, photo upload, or message submission)? | ☐            |
| 1.3 | Consent linkage  | Is user consent (checkbox or acknowledgment) tied to the notice during booking or messaging?                                          | ☐            |
| 1.4 | Language clarity | Is the privacy notice written in **plain English**, accessible to non-legal users?                                                    | ☐            |


The architectural sources provided offer detailed insights into the plugin's settings, data flow, email templates, and administrative features, but they **do not contain information** concerning the implementation of a user-facing **Privacy Policy/Notice** or associated consent mechanisms.

Therefore, based strictly on the provided material, we cannot confirm whether the options listed in your table are implemented in the plugin.

| # | Check Item | Description | Implemented? |
| :--- | :--- | :--- | :--- |
| **1.1** | Notice exists | Does the plugin provide or link to a **Privacy Policy / Notice** page? | **Unsupported by sources** |
| **1.2** | Notice placement | Is the notice clearly visible **before or at the time of data collection** (e.g., booking form, photo upload, or message submission)? | **Unsupported by sources** |
| **1.3** | Consent linkage | Is user consent (checkbox or acknowledgment) tied to the notice during booking or messaging? | **Unsupported by sources** |
| **1.4** | Language clarity | Is the privacy notice written in **plain English**, accessible to non-legal users? | **Unsupported by sources** |

### Summary of Relevant Architectural Details

While the sources do not discuss legal privacy compliance features for end-users, they extensively cover systems that *would* be relevant to such implementation, none of which explicitly include privacy controls:

1.  **Settings and Configuration:** The General Settings defined in `core/admin/api-settings.php` control fundamental features like the Calendar view, Legend display, Availability rules, and **Post-booking action (confirmation message or redirect URL)**. There is no mention of an option to link to or display a privacy policy within these settings.
2.  **Form Structure:** The mechanism for defining booking forms relies on a **custom, shortcode-like syntax** which is parsed by `core/form_parser.php`. The limitation of this parser is noted: the recognized field types are **hardcoded into the regular expression**, making it difficult to add new custom field types, such as a legally required consent checkbox, without modifying the core files.
3.  **Data Handling:** The plugin’s core architectural files establish methods for retrieving and storing booking data and custom booking metadata (via the `booking_options` serialized array in the custom `{$wpdb->prefix}booking` table). However, the documentation focuses on the *technical storage* rather than the *legal handling* or required user consent for that data.
4.  **Admin Notices vs. Privacy Notices:** The plugin features a sophisticated **WPBC\_Notices** class for persistent, dismissible warnings shown to the administrator (e.g., warnings about downgrading from a paid version). These administrative notices are distinct from a legal privacy notice intended for the public or site visitors at the time of data submission.
5.  **Internationalization (i18n):** The plugin has a robust translation engine (`core/wpbc-translation.php`) and date localization system (`core/wpbc_functions_dates.php`). If a privacy notice were implemented, this system would ensure it is displayed in the correct language. However, the content of any such notice is not analyzed.


Based on the detailed architectural and functional analyses provided in the sources, there is **no evidence** that the **Privacy Notice Availability** option has been implemented within the core components of the plugin.

Therefore, the marking is **1**.

### **Rating: 1** (The feature is not implemented at all.)

---

### **Justification from Architectural Sources**

The sources comprehensively detail administrative settings, asset loading, API integrations, and email workflows, yet they contain **no mention** of classes, settings fields, database columns, or extension hooks related to legal compliance, privacy policies, or explicit user consent mechanisms.

*   **Lack of Corresponding Settings (1.1, 1.2, 1.3):** The general settings file (`core/admin/api-settings.php`) defines fields for the Calendar view, Legend display, Availability rules, and **Post-booking action (confirmation message or redirect URL)**. There is no documented setting within the `WPBC_Settings_API_General` class that allows an administrator to link to, upload, or configure the display of a Privacy Notice or collect user consent.
*   **Form Extensibility Limitations (1.3):** Implementing mandatory consent (a checkbox or acknowledgment field) would require modifications to the booking form. The core logic for reading and parsing the booking form syntax resides in `core/form_parser.php`. A notable limitation of this file is that the list of supported field shortcode types is **hardcoded into the regular expression**. This rigidity makes it difficult to add new, custom field types, such as a legally required consent checkbox, without modifying the core parser code.
*   **Data Handling Focus:** The architecture focuses on the *technical storage* of booking metadata, which is stored as a single, serialized array in the `booking_options` column of the custom `{$wpdb->prefix}booking` table. While the data is stored, the sources do not specify the legal *consent* obtained for that storage.
*   **Localization (1.4 - Not Applicable):** While the plugin has a robust internationalization engine (`core/wpbc-translation.php`) capable of localizing text strings and displaying them in plain language, the absence of the underlying legal content itself makes the linguistic capability irrelevant for this check item.

This implementation overview draws on the architectural components detailed in the sources, focusing on integrating the "Privacy Notice Availability" check items (1.1–1.4) into the plugin’s existing frameworks.

The implementation requires modifications across three main architectural layers: the **Settings API** (for configuration), the **Front-end Rendering** (for placement and display), and the **Booking Workflow** (for validation and storage).

---

## High-Level Implementation Overview

### 1. Configuration: Implementing the Admin Settings (1.1, 1.4)

The first step is to provide the administrator with fields to define the policy content, placement, and enablement, leveraging the plugin's custom settings framework.

*   **Settings Definition:** A new section must be added to the General Settings page, defined in the `WPBC_Settings_API_General` class located in `core/admin/api-settings.php`. This class programmatically defines all configuration options within its `init_settings_fields()` method.
*   **New Fields Required:**
    *   **Privacy Notice Content (1.1, 1.4):** A large text area field to allow the administrator to input the actual privacy policy text or a hyperlink to the policy. This text field would use the core Internationalization (i18n) engine, utilizing the custom inline translation shortcode format, **[lang=xx\_XX]...[/lang]**, defined in `core/wpbc-translation.php`. This ensures the notice is displayed in **plain English** or the active user locale.
    *   **Consent Required (1.3):** A checkbox setting to enable or disable mandatory user consent.
    *   **Placement Logic (1.2):** A setting (e.g., a select dropdown) to control where the notice and checkbox should be dynamically injected into the booking form.
*   **Settings Storage:** These new options will be saved and managed using the plugin’s standard data abstraction layer functions like `update_bk_option()`, as defined in `core/wpbc-core.php`.

### 2. Front-End Integration: Display and Placement (1.2, 1.3)

The notice and the required consent field must be injected directly into the booking form, clearly visible **before** data submission.

*   **Form Injection:** The plugin centralizes form rendering logic within the `wpdev_booking` class (in `core/lib/wpdev-booking-class.php`). The safest method to integrate the notice is by hooking into a global form filter, such as **`wpdev_booking_form`**. This ensures the new content is injected consistently, whether the form is rendered via a shortcode **[booking]** or via the **BookingWidget**.
*   **Content Display (1.2):** The filter handler would retrieve the configuration (link/text) using **`get_bk_option()`** and generate the necessary HTML for the privacy notice, ensuring it is clearly visible.
*   **Consent Field Creation (1.3):** The implementation must inject a consent field (likely a checkbox). If the form parser (`core/form_parser.php`) does not support a dedicated "consent" field type, an existing, supported shortcode type (e.g., a simple checkbox) must be utilized to ensure the field’s data is collected upon submission.

### 3. Backend Workflow: Validation and Storage (1.3)

After the user submits the form via AJAX, the server must validate that consent was given and then store the user's acknowledgment with the booking record.

*   **AJAX Interception:** The booking submission is handled dynamically by the central AJAX router, `core/lib/wpbc-ajax.php`, which ensures security via nonce verification. The submission handling functions must be modified to intercept the request **before** the final `wpbc_booking_save()` call.
*   **Server-Side Validation (1.3):** If the administrator enabled the mandatory consent option in step 1, the AJAX validation logic must check if the injected consent field was checked/acknowledged. If consent is missing, the server should return an error response, preventing the final database insertion.
*   **Data Persistence:** If validation succeeds, the record of user consent must be saved. This can be achieved using the plugin’s dedicated booking metadata functionality, utilizing the function **`wpbc_save_booking_meta_option($booking_id, $option_arr)`** from `core/wpbc-core.php`. This function serializes the custom data and stores it in the **`booking_options`** column of the custom `{$wpdb->prefix}booking` table, which is the standard mechanism for storing per-booking custom fields.
*   **Post-Submission Hooks:** After successful booking creation, developers can leverage action hooks documented in the Developer API (`core/wpbc-dev-api.php`), such as **`wpbc_track_new_booking`** or **`wpbc_booking_approved`**, to trigger any necessary side effects related to privacy logging or external system compliance.



---

### **2. Information Disclosure Requirements**

| #   | Check Item             | Description                                                                                                               | Implemented? |
| --- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------- | ------------ |
| 2.1 | Data collected         | Lists all personal data collected: name, contact info, booking details, uploaded photos, message content.                 | ☐            |
| 2.2 | Purpose of processing  | Explains *why* data is collected — e.g., service delivery, communication, invoicing.                                      | ☐            |
| 2.3 | Lawful basis           | States the lawful bases for processing (e.g., contract, consent, legitimate interest).                                    | ☐            |
| 2.4 | Third-party disclosure | Lists third-party integrations (e.g., Twilio, SendGrid, WhatsApp API) and their roles as processors.                      | ☐            |
| 2.5 | Retention period       | Specifies how long booking records, photos, and messages are stored.                                                      | ☐            |
| 2.6 | Data location          | Indicates whether data is stored in the UK, EU, or third countries and describes safeguards (e.g., SCCs for US services). | ☐            |


Based on the detailed architectural analysis of the plugin's code, the sources extensively describe *what* data is collected and *how* it is processed and stored internally, but they **do not contain information** indicating the implementation of user-facing disclosures regarding data practices, lawful basis, third-party processors, retention periods, or data location.

Therefore, we cannot confirm that the **Information Disclosure Requirements** outlined in your table are implemented for the end-user.

| # | Check Item | Description | Implemented? |
| :--- | :--- | :--- | :--- |
| **2.1** | Data collected | Lists all personal data collected: name, contact info, booking details, uploaded photos, message content. | **Unsupported by sources** |
| **2.2** | Purpose of processing | Explains *why* data is collected — e.g., service delivery, communication, invoicing. | **Unsupported by sources** |
| **2.3** | Lawful basis | States the lawful bases for processing (e.g., contract, consent, legitimate interest). | **Unsupported by sources** |
| **2.4** | Third-party disclosure | Lists third-party integrations (e.g., Twilio, SendGrid, WhatsApp API) and their roles as processors. | **Unsupported by sources** |
| **2.5** | Retention period | Specifies how long booking records, photos, and messages are stored. | **Unsupported by sources** |
| **2.6** | Data location | Indicates whether data is stored in the UK, EU, or third countries and describes safeguards (e.g., SCCs for US services). | **Unsupported by sources** |

### Insights on Data Collection and Processing from Sources (Contextual Information)

While the sources do not document the *disclosure* of this information to the end-user, they provide architectural details confirming that the plugin actively collects and processes data related to all items listed in your table:

#### **Data Collected (2.1)**
The architectural files confirm that the plugin is designed to handle user-submitted personal and booking data:

*   **Custom Form Data:** The plugin uses a custom form-parsing utility (`core/form_parser.php`) to capture fields submitted by the user (which would include name, contact info, and message content).
*   **Booking Details:** Booking records are stored in custom database tables (e.g., `{$wpdb->prefix}booking`, `{$wpdb->prefix}bookingdates`). Custom fields and internal settings related to the booking are stored as a **single, serialized array** in the **`booking_options`** column of the custom `{$wpdb->prefix}booking` table.

#### **Purpose of Processing (2.2)**
The operational workflow confirms data is processed for service delivery and communication:

*   **Communication:** Data is retrieved to populate **email templates** (subject, content, styling) and send transactional emails to both administrators and visitors following booking status changes (e.g., New Booking, Approved, Deleted).
*   **Administration:** Data is queried dynamically for administrative display on the **Booking Listing**, **Timeline View**, and **Dashboard Widget**. Administrative emails include **quick management shortcodes** like `[click2approve]` and `[click2decline]`, indicating the data is processed to facilitate workflow management.

#### **Third-Party Disclosure (2.4)**
The architecture supports integration with external services, confirming the *possibility* of third-party data transfer:

*   **Google Calendar:** The plugin features a complex **one-way synchronization** system that involves communication with the Google Calendar API v3. The core engine, `WPBC_Google_Calendar`, uses `wp_remote_get()` to send requests that include the Google API Key (stored as `booking_gcal_api_key`) and the Calendar ID. This process directly relates to sharing configuration data (API key, Calendar ID) with Google.
*   **Other Integrations:** The sources mention advanced synchronization features (ICS Import/Export) are often delegated to the **"Booking Manager"** companion plugin, confirming that external processors are necessary for core features.

#### **Retention Period (2.5)**
While no specific retention duration is defined in the documentation provided, the plugin's lifecycle management architecture addresses data cleanup upon deactivation:

*   **Data Deletion Control:** The deactivation logic in `core/any/activation.php` respects the user-configurable option **`booking_is_delete_if_deactive`**. This confirms that the administrator has explicit control over whether booking data is retained or permanently deleted upon plugin deactivation.

As previously established in our conversation, the components necessary to provide these disclosures (like a settings field to input the policy, a method to display it, and a mandatory consent checkbox) are **not implemented** within the analyzed code.


Based on the comprehensive architectural documentation provided, there is **no evidence** that the "Information Disclosure Requirements" (Check Items 2.1 through 2.6) have been implemented as user-facing disclosures within the plugin.

The sources detail *how* the plugin collects and manages data internally, but they do not document any mechanism, setting, or user interface element dedicated to *disclosing* this information (purpose, lawful basis, third parties, retention) to the user.

Therefore, the feature is marked:

### **Rating: 1** (The feature is not implemented at all.)

---

### **Justification from Architectural Sources**

The plugin's architecture confirms the existence of systems relevant to data handling (e.g., data is collected, and third-party systems are utilized), but critical components necessary for legal disclosure are missing:

1.  **Lack of Disclosure Settings (2.1, 2.2, 2.3, 2.5, 2.6):** The core settings files, such as `core/admin/api-settings.php` (which defines the `WPBC_Settings_API_General` class), meticulously define configuration for features like the Calendar view, Legend display, and Availability rules. However, there is no documented setting that allows an administrator to define or input text regarding the data collected, the purpose of processing, the lawful basis, retention periods, or data location.
2.  **Confirmed Data Processing without Disclosure:**
    *   **Data Collected (2.1) & Purpose (2.2):** The plugin actively processes data to facilitate **communication** (via tailored email templates like `WPBC_Emails_API_NewAdmin` and `WPBC_Emails_API_Approved`) and **service delivery** (saving booking details). Custom form data is saved as a serialized array in the `booking_options` column of the database. However, the sources confirm the data is collected and used without documenting any user disclosure describing *what* is collected or *why*.
    *   **Third-party Disclosure (2.4):** The plugin architecture explicitly details interaction with external entities via its synchronization features. The Google Calendar feature uses the `WPBC_Google_Calendar` class to communicate with the Google Calendar API v3 using `wp_remote_get()`, utilizing stored settings like the `booking_gcal_api_key`. This confirms the potential for data transfer to a third party (Google). Despite this technical reality, there is no documented mechanism to inform the end-user (site visitor) about this third-party processor.
3.  **Retention Period Control vs. Disclosure (2.5):** The system provides the administrator with lifecycle control over booking data via the `booking_is_delete_if_deactive` option (defined in `core/any/activation.php`). This control relates to cleanup upon deactivation, but it is a backend option for the site owner, not a user-facing disclosure specifying the length of data retention.


The implementation of the Information Disclosure Requirements (Check Items 2.1–2.6) necessitates extending the plugin’s administrative settings, utilizing its robust localization engine, and integrating disclosures into the front-end rendering and data storage workflows.

Here is a high-level overview drawing on the plugin's core architectural components:

---

## High-Level Implementation Overview: Information Disclosures

### 1. Administrative Configuration: Defining the Disclosures (2.1, 2.2, 2.3, 2.5, 2.6)

The initial phase requires creating the backend interface for the administrator to input all necessary legal text, utilizing the plugin’s custom settings framework.

*   **Settings Blueprint:** A new section dedicated to "Privacy and Disclosure" must be defined in the General Settings architecture, specifically by modifying the logic within the relevant settings API file (e.g., `core/admin/api-settings.php`). This new section would conceptually extend the `WPBC_Settings_API_General` class,.
*   **Input Fields:** Multiple large text area fields must be implemented to allow the administrator to input the verbose content for each required disclosure:
    *   **Data Collected (2.1):** A list of collected fields (name, email, dates).
    *   **Purpose of Processing (2.2):** Why the data is used (e.g., service delivery, communication),.
    *   **Lawful Basis (2.3):** The legal justification for processing.
    *   **Retention Policy (2.5):** The duration for which booking data is stored, referencing the plugin's date and time utility logic found in files like `core/wpbc-dates.php` and `core/wpbc_functions_dates.php`,.
    *   **Data Location (2.6):** Storage location (UK/EU/Third Countries).
*   **Localization (1.4, 2.1-2.6):** To ensure clarity across locales, the input fields should utilize the plugin’s custom inline translation system. This involves wrapping the field output with the `wpbc_lang()` function, enabling the administrator to embed translations directly via the custom shortcode format `[lang=xx_XX]...[/lang]`,.
*   **Persistence:** All new settings would be saved via the plugin's data abstraction layer wrappers, such as `update_bk_option()`.

### 2. Synchronization Integration: Third-Party Disclosures (2.4)

Since the plugin supports third-party synchronization, disclosure information related to external processors must be linked to the relevant administrative settings.

*   **Linking Disclosures:** The configuration fields for synchronization (e.g., on the Google Calendar sync page managed by `core/sync/wpbc-gcal.php`), should be updated.
*   **Contextual Display:** The UI helper functions for Google Calendar settings (`wpbc_gcal_settings_content_field_*()`), should be modified to display the third-party disclosure text (retrieved from Step 1) whenever the administrator is configuring or activating external synchronization. This emphasizes the data transfer to entities like Google Calendar.
*   **Third-Party Logging:** If the paid version is active, the synchronization logic in `wpbc_silent_import_all_events()`, (which queries the `wp_bookingtypes` table and loops through multiple resources) must also ensure that the audit logs confirm the administrator has acknowledged the third-party processor for each resource feed.

### 3. Front-End Display and Acknowledgment (1.2, 1.3, 2.1-2.6)

The comprehensive disclosures (from Step 1) must be displayed, and explicit user consent must be obtained.

*   **Content Injection (1.2):** The plugin’s core rendering controller, the `wpdev_booking` class (in `core/lib/wpdev-booking-class.php`), which handles shortcode rendering, must be targeted. A rendering filter, such as `wpdev_booking_form`,, would be used to retrieve the stored disclosure texts (all of 2.1–2.6) and inject them into the form HTML, ensuring visibility **before** submission.
*   **Consent Mechanism (1.3):** A mandatory consent field (e.g., a checkbox) must be inserted alongside the disclosure text. Since the form parser (`core/form_parser.php`) is noted as having hardcoded fields,, this consent field should either be injected directly via the rendering filter or require an extension of the parser using the documented filtering mechanism (if available).

### 4. Data Workflow and Persistence

*   **Validation:** The central AJAX request router (`core/lib/wpbc-ajax.php`), handles the real-time submission of booking data. The request processing logic must be modified to perform server-side validation, checking that the user's explicit consent field (from Step 3) was present and checked before allowing the booking to proceed.
*   **Audit Trail and Storage:** Upon successful validation, the record of user consent must be stored with the booking itself. This should utilize the core functionality for handling custom data via the **Booking Meta Functions**. The function `wpbc_save_booking_meta_option( $booking_id, $option_arr )` would be called immediately after the booking record is created, saving the acknowledgment (and potentially the timestamp or version of the policy) into the non-queryable `booking_options` serialized column of the `{$wpdb->prefix}booking` table,.
*   **Post-Submission Integration:** Developers can use the plugin's dedicated event hooks, such as **`wpbc_track_new_booking`** or **`wpbc_booking_approved`**, which are documented in the Developer API (`core/wpbc-dev-api.php`),, to trigger external logging or auditing systems related to the successful consent capture.


---

### **3. User Rights & Controls**

| #   | Check Item        | Description                                                                            | Implemented? |
| --- | ----------------- | -------------------------------------------------------------------------------------- | ------------ |
| 3.1 | Access rights     | Users can request a copy of their booking/photos/messages data.                        | ☐            |
| 3.2 | Correction rights | Users can correct inaccurate information.                                              | ☐            |
| 3.3 | Deletion rights   | Users can request deletion of their data (subject to business retention requirements). | ☐            |
| 3.4 | Objection rights  | Users can object to certain processing (e.g., marketing messages).                     | ☐            |
| 3.5 | Contact info      | Notice provides a **contact method for data requests** (e.g., DPO email).              | ☐            |

The provided sources, which detail the plugin's administrative workflow, APIs, and data handling, **do not confirm the implementation** of user-facing features enabling the formal exercise of data access, correction, deletion, or objection rights, nor do they document a contact method for such requests.

The plugin architecture focuses on **administrator control** over data management, not user self-service or formal compliance disclosures.

| # | Check Item | Description | Implemented? |
| :--- | :--- | :--- | :--- |
| **3.1** | Access rights | Users can request a copy of their booking/photos/messages data. | **Unsupported by sources** |
| **3.2** | Correction rights | Users can correct inaccurate information. | **Inferred, but undocumented as a formal right** |
| **3.3** | Deletion rights | Users can request deletion of their data (subject to business retention requirements). | **Unsupported by sources** |
| **3.4** | Objection rights | Users can object to certain processing (e.g., marketing messages). | **Unsupported by sources** |
| **3.5** | Contact info | Notice provides a **contact method for data requests** (e.g., DPO email). | **Unsupported by sources** |

### Detailed Analysis of User Rights & Controls

#### 3.1 Access Rights and 3.3 Deletion Rights (Unsupported)
The sources confirm that the plugin's data management and lifecycle systems are oriented toward the administrator's control, offering no evidence of user-initiated data retrieval or deletion:

*   **Data Deletion:** The administrator has explicit control over data retention via the **`booking_is_delete_if_deactive`** option, which controls whether data is deleted upon plugin deactivation. Furthermore, administrative AJAX endpoints are defined to allow the admin to trash or permanently delete bookings (`wpbc_ajax_TRASH_RESTORE`, `wpbc_ajax_DELETE_APPROVE`). There is no documented mechanism for a non-admin user to request or execute deletion.
*   **Data Access/Export:** While the Developer API (`core/wpbc-dev-api.php`) includes functions like `wpbc_api_get_booking_by_id()` and `wpbc_api_get_bookings_arr()` (deprecated), these are tools for third-party developers, not mechanisms for an end-user to request a copy of their data.

#### 3.2 Correction Rights (Inferred Feature)
The sources provide limited evidence suggesting a user *might* be able to edit a booking, though this is primarily documented as a premium feature enhancement rather than a universal correction right:

*   The email template sent to visitors following a new booking (`core/admin/page-email-new-visitor.php`) can include the dynamic shortcode **`[visitorbookingediturl]`**. This shortcode, noted as being available "for premium users", strongly implies the existence of a front-end URL that allows a visitor to correct submitted information.
*   However, the sources do not detail the user experience or workflow for this link, making it impossible to confirm if it meets the requirements of a formal correction right (e.g., correcting only inaccurate fields without changing resource/dates, or receiving confirmation of correction).

#### 3.4 Objection Rights (Unsupported)
While the ability to implement an opt-out mechanism exists for developers, the user-facing right is not documented:

*   The Email API (`core/wpbc-emails.php`) contains the filter **`wpbc_email_api_is_allow_send`**, which acts as a "central 'kill-switch'" that developers can use to programmatically prevent emails from being sent. This is an extension point to *implement* objection logic, but there is no default, user-facing mechanism (like a checkbox or preference center) for a user to express their objection to processing.

#### 3.5 Contact Info (Unsupported)
The General Settings file (`core/admin/api-settings.php`), which defines the settings catalog, does not document a field for providing public contact information for data requests (like a DPO email). The existing administrative notice systems are designed for administrative warnings (e.g., downgrade notice), not public privacy disclosures.



Based on the comprehensive architectural review of the plugin's core files, the rating for the implementation of **User Rights & Controls** (Check Items 3.1–3.5) is **2**.

### **Rating: 2** (The feature is minimally supported but not fully implemented as formal user rights.)

---

### **Justification from Architectural Sources**

The rating of **2** is assigned because the plugin architecture demonstrates **no explicit implementation** of user-facing systems for accessing (3.1), deleting (3.3), objecting to (3.4), or requesting contact information (3.5) for data rights. However, a specific component indicates that **correction rights (3.2)** may be implemented as a premium feature.

#### **Check Item 3.2: Correction Rights (Minimal, Premium Implementation Inferred)**
The file `core/admin/page-email-new-visitor.php` defines the email template sent to the visitor after a new booking. This template includes the dynamic shortcode **`[visitorbookingediturl]`**, which is explicitly noted as being available **"for premium users"**.

The existence of a dedicated shortcode that generates a URL for editing a booking strongly suggests that the functionality to allow a user to **correct inaccurate information (3.2)** has been implemented, although it is restricted to paid versions of the plugin.

#### **Check Items 3.1, 3.3, 3.4, 3.5: Access, Deletion, Objection, and Contact Info (Not Implemented)**

The sources reveal a strong focus on *administrator control* over data, with no corresponding documentation for user-initiated rights mechanisms:

*   **Access (3.1) & Deletion (3.3):** All documented data management tools are admin-facing. The AJAX router (`core/lib/wpbc-ajax.php`) handles admin actions for trashing, restoring, and permanently **deleting bookings**. Similarly, database access functions (e.g., `wpbc_api_get_booking_by_id()` in `core/wpbc-dev-api.php`) are part of the Developer API, intended for programmatic data retrieval by external developers, not end-user access requests.
*   **Objection (3.4):** While the Email API (`core/any/api-emails.php`) contains the filter `wpbc_email_api_is_allow_send` which acts as a "global 'kill switch'" to programmatically prevent emails from being sent, this is a developer extension point, not a user-facing mechanism for objecting to marketing or processing.
*   **Contact Information (3.5):** The administrative settings catalog, which defines the complete configuration blueprint, contains no documented fields for storing or displaying DPO/data request contact information.

The implementation of **User Rights & Controls** (Access, Correction, Deletion, and Objection) requires integrating administrative configuration, creating secure new AJAX endpoints, formalizing existing front-end features, and leveraging the plugin's internal hook system.

This high-level overview details how these requirements could be implemented using the existing plugin architecture.

---

## High-Level Implementation Overview: User Rights & Controls

### 1. Administrative Configuration (3.5 Contact Info & 3.4 Objection Enabling)

New settings must be added to the administrative interface to define the necessary contact information and enable the objection/opt-out mechanism.

*   **Define Contact Setting (3.5):** A new text field for the **Data Protection Officer (DPO) email address** or data request contact phone number must be added to the General Settings page. This configuration blueprint is managed by defining the field within the `init_settings_fields()` method of the **`WPBC_Settings_API_General`** class, located in `core/admin/api-settings.php`. This field, once saved via wrappers like `update_bk_option()`, makes the contact information retrievable via `get_bk_option()` for display in the privacy notice.
*   **Define Objection/Marketing Consent Setting (3.4 Enabling):** A new checkbox or switch should be added to the General Settings to enable or disable the display of the marketing objection field on the booking form.

### 2. User Access and Deletion Workflow (3.1 Access & 3.3 Deletion)

A formal, secure mechanism must be created for non-admin users to request or execute their rights.

*   **Front-End Access Interface:** A new shortcode (e.g., `[user_data_request]`) must be registered by the **`wpdev_booking`** class to display a dedicated page or form. This form would require the user to input identifying information (like email address and booking ID/hash) to initiate a request for data access (export) or deletion.
*   **Secure AJAX Endpoints (3.1 & 3.3):** Since these actions involve database interaction and potential data destruction, they must be processed via the **AJAX router** (`core/lib/wpbc-ajax.php`). New handler functions (`wpbc_ajax_USER_DATA_ACCESS`, `wpbc_ajax_USER_DATA_DELETE`) must be registered using the **`wpbc_ajax_action_list` filter**.
*   **Access Processing (3.1):** The new AJAX handler would retrieve the user's booking details using the established **Developer API**. Specifically, functions like **`wpbc_api_get_booking_by_id()`** would fetch the record, and the output would need to unserialize the custom data stored in the `booking_options` column of the custom booking table. The API is recommended as it shields the developer from the plugin's complex internal data structures.
*   **Deletion Processing (3.3):** The deletion handler would perform raw **`$wpdb`** queries against the custom booking tables (`{$wpdb->prefix}booking`) after verifying ownership. Crucially, the handler must fire the relevant cleanup hooks, such as **`wpbc_booking_delete`**, to trigger any necessary side effects or logging before permanently deleting the records.

### 3. Correction Rights Formalization (3.2 Correction)

The existing inferred functionality to allow users to edit their bookings must be integrated and formalized as a correction right.

*   **Formalize Premium Link:** The existing shortcode **`[visitorbookingediturl]`**, found in the visitor email template (`core/admin/page-email-new-visitor.php`), implies a working front-end edit page. This system must be formalized to clearly present the relevant booking data (retrieved via the Developer API) within the booking form rendering context.
*   **AJAX Update:** The form submission on the correction page must leverage the existing, secure AJAX endpoints for updating booking records defined in `core/lib/wpbc-ajax.php`.

### 4. Objection Rights Enforcement (3.4 Objection)

Enforcement primarily involves respecting the user's consent choice (collected in the front-end privacy form) at the point of email dispatch.

*   **Store User Preference:** When the user submits the booking form, the status of the objection field (e.g., marketing opt-out) must be captured and stored as custom metadata. This uses the metadata functions like **`wpbc_save_booking_meta_option($booking_id, $option_arr)`**, saving the setting into the `booking_options` serialized column of the booking record.
*   **Email Suppression Logic:** The Email API (`core/wpbc-emails.php` and `core/any/api-emails.php`) provides a global "kill switch" filter called **`wpbc_email_api_is_allow_send`**. A custom function must hook into this filter.
*   **Execute Objection:** The filter function must perform the following check:
    1. Retrieve the booking ID associated with the email being sent.
    2. Retrieve the user's stored objection preference via `wpbc_get_booking_meta_option()`.
    3. If the email is a marketing or non-essential communication AND the user has objected, the filter must return `false`, programmatically **preventing the email from being sent**.
 

---

### **4. Technical Transparency**

| #   | Check Item          | Description                                                                                            | Implemented? |
| --- | ------------------- | ------------------------------------------------------------------------------------------------------ | ------------ |
| 4.1 | Data flow clarity   | Does the plugin document **how data flows** between the booking system and external integrations?      | ☐            |
| 4.2 | API processors      | Identifies external processors like Twilio (SMS), SendGrid (email), WhatsApp Business API (messaging). | ☐            |
| 4.3 | Data minimization   | Confirms that only necessary data fields are sent to each third-party.                                 | ☐            |
| 4.4 | Processor contracts | Mentions that appropriate **Data Processing Agreements (DPAs)** exist for integrations.                | ☐            |

Based on the comprehensive architectural and functional analyses provided in the sources, there is **no evidence** that the **Technical Transparency** requirements, which involve user-facing documentation or legal statements about data handling, are implemented in the plugin.

The sources detail *how* external data communication works but do not document any feature that communicates this information to the end-user or site administrator as a measure of transparency.

| # | Check Item | Description | Implemented? |
| :--- | :--- | :--- | :--- |
| **4.1** | Data flow clarity | Does the plugin document **how data flows** between the booking system and external integrations? | **Unsupported by sources** |
| **4.2** | API processors | Identifies external processors like Twilio (SMS), SendGrid (email), WhatsApp Business API (messaging). | **Partially relevant, but unsupported as a disclosure** |
| **4.3** | Data minimization | Confirms that only necessary data fields are sent to each third-party. | **Unsupported by sources** |
| **4.4** | Processor contracts | Mentions that appropriate **Data Processing Agreements (DPAs)** exist for integrations. | **Unsupported by sources** |

### Detailed Analysis of External Integrations

While the transparency documentation (4.1-4.4) is absent, the sources clearly outline the plugin's interaction with external processors, which is highly relevant to this category:

#### **External API Processors (4.2)**

The plugin architecture explicitly supports and details integration with at least one major external API:

*   **Google Calendar:** The synchronization feature is a core component, managed by the **`WPBC_Google_Calendar`** class. The system makes requests to the Google Calendar API v3 using the standard WordPress function **`wp_remote_get()`**. This process involves sharing configuration data, such as the Google API Key (stored as `booking_gcal_api_key`), and transferring booking resource IDs.
*   **Email System:** The plugin employs a central Email API (`WPBC_Emails_API`) that sends transactional notifications via a wrapper around **`wp_mail()`**. While the sources do not mention specific email service providers like SendGrid, any transactional email relies on an external mail server or service configured by the administrator.

#### **Data Flow Clarity and Minimization (4.1, 4.3)**

The sources describe the *technical implementation* of data flow, but not a *transparency disclosure* for the end-user:

*   **Data Flow:** The flow is documented architecturally for developers. For instance, the Google Calendar synchronization is a **one-way process** designed to **import events** from GCal into the local database as bookings, directly affecting front-end availability.
*   **Data Minimization:** During the Google Calendar import, the plugin processes JSON data and uses the administrator's **field mapping configuration** (`booking_gcal_events_form_fields`) to assign event details (title, description) to local booking fields. This ensures only the necessary data *from* GCal is saved *locally*, but there is no documentation confirming minimization when data is transferred *to* a third-party, such as Google, or ensuring only necessary data fields are sent to third parties (4.3).

#### **Processor Contracts (4.4)**

The sources, which are technical analyses of code architecture, API design, and scheduling systems, do not contain any reference to legal requirements, policies, or the existence of **Data Processing Agreements (DPAs)**.


Based on the detailed architectural analysis of the plugin's code, there is **no evidence** that the **Technical Transparency** requirements (Check Items 4.1–4.4) have been implemented as user-facing or administrative disclosures.

Therefore, the marking is **1**.

### **Rating: 1** (The feature is not implemented at all.)

---

### **Justification from Architectural Sources**

The sources confirm that the plugin uses external integrations, making transparency disclosures relevant, but they provide no implementation details for the required communication or legal mechanisms:

1.  **Lack of Disclosure Settings (4.1, 4.3, 4.4):** The General Settings (managed by `WPBC_Settings_API_General`), email templates (managed by `WPBC_Emails_API`), and synchronization configuration files (e.g., `core/sync/wpbc-gcal.php`) define technical parameters (like API keys, schedules, or resource IDs) but **do not include any fields** that allow an administrator to input text regarding data flow, data minimization policies, or the existence of **Data Processing Agreements (DPAs)** (4.4).
2.  **API Identification is Technical, Not Disclosed (4.2):**
    *   The plugin architecture confirms interaction with external processors, most notably the **Google Calendar API v3**, which handles the one-way synchronization process.
    *   The Email API (`core/wpbc-emails.php`) uses a wrapper function, `wpbc_wp_mail()`, around the WordPress `wp_mail()` function, which relies on an external mail server or processor.
    *   However, the documentation focuses exclusively on the *technical implementation* of these integrations (using `wp_remote_get()`, error handling), not the mandated *disclosure* of the processor's identity or role to the end-user.
3.  **Data Minimization Focus is Internal (4.3):** While the synchronization process is described, the documentation only confirms that data mapping is utilized when importing data *from* Google Calendar. There is no documented implementation that confirms or discloses that only "necessary data fields are sent to each third-party" (4.3).

The implementation of Technical Transparency (disclosing data flow, processors, minimization, and contracts) requires the creation of structured data fields in the administrative backend and integrating the display of this information into the existing configuration and front-end rendering architecture.

This high-level overview leverages the plugin’s dedicated **Settings API**, **Synchronization Controller**, and **Email API** to meet requirements 4.1 through 4.4.

---

## High-Level Implementation Overview: Technical Transparency

### 1. Configuration: Defining Disclosures in the Admin Panel (4.1, 4.2, 4.4)

The process begins by using the plugin's custom settings framework to allow the administrator to input the required legal and technical information.

*   **Settings Blueprint Modification:** A new section dedicated to "Technical Disclosures" must be added to the General Settings. This requires modifying the crucial **`init_settings_fields()`** method within the **`WPBC_Settings_API_General`** class, defined in `core/admin/api-settings.php`.
*   **New Fields Required:** Using the standardized rendering methods of the abstract **`WPBC_Settings_API`**, several large text area fields should be added:
    *   **Processor Identification (4.2, 4.4):** Fields to list all third-party integrations (e.g., Google Calendar, inferred payment/SMS services) and input text confirming the existence of **Data Processing Agreements (DPAs)** (4.4) for those integrations.
    *   **Data Flow Description (4.1):** A descriptive field explaining how personal data flows to and from these external systems (e.g., the one-way synchronization process with the Google Calendar API v3).
    *   **Minimization Statement (4.3):** A statement outlining the principle of data minimization, confirming that only the necessary fields are transmitted to external processors.
*   **Localization (Plain English):** All input fields must utilize the plugin’s robust localization engine, specifically the custom **`[lang=xx_XX]...[/lang]`** shortcode syntax parsed by **`wpbc_lang()`**. This ensures the disclosures meet the requirement for clarity (plain English or the user's locale).
*   **Settings Persistence:** The entered disclosure content would be stored in the database as plugin options via the wrappers **`update_bk_option()`**.

### 2. Integration: Contextualizing Disclosure and Minimization (4.2, 4.3)

Disclosures related to specific external services should be visible to the administrator when they configure those services, and confirmation of data minimization must be implicit in the technical workflow.

*   **Google Calendar Contextual Notice (4.2, 4.4):** The configuration page for Google Calendar synchronization (`core/sync/wpbc-gcal.php`) already uses UI helper functions (like `wpbc_gcal_settings_content_field_*()`) to render settings. These functions should be updated to retrieve and display the relevant Processor/DPA disclosure content (from Step 1) directly on the sync page.
*   **Data Minimization Principle (4.3):** The sources note that the Google Calendar integration already uses an **"admin-configured field mapping"** (`booking_gcal_events_form_fields`) to map event details to *local* booking fields. This mapping setting provides a control point that should be referenced in the disclosure, confirming the administrator actively controls what data is processed.
*   **Custom Hook Interception (4.1, 4.3):** Since background tasks like Google Calendar import are executed by the custom cron scheduler via **`wpbc_silent_import_all_events()`**, developers can use the filter **`wpdev_bk_get_option`** to intercept settings used during automated cron runs. This allows programmatic checks to ensure essential privacy settings are active immediately before external API calls (e.g., `wp_remote_get()` used by the `WPBC_Google_Calendar` class) are made.

### 3. Presentation: Displaying Disclosures to the User

The collected information must be integrated into user-facing communication channels.

*   **Privacy Notice Integration:** The disclosure text fields (4.1–4.4) should be included as mandatory content within the overarching **Privacy Notice** implementation (as discussed in previous steps), ensuring users see all required information before submitting their data.
*   **Email Transparency:** If external email processors (like SendGrid) are used, the transactional emails sent via the **`WPBC_Emails_API`** should include relevant information. This can be achieved by injecting the transparency shortcodes into the email template content fields (defined in files like `page-email-new-visitor.php`), leveraging the **`wpbc_replace_params_for_booking`** filter.
*   **Debugging Tool Usage:** Developers can optionally use the global debugging utility **`wpbc_admin_show_top_notice()`** defined in `core/wpbc-debug.php` to dynamically display warnings to administrators if critical disclosure fields have been left empty.




---

### **5. Consent & Record Keeping**

| #   | Check Item                   | Description                                                                            | Implemented? |
| --- | ---------------------------- | -------------------------------------------------------------------------------------- | ------------ |
| 5.1 | Consent mechanism            | Clear opt-in checkbox (not pre-ticked) for marketing consent.                          | ☐            |
| 5.2 | Separate operational consent | Operational messages (e.g., booking confirmations) are excluded from marketing opt-in. | ☐            |
| 5.3 | Consent log                  | Plugin or system stores timestamp and IP of consent for each user.                     | ☐            |
| 5.4 | Withdrawal method            | Users can easily withdraw consent via profile or unsubscribe link.                     | ☐            |

Based on the comprehensive architectural review of the plugin's core files, the requirements for formal **Consent & Record Keeping** (Check Items 5.1–5.4) are **not explicitly implemented** as standard, ready-to-use features.

The sources provide strong foundational components that *could* be used to build these features, but they do not confirm the existence of user-facing consent fields, specific logging mechanisms for consent, or formal withdrawal methods.

| # | Check Item | Description | Implemented? |
| :--- | :--- | :--- | :--- |
| **5.1** | Consent mechanism | Clear opt-in checkbox (not pre-ticked) for marketing consent. | **Unsupported by sources** |
| **5.2** | Separate operational consent | Operational messages (e.g., booking confirmations) are excluded from marketing opt-in. | **Inferred from architecture, but unsupported as a formalized separation** |
| **5.3** | Consent log | Plugin or system stores timestamp and IP of consent for each user. | **Unsupported by sources** |
| **5.4** | Withdrawal method | Users can easily withdraw consent via profile or unsubscribe link. | **Unsupported by sources** |

### Detailed Analysis and Architectural Context

#### 5.1 Consent Mechanism (Marketing Opt-in Checkbox)
The sources do not document a pre-built marketing consent checkbox or an administrative setting to enable one:

*   **Form Structure Limitation:** The core logic for parsing booking forms is handled by `core/form_parser.php`. The analysis notes that the list of recognized shortcode field types is **hardcoded into the regular expression**, making it difficult to introduce a new, legally specific "consent" field type without modifying the core parser.
*   **Settings Absence:** The settings blueprint (`core/admin/api-settings.php`), which defines the complete catalog of general configuration options, does not include a field for enabling or configuring a marketing consent checkbox.

#### 5.2 Separate Operational Consent (Inferred but Not Formalized)
The plugin's architecture strongly supports separating marketing messages (if they were implemented) from operational messages, but the formal legal distinction is not documented:

*   **Transactional Email Segregation:** The plugin uses a highly modular and dedicated Email API pattern. Each email type (e.g., **Approved**, **Deleted**, **New Booking (Visitor)**) has its own dedicated class (e.g., `WPBC_Emails_API_Approved`, `WPBC_Emails_API_NewVisitor`). This modularity ensures that the critical operational confirmation messages are structurally separate from any non-operational messaging logic.
*   **Global Email Kill Switch:** The core `WPBC_Emails_API` abstract class provides the filter **`wpbc_email_api_is_allow_send`**. This filter acts as a "global 'kill switch'" that can be programmatically used by a developer to prevent *any* email from being sent based on custom logic (e.g., checking for consent status).

#### 5.3 Consent Log (Timestamp and IP)
The system has the technical capability to store custom data for each booking, but there is no evidence that it currently logs consent timestamps or IP addresses:

*   **Metadata Storage Mechanism:** The plugin uses specific functions like **`wpbc_save_booking_meta_option()`** to store custom booking metadata. This data is saved as a **single, serialized array** in the **`booking_options`** column of the custom `{$wpdb->prefix}booking` database table. This mechanism could be leveraged to store a consent timestamp and IP address, but the sources do not indicate that this is currently implemented.

#### 5.4 Withdrawal Method (Unsubscribe Link / Profile)
There is no documented user-facing mechanism for withdrawing consent:

*   **Lack of Profile Management UI:** The plugin focuses on shortcode rendering for booking forms and admin-side management, and it does not document a user-facing profile or account management interface where preferences could be updated.
*   **Correction Rights Implication:** While the admin email includes a `[visitorbookingediturl]` shortcode (for premium users) that implies a visitor can edit their booking, this feature is for **correcting inaccurate information** (Correction Rights 3.2), not for withdrawing consent or objecting to future processing (Objection Rights 3.4).

The implementation of **Consent & Record Keeping** (Check Items 5.1–5.4) is not explicitly present in the plugin's documented features. However, the plugin’s robust architecture provides tools that partially satisfy one requirement (5.2) and the necessary foundations to build the others.

Therefore, the feature implementation is rated **2**.

### **Rating: 2** (Minimal architectural foundations are present for separation of concerns, but user-facing consent, logging, and withdrawal mechanisms are absent.)

---

### **Justification from Architectural Sources**

#### **Not Implemented (Rating = 1 Contribution): 5.1, 5.3, 5.4**

*   **Consent Mechanism (5.1):** The architectural sources **do not document** the existence of a clear, non-pre-ticked marketing opt-in checkbox in the General Settings or booking form. Implementing such a field would require overcoming a noted architectural limitation: the booking form parser (`core/form_parser.php`) has supported shortcode types **hardcoded into the regular expression**, making it difficult to add new field types like a dedicated consent checkbox without modifying the core file.
*   **Consent Log (5.3):** While the plugin has the **technical capability** to store custom data for each booking using `wpbc_save_booking_meta_option($booking_id, $option_arr)`, this function stores the data as a **single, serialized array** in the `booking_options` column of the custom database table. There is no documentation or code analysis indicating that the plugin currently implements the logic to automatically capture and store the specific timestamp and IP address required for formal consent logging.
*   **Withdrawal Method (5.4):** There is **no documented mechanism** for users to withdraw consent, such as a self-service profile UI or an unsubscribe link. The inferred `[visitorbookingediturl]` shortcode, available to premium users, is designed for **correcting inaccurate booking information** (Correction Rights 3.2), not for managing ongoing consent preferences (Objection Rights 3.4).

#### **Architecturally Supported (Rating Increase): 5.2**

*   **Separate Operational Consent (5.2):** The plugin's architecture strongly supports the **separation of transactional emails** (operational messages) from any potential marketing emails.
    *   **Modular Email Classes:** The Email API uses highly modular, dedicated classes for each notification type (e.g., `WPBC_Emails_API_NewVisitor` for confirmation, `WPBC_Emails_API_Approved` for status change). This structural separation ensures operational emails are distinct.
    *   **Global Kill Switch:** The abstract `WPBC_Emails_API` class provides the filter **`wpbc_email_api_is_allow_send`**. This acts as a powerful "global 'kill switch'" that can be utilized by developers to programmatically prevent certain non-operational emails from being sent based on a user's stored consent status. This feature provides the necessary enforcement point if marketing consent were implemented.
 
The implementation of **Consent & Record Keeping** (Check Items 5.1–5.4) must leverage the plugin's core components: the custom **Settings API** for configuration, the **Shortcode/Form rendering logic** for display, the **Metadata functions** for record keeping, and the **Email API** for enforcement.

This implementation is complex because it requires injecting a new field into a system where form field types are traditionally **hardcoded** in `core/form_parser.php`.

---

## High-Level Implementation Overview: Consent & Record Keeping

### 1. Configuration: Enabling and Defining Consent Settings (5.1, 5.4)

The administrator must be able to enable the marketing opt-in requirement and define the method for withdrawal.

*   **Settings Definition:** New configuration options must be defined within the General Settings blueprint, which is managed by the concrete class **`WPBC_Settings_API_General`** located in `core/admin/api-settings.php`.
*   **New Fields:** Add fields to control:
    *   **Marketing Consent Toggle (5.1):** A checkbox to enable or disable the display and mandatory requirement of the marketing opt-in field.
    *   **Withdrawal Method URL (5.4):** A text field or editor area where the administrator can provide instructions or a link (e.g., to a "Manage Preferences" page) for withdrawing consent. This content should utilize the localization system (e.g., `wpbc_lang()` or `[lang=xx_XX]...[/lang]` shortcodes) to ensure clarity across languages.
*   **Settings Persistence:** The configured preferences are saved as options using data abstraction wrappers like `update_bk_option`.

### 2. Collection: Form Injection and Validation (5.1)

A clear, non-pre-ticked opt-in checkbox for marketing consent must be integrated into the booking submission form.

*   **Checkbox Injection (5.1):** Since the `core/form_parser.php` is difficult to extend, the safest way to include the checkbox is by hooking into the final form rendering filter, such as **`wpdev_booking_form`**, which is used by the main `wpdev_booking` class. The handler function must inject the required, non-pre-ticked HTML checkbox field and its accompanying disclosure text **before** submission.
*   **Server-Side Validation:** The booking submission is handled dynamically by the central **AJAX router** (`core/lib/wpbc-ajax.php`). The handler logic must check the submitted POST data:
    *   Validate that the technical/operational consent (tied to the privacy notice, as implemented in previous steps) is present.
    *   If the admin has enabled marketing consent, validate that the marketing opt-in checkbox was specifically checked. Failure to validate should return an error, preventing the booking save.

### 3. Record Keeping: Logging Consent (5.3)

Formal records of consent must be stored securely with the booking data.

*   **Metadata Storage:** The data must be recorded immediately after the booking is saved. This utilizes the plugin’s dedicated metadata management functions in **`core/wpbc-core.php`**.
*   **Log Data:** The function **`wpbc_save_booking_meta_option( $booking_id, $option_arr )`** would be used to store a serialized array in the `booking_options` column of the custom booking database table. This array must contain:
    *   The status of the marketing opt-in (Yes/No).
    *   The **timestamp** of consent.
    *   The user's **IP address** at the time of submission (which requires access to the server environment data during the AJAX call).

### 4. Enforcement and Management (5.2, 5.4)

The system must ensure that consent status is respected when sending emails and provide a functional mechanism for withdrawal.

*   **Email Separation and Enforcement (5.2):** The plugin's modular Email API uses distinct classes for transactional messages (e.g., `WPBC_Emails_API_Approved`) and is structurally separate from potential marketing messages. Enforcement is achieved using the global email "kill switch" filter:
    *   The filter **`wpbc_email_api_is_allow_send`** (defined in `core/any/api-emails.php`) must be hooked into.
    *   The hooked function must read the stored consent metadata (5.3). If the email type is defined as "marketing" AND the user did not opt-in, the filter returns `false`, preventing the message from being dispatched.
    *   Operational messages (e.g., booking confirmations) should be explicitly **excluded** from this marketing opt-in requirement, respecting the inferred distinction between template types.
*   **Withdrawal Method (5.4):** A custom AJAX endpoint must be implemented to process consent withdrawal:
    *   A new action, such as `wpbc_ajax_WITHDRAW_CONSENT`, must be securely registered using the **`wpbc_ajax_action_list`** filter in `core/lib/wpbc-ajax.php`.
    *   The handler would securely identify the user (e.g., via a unique hash included in the withdrawal link) and update the corresponding consent record in the booking metadata via `wpbc_save_booking_meta_option()`.
 

---

### **6. Display & Admin Configuration**

| #   | Check Item           | Description                                                                                    | Implemented? |
| --- | -------------------- | ---------------------------------------------------------------------------------------------- | ------------ |
| 6.1 | Configurable text    | Admin can customize the privacy notice text and link within plugin settings.                   | ☐            |
| 6.2 | Contextual display   | Different notices shown depending on context — e.g., booking, photo upload, or messaging page. | ☐            |
| 6.3 | Version control      | Keeps track of notice versions (important for proving compliance).                             | ☐            |
| 6.4 | Multilingual support | If site uses WPML or Polylang, privacy notices are translatable.                               | ☐            |

Based on the detailed architectural analysis of the plugin's components, there is **no evidence** that any configuration options for a user-facing **Privacy Notice** are implemented.

While the plugin is built on highly modular systems that possess the architectural capability for multilingual text input (6.4), the core feature enabling the administrator to define, track, or display a privacy notice remains unsupported in the documentation.

| # | Check Item | Description | Implemented? |
| :--- | :--- | :--- | :--- |
| **6.1** | Configurable text | Admin can customize the privacy notice text and link within plugin settings. | **Unsupported by sources** |
| **6.2** | Contextual display | Different notices shown depending on context — e.g., booking, photo upload, or messaging page. | **Unsupported by sources** |
| **6.3** | Version control | Keeps track of notice versions (important for proving compliance). | **Unsupported by sources** |
| **6.4** | Multilingual support | If site uses WPML or Polylang, privacy notices are translatable. | **Architecturally capable, but unsupported for this feature** |

### Detailed Analysis by Check Item

#### 6.1 Configurable Text (Unsupported)
The configuration blueprint for the plugin's General Settings is defined in the `WPBC_Settings_API_General` class. The sources confirm this class defines settings for features like **Calendar view**, **Legend display**, **Availability rules**, and **Post-booking action**. There is no documented setting within this structure that provides a field for the administrator to input or link to the text of a Privacy Policy or Notice.

#### 6.2 Contextual Display (Unsupported as a Privacy Feature)
The plugin has sophisticated architectural systems for displaying dynamic content based on context, but they are focused on administrative management, not user-facing privacy notices:

*   **Admin Notices:** The `WPBC_Notices` class provides a dedicated system for displaying persistent, dismissible warnings **to the administrator**. These notices are hooked into custom plugin actions (e.g., `wpbc_hook_*_page_header`) to ensure they appear only on specific plugin pages. This demonstrates the **capability** for contextual display, but the system is not documented as being used for user privacy compliance notices.
*   **Contextual Logic:** The primary front-end controller (`wpdev_booking` class) registers various shortcodes ([booking], [bookingform]) that could represent different contexts. However, there is no evidence that the output generated by these shortcodes is filtered to display different privacy notice text.

#### 6.3 Version Control (Unsupported)
The plugin uses version tracking mechanisms for its own lifecycle, but not specifically for legal text auditing:

*   **Plugin/Content Versioning:** The plugin tracks its overall version number (`booking_version_num`). Furthermore, the `WPBC_Welcome` class manages version tracking for the post-update "What's New" page, using a database option (`booking_activation_redirect_for_version`) to ensure the page is seen only once per new release.
*   While the architecture tracks plugin version and content updates, there is **no documented mechanism** tied to the settings framework to record the version number or modification history of a Privacy Notice text field itself, which is typically required for proving legal compliance.

#### 6.4 Multilingual Support (Architecturally Capable)
This check item is the only one with strong architectural support, even in the absence of the core feature it would support:

*   **Inline Translation System:** The file `core/wpbc-translation.php` defines a robust internationalization and localization engine. This system includes a custom feature using the `wpbc_lang()` function that parses strings containing the shortcode syntax **`[lang=xx_XX]...[/lang]`**. This allows administrators to embed multilingual content (e.g., "Hello[lang=fr_FR]Bonjour") directly within a single administrative text field.
*   **Third-Party Compatibility:** The translation engine includes **explicit compatibility logic for major multilingual plugins** like WPML and Polylang.

Therefore, if a configurable text field (6.1) were implemented, the plugin's translation engine **already possesses the architectural components** to ensure that content is fully translatable and correctly displayed based on the site's active locale.



Based on the sources, the implementation of the **Display & Admin Configuration** options is very low, as the foundational element of a configurable privacy notice (6.1) is not documented. However, the architectural capability for multilingual support (6.4) is strongly implemented.

Therefore, the overall implementation of this feature set is rated **3**.

### **Rating: 3** (The majority of configuration elements are missing, but the architectural requirement for multilingual support is fully met.)

---

### **Justification from Architectural Sources**

#### **Not Implemented (Rating = 1 Contribution): 6.1, 6.2, 6.3**

*   **Configurable Text (6.1):** The architectural blueprint for General Settings is defined in the `WPBC_Settings_API_General` class. The sources confirm this class defines configuration for calendar display, legends, and confirmation messages. There is **no documented field** within this settings catalog that allows an administrator to input or link to the text of a Privacy Notice.
*   **Contextual Display (6.2):** While the plugin has a dedicated and flexible system for contextual messaging via the `WPBC_Notices` class, this system is designed specifically for showing persistent, dismissible warnings **to the administrator** (e.g., the paid-to-free downgrade warning). There is no documented implementation for displaying user-facing privacy notices based on context (e.g., booking form vs. photo upload page).
*   **Version Control (6.3):** The plugin tracks its own internal version numbers and manages tracking for the administrative "What's New" page. However, there is **no documented mechanism** tied to the settings framework to record the version number or modification history of user-inputted legal text, which is necessary for compliance logging.

#### **Fully Implemented Capability (Rating Increase): 6.4**

The architectural capability for multilingual support is robust and highly implemented, even without the core text field present:

*   **Multilingual Support (6.4):** The plugin utilizes a comprehensive internationalization engine managed by `core/wpbc-translation.php`. This system ensures that if a text field were implemented, its content would be translatable:
    *   **Inline Translation System:** The plugin supports a custom shortcode syntax, **`[lang=xx_XX]...[/lang]`**, which can be used by the administrator to embed translations for various languages directly into a single text field. The `wpbc_lang()` function handles the parsing of this content.
    *   **Third-Party Compatibility:** The translation engine includes **explicit compatibility logic** for multilingual plugins like Polylang and WPML.
 

The implementation of the **Display & Admin Configuration** requirements (Check Items 6.1–6.4) requires leveraging the plugin's highly structured custom frameworks for settings, localization, and front-end rendering.

Here is a high-level overview of how these features would be integrated:

---

## High-Level Implementation Overview: Display & Admin Configuration

### 1. Configuration, Customization, and Localization (6.1, 6.4)

The primary goal is to provide the administrator with customizable text input and ensure that content is translatable using the plugin's built-in internationalization engine.

*   **Settings Blueprint Modification (6.1):** New settings fields must be added to the General Settings page by extending the concrete class **`WPBC_Settings_API_General`** in `core/admin/api-settings.php`. This file contains the crucial `init_settings_fields()` method which defines the complete catalog of all settings. New text fields are necessary to allow the admin to **customize the privacy notice text and link** [6.1].
*   **Multilingual Support (6.4):** To ensure the configurable text is **translatable** [6.4], the input fields must be configured to utilize the plugin's custom inline translation system. This engine, defined in `core/wpbc-translation.php`, allows the administrator to embed translations directly within a single text field using the shortcode syntax **`[lang=xx_XX]...[/lang]`**.

### 2. Contextual Rendering and Version Auditing (6.2, 6.3)

Disclosures need to be displayed correctly based on where the user is submitting data, and the accepted version of the policy must be logged.

#### Contextual Display (6.2)
To show different notices based on context (e.g., booking vs. photo upload), the administrator configuration (from Step 1) must include multiple input fields for different policies.

*   **Front-End Injection:** The core rendering logic, centralized in the **`wpdev_booking`** class and its associated filters, must be used. The **`wpdev_booking_form`** filter allows the plugin to retrieve the contextual policy text from the database (using `get_bk_option`) and **inject the appropriate notice HTML** [6.2] into the form before it is displayed to the user.
*   **Admin-Side Customization:** The plugin's existing architecture for UI helper functions (`core/any/admin-bs-ui.php`) can be used to ensure the notice displays consistently with the admin panel's aesthetic standards.

#### Version Control and Record Keeping (6.3)
The system must log the version of the notice the user consented to for compliance [6.3].

*   **Version Storage:** A setting field for the **Notice Version Number** (6.3) must be added in the Admin Configuration (Step 1).
*   **Metadata Persistence:** When a booking is submitted via the AJAX controller (`core/lib/wpbc-ajax.php`), the current version number is retrieved. This version number is then stored as custom metadata associated with the specific booking record.
*   **Storage Mechanism:** This record-keeping utilizes the plugin's dedicated **booking metadata functions** (`wpbc_save_booking_meta_option($booking_id, $option_arr)`), saving the serialized version information into the **`booking_options`** column of the custom booking database table. This creates a permanent audit trail linking the user's consent to the exact policy version they acknowledged.




---

### ✅ **Success Levels**

| Level                      | Description                                                                                                                     |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Basic**                  | A visible privacy notice link exists at booking and message submission.                                                         |
| **Good**                   | Notice includes data types, purpose, retention, rights, and processor list.                                                     |
| **Full Compliance (High)** | Contextual privacy notices per data type (booking/photos/messages), with consent logging, retention policy, and DPA references. |

---

Would you like me to create a **CSV checklist version** of this (so you can mark each compliance item as “Yes / Partial / No” for your plugin audit)?
