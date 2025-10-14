Perfect — let’s break down **“Receive auto-updates (on the way, arrived, delayed, completed)”** into **verifiable items** so you can check if the plugin really implements it.

---

## **Requirements Breakdown**

### 1. Trigger Events

* [ ] Updates can be triggered when the tradesperson changes the **job/booking status** in the admin (e.g., “On the way”).
* [ ] Status options available at minimum: **On the way, Arrived, Delayed, Completed**.
* [ ] Admin can customise/add additional statuses if needed.

Based on the analysis of the plugin's architecture, particularly the status management workflow, the ability to trigger updates based on job/booking status changes is implemented, but the specific, granular status options requested (like "On the way," "Arrived," "Delayed," "Completed") are **not explicitly mentioned** as core features.

Here is a breakdown of the implementation status for the requested "Trigger Events" options:

### Trigger Events Implementation Status

| Feature | Implementation Status (Based on Sources) | Supporting Architectural Evidence |
| :--- | :--- | :--- |
| Updates can be triggered when the tradesperson changes the **job/booking status** in the admin (e.g., “On the way”). | **Yes** | The core workflow is designed around admin-initiated status changes. The AJAX controller (`core/lib/wpbc-ajax.php`) handles crucial booking lifecycle actions such as `wpbc_ajax_UPDATE_APPROVE()`, which approves or denies a booking, and `wpbc_ajax_TRASH_RESTORE()`. Functions in the workflow toolbox (`core/wpbc_functions.php`) manage status changes like `wpbc_db__booking_approve` and `wpbc_auto_approve_booking`. Status updates trigger side-effects, including transactional emails and custom action hooks (e.g., `wpbc_booking_approved` or `wpbc_booking_delete`) that external modules can use for integration. |
| Status options available at minimum: **On the way, Arrived, Delayed, Completed**. | **No (Core status types differ)** | The sources primarily reference core, transactional booking statuses, not job execution statuses. The explicitly mentioned statuses that trigger the Email API are: **New Booking (Admin)**, **New Booking (Visitor)**, **Approved**, **Pending/Denied**, **Deleted**, and **Trash / Reject**. The system tracks **new, pending, and approved** bookings for the Dashboard Widget. The job execution statuses requested (On the way, Arrived, Delayed, Completed) are **not listed** as standard, core status types. |
| Admin can customise/add additional statuses if needed. | **Implied via Extensibility (Unconfirmed for Statuses)** | While the plugin is highly modular and uses a **Custom Hook System** (`add_bk_action`, `apply_bk_filter`) for internal extensibility, the sources do not specify a setting or hook that allows the administrator to easily define and manage **new, named booking statuses** beyond the core lifecycle stages (Approved, Pending, Deleted). The filtering system in the data engine (`core/admin/wpbc-sql.php`) is highly extensible, allowing custom SQL conditions to be injected, which theoretically could filter based on custom status meta data, but the front-end status UI customization is not supported by the sources. |

***

### Numerical Marking (1-10 Scale)

| Feature | Rating (1-10) | Justification (Based on Architectural Evidence) |
| :--- | :--- | :--- |
| Updates can be triggered the core lifecycle stages (Approved, Pending, Deleted). The filtering system in the data engine (`core/admin/wpbc-sql.php`) is highly extensible, allowing custom SQL conditions to be injected, which theoretically could filter based on custom status meta data, but the front-end status UI customization is not supported by the sources. |

***

### Numerical Marking (1-10 Scale)

| Feature | Rating (1-10) | Justification (Based on Architectural Evidence) |
| :--- | :--- | :--- |
| Updates can be triggered when the tradesperson changes the **job/booking status** in the admin (e.g., “On the way”). | **10** | The functionality to trigger status updates and side-effects (emails, hooks) based on an admin action is fundamental to the core workflow, handled by AJAX status handlers (e.g., `wpbc_ajax_UPDATE_APPROVE`). |
| Status options available at minimum: **On the way, Arrived, Delayed, Completed**. | **3** | While core statuses exist (**Approved, Pending, Deleted**), these are workflow statuses. The specific, detailed *job execution* statuses requested are **not documented** in the core status definitions, making the feature incomplete as requested. |
| Admin can customise/add additional statuses if needed. | **4** | The plugin features high extensibility via custom filters and hooks. However, the sources do not confirm a dedicated setting or simple hook for defining and integrating new custom *named statuses* into the admin UI without complex core modifications. |

The implementation of custom job execution statuses ("On the way," "Completed") requires leveraging the plugin's existing status management workflow, data abstraction layers for custom metadata, and its dedicated AJAX system for real-time administrative triggers.

This high-level overview details the implementation across three main architectural components:

### I. Data Model and Customization (Settings Layer)

The system must allow the administrator to define and customize the list of job statuses, which will be stored separately from the plugin's core transactional statuses (Approved, Pending, Deleted).

1.  **Status Definition Storage:** A new configuration field would be defined within the **`WPBC_Settings_API_General`** class in `core/admin/api-settings.php`. This field (e.g., a textarea accepting a list of statuses like "On the way, Delayed, Completed") would allow the administrator to customize the available options.
2.  **Configuration Retrieval:** This customized list of job statuses would be saved using the settings wrapper function **`update_bk_option()`** and retrieved using **`get_bk_option()`** for use in the administrative UI.
3.  **Booking Data Persistence:** The selected custom status for a specific job would be stored as booking metadata. This is achieved using the function **`wpbc_save_booking_meta_option()`**, which saves the custom data as a **serialized array** in the **`booking_options`** column of the custom database table.

### II. Triggering and Core Logic (AJAX and Workflow)

The change must be initiated by the administrator and processed securely in the background, updating both the database and the UI.

1.  **Admin UI Integration:** New buttons or a dynamic dropdown menu reflecting the custom statuses would be added to the booking list view's toolbar, utilizing the existing UI standardization helpers from `core/any/admin-bs-ui.php`.
2.  **Secure AJAX Endpoint:** A new custom AJAX action (e.g., `wpbc_ajax_UPDATE_JOB_STATUS`) must be registered using the **`wpbc_ajax_action_list` filter** located in the central AJAX router.
3.  **Security and Execution:** The PHP handler for this new AJAX action must strictly enforce **nonce verification** (using `wpbc_check_nonce_in_admin_panel()`) to prevent CSRF attacks. The function would then execute the database update using **`wpbc_save_booking_meta_option()`** to record the new status.
4.  **Triggering Side-Effects:** Immediately after the status is saved, a **custom internal action hook** (e.g., `wpbc_job_status_changed`) would be triggered using **`make_bk_action()`** (part of the plugin's custom hook system defined in `core/wpbc-core.php`).

### III. Notifications and Feedback (Email API and UI)

The status change must trigger notifications to the client and provide real-time feedback to the administrator.

1.  **Client Notification:**
    *   A new class (e.g., `WPBC_Emails_API_JobStatus`) would be defined, extending the **`WPBC_Emails_API`** abstract class.
    *   The custom action hook (`wpbc_job_status_changed`) would trigger the sending logic (`wpbc_send_email_job_status()`). This function would retrieve booking data, populate the notification using dynamic shortcodes (e.g., `[booking_id]`, `[dates]`), and use the reliable **`wpbc_wp_mail()`** wrapper to ensure proper header fixing and delivery to the client.
2.  **Admin Feedback and UI Update:**
    *   The AJAX response would follow the established pattern in `core/lib/wpbc-ajax.php`: instead of simply returning JSON, the handler would **echo a `<script>` block** back to the browser.
    *   This inline JavaScript would call a client-side function to dynamically update the booking row in the admin UI, reflecting the new status (e.g., changing the background color of the row to reflect "On the way") without requiring a full page reload.
    *   A dynamic success message could also be displayed to the administrator using the debugging utility **`wpbc_admin_show_top_notice()`**.

---

### 2. Client Notifications

* [ ] Clients receive **real-time notifications** when the status changes.
* [ ] Supported channels:

  * [ ] Email (default, required).
  * [ ] SMS (if Twilio/SendGrid integrated).
  * [ ] WhatsApp (if WhatsApp Business API integrated).
* [ ] Notifications include:

  * Service booked.
  * Job date/time.
  * New status (e.g., “Your tradesperson is on the way”).

The implementation status of the requested Client Notifications, based on the architectural components analyzed in the sources, is highly robust for the **Email** channel but lacks support for external messaging services.

### Status Implementation Summary

| Feature | Implementation Status | Justification (Based on Sources) |
| :--- | :--- | :--- |
| Clients receive **real-time notifications** when the status changes. | **Yes (via transactional email)** | The core workflow triggers notifications immediately upon an administrative action. Status changes (like approval or deletion) are executed via **AJAX endpoints** (e.g., `wpbc_ajax_UPDATE_APPROVE()`, `wpbc_ajax_DELETE_APPROVE()`). These handlers are designed to execute necessary side-effects, such as sending confirmation emails, immediately after the booking status is updated in the database. |
| Supported channels: Email (default, required). | **Yes (Foundational)** | Email is the primary and centralized communication method. Every critical booking status change—including **Approved**, **Pending/Denied**, **Deleted**, and **New Booking (Visitor)**—is managed by a dedicated class that extends the `WPBC_Emails_API` framework. The system uses the `wpbc_wp_mail()` wrapper to enhance deliverability. |
| Supported channels: SMS (if Twilio/SendGrid integrated). | **No** | The provided sources document extensive integration with third-party services for calendar synchronization (ICS feeds, Google Calendar API), but they **do not mention** any architectural support, APIs, or integration logic for SMS gateways like Twilio or SendGrid. |
| Supported channels: WhatsApp (if WhatsApp Business API integrated). | **No** | There is **no evidence** in the sources of integration with the WhatsApp Business API or any other dedicated chat/messaging platform. |
| Notifications include: Service booked. Job date/time. New status (e.g., “Your tradesperson is on the way”). | **Yes (via dynamic shortcodes)** | The content of transactional emails is highly customizable and dynamic using **shortcodes**. The system includes helper functions to generate an array of dynamic values for substitution. This dynamic content includes essential booking details: |
| | | *   **Booking Details (Service, Date/Time):** Shortcodes like **`[booking_id]`**, **`[dates]`**, **`[resource_title]`**, and **`[moderatelink]`** are available and listed in the help section for email settings. The email sending logic retrieves booking data to populate these shortcodes just before dispatch. |
| | | *   **New Status:** The notification itself serves as the status update. When an administrator approves a booking, the content defined in the **`page-email-approved.php`** template is sent, clearly communicating the new status to the client. Similarly, the `[denyreason]` shortcode is available in the **Deleted** and **Pending/Denied** templates to explain the status change. |

### Conclusion

The plugin fully implements **transactional, status-based client notifications via Email** using a modular, shortcode-driven API. However, the specific status options mentioned (like "On the way") are not confirmed as core statuses (as discussed in prior conversation) and the ability to send notifications via SMS or WhatsApp is **not supported** by the analyzed architecture.
The implementation of advanced, multi-channel client notifications requires building new channel APIs (for SMS and WhatsApp) and integrating them seamlessly into the existing booking status workflow.

This high-level overview outlines how to implement **real-time, channel-aware client notifications** by leveraging the plugin’s robust object-oriented architecture and custom hook system:

### I. Channel Integration and Configuration

The first step is to establish the infrastructure for the new messaging channels (SMS/WhatsApp) and integrate the required API keys into the settings system.

1.  **Settings Integration:** New settings fields for SMS Gateway API keys (e.g., Twilio Account SID, Auth Token) and WhatsApp Business API credentials would be defined within the **`WPBC_Settings_API_General`** concrete class. These settings would be managed and persisted via the custom settings framework.
2.  **New Messaging APIs (SMS/WhatsApp):** New abstract classes (e.g., `WPBC_SMS_API` and `WPBC_WhatsApp_API`) would be created, following the standardized pattern of the existing **`WPBC_Emails_API`**.
    *   These new classes would handle the specific formatting and external communication logic.
    *   API requests to external gateways (Twilio, WhatsApp) would be executed using secure HTTP functions, similar to how the Google Calendar synchronization uses **`wp_remote_get()`**.
3.  **Template and Dynamic Content:** The new channel APIs must include a **shortcode replacement engine** to dynamically populate message content (e.g., `[dates]`, `[resource_title]`) with booking details, mirroring the functionality used by the Email API to generate transactional emails.

### II. Triggering and Dispatch Controller

The custom status change (e.g., "On the way") serves as the primary trigger for channel-aware notification dispatch.

1.  **Custom Status Hook Integration:** When the administrator changes the job status via the custom administrative AJAX action (e.g., `wpbc_ajax_UPDATE_JOB_STATUS`), the handler must execute a **`make_bk_action()`** hook (e.g., `wpbc_client_status_update`) using the custom internal hook system defined in `core/wpbc-core.php`.
2.  **Central Dispatch Controller:** A new controller function would listen to this custom hook. This controller's responsibilities include:
    *   Retrieving the current booking details using the stable Developer API function, such as **`wpbc_api_get_booking_by_id()`**.
    *   Retrieving the client's preferred communication channel (Email, SMS) and ensuring they have not opted out, likely checking data stored as **"booking meta options"** via `wpbc_get_booking_meta_option()`.
    *   Translating and Localizing Content: Ensuring the message content and dates are correct. Date/time strings must be processed using the specialized functions in `core/wpbc_functions_dates.php`, such as **`wpbc_datetime_localized()`** and **`wpbc_get_redable_dates()`**, to respect the user's locale and timezone offset.

### III. Channel-Aware Delivery and Fallback

The final stage ensures reliability by prioritizing the selected channel while using email as a mandatory fallback.

1.  **Channel Priority and Execution:**
    *   The Dispatch Controller prioritizes sending via the highest-priority enabled channel (e.g., WhatsApp > SMS > Email).
    *   It executes the **`send()`** method of the chosen API (e.g., `WPBC_SMS_API->send()`).
2.  **Operational Email Guarantee:** Since email is required, it acts as the guaranteed operational channel. Even if SMS or WhatsApp is used, a record or simplified notification should be sent via the standard **`WPBC_Emails_API`**.
3.  **Global Block Check:** Before any email is dispatched (including the operational email notifications), the system should check the **`wpbc_email_api_is_allow_send`** filter, which acts as a global "kill switch" to programmatically prevent sending, ensuring compliance with any high-level suppression rules.

---

### 3. Admin Controls

* [ ] Admin can **enable/disable** auto-updates.
* [ ] Admin can **customise message templates** per status (with placeholders like {client_name}, {status}, {time}).
* [ ] Admin can **choose which channels** each status is sent to (email only, SMS + email, etc.).
* [ ] Option to trigger updates **manually** (send “On the way” message without changing full job status).

Based on the comprehensive architectural review of the source files, here is the implementation status of the requested Admin Controls, drawing on details regarding the Settings API, Email API, and core utility functions:

### Admin Controls Implementation Status

| Feature | Implementation Status (Based on Sources) | Supporting Architectural Evidence |
| :--- | :--- | :--- |
| Admin can **enable/disable** auto-updates. | **Partial (Related: Translation Updates Only)** | The sources **do not explicitly mention** a global control to enable/disable auto-updates for the main plugin code itself. However, the system includes functionality to manage specific automated processes: |
| | | *   **Translation Updates:** The `core/wpbc-translation.php` file contains logic (`wpbc_update_translations__from_wp()`) for updating language packs programmatically using `WP_Upgrader`. The System Info panel also includes tools for force-updating translation files. |
| | | *   **Google Calendar Sync:** The scheduler controls the automation of GCal imports via `WPBC()->cron->update()`, which can be enabled/disabled via settings. |
| Admin can **customise message templates** per status (with placeholders like {client\_name}, {status}, {time}). | **Yes (Fully Implemented for Email)** | The Email API is specifically designed for this purpose: |
| | | *   **Per Status Templates:** Each critical status (Approved, Deleted, Pending/Denied, New Admin/Visitor) has a dedicated email template file (e.g., `page-email-approved.php`, `page-email-deleted.php`). Each template is managed by a class (e.g., `WPBC_Emails_API_Approved`) that defines fields for customizing the subject and content. |
| | | *   **Placeholders/Shortcodes:** The system supports dynamic placeholders, referred to as **shortcodes** (e.g., `[booking_id]`, `[dates]`, `[resource_title]`, `[moderatelink]`). The sending logic includes a function (e.g., `wpbc__get_replace_shortcodes__email_approved()`) that retrieves booking data and generates an array of dynamic values for substitution. The function `wpbc_get_email_help_shortcodes()` dynamically generates a list of available shortcodes for the admin interface. |
| Admin can **choose which channels** each status is sent to (email only, SMS + email, etc.). | **Partial (Email Channel Control Only)** | The system is architecturally focused on **Email** as the channel, with control available only at the template level. |
| | | *   **Channel Selection (Email):** Each dedicated email class (e.g., `WPBC_Emails_API_Approved`) includes fields for **enabling/disabling** that specific notification. |
| | | *   **No Multi-Channel Support:** The sources provide **no evidence** of architectural support or integration with SMS, WhatsApp, or other multi-channel APIs. The only communication channel detailed is the sophisticated Email API. |
| Option to trigger updates **manually** (send “On the way” message without changing full job status). | **No (Only Transactional Status Changes Supported)** | The core status change workflow is strictly **transactional** (status change triggers the email), not conversation-driven: |
| | | *   **Auto-Triggered:** Emails (like approval or denial) are sent automatically as a result of an administrative **status change**. Quick actions like `[click2approve]` in admin emails directly change the status. |
| | | *   **No Ad-Hoc Sending:** There is **no evidence** of a separate mechanism, UI, or function that allows the administrator to compose and send a *manual, ad-hoc message* (like "On the way") without triggering a full, core job status change or utilizing the system's predefined transactional templates. |


Based on the sources and our conversation history regarding the plugin's architecture, here is the implementation rating for the requested Admin Controls:

| Feature | Rating (1-10) | Justification (Based on Architectural Evidence) |
| :--- | :--- | :--- |
| Admin can **enable/disable** auto-updates. | **5** | Control exists for specific automated features: the system schedules **Google Calendar imports** using the `WPBC()->cron->update()` method, and manages the update and display of **translation files**. However, the sources **do not confirm** a global setting to enable or disable automatic updates for the core plugin code itself. |
| Admin can **customise message templates** per status (with placeholders like {client\_name}, {status}, {time}). | **10** | This functionality is **fully implemented** for the primary communication channel (Email). Each key status (Approved, Deleted, Pending) has a dedicated email template class (e.g., `WPBC_Emails_API_Approved`) that defines customizable fields for subject, content, and styling. The templates support dynamic content via **shortcodes** (placeholders) such as `[booking_id]`, `[dates]`, `[resource_title]`, and `[denyreason]` which are processed by functions like `wpbc__get_replace_shortcodes__email_approved()`. |
| Admin can **choose which channels** each status is sent to (email only, SMS + email, etc.). | **4** | Control is implemented only at the **transactional email template level**. Each email class includes fields for **enabling/disabling** that specific notification type. This allows for granular control within the Email channel. However, the sources provide **no architectural evidence** supporting multi-channel functionality (e.g., SMS or WhatsApp integration) that would allow selection across different media. |
| Option to trigger updates **manually** (send “On the way” message without changing full job status). | **1** | The existing notification workflow is strictly **transactional**: emails are sent **automatically** only when a core booking status is changed (Approved, Deleted, etc.). There is **no evidence** in the sources of a dedicated UI, AJAX endpoint, or logic that allows an administrator to compose and send an **ad-hoc manual message** (e.g., "On the way") without triggering a full, core status change. |



The implementation of these advanced administrative controls and multi-channel features requires extending the plugin's established object-oriented frameworks, including the **Custom Settings API**, the **AJAX Controller**, and the **Email API Pattern**.

Here is a high-level overview of the implementation strategy:

### I. Settings and Control Configuration

This phase involves adding new fields to the existing configuration framework to manage messaging channels, update behavior, and manual templates.

1.  **Auto-Update Control:**
    *   A new setting for enabling/disabling plugin auto-updates would be defined by extending the concrete class **`WPBC_Settings_API_General`** in `core/admin/api-settings.php`.
    *   Although the sources do not detail the corresponding WordPress filter (e.g., `auto_update_plugin`), the plugin already demonstrates the ability to manage automated processes like Google Calendar synchronization using `WPBC()->cron->update()` and customizes the UI for translation updates. The new setting would control the logic attached to the appropriate WordPress filter to enforce the admin's preference.

2.  **Channel Selection per Status:**
    *   The configuration UI for each email template (e.g., **Approved**, **Deleted**) is managed by a dedicated class (e.g., `WPBC_Emails_API_Approved`) and its corresponding settings page.
    *   These pages would be extended to include checkboxes for new channels (SMS, WhatsApp), using the standardized rendering methods of the **`WPBC_Settings_API`**. The settings would be persisted as options in the database using the same mechanisms employed by the existing template settings.

3.  **Customizing Message Templates (Leveraging Existing Architecture):**
    *   The ability to customize content with placeholders is already robustly implemented via the **Email API**.
    *   Administrators can define content and subject lines using dedicated settings tabs. The system already supports dynamic shortcodes (placeholders) like **`[booking_id]`**, **`[dates]`**, **`[resource_title]`**, and **`[denyreason]`**. New custom statuses (e.g., "On the way") would rely on this same shortcode replacement engine.

### II. Multi-Channel API Integration

The plugin's architecture lacks native SMS/WhatsApp support. This requires creating new API handlers structured identically to the existing Email API.

1.  **New Abstract API Classes:** Define new abstract classes (e.g., `WPBC_SMS_API` and `WPBC_WhatsApp_API`) that mirror the inheritance and structure of the **`WPBC_Emails_API`**.
2.  **External Communication Logic:** These new classes would handle the specific logic for communicating with external services (Twilio, WhatsApp Business API). This external communication should use secure WordPress functions like **`wp_remote_get()`** for HTTP requests, similar to the Google Calendar sync integration.
3.  **Channel-Aware Dispatch:** A central Dispatch Controller would be introduced (likely triggered by a custom action hook). For a given status update, this controller would read the channel preferences defined in the settings and call the appropriate **`send()`** method on the chosen API object (`WPBC_Emails_API`, `WPBC_SMS_API`, etc.).

### III. Manual Messaging and Triggering

The current workflow is strictly transactional (status change triggers notification). Implementing manual sending requires creating a new, dedicated flow using the AJAX controller.

1.  **Admin UI for Manual Sending:** A dedicated form or quick-action button would be added to the booking details view in the admin panel. The UI components would be built using the procedural helper functions found in `core/any/admin-bs-ui.php`.
2.  **New AJAX Endpoint:** A new custom AJAX action (e.g., `wpbc_ajax_send_manual_message`) would be registered using the **`wpbc_ajax_action_list` filter** found in `core/lib/wpbc-ajax.php`.
3.  **Security and Logic:** The handler for this new endpoint must begin with **nonce verification** (using functions like `wpbc_check_nonce_in_admin_panel()`) to ensure the request is secure against CSRF.
4.  **Message Logging (Conversation Management Prerequisite):** The manual message content must be logged and linked to the **`booking_id`**. If a centralized conversation system is not yet in place, the message could be saved as **custom booking metadata** using the **`wpbc_save_booking_meta_option()`** function, storing the data as a serialized array in the **`booking_options`** column of the custom booking table.
5.  **Dispatch:** Once logged, the manual message handler would call the Channel-Aware Dispatch Controller (Section II) to execute delivery via the selected channel(s).


---

### 4. Conversation History

* [ ] All auto-updates appear in the **client’s conversation history** (alongside two-way messaging if available).
* [ ] Admin can see which auto-updates were sent, with **timestamp and delivery status**.

Based on the analysis of the plugin's architecture, particularly its data handling and notification systems, the requested **Conversation History** options are **not implemented** as a centralized, logging, or multi-message feature.

The plugin manages transactional emails effectively, but it lacks the core structural components to record and display a full history of communications or auto-updates.

Here is a detailed breakdown of the implementation status:

### Conversation History Implementation Status

| Feature | Implementation Status (Based on Sources) | Supporting Architectural Evidence |
| :--- | :--- | :--- |
| All auto-updates appear in the **client’s conversation history** (alongside two-way messaging if available). | **No** | The plugin operates on a transactional email model, where admin actions trigger specific email dispatches (e.g., `wpbc_send_email_approved()`). There is **no evidence** of a dedicated database table or a conversation view UI component within the administrative pages (such as the Booking Listing or Timeline views) that is designed to log and display a chronological history of these outbound auto-updates or any inbound messages. |
| Admin can see which auto-updates were sent, with **timestamp and delivery status**. | **No (Partial Traceability Implied)** | The Email API focuses on reliable dispatch, utilizing the `wpbc_wp_mail()` wrapper to fix the Sender header for deliverability. The process involves generating dynamic content via shortcodes (e.g., `[booking_id]`, `[dates]`) and triggering the email upon a status change. |
| | | However, the sources **do not mention** any system for logging the precise **timestamp** of the successful *send attempt* or recording the external **delivery status** (e.g., "delivered" vs. "bounced") for these transactional auto-updates. Critical administrative logging functions mentioned, like `wpbc_db_get_number_new_bookings()` and `wpbc_db__add_log_info` (implied in the workflow), focus on booking status and errors, not message delivery records. |

### Architectural Context for Missing Feature

The plugin's architectural philosophy relies on:

1.  **Transactional Emails:** Communication is handled by discrete, highly configurable transactional email classes (e.g., `WPBC_Emails_API_Approved`, `WPBC_Emails_API_Deleted`). These are one-way updates triggered by status changes.
2.  **Serialized Metadata:** Custom data for a booking is stored in a **serialized array** within the **`booking_options`** column of the custom booking table. This format is **inefficient and not queryable** for tasks like filtering a message log by timestamp or sender, which is necessary for a conversation history feature.
3.  **Data Delegation:** The UI relies on the data engine (`core/admin/wpbc-sql.php`) to fetch booking data, but this engine is tailored for filtering based on core booking status and dates, not for querying a dedicated conversation history.

***

### Numerical Marking (1-10 Scale)

| Feature | Rating (1-10) | Justification (Based on Architectural Evidence) |
| :--- | :--- | :--- |
| All auto-updates appear in the **client’s conversation history** (alongside two-way messaging if available). | **1** | There is no evidence of a dedicated database table, messaging log, or centralized UI view designed to capture and display a historical conversation thread or a chronological record of auto-updates for a specific booking. |
| Admin can see which auto-updates were sent, with **timestamp and delivery status**. | **2** | While the system confirms the *action* of sending the email (e.g., `wpbc_send_email_approved()`), the sources provide no data structures or logging mechanisms for recording the **timestamp of the successful send** or the **delivery status** (e.g., confirmation from an external SMTP provider) of the transactional auto-update message. |


The implementation of **Conversation History** would fundamentally change the plugin’s data model from a transactional email system to a persistent logging system. This requires creating a dedicated, queryable database table and hooking into the plugin's existing status change and communication workflow to ensure all auto-updates are logged and displayed.

Here is a high-level overview of the implementation strategy:

### I. Data Model and Persistence (Logging Foundation)

The current architecture relies on storing custom data as a serialized array in the `booking_options` column, which is unsuitable for conversational history and filtering. A new dedicated structure is necessary.

1.  **Custom Database Table:** A new database table (e.g., `wp_booking_messages`) would be created to store every communication event. Fields would include `booking_id`, `sender_type` (Admin/Client/System), `channel` (Email/SMS/Portal), `timestamp`, `content`, and crucial `delivery_status`.
2.  **Lifecycle Integration:** The creation of this new table would be orchestrated during plugin activation by hooking into the custom action **`make_bk_action( 'wpbc_activation' )`**. This ensures the setup runs reliably when the plugin is installed or updated, managed by the lifecycle class `WPBC_Install`.
3.  **Data Persistence Wrappers:** New secure functions would be implemented for inserting and retrieving messages from this table, leveraging the global **`$wpdb`** object with prepared statements for security, following the pattern used by the Dates Engine for database queries.

### II. Logging Auto-Updates (Transaction History)

The primary goal is to log every message sent via the Email API.

1.  **Logging Hook Injection:** Every core email sending function (e.g., `wpbc_send_email_approved()`, `wpbc_send_email_deleted()`) must be modified. Immediately before the function calls the core `$mail_api->send()` method (which wraps `wpbc_wp_mail()`), it must trigger a new logging function.
2.  **Capturing Data:** The logging function captures the full, shortcode-processed email body, the `booking_id`, and the `timestamp` of the send request. The timestamp must be accurately recorded, potentially using the localization engine functions in `core/wpbc_functions_dates.php` (like `wpbc_datetime_localized()`) to ensure timezone accuracy.
3.  **Delivery Status Tracking:**
    *   **Send Attempt Status:** The logging function records whether the `wpbc_wp_mail()` wrapper successfully initiated the send attempt.
    *   **Delivery Status:** For external channels (SMS, WhatsApp, or specialized transactional email services), API responses would be captured and processed, updating the `delivery_status` field in the message table to provide feedback on "delivery status".

### III. Administrative Interface (Conversation View)

The history must be displayed chronologically within the administrative dashboard.

1.  **Admin UI Location:** A new, dedicated section would be added to the single booking details page within the administrative interface. This section would utilize the standardized Meta Box UI helpers provided in `core/wpbc_functions.php`: **`wpbc_open_meta_box_section()`** and **`wpbc_close_meta_box_section()`**.
2.  **Rendering the Thread:** The view function for this section would query the new `wp_booking_messages` table for all records associated with the current `booking_id`, ordered by `timestamp`. The output would display the chronological feed, showing the **sender, timestamp, and channel** for each auto-update or manual message.
3.  **Real-time Admin Feedback:** When an administrator sends a manual message (or an auto-update is processed via AJAX), the AJAX handler (located in `core/lib/wpbc-ajax.php`) would not only update the database but also echo inline JavaScript (following the existing pattern) to trigger a client-side function that refreshes the conversation history section without a full page reload.


---

### 5. Reliability & Compliance

* [ ] If delivery fails (SMS/email bounce), system logs the failure.
* [ ] Auto-updates are classified as **operational notifications** (not marketing).
* [ ] GDPR: Clients cannot opt out of these if they have an active booking, but opt-out applies after job completion.

Based on the sources, the options related to **Reliability & Compliance** for messaging are **partially implemented** through the Email API and core data handling, but the system lacks explicit features for logging delivery failures, formal message classification (operational vs. marketing), or GDPR-specific client opt-out rules.

Here is a breakdown of the implementation status:

### Reliability & Compliance Implementation Status

| Feature | Implementation Status (Based on Sources) | Supporting Architectural Evidence |
| :--- | :--- | :--- |
| If delivery fails (SMS/email bounce), system logs the failure. | **No** | The plugin’s focus is on reliably dispatching emails using the `wpbc_wp_mail()` wrapper, which attempts to fix the Sender header to improve deliverability. However, the sources **do not contain evidence** of a system for receiving, processing, or logging external feedback on delivery failure (such as an email bounce notification or an external API error code from an SMS gateway). The closest logging functionality is found in debugging tools (`core/wpbc-debug.php`) which can retrieve the last database error (`$EZSQL_ERROR`), but this pertains to database failures, not email delivery failures. |
| Auto-updates are classified as **operational notifications** (not marketing). | **Partial (Implied Operational Nature)** | The plugin architecture defines email notifications strictly as **transactional** updates tied to booking lifecycle status changes (e.g., **Approved**, **Deleted**, **New Booking**). This transactional nature implies they are operational. Furthermore, the **Email API** provides a powerful mechanism that could be used for filtering: the **`wpbc_email_api_is_allow_send`** filter acts as a "global kill switch" that can programmatically prevent *any* email from being sent. This filter could be leveraged by an external system to enforce a classification policy, but the sources **do not indicate** that the plugin formally classifies emails as "operational" versus "marketing". |
| GDPR: Clients cannot opt out of these if they have an active booking, but opt-out applies after job completion. | **No (GDPR-specific opt-out not supported)** | While the plugin supports enabling/disabling individual transactional email notifications, and the deletion logic respects administrator settings (`booking_is_delete_if_deactive`) upon deactivation, there is **no explicit functionality** documented for: |
| | | *   **GDPR compliance**. |
| | | *   Allowing clients to manage their own messaging preferences based on booking status (active vs. complete). |
| | | *   Providing a client-facing opt-out mechanism for notifications. |
| | | The only mechanism for blocking emails is the global filter **`wpbc_email_api_is_allow_send`**, which is system-controlled, not client-controlled based on active status. |


Based on the analysis of the plugin's data handling, Email API, and core utility files, here is the rating (1-10 scale) for the requested Reliability & Compliance features:

| Feature | Rating (1-10) | Justification (Based on Architectural Evidence) |
| :--- | :--- | :--- |
| If delivery fails (SMS/email bounce), system logs the failure. | **1** | The plugin uses a wrapper (`wpbc_wp_mail()`) around the core WordPress mail function to improve deliverability by fixing the Sender header. However, the sources **do not contain any architecture** (like a webhook receiver, logging table, or API error processing) designed to receive, process, or record external feedback concerning delivery failures (e.g., email bounces or SMS API failure codes). Logging functions mentioned in the debug toolkit are primarily for database errors. |
| Auto-updates are classified as **operational notifications** (not marketing). | **7** | The notifications are inherently **transactional** (e.g., approval, denial, deletion), meaning they serve an operational purpose tied to the booking lifecycle. The architecture is highly controllable: individual templates can be enabled/disabled, and the global filter `wpbc_email_api_is_allow_send` acts as a "kill switch" that could be used programmatically to enforce a classification rule. However, the plugin **does not explicitly define** or provide settings for classifying messages as "marketing" versus "operational." |
| GDPR: Clients cannot opt out of these if they have an active booking, but opt-out applies after job completion. | **2** | The plugin provides mechanisms for data management, such as setting up data cleanup upon plugin deactivation. It also provides granular control over which transactional emails are sent. However, the sources **do not support** the implementation of a client-side opt-out preference that is dynamically enforced based on the live status of the booking (i.e., whether the job is "active" or "completed"). The complex, status-aware compliance rule requested is not present. |


This implementation overview addresses the requirements for **Reliability & Compliance** by focusing on leveraging the plugin's existing custom settings framework, the transactional **Email API** pattern, and the **pseudo-cron system** for automated enforcement and logging.

### I. Logging Failures and Tracking Delivery Status

Since the plugin currently lacks a mechanism to log delivery failures, a prerequisite is establishing a logging mechanism that intercepts the outbound communication process.

1.  **Centralized Logging (Data Model Prerequisite):** The implementation relies on the creation of a dedicated database table (e.g., `wp_booking_messages` as discussed previously) to store every communication attempt, including the field for `delivery_status` [I]. This table creation must be hooked into the activation sequence using a custom action.
2.  **Intercepting Outbound Attempts:**
    *   The core wrapper function for mail sending, **`wpbc_wp_mail()`**, must be enhanced. This function wraps WordPress's `wp_mail()` and installs a temporary class that hooks into `phpmailer_init` to fix the Sender header for deliverability.
    *   The wrapper should be modified to catch non-successful results from the native mail function, logging a "Send Failure" record to the new message table with the attempt timestamp.
3.  **External Delivery Status:** For channels like SMS or specialized transactional email services, the response from the external API (e.g., Twilio) must be captured and parsed. Robust error checking already exists in the synchronization logic (checking for `WP_Error` objects and non-200 HTTP status codes); this pattern should be reused to update the `delivery_status` record to "Delivered," "Bounced," or "API Error".

### II. Message Classification and Opt-Out Enforcement

This addresses the need to distinguish between operational and marketing messages and enforce client-specific opt-out rules.

1.  **Operational Classification:**
    *   The plugin's auto-updates (Emails) are already **transactional** (operational) as they are tied to lifecycle status changes (Approved, Deleted, etc.).
    *   The **`WPBC_Emails_API`** abstract class provides the filter **`wpbc_email_api_is_allow_send`**, which acts as a "global kill switch" that can programmatically prevent *any* email from being sent.
    *   Implementation would involve hooking into this filter to check the message type (which can be inferred from the sending class, e.g., `WPBC_Emails_API_Approved`) and compare it against the client's preference, ensuring operational messages are not blocked.

2.  **Status-Aware GDPR Opt-Out:**
    *   **Data Storage:** The client's opt-out preference and the booking status (Active or Completed) would need to be stored as **booking meta options** using **`wpbc_save_booking_meta_option()`**, saved as a serialized array in the `booking_options` column.
    *   **Enforcement Logic:** The enforcement function (listening to `wpbc_email_api_is_allow_send`) would retrieve the client's preference and the booking status (Active/Completed) via **`wpbc_get_booking_meta_option()`**. If the booking is active, the preference is overridden, and the operational message is allowed to send.

### III. Automated Data Retention

Enforcing data retention rules requires configuration and scheduled execution, leveraging the plugin’s dedicated cron system.

1.  **Configuration:** A new setting field for retention duration (e.g., "Delete records older than X months") would be defined using the custom Settings API framework. This is done via the **`WPBC_Settings_API_General`** concrete class.
2.  **Scheduling the Task:** The plugin uses a traffic-dependent **custom pseudo-cron system** implemented by the **`WPBC_Cron`** class.
    *   A new recurring task, "Compliance Cleanup," would be scheduled using the public methods of the global `WPBC()->cron` object.
    *   The task logic would be attached to a custom action hook (using `make_bk_action`).
3.  **Executing Cleanup:** When triggered by the pseudo-cron system, the task function would execute a secure, prepared **SQL query** (using `$wpdb`) to delete message records from the messages table that are older than the configured duration retrieved via `get_bk_option`.
---

## **Success Criteria Checklist**

✅ Status options (On the way, Arrived, Delayed, Completed) are available and trigger notifications.
✅ Clients receive notifications in real time when status changes.
✅ Notifications are customisable by admin (templates + channels).
✅ Auto-updates appear in conversation history and logs.
✅ Delivery failures are logged.
✅ Compliance: operational updates only, GDPR-safe.

---

👉 Would you like me to **also map these auto-updates into the earlier “two-way messaging hub” feature**, so you can check if the plugin merges them into the same conversation thread (instead of sending them separately)? That way we can see if the plugin is siloed or unified in messaging.
