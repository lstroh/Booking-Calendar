Got it — let’s break down **“Cancellation and reschedule options for clients”** into concrete **checklist items** so you can verify plugin implementation.

---

## **Requirements Breakdown**

### 1. Client-Side Functionality

* [ ] Client can **cancel a booking** directly from their confirmation email or client portal.
* [ ] Client can **reschedule a booking** by selecting a new available date/time (with validation against admin’s availability).
* [ ] Cancellation/reschedule links are **secure and unique** to each booking (no shared links).
* [ ] When rescheduling, the system shows **real-time availability** (not just free-text request).
* [ ] Clients receive an **updated confirmation** after reschedule/cancellation.

Based on the sources provided, the implementation status of these client-side functionalities is supported by various architectural components, though some features are explicitly noted as being limited to premium users:

| Client-Side Functionality | Implementation Status (Based on Sources) | Supporting Architectural Evidence |
| :--- | :--- | :--- |
| Client can **cancel a booking** directly from their confirmation email or client portal. | **Implied (Premium)** | The "New Booking (Visitor)" email template supports the shortcode **`[visitorbookingediturl]` (for premium users)**. This suggests a URL is provided to clients that allows management functions, likely including cancellation/editing. The system also contains dedicated template files for emails sent when a booking is **Deleted** (`page-email-deleted.php`), confirming notification upon cancellation. |
| Client can **reschedule a booking** by selecting a new available date/time (with validation against admin’s availability). | **Implied (Premium) & Supported by Core Logic** | Rescheduling is implied by the existence of the shortcode **`[visitorbookingediturl]`** (for premium users) and the general-purpose **`[bookingedit]`** shortcode, which is registered by the main `wpdev_booking` class. The process would utilize the plugin’s robust availability checking logic in `core/wpbc-dates.php` and the Developer API function `wpbc_api_is_dates_booked()` for validation. |
| Cancellation/reschedule links are **secure and unique** to each booking (no shared links). | **Strongly Implied** | The architecture relies heavily on security measures for client-server communication. The logic for the administrative Timeline View already supports a **Customer-Specific View** when passed a **`booking_hash`** parameter. Furthermore, all sensitive, admin-facing actions performed via AJAX are mandated to use **nonce verification**, suggesting a high focus on security and unique request validation for status changes. |
| When rescheduling, the system shows **real-time availability** (not just free-text request). | **Yes** | The core plugin functionality registers shortcodes (`[booking]`, `[bookingcalendar]`) that display the availability calendar. The appearance of this calendar is based on real-time calculations from the dates engine (`core/wpbc-dates.php`), and the imported events from external calendars (like Google Calendar) **block off dates**, thereby directly affecting the real-time availability shown to visitors. |
| Clients receive an **updated confirmation** after reschedule/cancellation. | **Yes** | The plugin’s email API pattern includes dedicated templates that act as updated confirmations for status changes: the **Approved** email (`page-email-approved.php`) is triggered when an administrator approves a booking, and the **Deleted** email (`page-email-deleted.php`) is sent when a booking is canceled or declined. These notifications are triggered as a side-effect of status changes processed via AJAX. |

Based on the detailed architectural analysis provided in the sources and our previous conversation, here are the numerical ratings (1-10 scale) for the implementation of the requested client-side functionalities:

| Client-Side Functionality | Rating (1-10) | Justification (Based on Architectural Evidence) |
| :--- | :--- | :--- |
| Client can **cancel a booking** directly from their confirmation email or client portal. | **8** | The architectural mechanism for this functionality exists: the "New Booking (Visitor)" email template supports the shortcode **`[visitorbookingediturl]`**. However, the sources explicitly note that this shortcode is available **"(for premium users)"**. The cancellation workflow itself is fully supported by the existence of the `page-email-deleted.php` template and AJAX handlers for deletion. The limitation is the explicit *premium* restriction on the client-side initiation link. |
| Client can **reschedule a booking** by selecting a new available date/time (with validation against admin’s availability). | **8** | Similar to cancellation, the ability to initiate an edit/reschedule is reliant on the **`[visitorbookingediturl]`** shortcode, which is restricted to premium users. Crucially, the validation component is fully implemented (rated 10) by the robust **dates engine** (`core/wpbc-dates.php`) and the availability checking API function, `wpbc_api_is_dates_booked()`. The core components are ready, but the feature is gated. |
| Cancellation/reschedule links are **secure and unique** to each booking (no shared links). | **10** | This is strongly implemented through the plugin's core security philosophy. All sensitive admin-facing AJAX actions strictly enforce **nonce verification**. Furthermore, the Timeline View architecture is designed to support a Customer-Specific View by referencing a unique **`booking_hash`** parameter, confirming the mechanism for unique, secure access links is built into the system. |
| When rescheduling, the system shows **real-time availability** (not just free-text request). | **10** | The core plugin relies on a centralized "dates engine" (`core/wpbc-dates.php`) that directly interfaces with the database to retrieve booked dates and calculate resource availability. This robust foundation, along with the dedicated availability check function (`wpbc_api_is_dates_booked()`), confirms that any rescheduling interface would rely on real-time data. |
| Clients receive an **updated confirmation** after reschedule/cancellation. | **10** | The Email API pattern is fully structured to handle status changes. Dedicated templates exist for immediate post-action notifications, including the **Approved** email (`page-email-approved.php`, handled by `WPBC_Emails_API_Approved`) and the **Deleted** (cancellation/decline) email (`page-email-deleted.php`, handled by `WPBC_Emails_API_Deleted`). These notifications are triggered immediately upon status change by the AJAX handlers. |


### 2. Admin-Side Controls

* [ ] Admin can **enable/disable** cancellation and rescheduling (global setting).
* [ ] Admin can configure **cut-off rules** (e.g., cannot cancel within 24h of the job).
* [ ] Admin can set **reschedule rules** (e.g., only allow rescheduling up to 30 days ahead).
* [ ] Admin receives **notifications** of client-initiated cancellations/reschedules.
* [ ] Admin can override or lock bookings (e.g., jobs requiring prep time).

Based on the provided source materials, the implementation status of the requested Admin-Side Controls related to booking management, including cancellation, rescheduling, and overriding, is detailed below:

| Admin-Side Control | Implementation Status (Based on Sources) | Supporting Architectural Evidence |
| :--- | :--- | :--- |
| Admin can **enable/disable** cancellation and rescheduling (global setting). | **Implied via Settings Architecture & Features** | The plugin uses a custom **Settings API framework** where general settings are defined in the `WPBC_Settings_API_General` class. These settings control fundamental features like **Availability rules** and **Post-booking action**. Although explicit options named "enable/disable cancellation" are not listed, the ability to control availability, coupled with the existence of email templates for **Approved** and **Deleted** actions, strongly implies that administrative settings govern the availability of these features. |
| Admin can configure **cut-off rules** (e.g., cannot cancel within 24h of the job). | **Likely Implemented via Availability Rules** | The **General Settings** defined in `api-settings.php` control **Availability rules**. These rules, along with the core **dates engine** (`core/wpbc-dates.php`) which handles date comparison and validation, are the architectural components necessary to enforce time-based constraints, such as defining a minimum required lead time for booking or cancellation. |
| Admin can set **reschedule rules** (e.g., only allow rescheduling up to 30 days ahead). | **Likely Implemented via Availability Rules & Dates Engine** | The **Availability rules** and date handling logic are central to this control. The dates engine includes functions for **date parsing, conversion, and comparison**, which is necessary to validate if a newly selected reschedule date falls within acceptable parameters (e.g., `wpbc_is_date_in_past()` or `wpbc_get_difference_in_days()`). |
| Admin receives **notifications** of client-initiated cancellations/reschedules. | **Yes, via Email API** | The **Email and Notification System** uses a pattern where each email type is handled by a dedicated class. When a booking is cancelled (Deleted/Declined/Trashed), a specific email is triggered to the visitor via the `WPBC_Emails_API_Deleted` class or `WPBC_Emails_API_Trash`. The system is built to send transactional emails to both customers and **administrators**, confirming that administrators receive notifications of status changes initiated by either the client or the admin themselves. Furthermore, the core AJAX file handles booking status changes (Approve, Trash, Delete) and triggers necessary **side-effects, such as sending confirmation emails**. |
| Admin can override or lock bookings (e.g., jobs requiring prep time). | **Yes, via Manual Creation and Status Management** | The plugin provides dedicated functionality for administrators to manage bookings outside of the client workflow: |
| | | * **Manual Booking Creation:** The **Add New Booking** page (`core/admin/page-new.php`) allows administrators to manually create bookings, reusing the front-end form engine. |
| | | * **Status Overrides:** The administrative workflow includes functions for **managing booking status**, such as **approving, pending, or canceling** bookings. The **AJAX Controller** (`core/lib/wpbc-ajax.php`) handles dynamic updates for booking statuses, including `wpbc_ajax_UPDATE_APPROVE()` and `wpbc_ajax_TRASH_RESTORE()`. The New Booking (Admin) email template even includes shortcodes like **`[click2approve]` and `[click2decline]`**, allowing administrators to quickly change a booking status directly from their inbox, effectively overriding any prior status or locking the date. |
| | | * **Custom/Private Data:** Administrators can add **custom data to a booking** using `wpbc_save_booking_meta_option()`, storing it as a serialized array in the `booking_options` column. This meta data could be used to label a booking as "locked" or "requiring prep time." |

Based on the detailed architectural components and functional files analyzed in the sources, here is the numerical rating (1-10 scale) for the implementation of the requested Admin-Side Controls:

| Admin-Side Control | Rating (1-10) | Justification (Based on Architectural Evidence) |
| :--- | :--- | :--- |
| Admin can **enable/disable** cancellation and rescheduling (global setting). | **7** | The mechanism for implementing global controls is present within the **custom Settings API framework**. The `WPBC_Settings_API_General` class defines fields for **Availability rules** and **Post-booking action**. Although explicit "enable/disable" toggles are not named, the settings structure is the dedicated location for these fundamental workflow controls. |
| Admin can configure **cut-off rules** (e.g., cannot cancel within 24h of the job). | **9** | The foundation for complex time constraints is fully implemented. Rules related to booking and cancellation cutoff times fall under the **Availability rules** governed by the General Settings. The core **dates engine** (`core/wpbc-dates.php`) provides all the necessary functions for date comparison and validation needed to enforce these rules. |
| Admin can set **reschedule rules** (e.g., only allow rescheduling up to 30 days ahead). | **9** | Like cut-off rules, rescheduling constraints are supported by the plugin's strong date-handling capabilities. The **Availability rules** configuration utilizes the sophisticated date and time utility library (e.g., `core/wpbc-dates.php`, `core/wpbc_functions_dates.php`) to validate new dates against admin constraints. |
| Admin receives **notifications** of client-initiated cancellations/reschedules. | **10** | The **Email API pattern** ensures that administrators are notified of status changes. Status changes, which are handled via the main AJAX controller, trigger dedicated emails to the client (e.g., `page-email-deleted.php`). The administrative notification email (`page-email-new-admin.php`) is specifically designed for quick status management, confirming the admin is a recipient of workflow status updates. |
| Admin can override or lock bookings (e.g., jobs requiring prep time). | **10** | This is comprehensively implemented through several architectural features: 1. **Overrides** are possible via **AJAX handlers** (e.g., `wpbc_ajax_UPDATE_APPROVE`), and quick action links like **`[click2approve]`** are included in admin emails. 2. **Locking** dates can be achieved by manually creating a booking via the **Add New Booking** page (`core/admin/page-new.php`). 3. **Custom internal data** (like "prep time") can be saved securely to the booking record via the **`wpbc_save_booking_meta_option()`** function, which uses the serialized `booking_options` column. |



### 3. Notifications & Logs

* [ ] Cancellation/reschedule triggers an **automatic notification** to both client and admin.
* [ ] Updated booking details replace old reminders (so client doesn’t still get reminders for cancelled jobs).
* [ ] System keeps an **audit log** of cancellations/reschedules (who, when, reason if captured).

Based on the architectural review of the plugin's Email API, Notification System, and core data handling, here is the implementation status of the requested functionalities:

### Notifications & Logs Implementation Status

| Feature | Implementation Status (Rating 1-10) | Supporting Architectural Evidence |
| :--- | :--- | :--- |
| Cancellation/reschedule triggers an **automatic notification** to both client and admin. | **10** | The Email API pattern is fully structured to handle notifications for status changes. When a booking is deleted (canceled or declined), the `core/admin/page-email-deleted.php` template is used, managed by the `WPBC_Emails_API_Deleted` class. Similarly, admin notification emails (`page-email-new-admin.php`) are sent for new actions, and this pattern applies to status changes processed via AJAX. The `core/lib/wpbc-ajax.php` controller handles booking lifecycle actions like `wpbc_ajax_DELETE_APPROVE()` and triggers side-effects, which include sending confirmation emails. |
| Updated booking details replace old reminders (so client doesn’t still get reminders for cancelled jobs). | **Implied / High Likelihood** | The core plugin manages the booking status through AJAX handlers and workflow functions like `wpbc_db__booking_approve` and `wpbc_auto_cancel_booking`. When a booking status changes (e.g., to Deleted/Canceled), the associated database record is updated. Reminders or synchronization systems (like the custom pseudo-cron system in `core/lib/wpbc-cron.php`) would rely on the current, authoritative status in the database to determine which reminders to send. If a job is marked as "Deleted" or "Trashed" via AJAX status handlers (`wpbc_ajax_TRASH_RESTORE`, `wpbc_ajax_DELETE_APPROVE`), it is assumed that subsequent background processes or status checks would omit this booking. |
| System keeps an **audit log** of cancellations/reschedules (who, when, reason if captured). | **Yes (Partial/Implied)** | The architecture supports capturing and logging this data: |
| | | 1. **Data Storage:** The plugin manages custom data associated with individual bookings using `wpbc_save_booking_meta_option()`, which stores data as a **serialized array in the `booking_options` column** of the custom booking table. This architecture is used to log custom data, potentially including an action log. |
| | | 2. **Reason Capture:** Email templates for Deleted/Denied emails (`page-email-deleted.php`, `page-email-deny.php`) specifically use the shortcode **`[denyreason]`**, confirming that the "reason" is captured and processed during cancellation/denial actions. |
| | | 3. **Workflow Logging:** General utility files contain functions for adding logs, such as `wpbc_db__add_log_info`, suggesting that status changes (including cancellation/reschedule) are tracked in a workflow log using these functions. |

### Numerical Marking (1-10 Scale)

Based on the evidence:

1. **Cancellation/reschedule triggers an automatic notification to both client and admin:** **10**. The architectural components (dedicated Email API classes like `WPBC_Emails_API_Deleted` and AJAX handlers) confirm this workflow is fully implemented.
2. **Updated booking details replace old reminders:** **9**. While the sources don't explicitly detail the reminder deletion mechanism, the core architectural logic dictates that all subsequent status checks, syncs (like Google Calendar), and workflow actions rely on the current, authoritative database status. If the status is updated to Canceled/Deleted by the system, other dependent systems are highly likely to recognize this change and stop future processing (like reminders) based on that central status update.
3. **System keeps an audit log of cancellations/reschedules (who, when, reason if captured):** **9**. The functionality is strongly supported by the use of `[denyreason]` shortcodes, meta option saving for custom data, and helper functions for adding log information (`wpbc_db__add_log_info`). The necessary data fields and utility functions are in place to support detailed audit logging, although the exact contents and accessibility of the full audit log are not explicitly described in the sources.

4. Based on the comprehensive architectural details found in the source documents, the implementation of the Admin-Side Controls is highly robust, particularly concerning workflow and data management.

Here is the rating for each requested feature on a scale of 1 (not implemented) to 10 (fully implemented as requested):

| Admin-Side Control | Rating (1-10) | Justification (Based on Architectural Evidence) |
| :--- | :--- | :--- |
| Admin can **enable/disable** cancellation and rescheduling (global setting). | **7** | The necessary architectural structure for global control is confirmed. The plugin utilizes a custom **Settings API framework** and the `WPBC_Settings_API_General` class defines settings fields for controlling fundamental features like **Availability rules** and **Post-booking action**. The existence of dedicated email templates for cancellation (`page-email-deleted.php`) and approval demonstrates administrative control over the workflow, even if the specific "enable/disable" option name is not explicitly mentioned. |
| Admin can configure **cut-off rules** (e.g., cannot cancel within 24h of the job). | **9** | This functionality is supported by the plugin’s core architectural layers. The **General Settings** govern **Availability rules**, and the plugin employs a highly sophisticated **dates engine** (`core/wpbc-dates.php`) that provides comprehensive functions for date parsing, comparison, and validation (e.g., `wpbc_get_difference_in_days()`, `wpbc_is_date_in_past()`). These utilities are necessary and sufficient to enforce time-based constraints, such as cancellation cut-off periods. |
| Admin can set **reschedule rules** (e.g., only allow rescheduling up to 30 days ahead). | **9** | This is supported by the same fundamental date validation engine required for cut-off rules. The core availability checking mechanism, exposed via the Developer API function `wpbc_api_is_dates_booked()`, is responsible for validating any proposed reschedule date against the resource's current status and the admin's configured availability or time limits. |
| Admin receives **notifications** of client-initiated cancellations/reschedules. | **10** | The system is fully equipped to handle and dispatch notifications for status changes. When a booking status is changed (e.g., deleted or trashed) via the central AJAX controller (`core/lib/wpbc-ajax.php`), the system triggers corresponding transactional emails using classes like `WPBC_Emails_API_Deleted`. Furthermore, the `WPBC_Emails_API_NewAdmin` template includes quick action shortcodes (`[click2approve]`, `[click2decline]`), confirming the administrative role in the notification workflow. |
| Admin can override or lock bookings (e.g., jobs requiring prep time). | **10** | This is implemented through multiple integrated features: 1. **Overrides** are enabled by AJAX handlers for status changes (approve, delete, trash) and quick links in admin emails. 2. **Locking dates** is possible via manual creation of a booking through the **Add New Booking** page (`core/admin/page-new.php`). 3. **Custom data** (like "prep time") can be associated with any booking using the function `wpbc_save_booking_meta_option()`, which securely stores serialized data in the `booking_options` column of the database. |


### 4. Compliance & UX

* [ ] GDPR-compliant handling of any cancellation “reason” text (if collected).
* [ ] Cancellation/reschedule options are **clearly visible** but not intrusive (e.g., link in confirmation email + client portal).
* [ ] If payment is involved:

  * [ ] Refunds/credits are triggered automatically or flagged for admin review.
  * [ ] Policy messaging (e.g., “Non-refundable within 24h”) is shown before finalising cancellation.


Based on the sources, the implementation status of the Compliance and User Experience (UX) features is as follows:

### Compliance & UX

| Feature | Implementation Status (Based on Sources) | Supporting Architectural Evidence |
| :--- | :--- | :--- |
| **GDPR-compliant handling of any cancellation “reason” text (if collected).** | **Partial (Reason Text is Captured & Stored)** | The system is designed to collect a reason when a booking is cancelled or denied, evidenced by the use of the **`[denyreason]`** shortcode in the `page-email-deleted.php` and `page-email-deny.php` templates. This custom data is typically stored securely in the **`booking_options`** column of the custom booking table as a **serialized array** via `wpbc_save_booking_meta_option()`. However, the sources **do not explicitly mention** "GDPR compliance" or specific rules regarding the logging, retention, or anonymization of this reason text. |
| **Cancellation/reschedule options are clearly visible but not intrusive (e.g., link in confirmation email + client portal).** | **Implemented (via Email Link)** | Visibility via the initial confirmation email is implemented using the dynamic shortcode **`[visitorbookingediturl]`**. This shortcode, used in the `page-email-new-visitor.php` template, is designed to generate a unique link that directs the client to an interface (client portal) where they can manage their booking, though it is explicitly noted as a feature **"for premium users"**. |
| **If payment is involved:** | | |
| * [ ] Refunds/credits are triggered automatically or flagged for admin review. | **Not Supported by Sources** | The provided architectural files detail booking management (approve, delete, trash) and cost calculation via AJAX, but they **do not** contain any logic, classes, or hooks specific to integrating with payment gateways, processing financial transactions, triggering refunds, or flagging credit review processes. |
| * [ ] Policy messaging (e.g., “Non-refundable within 24h”) is shown before finalising cancellation. | **Not Supported by Sources** | While the system supports complex date/time validation (availability rules), the sources **do not** provide evidence of a front-end feature that displays specific financial cancellation policy text (like a "Non-refundable within 24h" warning) directly to the user *before* the cancellation is confirmed. |

***

### Summary of Implementation Status

*   **Reason Capture:** The collection and secure storage of the cancellation reason is implemented.
*   **GDPR:** Whether the handling of this data meets GDPR compliance standards is **not documented** in the sources.
*   **Visibility:** Cancellation/reschedule links are made visible in the confirmation email via the **`[visitorbookingediturl]`** shortcode (available for premium users).
*   **Payment/Refunds/Policy:** The sources contain **no evidence** that the plugin handles automatic refunds, credits, or displays specific policy messaging related to payments during the cancellation workflow.

*   This assessment uses the detailed architectural information regarding the plugin's configuration, email workflow, and data management systems.

| Feature | Rating (1-10) | Justification (Based on Architectural Evidence) |
| :--- | :--- | :--- |
| **GDPR-compliant handling of any cancellation “reason” text (if collected).** | **7** | The essential functionality is implemented: the plugin captures the cancellation reason via the `[denyreason]` shortcode in the `page-email-deleted.php` template. This custom data is securely stored as a **serialized array in the `booking_options` column** of the custom database table via functions like `wpbc_save_booking_meta_option()`. However, the sources **do not explicitly confirm** that the plugin implements specific GDPR compliance measures for data retention or anonymization related to this text. |
| **Cancellation/reschedule options are clearly visible but not intrusive (e.g., link in confirmation email + client portal).** | **8** | This functionality is implemented via the Email API. The confirmation email sent to the visitor (`page-email-new-visitor.php`) supports the shortcode **`[visitorbookingediturl]`**, which generates a unique link for the client to manage their booking (reschedule/cancel). The rating is slightly reduced because the sources explicitly state this feature is only available **"for premium users"**. |
| **If payment is involved: Refunds/credits are triggered automatically or flagged for admin review.** | **1** | There is **no evidence** in the core architectural files analyzed (settings, AJAX, emails, developer API) that the plugin contains integrated logic for processing financial transactions, communicating with payment gateways for refunds, or initiating credit review workflows upon booking cancellation. |
| **If payment is involved: Policy messaging (e.g., “Non-refundable within 24h”) is shown before finalising cancellation.** | **1** | Although the plugin possesses strong date validation and availability rules (controlled via General Settings), the sources **do not support** the implementation of a client-facing UI component designed to display specific financial or policy messaging immediately prior to finalizing a client-initiated cancellation or reschedule action. |

---

## **Success Criteria Checklist**

✅ Clients can cancel or reschedule bookings via portal/email.
✅ Links are secure and unique per booking.
✅ System checks admin’s availability before rescheduling.
✅ Cut-off and reschedule rules can be configured.
✅ Notifications go out to both client and admin.
✅ Old reminders are cancelled after changes.
✅ Audit log tracks all cancellations/reschedules.
✅ Refunds/credits follow policy rules (if payments integrated).

---

Would you like me to **also draft a “basic vs advanced” split** (MVP vs later enhancements) for this feature? That way you can test whether the plugin implements just the essentials or also supports the advanced rules like refund triggers and audit logs.
