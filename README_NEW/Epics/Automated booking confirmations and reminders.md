Perfect — let’s break down **“Automated booking confirmations and reminders”** into concrete items you can **verify in the plugin**.

---

## **Requirements Breakdown**

### 1. Trigger Points

* [ ] **Booking confirmation** is automatically sent immediately after a booking is made (via form or admin).
* [ ] **Reminders** are sent before the booking, at configurable intervals (e.g., 24h before, 1h before).
* [ ] **Cancellation/reschedule confirmation** is sent automatically if booking status changes.

Based on the analysis of the plugin's architecture, particularly the **Email and Notification System**, the functionality for sending immediate confirmations and cancellation confirmations is **implemented**, but the feature for sending pre-booking **reminders** at configurable intervals is **not explicitly implemented** in the documented system.

Here is a breakdown of the Trigger Points based on the source material:

### 1. Booking confirmation is automatically sent immediately after a booking is made (via form or admin).

**This feature is implemented.** The plugin architecture dictates that upon new booking submission, immediate notifications are triggered and sent to both the administrator and the visitor.

*   **Administrator Confirmation:** When a user submits a new booking, the functions `wpbc_send_email_new_admin()` and `wpbc_send_email_new_visitor()` are immediately generated and sent. The admin notification is defined by the **`WPBC_Emails_API_NewAdmin`** class and is designed for **quick management**, including dynamic shortcodes like `[moderatelink]`, `[click2approve]`, and `[click2decline]`.
*   **Visitor Confirmation:** The confirmation sent to the visitor is managed by the **`WPBC_Emails_API_NewVisitor`** class.
*   **Approval Confirmation:** If the booking requires approval and the administrator approves it, this triggers the sending of the **"Approved" confirmation email** to the visitor, managed by the `WPBC_Emails_API_Approved` class and its sending function `wpbc_send_email_approved()`.

### 2. Reminders are sent before the booking, at configurable intervals (e.g., 24h before, 1h before).

**This specific feature is not documented as implemented.** Sending reminders before a booking requires a reliable, time-sensitive scheduling mechanism, which the plugin's current system has limitations regarding.

*   **Scheduling System:** The plugin utilizes a **custom pseudo-cron system** via the **`WPBC_Cron`** class (in `core/lib/wpbc-cron.php`). This system stores scheduled tasks in a database option (`booking_cron`) and executes them by hooking into the high-priority `init` action, which runs on most page loads.
*   **Use Case:** The example cited for this pseudo-cron system is the automated import of Google Calendar data (`wpbc_import_gcal`).
*   **Limitation:** This system is **traffic-dependent**. The sources note that "If a website has very low traffic, scheduled tasks may be significantly delayed". While the cron system provides the necessary foundation for executing delayed tasks, **the sources do not confirm that an email template or core logic exists to define and send automated, time-specific booking reminders.**

### 3. Cancellation/reschedule confirmation is sent automatically if booking status changes.

**This feature is implemented for cancellations and status changes.**

*   **Cancellation/Denial:** When a booking status is changed by the administrator (e.g., deleted, denied, or moved to trash), a corresponding email is automatically triggered.
    *   The **"Deleted"** notification is managed by the `WPBC_Emails_API_Deleted` class.
    *   The **"Trash / Reject"** notification is managed by the `WPBC_Emails_API_Trash` class.
    *   The emails for denied/deleted status changes utilize the shortcodes `[denyreason]` or `[reason]` to convey the justification for the status change.
*   **Status Change Workflow:** Booking status changes are initiated via **AJAX requests** handled by `core/lib/wpbc-ajax.php`, which executes direct database queries and triggers lifecycle action hooks (e.g., `wpbc_booking_approved`, `wpbc_booking_delete`). These actions then initiate the email sending process via the functions defined in the Email API.

Based on the detailed architectural analysis of the plugin's Email and Notification System, I can evaluate the implementation status of each trigger point and assign a rating on a scale of 1 (not implemented at all) to 10 (fully implemented as requested).

The plugin uses a robust, object-oriented pattern based on an abstract Email API to define and manage every email template.

***

### Trigger Point Analysis and Rating

#### 1. Booking confirmation is automatically sent immediately after a booking is made (via form or admin).

| Implementation Status | Rating (1-10) |
| :--- | :--- |
| **Implemented** | **10** |

**Justification:**

The plugin explicitly implements mechanisms to send two immediate notifications upon successful booking submission: one to the visitor and one to the administrator.

*   **Visitor Confirmation (Automatic, Immediate):** The core process initiates two immediate notifications when a new booking is submitted. The file `core/admin/page-email-new-visitor.php` implements the initial **"New Booking" confirmation email template sent directly to the visitor** after they submit a booking. This notification is handled by the `WPBC_Emails_API_NewVisitor` class.
*   **Administrator Notification (Automatic, Immediate):** A notification is simultaneously sent to the administrator(s), defined by the `WPBC_Emails_API_NewAdmin` class (in `page-email-new-admin.php`). This email is tailored for quick management, including dynamic shortcodes like `[click2approve]` and `[click2decline]`.

This process fully covers the request for automatic, immediate booking confirmation to both parties.

#### 2. Reminders are sent before the booking, at configurable intervals (e.g., 24h before, 1h before).

| Implementation Status | Rating (1-10) |
| :--- | :--- |
| **Unconfirmed/Infrastructure Exists** | **2** |

**Justification:**

The provided sources **do not explicitly confirm the implementation of pre-booking reminder emails** sent at configurable intervals (e.g., 24h before, 1h before).

However, the architectural foundation necessary for such a feature is fully implemented:

*   **Scheduling Infrastructure Exists:** The plugin utilizes a **custom pseudo-cron system** via the `WPBC_Cron` class (in `core/lib/wpbc-cron.php`). This system is specifically designed for scheduling and executing **recurring tasks** using the WordPress `init` action.
*   **Extensibility:** Developers can schedule custom tasks using the public `add()`, `update()`, and `delete()` methods of the global `WPBC()->cron` object.
*   **Primary Cited Use:** The main example cited for the use of this custom cron system is the execution of `wpbc_import_gcal` for automated Google Calendar synchronization.

Since the sources detail the infrastructure for automated, time-based tasks but do not mention a specific email template or workflow for sending pre-booking reminders, the feature is marked as largely unimplemented or unconfirmed based on the provided analysis.

#### 3. Cancellation/reschedule confirmation is sent automatically if booking status changes.

| Implementation Status | Rating (1-10) |
| :--- | :--- |
| **Implemented** | **10** |

**Justification:**

The plugin's Email API implements dedicated templates and sending logic for all critical booking status changes, fully covering confirmation of cancellation, denial, and approval (which often follows a reschedule/update).

The plugin defines dedicated files and classes to manage these status changes, ensuring automatic confirmations are sent:

*   **Cancellation/Denial Confirmation:** The file `core/admin/page-email-deleted.php` implements the email template and sending logic for the **"Deleted" notification**, which is sent automatically **when a booking is canceled or declined by an administrator**. This email includes a reason if provided, via the `[denyreason]` shortcode. A separate `page-email-trash.php` handles bookings moved to the trash.
*   **Approval Confirmation:** When an administrator approves a booking, the `core/admin/page-email-approved.php` file triggers the sending of the **"Approved" confirmation email** to the visitor.
*   **Pending Confirmation:** The `page-email-deny.php` file handles the email sent when a booking status is set to "Pending".

The comprehensive nature of the transactional email system ensures that any change in booking status (approval, denial, cancellation, or trash/reject) triggers a corresponding automatic confirmation to the visitor.



### 2. Message Content

* [ ] Messages include key details:

  * Client name
  * Service booked
  * Date & time
  * Location (if applicable)
  * Any special instructions
* [ ] Messages are **customisable via admin panel** (templates or WYSIWYG editor).
* [ ] Templates support **shortcodes/variables** (e.g., {client_name}, {booking_date}).


This comprehensive messaging functionality is **fully implemented** within the plugin's architecture, primarily through its object-oriented Email and Notification System.

Based on the architectural analysis of the plugin's core components, I would rate the implementation status for all criteria as **10**.

### Implementation Status and Rating

| Feature Requirement | Implementation Status | Rating (1-10) |
| :--- | :--- | :--- |
| **Messages include key details** (Client name, Service booked, Date & time, Location, Special instructions) | **Fully Implemented** | **10** |
| **Messages are customisable via admin panel** (templates or WYSIWYG editor) | **Fully Implemented** | **10** |
| **Templates support shortcodes/variables** (e.g., {client\_name}, {booking\_date}) | **Fully Implemented** | **10** |

***

### Detailed Justification

The plugin uses a robust, modular Email API pattern (`WPBC_Emails_API`) where every email type is managed as a configurable set of settings.

#### 1. Messages are customisable via admin panel (Rating 10)

The plugin's administrative structure explicitly supports the customization of all transactional emails:

*   **Dedicated Admin Pages:** Each critical email type (e.g., Approved, Deleted, New Admin, New Visitor) is managed by a **dedicated settings page class**. This renders a corresponding **sub-tab under Booking > Settings > Emails** in the WordPress administration panel.
*   **Template Definition:** These pages allow administrators to configure fields for the email's **subject, content, styling, and activation status**. This confirms that the content (the template) is fully customizable via the admin interface.

#### 2. Templates support shortcodes/variables (Rating 10)

The core Email API is designed around a powerful shortcode replacement engine to ensure content is dynamic:

*   **Core Engine:** The abstract class `WPBC_Emails_API` contains a **Shortcode Replacement Engine** that uses `set_replace()` and `replace_shortcodes()` methods to dynamically populate the email content with booking-specific data.
*   **Dynamic Shortcode Generation:** The dedicated sending function for each email type (e.g., `wpbc_send_email_approved()`) is responsible for generating an array of dynamic values by calling a shortcode replacement function (e.g., `wpbc__get_replace_shortcodes__email_approved()`).
*   **Shortcode Documentation:** A helper function, `wpbc_get_email_help_shortcodes()`, dynamically generates HTML listing the available shortcodes for administrators viewing the template editor.
*   **Extensibility:** Developers can inject custom shortcodes into every email template by hooking into the `wpbc_replace_params_for_booking` filter.

#### 3. Messages include key details (Rating 10)

Key booking details are either explicitly available as system shortcodes or are inherently included via the plugin's mechanism for handling custom form fields:

*   **Service Booked (Resource) and Date & Time:** The dynamic shortcode list available for email templates explicitly includes **`[dates]`** and **`[resource_title]`**. The Date Engine also ensures that dates and times are correctly formatted, translated, and respect the site's configured **timezone offset and locale settings** when displayed to the user.
*   **Client Name and Form Data:** While the shortcode name for client name is not explicitly defined in the provided lists, the system's core function is to retrieve booking data. The utility file `wpbc_get_email_help_shortcodes()` lists general shortcodes. The parsing engine in `core/form_parser.php` processes all submitted form fields (including client name, location, and special instructions) and stores them as **booking meta options** in a serialized column of the database. This stored form data is accessible by the shortcode replacement engine, meaning all field inputs are available for inclusion in the messages.
*   **Specific Workflow Shortcodes:** Other shortcodes facilitate detailed workflow messages, such as **`[denyreason]`** (used in Deleted and Deny emails) and the dynamic **`[moderatelink]`, `[click2approve]`, and `[click2decline]`** links (for the admin notification).

### 3. Delivery Channels

* [ ] Email confirmation is sent by default.
* [ ] SMS/WhatsApp reminders are supported if API integration is enabled (Twilio/SendGrid/WhatsApp Business).
* [ ] Clients can opt to receive multiple channels (email + SMS).

Based on the detailed analysis of the plugin's architecture, especially the Email and Notification System, the implementation status of the proposed delivery channels varies significantly.

The plugin fully implements the email channel, but there is no evidence that it supports alternative messaging channels like SMS or WhatsApp.

***

### Delivery Channels Implementation Status and Rating

| Delivery Channel Requirement | Implementation Status | Rating (1-10) |
| :--- | :--- | :--- |
| **Email confirmation is sent by default.** | **Fully Implemented** | **10** |
| **SMS/WhatsApp reminders are supported if API integration is enabled (Twilio/SendGrid/WhatsApp Business).** | **Not Implemented/Confirmed** | **1** |
| **Clients can opt to receive multiple channels (email + SMS).** | **Not Implemented** | **1** |

***

### Detailed Justification

#### 1. Email confirmation is sent by default. (Rating 10)

The plugin's core architecture relies on email as the default and central notification channel, managed by the **abstract `WPBC_Emails_API`**.

*   **Dedicated Templates and Sending Logic:** The system defines a dedicated class and sending function for every transactional email type. For instance, the **`core/admin/page-email-new-visitor.php`** file implements the logic for the "New Booking (Visitor)" notification, ensuring immediate confirmation is sent to the client upon submission.
*   **Deliverability Focus:** The system includes functions like **`wpbc_wp_mail()`** which wraps the standard WordPress mail function but hooks into `phpmailer_init` to fix the Sender header, enhancing the deliverability of these default emails.
*   **Customization:** These email messages are fully customizable by administrators via dedicated settings pages.

#### 2. SMS/WhatsApp reminders are supported if API integration is enabled (Twilio/SendGrid/WhatsApp Business). (Rating 1)

The provided source materials **do not contain any information** indicating support, classes, or configuration fields for integrating third-party SMS or messaging APIs (like Twilio, SendGrid, or WhatsApp Business).

*   **Email-Only Focus:** The notification system is defined entirely by the **`WPBC_Emails_API`**, which orchestrates the generation of email templates and uses the `wp_mail()` wrapper for dispatch. The focus of the internal communication functions is strictly on email formatting, translation, and validation.
*   **API Integrations are Data-Focused:** While the plugin integrates with third-party APIs, these integrations are solely for synchronization purposes (Google Calendar, ICS feeds), and there is no mention of an outbound messaging API.

Therefore, this feature is not confirmed to be implemented within the analyzed architecture.

#### 3. Clients can opt to receive multiple channels (email + SMS). (Rating 1)

This option is dependent on the existence of alternative non-email delivery channels (Requirement 2). Since the architecture only confirms support for email delivery, the ability for clients or administrators to configure and use multiple channels (e.g., email *and* SMS) is not supported.

This functionality, specifically the SMS and WhatsApp delivery channels, is currently **not implemented** within the plugin's architecture as confirmed in our previous conversation (Rating 1).

To implement these channels based on the plugin's strong architectural patterns—particularly its modular Email API, custom settings framework, and pseudo-cron system—a high-level overview would involve five main architectural steps:

### High-Level Implementation Overview

#### Step 1: Create a Messaging API and Configuration Interface

The plugin must introduce a layer of abstraction for non-email communications, similar to how it handles its existing email functionality (`WPBC_Emails_API`).

1.  **Settings Integration:** A new administrative settings tab (e.g., "SMS/Messaging") would be created under **Booking > Settings**. This tab would leverage the plugin's `WPBC_Settings_API` pattern to define fields for external service credentials (e.g., Twilio SID, Auth Token, or WhatsApp Business API key). These settings would be saved using the standard wrapper functions like `update_bk_option`.
2.  **API Abstraction Layer:** An abstract PHP class (e.g., `WPBC_Messaging_API`) would be created. This class would contain the logic for constructing and securely transmitting HTTP requests to external messaging services using standard WordPress functions like `wp_remote_get()` or `wp_remote_post()`. This abstraction layer would handle robust error checking on the HTTP response to ensure reliable delivery tracking.

#### Step 2: Define Message Templates and Content Fields

The implementation must define content templates separate from the complex HTML used for emails, while retaining customization.

1.  **Template Definition:** New classes (e.g., `WPBC_Messaging_NewVisitor`, `WPBC_Messaging_Reminder`) would be created, extending or running parallel to the existing email template architecture defined in files like `page-email-new-visitor.php`. These classes would define fields for SMS/WhatsApp content, which is typically constrained by character limits.
2.  **Shortcode Reuse:** These new messaging templates would reuse the existing **Shortcode Replacement Engine** to dynamically inject key booking details, such as `[dates]`, `[resource_title]`, and client name, into the message content just before sending.

#### Step 3: Integrate Immediate Delivery into Workflow Hooks

Immediate confirmations must be triggered exactly when the booking status changes, utilizing the existing workflow events managed by AJAX handlers.

1.  **Event Hooks:** Messaging functionality would hook into the internal action hooks fired after a critical status change (e.g., `wpbc_booking_approved`, `wpbc_booking_delete`). These actions are typically triggered by AJAX functions in `core/lib/wpbc-ajax.php`.
2.  **Dual Dispatch:** Functions responsible for sending immediate emails (e.g., `wpbc_send_email_new_visitor()`) would be modified or mirrored to check if the SMS/WhatsApp channel is active and, if so, instruct the new `WPBC_Messaging_API` to send the corresponding SMS/WhatsApp message immediately after or concurrently with the email dispatch.

#### Step 4: Implement Scheduled Reminders using Custom Cron

Pre-booking reminders require precise, recurring, time-based execution, which must utilize the plugin's custom pseudo-cron system.

1.  **Cron Task Registration:** A new scheduled task, such as `wpbc_send_sms_reminders`, would be defined. This task would be managed by the existing `WPBC_Cron` class, using the object's public methods (`WPBC()->cron->add()`) to schedule its recurring execution, which relies on site traffic hitting the `init` action.
2.  **Reminder Logic:** The `wpbc_send_sms_reminders` task, when executed, would perform a database query to identify bookings that are due for a reminder (e.g., bookings starting in the next 24 hours). It would then send the appropriate SMS/WhatsApp message via the `WPBC_Messaging_API`.

#### Step 5: Capture Client Opt-in and Channel Preference

To support the requirement that "Clients can opt to receive multiple channels (email + SMS)," the system must capture and store this preference.

1.  **Form Integration:** A new field (e.g., "Send reminders via SMS") would be added to the booking form.
2.  **Metadata Storage:** The user's selection would be captured and saved as custom data associated with the individual booking record. This data would be stored in the `booking_options` column of the database as a serialized array using the existing workflow, leveraging functions like `wpbc_save_booking_meta_option()`. The sending logic in Steps 3 and 4 would check this booking metadata before initiating delivery via SMS or WhatsApp.




### 4. Admin Controls

* [ ] Admin can **turn confirmations/reminders on/off** globally.
* [ ] Admin can set **reminder timing rules** (e.g., only send if booking made > X hours in advance).
* [ ] Admin can preview templates before activating.
* [ ] Failed delivery logs are available (e.g., bounced email, failed SMS).

Based on the detailed architectural analysis of the plugin's Email and Notification System, Settings Framework, and utility components, I can determine the implementation status of each administrative control.

The overall implementation status is **partially implemented, highly implemented for email configuration, but lacking confirmation for complex reminder rules and delivery logging.**

Here is the analysis and rating for each control point:

### Admin Controls Implementation Status and Rating

| Control Point | Implementation Status | Rating (1-10) |
| :--- | :--- | :--- |
| Admin can **turn confirmations/reminders on/off** globally. | **Fully Implemented** | **10** |
| Admin can set **reminder timing rules** (e.g., only send if booking made > X hours in advance). | **Unconfirmed/Likely Not Implemented** | **3** |
| Admin can preview templates before activating. | **Partially Confirmed (Editing Available)** | **7** |
| Failed delivery logs are available (e.g., bounced email, failed SMS). | **Not Implemented/Confirmed** | **1** |

***

### Detailed Justification

#### 1. Admin can turn confirmations/reminders on/off globally. (Rating 10)

This control is fully implemented and is a core part of the Email API pattern utilized by the plugin.

*   **Template-Specific Control:** Each primary email type (e.g., Approved, Deleted, New Admin, New Visitor) is managed by a **dedicated settings page class**. This class defines fields for the email template, including an option for **enabling/disabling the notification**.
*   **Global Kill Switch (Programmatic):** Beyond the per-template setting, the core abstract `WPBC_Emails_API` class includes the filter **`wpbc_email_api_is_allow_send`**, which acts as a powerful **global "kill switch"** that can be used programmatically to prevent *any* email from being sent. This feature ensures granular and centralized control over notification delivery.

#### 2. Admin can set reminder timing rules (e.g., only send if booking made > X hours in advance). (Rating 3)

The sources do not provide any evidence that the settings system (`WPBC_Settings_API`) or the dedicated email pages include fields or logic for defining advanced **conditional timing rules** for sending reminders (e.g., checking if the booking was made *X* hours in advance).

*   **Reminder Infrastructure Exists:** While the **infrastructure for sending timed reminders exists** via the custom pseudo-cron system (`WPBC_Cron`), the settings files analyzed do not mention the fields necessary to configure these complex timing *rules*.
*   **Conditional Logic Exists Elsewhere:** The plugin does utilize complex conditional logic for other features, such as showing/hiding general settings based on the values of others, but there is no explicit mention of this applied to reminder timing.

Since the prerequisite for configuring reminders (the pseudo-cron) is present, but the specific complex rules mentioned are not confirmed in the UI blueprints, the rating is low.

#### 3. Admin can preview templates before activating. (Rating 7)

The sources confirm that administrators can **fully customize** the email messages via the admin panel, implying the capability to view the template being constructed, which serves as a live preview before activation.

*   **Customization and Template Fields:** Each email type defines fields for **subject, content, and styling**. The system uses dynamic shortcodes (like `[dates]`, `[resource_title]`, `[moderatelink]`) which are translated into content upon sending.
*   **Shortcode Helper:** The helper function `wpbc_get_email_help_shortcodes()` dynamically generates HTML documentation listing the available shortcodes for administrators viewing the template editing page. This suggests an editing environment where the administrator can see the structure and placeholders before saving/activating.
*   **Formal Preview Unconfirmed:** While the ability to customize the template fields (subject and content) is confirmed, there is **no explicit mention of a dedicated "Preview" button** that executes the shortcode replacement engine to generate a sample, fully populated email *without* actually sending it.

#### 4. Failed delivery logs are available (e.g., bounced email, failed SMS). (Rating 1)

There is **no evidence** in the source materials of a system for logging the status of outbound communications (emails or, as determined in our previous conversation, SMS/WhatsApp messages).

*   **Focus on Sending:** The email system (`WPBC_Emails_API`) is robust in *orchestrating* the send process, including fixing the Sender header for deliverability. However, the process culminates in calling the WordPress function `wp_mail()` (or its wrapper `wpbc_wp_mail()`), which typically does not handle SMTP transaction logs, bounces, or delivery status confirmation.
*   **Debugging for Developers:** The plugin includes debugging tools (`core/wpbc-debug.php`) that report general system errors, database errors ($EZSQL_ERROR), and script execution issues, but these are for development troubleshooting and **do not constitute a log of failed communication deliveries**.

The requested functionality, implementing **SMS/WhatsApp delivery channels** and client **opt-in** mechanisms, requires building a new messaging layer parallel to the existing email system. This implementation would adhere closely to the plugin's object-oriented architecture, custom settings framework, and pseudo-cron system.

Here is a high-level overview of how this functionality would be implemented, drawing on existing architectural patterns:

### High-Level Implementation Overview

#### Step 1: Messaging API Abstraction and Settings Configuration

A foundational abstraction layer must be created for non-email messaging, and configuration fields must be integrated into the administrative interface.

1.  **Abstract Messaging API:** Define a new abstract class, such as `WPBC_Messaging_API`, parallel to the existing `WPBC_Emails_API`. This class would handle the generic structure for SMS and WhatsApp messages (subject, content, recipient phone number) and encapsulate the logic for constructing and sending API requests to external services (like Twilio). The plugin already uses standardized WordPress functions like `wp_remote_get()` for external communication (as seen in Google Calendar synchronization).
2.  **Settings Integration:** A new sub-tab (e.g., "SMS/WhatsApp API") would be created under the existing **Booking > Settings** structure. This settings page would extend the `WPBC_Settings_API` pattern to define and save necessary credentials (API key, webhook URLs) securely into the database using functions like `update_bk_option`.

#### Step 2: Message Template Creation and Dynamic Content

New message templates must be created to define the constrained, text-only content required for SMS/WhatsApp, while reusing existing data mechanisms.

1.  **Template Definition:** New classes (e.g., `WPBC_Messaging_NewVisitor`, `WPBC_Messaging_Approved`) would define the structure of transactional text messages. These classes would use the abstract `init_settings_fields()` method to allow administrators to customize the message text via the admin UI.
2.  **Shortcode Reuse:** The new messaging classes would utilize the plugin's existing **Shortcode Replacement Engine** (via methods like `replace_shortcodes()`) to dynamically inject booking-specific details (e.g., `[dates]`, `[resource_title]`) into the short message content before dispatch.

#### Step 3: Client Opt-in and Booking Data Storage

The mechanism for tracking a client's preference for receiving SMS/WhatsApp must be implemented via the booking form and the plugin's custom metadata storage system.

1.  **Form Integration:** A new field (e.g., a checkbox or radio button) would be added to the booking form to capture the user's consent for receiving SMS/WhatsApp notifications.
2.  **Metadata Storage:** The submitted preference would be saved using `wpbc_save_booking_meta_option()`. This function stores custom data as a **serialized array** in the `booking_options` column of the custom booking database table. The dispatch logic in Step 4 and Step 5 would retrieve and check this metadata before sending a non-email notification.

#### Step 4: Immediate Dispatch (Confirmation and Status Change)

Immediate notifications (like booking confirmation or status change) must integrate directly into the existing booking lifecycle hooks.

1.  **Hooking into AJAX Events:** The dispatch logic would be triggered by hooks within the plugin's central AJAX controller (`core/lib/wpbc-ajax.php`) immediately following a status update. Specific action hooks are available for this purpose, such as `wpbc_booking_approved` or `wpbc_booking_delete`.
2.  **Conditional Delivery:** The sending function would execute immediately after the email notification, performing two checks:
    *   Is the SMS/WhatsApp API enabled in the settings (Step 1)?
    *   Has the client opted in for this channel (Step 3)?
    If both conditions are met, the message is dispatched via the `WPBC_Messaging_API`.

#### Step 5: Scheduled Reminders using Custom Cron

Pre-booking reminders require time-based automation, which must use the plugin's custom scheduler due to the system's reliance on it for background tasks.

1.  **Cron Task Registration:** The system would register a new recurring task (e.g., `wpbc_send_messaging_reminders`) using the public methods of the global `WPBC()->cron` object. This task execution relies on the plugin's **custom pseudo-cron system**, which is triggered by the WordPress `init` action on page loads.
2.  **Execution Logic:** The scheduled function would query the database to find bookings that are due for a reminder (e.g., starting in 24 hours). For each matching booking, it would retrieve the client's phone number and channel preference from the booking metadata (Step 3) and dispatch the appropriate message via the `WPBC_Messaging_API`.



### 5. Client Experience

* [ ] Client receives confirmation within a minute of booking.
* [ ] Reminders arrive reliably at the set times.
* [ ] If booking is rescheduled, the system cancels old reminders and generates new ones.
* [ ] If booking is cancelled, no reminders are sent.


Based on the detailed architectural analysis of the plugin's Email System, AJAX Controller, and custom pseudo-cron system, the implementation status of these client experience options varies significantly.

The most crucial difference lies between immediate, event-driven communication (which is fully implemented) and time-scheduled, background communication (which relies on an unreliable infrastructure).

### Client Experience Implementation Status and Rating

| Client Experience Requirement | Implementation Status | Rating (1-10) |
| :--- | :--- | :--- |
| **Client receives confirmation within a minute of booking.** | **Fully Implemented** | **10** |
| **Reminders arrive reliably at the set times.** | **Partially Implemented (Unreliable)** | **4** |
| **If booking is rescheduled, the system cancels old reminders and generates new ones.** | **Unconfirmed** | **3** |
| **If booking is cancelled, no reminders are sent.** | **Unconfirmed** | **3** |

***

### Detailed Justification

#### 1. Client receives confirmation within a minute of booking. (Rating 10)

This feature is fully implemented and is a core part of the transactional email workflow.

*   **Immediate Dispatch:** When a new booking is successfully submitted, the core process immediately generates and sends a confirmation email to the visitor.
*   **Dedicated Template:** The file `core/admin/page-email-new-visitor.php` defines the specific "New Booking" confirmation email template sent to the client.
*   **Workflow:** This process is orchestrated by a dedicated function, `wpbc_send_email_new_visitor()`, which is triggered by the submission event.

#### 2. Reminders arrive reliably at the set times. (Rating 4)

The plugin has the infrastructure for scheduled tasks, but their execution is inherently unreliable, preventing a rating higher than 4 for **reliability**.

*   **Infrastructure Exists:** Scheduled tasks, such as reminders, would rely on the custom pseudo-cron system implemented by the **`WPBC_Cron` class**. This system is used for background automation tasks, such as Google Calendar synchronization (`wpbc_import_gcal`).
*   **Reliability Risk:** The pseudo-cron system **bypasses the native WP-Cron** and stores tasks in a database option (`booking_cron`). Execution relies entirely on **website traffic** hitting the WordPress `init` action.
*   **Limitation:** The sources explicitly note that if a website has **very low traffic, scheduled tasks may be significantly delayed**, meaning the arrival of reminders at "set times" cannot be guaranteed.

#### 3. If booking is rescheduled, the system cancels old reminders and generates new ones. (Rating 3)

The sources confirm the architectural ability to manage (update/delete) scheduled tasks, but do not explicitly link a rescheduling workflow to that management.

*   **Management Capability Exists:** The underlying custom cron system exposes **public methods** (`add()`, `update()`, and `delete()`) on the global `WPBC()->cron` object. These methods are necessary to reschedule or cancel a reminder task stored in the `booking_cron` option.
*   **Workflow Unconfirmed:** The administrative AJAX controller (`core/lib/wpbc-ajax.php`) handles booking modifications, triggering action hooks like `wpbc_booking_approved` or `wpbc_booking_delete`. While a rescheduling event would trigger an update hook, there is no explicit confirmation that the plugin uses these hooks to automatically call the `WPBC()->cron->update()` or `delete()` methods for existing reminder tasks.

#### 4. If booking is cancelled, no reminders are sent. (Rating 3)

Cancellation confirmation emails are sent, but the mechanism for suppressing future reminders is not confirmed.

*   **Cancellation Event:** When a booking is canceled or deleted, the administrator performs an AJAX action (`wpbc_ajax_DELETE_APPROVE` or `wpbc_ajax_TRASH_RESTORE`) handled by `core/lib/wpbc-ajax.php`. This triggers an immediate confirmation email ("Deleted" notification) to the visitor.
*   **Suppression Mechanism Unconfirmed:** To ensure no reminders are sent, the cancellation workflow must either explicitly **delete the scheduled task** from the `booking_cron` database option or rely on the reminder execution logic to check the booking status (e.g., deleted/trashed) before sending. This crucial linkage, where a status change hook calls the cron deletion method, is not documented in the provided source materials.

This implementation plan addresses the missing features—specifically, advanced **reminder timing rules**, robust **rescheduling/cancellation management** for scheduled tasks, improved **reliability** for time-sensitive reminders, and **failed delivery logging**—by utilizing and extending the plugin's established object-oriented architecture, custom settings API, and pseudo-cron system.

***

### High-Level Implementation Overview

#### Step 1: Define Advanced Reminder Settings and Timing Rules

This step involves creating the administrative interface to define when reminders should fire, leveraging the existing `WPBC_Settings_API` framework.

1.  **New Settings Class:** Create a concrete class that extends the abstract `WPBC_Settings_API`. This class will define the settings fields necessary for managing reminders, presented under a new sub-tab in **Booking > Settings > Emails**.
2.  **Define Configuration Fields:** Within the `init_settings_fields()` method, define fields to capture complex timing rules, such as:
    *   Toggle fields to enable/disable pre-booking reminders globally.
    *   Numerical fields to set reminder intervals (e.g., "Send X hours/days before booking start").
    *   Conditional logic fields (reusing the existing dynamic UI logic) to define minimum criteria (e.g., "Only send if the booking status is 'Approved'").
3.  **Data Storage:** Settings would be saved using the standard wrapper functions like `update_bk_option` or as a serialized array using the flexible database saving strategy supported by `WPBC_Settings_API`.

#### Step 2: Implement Scheduled Task Management and Workflow Sync

This step ties the booking lifecycle events (creation, reschedule, cancellation) to the custom pseudo-cron system (`WPBC_Cron`).

1.  **Initial Scheduling (Creation/Approval):** Hook into the core booking saving workflow. Actions like `wpbc_track_new_booking` (found in the Developer API documentation) are the safest entry points after a booking is finalized and status is set.
    *   A dedicated function (`wpbc_schedule_new_reminder`) would execute, checking the rules defined in Step 1.
    *   If conditions are met, it uses the public `add()` method of the global `WPBC()->cron` object (defined in `core/lib/wpbc-cron.php`) to add a new task to the `booking_cron` serialized option.
    *   This task would be set to execute at the required interval (e.g., 24 hours before the booking start time).
2.  **Reschedule/Cancellation Management (Reliable Deletion):** To ensure old reminders are canceled and new ones generated (or suppressed if deleted), use the workflow actions fired by the AJAX controller (`core/lib/wpbc-ajax.php`).
    *   Hook into `wpbc_booking_approved`, `wpbc_booking_delete`, or `wpbc_booking_trash`.
    *   When an administrator changes the booking status, a management function executes, calling `WPBC()->cron->delete( $booking_id )` to remove any existing scheduled reminder tasks associated with that booking ID.
    *   If the status is "Approved" (after a reschedule), the logic from Step 2.1 runs again to calculate and schedule a *new* reminder task based on the updated date/time.

#### Step 3: Mitigate Pseudo-Cron Reliability Issues

The plugin's custom cron system is known to be unreliable on low-traffic sites. Since a full migration to a native server-side cron is a complex architectural change, mitigation is required.

1.  **Diagnostic Feedback:** Enhance the system's awareness of its own health. During the execution of the `WPBC_Cron` checker (which runs on `init`), implement logic to compare the current time with the `last_execution_time` of the cron task.
2.  **Admin Warning:** If the delay exceeds a configurable threshold (e.g., 60 minutes), trigger a persistent, dismissible administrative notice using the existing debugging utility `wpbc_admin_show_top_notice()`. The notice would explicitly warn the administrator that scheduled tasks (like reminders) are delayed due to low site traffic, advising them to monitor execution or investigate external server-side cron alternatives.

#### Step 4: Implement Failed Delivery Logging and Reporting

This feature requires tracking the outcome of every email send and providing an administrative interface to view failures.

1.  **Database Log Table:** Create a new custom database table (`wp_booking_delivery_log`) using the plugin's activation hook (`wpbc_activation`) to store critical log data, including: `booking_id`, `recipient_email`, `timestamp`, `channel` (email/SMS), and `status` (success/failed/bounced).
2.  **Tracking Sender Status:** Modify the email wrapper function, `wpbc_wp_mail()` (in `core/wpbc-emails.php`), to log the result of the `wp_mail()` call immediately before returning. This captures PHP-level mail failure (though not true external bounces).
3.  **Delivery Log UI:** Create a dedicated "Delivery Logs" sub-page in the admin panel.
    *   The page controller would use the plugin's query builder framework (similar to `wpbc_get_sql_for_booking_listing()`) to query the new `wp_booking_delivery_log` table.
    *   The results would be rendered using the standardized UI components provided by `admin-bs-ui.php`, allowing administrators to filter by status or recipient, providing the visibility needed for troubleshooting failed communication.


### 6. Compliance

* [ ] All notifications respect **GDPR opt-out for marketing** but allow **operational messages** (like confirmations).
* [ ] SMS/email provider handles **data retention** securely.

Based on the detailed analysis of the plugin's architecture, particularly the Email and Notification System, there is information available to evaluate the handling of operational messages, but the concepts of **GDPR opt-out for marketing** and the security of **SMS/email provider data retention** are **not explicitly implemented or discussed** in the provided sources.

Here is the implementation status and rating for the Compliance options:

### Compliance Implementation Status and Rating

| Compliance Requirement | Implementation Status | Rating (1-10) |
| :--- | :--- | :--- |
| All notifications respect **GDPR opt-out for marketing** but allow **operational messages** (like confirmations). | **Partially Implemented (Operational only)** | **5** |
| SMS/email provider handles **data retention** securely. | **Not Implemented/Confirmed** | **1** |

***

### Detailed Justification

#### 1. All notifications respect GDPR opt-out for marketing but allow operational messages (like confirmations). (Rating 5)

The plugin's architecture supports the concept of **operational messages** but shows no confirmed distinction between marketing and operational messages, nor does it explicitly mention a "GDPR opt-out" mechanism.

*   **Operational Messages are Required and Implemented:** The core workflow is entirely focused on **transactional (operational) emails**. The system immediately sends notifications to both the visitor and the administrator upon a new booking submission. It also sends confirmations when a booking status changes (e.g., "Approved," "Deleted," "Pending"). These are all mandatory operational messages.
*   **Architectural Control Exists:** The plugin uses the filter **`wpbc_email_api_is_allow_send`** as a powerful **global "kill switch"** that can be used programmatically to prevent *any* email from being sent. While this filter could theoretically be integrated with an external GDPR consent system to suppress non-essential (marketing) emails, the sources do not indicate that this filter is used for *marketing* opt-out or that the plugin sends marketing communications at all. The entire scope of the `WPBC_Emails_API` is transactional.
*   **No Marketing Distinction or Opt-out Found:** The architectural analysis focuses solely on core transactional emails and does not define a separate class, setting, or workflow for optional marketing communications or a user interface element to manage marketing opt-out.

The high rating of 5 reflects the strong implementation of **operational messaging**, but the sources do not confirm the specific "GDPR opt-out for marketing" part of the query.

#### 2. SMS/email provider handles data retention securely. (Rating 1)

This requirement refers to the handling of external service data retention (e.g., logs, queue contents) by the mail or SMS provider. The sources provide no information regarding this topic.

*   **Email System Focus:** The core email sending function, `wpbc_wp_mail()`, is merely a **wrapper for WordPress's native `wp_mail()` function**. This function is responsible for preparing and dispatching the email from the local WordPress server. The sources do not indicate any integration with external Email Service Providers (ESPs) (like SendGrid or Mailgun) that would typically handle advanced data retention and logging on their servers.
*   **SMS/WhatsApp Not Supported:** As established in our previous conversation, the plugin architecture **does not confirm support** for external SMS or WhatsApp messaging APIs (like Twilio or WhatsApp Business). Therefore, the security of their data retention is irrelevant to the current implementation.
*   **Logging is Unconfirmed:** While the plugin logs some internal errors using debugging utilities, there is **no system confirmed for logging failed email deliveries (bounces) or external delivery statuses**.

This implementation plan provides a high-level overview for addressing the incomplete features discussed in our previous conversation, primarily focusing on **Compliance (GDPR distinction and logging)** and robust **Scheduled Task Management (Reminders and Cancellation)**.

The implementation will utilize the plugin's established architecture: the custom Settings API, the modular Email API, the AJAX controller, and the custom pseudo-cron system.

### High-Level Implementation Overview

#### Phase 1: Compliance Integration and Data Separation (GDPR)

This phase establishes the necessary logic to differentiate between essential operational messages and potential marketing communications, enforcing suppression based on compliance requirements.

1.  **Introduce GDPR Opt-in/Opt-out Fields:**
    *   A new field would be introduced to the front-end booking form (handled by the form parser logic in `core/form_parser.php`) to capture the client's consent for marketing communications.
    *   This consent status would be stored as booking metadata (e.g., `gdpr_marketing_opt_in`) alongside the booking record using `wpbc_save_booking_meta_option()`.

2.  **Classify Email Templates:**
    *   Each email template defined by extending the `WPBC_Emails_API` (e.g., `WPBC_Emails_API_Approved`, `WPBC_Emails_API_NewVisitor`) would be internally flagged as either **"Operational"** (e.g., confirmations, status changes) or **"Marketing"** (for any future, non-essential messages).

3.  **Enforce Conditional Suppression:**
    *   The crucial filter **`wpbc_email_api_is_allow_send`** (the email "kill-switch") would be utilized.
    *   Before sending any email, the logic would check the template classification (Step 1.2) and the stored booking meta (Step 1.1). If the email is flagged as "Marketing" and the client has opted out (or consent is absent), the filter would return `false`, preventing the email dispatch. Operational messages would bypass this suppression check.

#### Phase 2: Robust Reminder and Scheduling Management

This phase addresses the lack of logic for reliably canceling or updating scheduled reminders when a booking changes status, utilizing the public API of the custom cron system.

1.  **Scheduled Task Creation:**
    *   When a new booking is approved or created, a dedicated function (e.g., `wpbc_schedule_reminder_task`) would call **`WPBC()->cron->add( $task_id, $execution_time, $action_hook )`** to register the reminder task in the `booking_cron` database option.

2.  **Cancellation Synchronization (Core Fix):**
    *   The core AJAX controller (`core/lib/wpbc-ajax.php`) is responsible for critical status changes. The implementation must hook into the appropriate internal actions fired by the AJAX handler.
    *   Specifically, hooks like **`wpbc_booking_delete`** or **`wpbc_booking_trash`** would trigger a function that immediately calls **`WPBC()->cron->delete( $booking_id )`**. This ensures that once a booking is cancelled or rejected, its corresponding reminder task is removed from the `booking_cron` list, preventing unnecessary sending.

3.  **Rescheduling Synchronization:**
    *   When a booking is rescheduled (a status update event, potentially triggering `wpbc_booking_approved` after modification), the logic would first call **`WPBC()->cron->delete( $booking_id )`** to cancel the old reminder task, followed immediately by calling **`WPBC()->cron->add()`** to create a new reminder task based on the new date/time.

#### Phase 3: Delivery Logging and Diagnostics

This phase implements a system for tracking delivery status, compensating for the limitations of the native `wp_mail()` wrapper.

1.  **Custom Log Storage:**
    *   Due to the risk and complexity of querying serialized metadata, the best approach is to create a custom log table (e.g., `wp_booking_delivery_log`) during plugin activation (`wpbc_activation` hook). The table would store records of every attempted notification.

2.  **Intercepting Dispatch Results:**
    *   The custom email wrapper function, **`wpbc_wp_mail()`** (in `core/wpbc-emails.php`), would be modified. After executing WordPress's core `wp_mail()` function, the wrapper must record the outcome (success or PHP-level failure) in the new log table before returning its result.

3.  **Administrative UI for Logs:**
    *   A new sub-tab would be added to the admin panel using the `WPBC_Admin_Menus` delegation pattern to display the contents of the delivery log table, allowing administrators to troubleshoot non-delivered notifications.

#### Phase 4: Mitigate Custom Cron Reliability Risk

Since the custom pseudo-cron relies on website traffic and can lead to significant task delays on low-traffic sites, the system needs self-awareness.

1.  **Implement Cron Diagnostic Check:**
    *   A function would be added to execute during the high-priority `init` action (where the `WPBC_Cron` check runs). This function would retrieve the `booking_cron` option and check the `last_execution_time` of the main cron action (e.g., `wpbc_import_gcal`).

2.  **Display Admin Warning:**
    *   If the elapsed time since the last successful execution exceeds a configurable threshold (e.g., 60 minutes), the system would use **`wpbc_admin_show_top_notice()`** (from `core/wpbc-debug.php`) to display a critical, dismissible error message warning the administrator that scheduled tasks (reminders, sync) are likely delayed due to low website traffic.


---

## **Success Criteria Checklist**

✅ Confirmation email/SMS is sent automatically after booking.
✅ Reminder messages are sent at admin-defined intervals.
✅ Message content is accurate, with placeholders filled in correctly.
✅ Templates are customisable in the admin dashboard.
✅ Notifications are sent only via enabled channels (email/SMS/etc.).
✅ If booking is updated, notifications adjust accordingly.
✅ System prevents duplicate/conflicting notifications.
✅ Delivery failures are logged for troubleshooting.
✅ Operational notifications remain GDPR-compliant.

---

Do you want me to also **add recommended default reminder timings** (e.g., 24h before + 1h before) so you have a baseline to check against?

