Excellent — this is a key **security and accountability feature**. Let’s break down:

> **Security → Audit → Audit trail for bookings, rewards, and messaging**
> Type: *Must-Have* | Effort: *8* | Priority: *High*

Below is a **structured checklist** you can use to test if your WordPress plugin already supports or partially supports this capability.

---

## 🧾 Audit Trail for Bookings, Rewards, and Messaging — Implementation Breakdown

### **1. Core Audit Logging Framework**

| #   | Check Item              | Description                                                                                                     | Implemented? |
| --- | ----------------------- | --------------------------------------------------------------------------------------------------------------- | ------------ |
| 1.1 | Audit table or log file | Does the plugin store an internal audit trail table or structured log (e.g., in custom DB table or WP options)? | ☐            |
| 1.2 | Event type tracking     | Are different types of actions logged — e.g. booking changes, reward usage, or messages sent/received?          | ☐            |
| 1.3 | Immutable record        | Are logs read-only or protected from being edited/deleted by non-admins?                                        | ☐            |
| 1.4 | Timestamping            | Does every log record include a timestamp (UTC ideally)?                                                        | ☐            |


Based on the sources, the plugin implements mechanisms for **tracking workflow changes** and **notifying administrators** of key events (like booking status changes), but it **does not implement a formal, queryable Core Audit Logging Framework** designed for historical review and immutability.

The logging mechanism that exists is integrated into the booking metadata structure, which has significant architectural limitations.

Here is the implementation status of the requested audit logging framework:

| # | Check Item | Implemented? |
| :--- | :--- | :--- |
| **1.1** | **Audit table or log file** | **☒ (Stored as serialized metadata)** |
| **1.2** | **Event type tracking** | **☒ (Workflow events tracked)** |
| **1.3** | **Immutable record** | **☐** |
| **1.4** | **Timestamping** | **☐** |

### Implementation Details

#### 1.1 Audit table or log file (Does the plugin store an internal audit trail table or structured log?)

**Partially Implemented (as serialized metadata).** The plugin stores log information directly within the database, but not in a dedicated, queryable audit table.

*   **Location:** Workflow logs (messages, status changes, notes) are stored as part of the custom data associated with an individual booking, known as **"booking meta options"**.
*   **Structure:** This data is stored as a **single, serialized array** in the **`booking_options` column** of the plugin's custom database table, typically named `{$wpdb->prefix}booking`.
*   **Limitation:** This serialization approach is explicitly identified as **inefficient, not queryable, and detrimental to database normalization**. This architectural flaw prevents easy retrieval or complex filtering of audit data, which is necessary for a robust audit framework.

#### 1.2 Event type tracking (Are different types of actions logged?)

**Implemented (Workflow events).** The plugin tracks critical administrative actions related to the booking lifecycle:

*   **Status Changes:** Functions related to managing booking status (e.g., `wpbc_db__booking_approve`, `wpbc_auto_approve_booking`, `wpbc_auto_cancel_booking`) integrate with the database and **trigger emails/logs**.
*   **Extensibility Hooks:** The **Developer API** (`core/wpbc-dev-api.php`) documents specific hooks that fire upon key events, signaling that change tracking is designed into the workflow:
    *   `wpbc_track_new_booking` (Creation/Insertion).
    *   `wpbc_booking_approved` (Status Update).
    *   `wpbc_deleted_booking_resources` (Deletion).

#### 1.3 Immutable record (Are logs read-only or protected from being edited/deleted by non-admins?)

**Not Implemented/Not Documented.** The sources do not contain information indicating that logged workflow entries (messages, notes) are cryptographically protected or rendered read-only to prevent editing by administrators or privileged users. Access to this data is managed via the plugin's **Role-Based Access Permissions** (RBA) system, which relies on WordPress capabilities.

#### 1.4 Timestamping (Does every log record include a timestamp?)

**Not Explicitly Confirmed.** While logs are recorded for workflow changes, the sources do not specifically confirm that **every** log record includes a timestamp, or that the timestamp adheres to the UTC standard. The plugin generally handles timezones robustly for date *display* using functions like `wpbc_datetime_localized()`, but this does not guarantee UTC recording for internal audit events.

***

In summary, the plugin has the **capability to track *what* happened** (workflow changes) and **where to store the data** (serialized metadata), but it lacks the necessary structural integrity (dedicated table) and security controls (immutability, dedicated timestamps) required for a formal, reliable Audit Logging Framework.


The implementation of the **Core Audit Logging Framework** is marked as a **5** on a scale of 1 to 10.

### Rationale for the Score

The score of 5 reflects that the plugin has successfully implemented the fundamental requirement of **change tracking** (Check Item 1.2) by integrating logging into its core workflow. However, this is critically offset by the **architectural flaw** in how log data is stored, which prevents the logs from serving as an efficient, queryable, or immutable audit history.

| # | Check Item | Implementation Status (Based on Sources) | Summary |
| :--- | :--- | :--- | :--- |
| **1.1** | **Audit table or log file** | **Partially Implemented (Stored as serialized metadata)** | Log data is physically stored in the database within the **`booking_options` column** as a **single serialized array**. This method is explicitly identified as **inefficient, not queryable, and detrimental to database normalization**, which fundamentally cripples its function as an audit trail. |
| **1.2** | **Event type tracking** | **Implemented (Workflow events tracked)** | The system effectively tracks major workflow changes. Administrative actions like updating or deleting bookings trigger specific, documented action hooks (e.g., `wpbc_track_new_booking`, `wpbc_booking_approved`, `wpbc_deleted_booking_resources`) that developers can use to trigger auditing. Workflow logs (notes and status changes) are recorded via utilities like `wpbc_db__add_log_info`. |
| **1.3** | **Immutable record** | **Not Implemented/Not Documented.** | The sources do not provide evidence that logged workflow entries are protected, cryptographically sealed, or rendered read-only to prevent modification by administrators or privileged users [Conversation History 6]. |
| **1.4** | **Timestamping** | **Not Explicitly Confirmed.** | While the date engine handles complex **localization and timezone awareness** for display purposes (converting UTC values into localized strings via `wpbc_datetime_localized`), the sources do not explicitly confirm that all internal log records are saved using a consistent UTC timestamp standard [Conversation History 6]. |

The score is heavily weighted by the structural implementation of Check Item 1.2 (Change tracking) but is held low by the fundamental limitations of Check Item 1.1 (Storage structure).


The implementation of the **Core Audit Logging Framework** necessitates a foundational shift in data architecture to overcome the existing limitation of storing logs as an **inefficient, non-queryable serialized array** in the `booking_options` column.

The implementation would be conducted by integrating a new database structure with the plugin's core components: the custom hook system, the AJAX Controller, and the Admin UI factory.

### I. Data Persistence and Immutability (Audit Table Foundation)

The primary step is establishing a reliable, queryable storage mechanism designed specifically for auditing:

1.  **Dedicated Audit Log Table:** A new, normalized custom database table (e.g., `{$wpdb->prefix}audit_log`) must be created during the plugin's installation sequence. The creation logic would be hooked into the custom plugin lifecycle action, **`make_bk_action( 'wpbc_activation' )`**, ensuring it runs during plugin activation or update.
2.  **Normalized Structure:** This table must store dedicated, queryable columns, including fields for `log_id`, `user_id`, `timestamp` (ideally UTC), `action_type` (e.g., VIEW, CREATE, FAILED\_AUTH), `resource_id` (booking or resource context), and `details` (for any supplementary payload).
3.  **Data Interaction:** All logging functions must use **direct, prepared SQL insertions** via the global `$wpdb` object, ensuring security against SQL injection, a practice already followed by the plugin's date and query engines. Logs stored here should be treated as **read-only records** to maintain immutability.

### II. Change Tracking (Workflow and Event Logging)

This focuses on capturing major lifecycle events and tracking who performed the action and when, fulfilling the change tracking requirement (Check Item 4.2):

1.  **Leveraging Existing Hooks:** The system would utilize the plugin's internal action and filter system. Critical events would be captured by hooking into documented Developer API actions:
    *   **Creation/Update:** Hook into `wpbc_track_new_booking` (for creation) or `wpbc_booking_approved` (for status changes).
    *   **Deletion:** Hook into `wpbc_deleted_booking_resources` after a booking is permanently removed.
2.  **AJAX Controller as Choke Point:** The central AJAX router (`core/lib/wpbc-ajax.php`) executes sensitive administrative actions (e.g., `wpbc_ajax_UPDATE_APPROVE`, `wpbc_ajax_DELETE_APPROVE`). The logging handler would be triggered immediately after the database modification in these functions, guaranteeing that the log records the final change status.

### III. Access Logging and Security Failure Auditing

This addresses the currently missing functionality of tracking successful views/downloads and unauthorized attempts (Check Item 4.1):

1.  **Gated Access Logging (Views/Downloads):**
    *   A new AJAX endpoint would be registered (using the **`wpbc_ajax_action_list`** filter) to serve sensitive administrative content, such as logs or photos.
    *   The handler for this endpoint must first verify the user's authority (checking roles and/or resource ownership, utilizing logic similar to the existing Multi-User support functions).
    *   A successful authentication would trigger an `ACTION_TYPE: VIEW` entry in the `{$wpdb->prefix}audit_log` table before the content is streamed.
2.  **Security Failure Logging:**
    *   Logic would be inserted directly into the core security wrappers of the AJAX controller. If a security check, specifically **nonce verification** using `wpbc_check_nonce_in_admin_panel()`, fails, the process must first record a dedicated log entry (`ACTION_TYPE: FAILED_AUTH`) before denying the action or displaying an error message.

### IV. Administrative UI for Audit Review (Audit View)

A user-facing interface is required to query the new data store (Check Item 4.3):

1.  **Admin Page Registration:** A new administrative submenu page ("Audit Log") would be registered by instantiating the **`WPBC_Admin_Menus`** class. The content rendering would be delegated using the custom hook **`wpbc_page_structure_show`**.
2.  **Display Engine:** The rendering function would execute secure, filtered queries against the new `{$wpdb->prefix}audit_log` table, utilizing the complex **query building** functions inherent to the plugin's data engine.
3.  **UI Components:** The page would leverage the procedural helper functions from `core/any/admin-bs-ui.php` (e.g., `wpbc_bs_display_tab`, `wpbc_bs_select`) to construct a visually consistent, filterable log viewer.


---

### **2. Booking Audit Coverage**

| #   | Check Item                       | Description                                                                      | Implemented? |
| --- | -------------------------------- | -------------------------------------------------------------------------------- | ------------ |
| 2.1 | Booking created                  | Logs when a new booking is created (by whom, for which client).                  | ☐            |
| 2.2 | Booking updated                  | Logs any changes (date, time, assigned staff, price, status).                    | ☐            |
| 2.3 | Booking cancelled or rescheduled | Logs client/admin/staff who triggered cancellation or change.                    | ☐            |
| 2.4 | Status transitions               | Logs each state change (e.g., “pending → confirmed”, “in progress → completed”). | ☐            |

The architectural analysis confirms that the plugin implements mechanisms to track and log core booking workflow events, changes, status transitions, and user actions, fulfilling the requirements for **Booking Audit Coverage**.

The audit data, which includes logs and notes, is physically stored as **serialized metadata** in the custom database table's `booking_options` column. Although this storage method is explicitly noted as **inefficient and not queryable**, the capability to record these events exists and is triggered by the plugin's workflow.

| #   | Check Item | Implemented? |
| :--- | :--- | :--- |
| **2.1** | **Booking created** | **☒** |
| **2.2** | **Booking updated** | **☒** |
| **2.3** | **Booking cancelled or rescheduled** | **☒** |
| **2.4** | **Status transitions** | **☒** |

### Implementation Details

#### 2.1 Booking created
**Implemented.** The plugin tracks new booking insertions:
*   **Workflow Hook:** The **Developer API** documents the existence of the action hook `wpbc_track_new_booking`, which fires upon the creation of a new booking.
*   **Data Source:** Log information related to the creation is recorded via utility functions (e.g., `wpbc_db__add_log_info`) and stored as booking metadata [Conversation History 6].

#### 2.2 Booking updated
**Implemented (for status and notes).** The system tracks and logs major administrative updates related to the booking status and internal notes:
*   **Administrative Control:** Updates are handled by the central **AJAX request router** (`core/lib/wpbc-ajax.php`), specifically using functions like `wpbc_ajax_UPDATE_APPROVE`. These sensitive, admin-facing functions are protected by Nonce verification and execute direct, prepared database queries for security and immediate status change.
*   **Programmatic Updates:** External developers can also trigger updates (which function as edits/changes) using the high-level API function `wpbc_api_booking_add_new()`.

#### 2.3 Booking cancelled or rescheduled
**Implemented.** Both cancellation and rescheduling (an update to dates/times) are tracked by the core workflow:
*   **Cancellation:** Cancellation and deletion actions are handled by AJAX functions (`wpbc_ajax_DELETE_APPROVE`, `wpbc_ajax_TRASH_RESTORE`) and trigger explicit hooks like `wpbc_deleted_booking_resources`. Functions for auto-cancellation (`wpbc_auto_cancel_booking`) also exist.
*   **User Tracking:** Tracking *who* performed the action (admin/staff) is supported by the necessity of **Nonce verification** for all sensitive AJAX actions, ensuring the user ID and permissions are checked before processing the request.

#### 2.4 Status transitions
**Implemented.** Logging status changes is a core function of the plugin's workflow tracking:
*   **Dedicated Hooks:** Status changes fire action hooks such as `wpbc_booking_approved`.
*   **Workflow Functions:** Status management functions like `wpbc_db__booking_approve` and `wpbc_auto_approve_booking` handle the database update and trigger associated actions, such as **sending emails**. The email system itself is highly granular, with dedicated templates for *Approved*, *Pending/Deny*, *Deleted*, and *Trash/Reject* status changes.



The implementation of **Booking Audit Coverage** is assessed as a **7** on a scale of 1 to 10.

### Rationale for the Score

The score is high because the plugin successfully implements mechanisms to track and log **all four specific workflow changes** requested (creation, updates, cancellation, and status transitions) using explicit action hooks and internal functions, ensuring traceability [Conversation History 6].

However, the score is penalized because the underlying data storage mechanism prevents the collected logs from functioning as an efficient, queryable, or performant historical audit trail, thereby hindering "full implementation as requested."

| # | Check Item | Implementation Status | Justification & Architectural Note |
| :--- | :--- | :--- | :--- |
| **2.1** | **Booking created** | **Implemented.** | Tracked via the Developer API action hook `wpbc_track_new_booking` and associated log utility functions [Conversation History 6]. |
| **2.2** | **Booking updated** | **Implemented.** | Status changes and updates are handled by secured AJAX functions (e.g., `wpbc_ajax_UPDATE_APPROVE`). The system records internal notes and messages via utility functions like `wpbc_db__add_log_info` [Conversation History 6]. |
| **2.3** | **Booking cancelled or rescheduled** | **Implemented.** | Cancellation/deletion triggers hooks like `wpbc_deleted_booking_resources` [Conversation History 6]. The tracking of *who* performs the action (admin/staff) is supported by **Nonce verification** checks required for all sensitive administrative AJAX actions [Conversation History 6]. |
| **2.4** | **Status transitions** | **Implemented.** | State changes (e.g., pending to approved) utilize functions like `wpbc_db__booking_approve` and fire dedicated action hooks such as `wpbc_booking_approved`. Status changes also trigger the highly granular email notification system [Conversation History 6]. |
| **Architectural Limitation** | **Data Persistence** | **Flawed.** | Log data is stored as a **single, serialized array** in the **`booking_options` column** of the custom database table. This approach is explicitly noted as being **inefficient, not queryable, and detrimental to database normalization**, preventing robust historical reporting and filtering. |



---

### **3. Rewards & Incentives Audit Coverage**

| #   | Check Item                      | Description                                                                | Implemented? |
| --- | ------------------------------- | -------------------------------------------------------------------------- | ------------ |
| 3.1 | Reward creation                 | Logs creation or assignment of reward/credit (by admin or automated rule). | ☐            |
| 3.2 | Reward redemption               | Logs client redemption events and the booking it applies to.               | ☐            |
| 3.3 | Reward expiration or adjustment | Logs expiry or manual edits to reward balances.                            | ☐            |

Based on the provided sources and our conversation history, the plugin's architecture focuses on core booking, synchronization, and administrative workflow management. There are **no components, classes, hooks, or documented functions** related to managing or logging features specifically identified as "**Rewards & Incentives**" or "**Credit Balances**" [1-234, Conversation History 1-7].

Therefore, none of the options for **Rewards & Incentives Audit Coverage** are implemented in the plugin.

| # | Check Item | Implemented? |
| :--- | :--- | :--- |
| **3.1** | **Reward creation** | **☐** |
| **3.2** | **Reward redemption** | **☐** |
| **3.3** | **Reward expiration or adjustment** | **☐** |

### Supporting Architectural Context

While the requested feature is absent, the plugin does have robust architectural components that would be required to implement it:

1.  **Change Tracking Mechanism:** The plugin uses action hooks (like `wpbc_track_new_booking`, `wpbc_booking_approved`, or `wpbc_deleted_booking_resources`) to log workflow status changes. This hook system would be essential for logging reward creation or redemption events.
2.  **Custom Data Storage (Meta Options):** Any reward balance or redemption record would likely be stored as a **"booking meta option"** (custom data associated with an individual booking). This data is stored as a **single serialized array** in the `booking_options` column of the custom database table.
3.  **Cron System:** If reward expiration needed automation, the plugin's custom, traffic-dependent **pseudo-cron system** (`WPBC_Cron` class) could be used to schedule checks on expiration dates.
4.  **Developer API:** The `core/wpbc-dev-api.php` file provides functions like `wpbc_api_booking_add_new()` for programmatic interaction, which could be leveraged to automatically apply rewards during booking creation.

Based on the comprehensive analysis of the plugin's architecture, the implementation score for **Rewards & Incentives Audit Coverage** is a **1** on a scale of 1 to 10.

### Rationale for the Score

The score of 1 reflects the fact that there are **no implemented features** related to the concept of "Rewards," "Incentives," or "Credit Balances" documented anywhere in the provided sources. Therefore, audit coverage for these non-existent features cannot be implemented.

| # | Check Item | Implementation Status | Justification |
| :--- | :--- | :--- | :--- |
| **3.1** | **Reward creation** | **Not Implemented.** | The sources do not define any functions, classes, settings, or administrative UI components for creating or assigning rewards or credit balances (by admin or automated rules) [1-234, Conversation History 7]. |
| **3.2** | **Reward redemption** | **Not Implemented.** | There is no mechanism to log client redemption events, as the concept of rewards or redemption itself is absent from the documented architectural components [1-234, Conversation History 7]. |
| **3.3** | **Reward expiration or adjustment** | **Not Implemented.** | Although the plugin possesses a custom pseudo-cron system (`WPBC_Cron`) that *could* be used for scheduling expiration checks, no logic or data structure for tracking reward expiration or manual balance adjustments exists in the sources [1-234, Conversation History 7]. |

The architectural components, such as the event


The implementation of **Rewards & Incentives Audit Coverage** requires introducing the core Rewards feature first, along with a dedicated, queryable database structure for logging, correcting the architectural debt associated with storing logs as serialized metadata.

This high-level overview leverages the plugin's existing **Custom API Frameworks** (Settings, Hooks, Cron) to ensure seamless integration.

### I. Core Data Architecture and Audit Persistence

This phase establishes the necessary normalized database structure required for tracking reward balances and generating an efficient audit trail, bypassing the architectural limitations of current metadata storage.

| Component | Implementation Action | Rationale / Source |
| :--- | :--- | :--- |
| **Database Tables (3.1, 3.2, 3.3)** | Create two new, normalized custom database tables during plugin activation: `{$wpdb->prefix}rewards` (stores current balances/assignments) and `{$wpdb->prefix}reward_log` (stores immutable audit records of creation, redemption, expiration, adjustment). | This resolves the inherent structural limitation where data is stored as a **single serialized array in `booking_options`**, which is explicitly noted as **inefficient and not queryable**. Table creation is hooked into `make_bk_action( 'wpbc_activation' )`. |
| **Audit Record Structure** | The `reward_log` table structure must include fields for `reward_id`, `user_id`, `booking_id` (if redeemed), `timestamp`, `action_type` (CREATE, REDEEM, EXPIRE, ADJUST), and `amount`. | This structure ensures that logs are timestamped and link redemption back to the specific booking, fulfilling the audit requirements. |
| **Secure Querying** | Implement dedicated functions for inserting and retrieving reward data using **direct, prepared `$wpdb` SQL queries** to ensure security and performance. | The plugin's data engine already relies on secure prepared statements for database operations involving bookings and dates. |

### II. Reward Logic and Workflow Integration (Audit Coverage)

This phase integrates reward events into the core booking and status workflow, ensuring that every significant action is logged immediately.

| Audit Check Item | Implementation Action | Rationale / Source |
| :--- | :--- | :--- |
| **3.1 Reward creation** | Hook into core booking insertion events (e.g., `wpbc_track_new_booking`, documented in the Developer API) to trigger automated reward assignment based on predefined rules. Immediately, save a `CREATE` entry to the `reward_log` table. | The Developer API provides stable hooks that fire upon booking events, making them the safest extension point for event-driven logic. |
| **3.2 Reward redemption** | Integrate the redemption logic into the **booking form processing pipeline** (filters like `wpdev_booking_form`) and cost calculation engine. When a client successfully applies a reward during submission, the process must commit a `REDEEM` entry to the `reward_log` table, linking the `booking_id`. | The plugin's front-end forms and widgets rely on centralized filters for output customization. |
| **3.3 Reward expiration or adjustment (Manual)** | Implement administrative **AJAX actions** (via the **AJAX Controller**) for manual adjustments to a reward balance. These functions must enforce **Nonce verification** for security and record an `ADJUST` entry in the `reward_log` table immediately after the database change. | All sensitive administrative functions must enforce Nonce verification (`wpbc_check_nonce_in_admin_panel()`) to prevent CSRF attacks. |

### III. Automation and Administrative UI

This phase implements the scheduling logic for automated checks and provides an administrative interface for reward management.

| Component | Implementation Action | Rationale / Source |
| :--- | :--- | :--- |
| **Expiration Automation (3.3)** | Define a recurring, low-priority cron task (e.g., `wpbc_cron_check_rewards`) using the public methods of the **`WPBC_Cron`** class. This pseudo-cron task runs on page loads and checks the `wpbc_rewards` table for expired entries, logging an `EXPIRE` action in the audit trail when applicable. | The plugin uses a custom, traffic-dependent pseudo-cron system hooked into the `init` action for background tasks like Google Calendar sync. |
| **Administrative UI** | Register a new top-level or submenu page (e.g., "Rewards") using the **`WPBC_Admin_Menus`** class. The page content must be rendered by hooking into the content delegation action `wpbc_page_structure_show`. | This standardizes the menu creation process and ensures assets are loaded efficiently on the specific page only. |
| **UI Components** | Use the procedural helper functions from **`core/any/admin-bs-ui.php`** to render tables, buttons, and input fields for managing reward types and viewing the detailed `reward_log` audit history. | This ensures the new interface maintains the consistent, Bootstrap-styled appearance of the rest of the administrative backend. |






---

### **4. Messaging Audit Coverage**

| #   | Check Item             | Description                                                           | Implemented? |
| --- | ---------------------- | --------------------------------------------------------------------- | ------------ |
| 4.1 | Outgoing messages      | Logs emails/SMS/portal messages sent (subject, recipient, timestamp). | ☐            |
| 4.2 | Incoming messages      | Logs client replies or messages received.                             | ☐            |
| 4.3 | Delivery status        | Logs message delivery events (sent, delivered, failed).               | ☐            |
| 4.4 | Message edits/deletion | Logs if messages are deleted or altered (if allowed).                 | ☐            |


The plugin utilizes a powerful, configurable **Email API** framework, but the sources **do not provide evidence** that a dedicated, comprehensive **Messaging Audit Coverage** system is implemented to log the details, delivery status, or content of specific messages sent or received.

The implementation status relies heavily on the plugin's existing workflow logging system, which tracks *events* that trigger message sending (e.g., booking approval) [Conversation History 6].

| # | Check Item | Implemented? |
| :--- | :--- | :--- |
| **4.1** | **Outgoing messages** | **☒ (Workflow event logging only)** |
| **4.2** | **Incoming messages** | **☐** |
| **4.3** | **Delivery status** | **☐** |
| **4.4** | **Message edits/deletion** | **☐** |

### Implementation Details

#### 4.1 Outgoing messages (Logs emails/SMS/portal messages sent)
**Partially Implemented (as workflow log).** The core audit structure logs the event that *causes* the email to be sent, rather than the details of the message itself [Conversation History 6].

*   **Triggering Event Logged:** The plugin tracks critical status changes (like approval, denial, deletion) via action hooks (e.g., `wpbc_track_new_booking`, `wpbc_booking_approved`) and workflow functions (e.g., `wpbc_db__booking_approve`). This event history is recorded as **booking metadata**.
*   **Email Sending:** Sending is handled by dedicated functions (e.g., `wpbc_send_email_approved()`) which use the `WPBC_Emails_API` framework. This API defines template fields for subject and content and uses a shortcode replacement engine to personalize the message just before dispatching via `wp_mail()`.
*   **Limitation:** The current structure logs that the booking was approved (event tracking), but the sources do not confirm that the audit trail captures the specific **subject, final content, and recipient** of the resulting transactional email.

#### 4.2 Incoming messages (Logs client replies or messages received)
**Not Implemented.** The plugin does not appear to have an integrated messaging or logging system for capturing external client replies [Conversation History 7].

*   **Reply Facilitation:** The administrative notification email (`page-email-new-admin.php`) includes an `enable_replyto` option which facilitates direct communication by setting the email's Reply-To header to the visitor's address. However, the management of the ensuing conversation is external to the plugin's documented audit system.

#### 4.3 Delivery status (Logs message delivery events)
**Not Implemented.** The email system's primary focus regarding delivery is **improvement and validation**, not auditing the final delivery status.

*   The file `core/wpbc-emails.php` provides a wrapper, `wpbc_wp_mail()`, that hooks into `phpmailer_init` to fix the Sender header for **improved deliverability**.
*   A global filter, `wpbc_email_api_is_allow_send`, functions as a programmatic "kill switch" to prevent emails from being sent based on custom logic.
*   There is no evidence of logic that records whether the message was accepted by the final mail server (sent) or confirmed delivered (via external API), which would be required for robust delivery status logging.

#### 4.4 Message edits/deletion (Logs if messages are deleted or altered)
**Not Implemented.** The core plugin does not track changes or version history for its email templates or administrative log entries [Conversation History 6].

*   **Template Editing:** Email templates are configurable options stored in the database. Editing these options is a standard administrative function, but the plugin does not provide audit history or versioning for these changes.
*   **Log Immutability:** Workflow logs and notes are stored as **serialized metadata** [190, Conversation History 6]. The current architecture **lacks immutability protection**, meaning logs could potentially be edited or deleted by an administrator without the action itself being audited [Conversation History 6].


Based on the analysis of the sources, the implementation score for **Messaging Audit Coverage** is a **3** on a scale of 1 to 10.

### Rationale for the Score

The plugin has a sophisticated Email API and tracks the administrative *events* that trigger message sending (e.g., booking approval). However, it critically lacks auditing for the message *content*, *delivery status*, and any form of *incoming replies*, preventing it from meeting the requirements for comprehensive messaging audit coverage.

| # | Check Item | Implementation Status | Justification and Limitation |
| :--- | :--- | :--- | :--- |
| **4.1** | **Outgoing messages** | **Partially Implemented (Workflow Only).** | The plugin logs the **workflow event** that causes a message to be sent (e.g., `wpbc_track_new_booking`, `wpbc_booking_approved`) [192, 233, Conversation History 6]. The Email API defines the subject and content using configurable templates, which include dynamic shortcodes. **However, the sources do not confirm that the final rendered subject, full recipient list, or content of the dispatched message is captured in the audit log.** |
| **4.2** | **Incoming messages** | **Not Implemented.** | The plugin does not appear to have an integrated messaging or logging system to capture client replies or incoming messages [Conversation History 7]. While the Admin Notification email template allows setting the Reply-To header to the visitor's address for **direct communication**, the resulting conversation audit is external to the plugin's documented architecture. |
| **4.3** | **Delivery status** | **Not Implemented.** | The email system is engineered for **improved deliverability** (e.g., fixing the Sender header via a `phpmailer_init` hook in `wpbc_wp_mail()`), and includes a **global kill switch** filter (`wpbc_email_api_is_allow_send`). However, the sources do not mention any mechanism to log or track the message's final delivery status (sent, delivered, failed) with the receiving mail server. |
| **4.4** | **Message edits/deletion** | **Not Implemented.** | Workflow logs and notes, which are stored as **serialized booking metadata**, **lack immutability protection**. If template changes are considered, the plugin does not provide audit history or versioning for administrator changes to email templates, which are configurable options. |



The implementation of **Messaging Audit Coverage** would necessitate addressing the architectural debt of storing logs as non-queryable serialized data and integrating robust interception points within the plugin's powerful **Email API (WPBC_Emails_API)** and settings framework.

This high-level overview details the necessary changes across data persistence, message interception, and administrative audit trails.

### I. Data Persistence: Normalized Audit Log Table

The current practice of storing logs as a serialized array in the `booking_options` column is explicitly noted as **inefficient and non-queryable**. A secure, queryable audit framework is critical for messaging.

1.  **Custom Database Table:** A new, normalized custom database table (e.g., `{$wpdb->prefix}message_log`) must be created during plugin activation, leveraging the custom lifecycle hook `make_bk_action( 'wpbc_activation' )`.
2.  **Normalized Structure:** This table would store columns necessary for comprehensive auditing, including: `log_id`, `booking_id`, `template_id`, `timestamp`, `direction` (Outgoing/Incoming), `recipient`/`sender`, `subject`, `full_content`, and `delivery_status` (4.1, 4.3).
3.  **Immutability:** Log insertion functions would rely on secure, direct `$wpdb` insertions using prepared statements.

### II. Outgoing Message and Delivery Status Logging (4.1, 4.3)

This implementation requires intercepting the message content immediately before it is dispatched and tracking its subsequent delivery status.

| Architectural Component | Implementation Action | Rationale / Source |
| :--- | :--- | :--- |
| **Message Interception (4.1)** | Insert logging logic directly into the wrapper function **`wpbc_wp_mail()`** defined in `core/wpbc-emails.php`. This function is the central point where messages are prepared for dispatch via `wp_mail()`. | Intercepting here ensures capture of the **final, fully processed content** (subject, body, recipient) after shortcode replacement and translation have occurred. |
| **Delivery Status Tracking (4.3)** | The plugin relies on `wp_mail()`, which does not provide delivery status. Implementation requires external integration:
    1. Augment `wpbc_wp_mail()` to obtain a **unique message ID** from an external mail service (e.g., transactional email API).
    2. Log the unique ID and `DELIVERY_STATUS: PENDING` to the `message_log` table.
    3. Implement a **webhook listener** outside the core plugin architecture (or via a new dedicated AJAX endpoint) to accept callbacks from the external mail service, updating the `delivery_status` to `DELIVERED` or `FAILED` in the database. | The plugin's AJAX Controller (`core/lib/wpbc-ajax.php`) is the central router for dynamic requests and would need to be extended via the `wpbc_ajax_action_list` filter to handle webhook receipts securely. |

### III. Template Edits and Settings Auditing (4.4)

Since email templates are managed as configurable options extending the `WPBC_Settings_API`, template auditing must be linked to the settings saving process.

*   **Settings Interception:** Hook into the filter that fires after settings are saved: `wpbc_fields_after_saving_to_db`.
*   **Change Tracking:** The handler function would inspect the `$fields_array` to detect if a specific email template class (e.g., `WPBC_Emails_API_Approved`) was saved.
*   **Audit Log:** If a change is detected, log the `user_id`, `timestamp`, and a diff of the old vs. new template content into the new `message_log` table (or a dedicated settings audit table), creating an immutable record of the change.

### IV. Incoming Messages (Client Replies) (4.2)

The current architecture only provides **Reply-To functionality** on admin notification emails, routing replies externally. Capturing and logging incoming messages requires a new integration point.

*   **Admin UI Integration:** Create a new "Messaging History" view on the administrative page for individual bookings, utilizing the custom **UI helper functions** provided by `core/any/admin-bs-ui.php`.
*   **External Data Ingestion:** This feature would rely on an external service (e.g., a dedicated mailbox) or companion plugin to pipe client replies back into the system. When a reply is received, the system must use the plugin's Developer API (`core/wpbc-dev-api.php`) to locate the `booking_id` and insert an `DIRECTION: INCOMING` log entry into the `message_log` table using the **`wpbc_api_booking_add_new()`** framework.



---

### **5. Security & Access to Logs**

| #   | Check Item             | Description                                                                   | Implemented? |
| --- | ---------------------- | ----------------------------------------------------------------------------- | ------------ |
| 5.1 | Role-based visibility  | Only authorized users (admin/supervisor) can view audit logs.                 | ☐            |
| 5.2 | Export restriction     | Export of audit data is restricted or password-protected.                     | ☐            |
| 5.3 | Log retention settings | Admin can configure how long logs are retained (e.g., 6 months, 12 months).   | ☐            |
| 5.4 | Integrity protection   | Logs cannot be deleted or altered without admin override or record of action. | ☐            |


The sources indicate that the plugin implements **Role-based visibility** for accessing administrative pages where workflow logs are displayed, but it **does not implement** dedicated features for **log retention settings, integrity protection (immutability), or secure export of audit data**.

Here is the implementation status of the **Security & Access to Logs** check items:

| # | Check Item | Implemented? |
| :--- | :--- | :--- |
| **5.1** | **Role-based visibility** | **☒** |
| **5.2** | **Export restriction** | **☐** |
| **5.3** | **Log retention settings** | **☐** |
| **5.4** | **Integrity protection** | **☐** |

### Implementation Details

#### 5.1 Role-based visibility (Only authorized users can view audit logs)

**Implemented (via Admin Permissions).** While there is no dedicated "Audit Log" viewer, access to the primary administrative interface where workflow logs and booking metadata are visible is restricted based on user capabilities defined in the **Admin Panel Permissions** setting.

*   **Access Control:** The system's **`WPBC_Admin_Menus`** class registers administrative pages and simplifies permission management by mapping simplified user roles to the correct WordPress capabilities. Access to these sensitive areas, such as the Timeline View or Booking Listing (where logs are associated with bookings), is inherently restricted to authorized personnel.

#### 5.2 Export restriction (Export of audit data is restricted or password-protected)

**Not Implemented.** The plugin does not support secure, restricted export of audit data.

*   **Existing Export:** Export functionality is primarily for calendar synchronization via public **.ics feed URLs** or is **delegated** to a companion plugin (like "Booking Manager") for advanced export.
*   **Security Deficiency:** There is no evidence in the sources that the plugin or its delegated companions implement features for encrypting, tokenizing, or password-protecting exported job data or audit logs [Conversation History 4, 5].

#### 5.3 Log retention settings (Admin can configure how long logs are retained)

**Not Implemented.** The plugin lacks features for configuring automated, time-based deletion of logs or booking data.

*   **Automation Absence:** Although the plugin uses a custom, traffic-dependent **pseudo-cron system** (`WPBC_Cron`) for scheduled tasks like Google Calendar imports, the sources do not document that this system is utilized for routine data retention or auto-deletion of logs [Conversation History 2].
*   **Limited Cleanup:** The only mass data cleanup mechanism described is the **conditional deletion of all plugin data** upon deactivation, which only occurs if the administrator has explicitly enabled the `booking_is_delete_if_deactive` option.

#### 5.4 Integrity protection (Logs cannot be deleted or altered)

**Not Implemented.** The architectural design of log storage fundamentally prevents immutability and integrity protection.

*   **Storage Limitation:** Workflow logs and notes are stored as **serialized metadata** within the **`booking_options` column** of the custom database table (`{$wpdb->prefix}booking`) [190, Conversation History 6].
*   **Immutability Lack:** This storage approach means the logs are not inherently tamper-proof, and the plugin **lacks immutability protection** [Conversation History 6]. An administrator with database access or access to backend functions that modify this serialized array could potentially delete or alter log entries without the change itself being recorded.


Based on the detailed analysis of the security mechanisms and data persistence methods, I would mark the implementation of **Security & Access to Logs** as a **4** on a scale of 1 to 10.

### Rationale for the Score

The score reflects that while foundational access control is implemented, the critical features concerning data lifecycle management (retention) and log integrity (immutability) are missing, largely due to architectural limitations in how logs are stored.

| # | Check Item | Implementation Status (Based on Sources) | Why 4/10? |
| :--- | :--- | :--- | :--- |
| **5.1** | **Role-based visibility** | **Implemented.** | Access to administrative pages, including those showing booking metadata/workflow logs, is managed by the plugin's **Admin Panel Permissions** and controlled by the **`WPBC_Admin_Menus`** class, which maps roles to WordPress capabilities. |
| **5.2** | **Export restriction** | **Not Implemented.** | The plugin's mechanisms for data export (public .ics feed URL, or functionality delegated to the "Booking Manager" companion plugin) lack any documented features for tokenization, encryption, or password protection for audit data [12, 21, Conversation History 4, 5]. |
| **5.3** | **Log retention settings** | **Not Implemented.** | There is no documented system for configuring automated log retention rules (e.g., auto-deleting logs after six months). The only mass cleanup action is conditional upon the administrator explicitly opting into data deletion when the plugin is deactivated (`booking_is_delete_if_deactive`) [38, Conversation History 2]. |
| **5.4** | **Integrity protection** | **Architecturally Prevented.** | Workflow logs and meta options are stored as a **single serialized array** in the **`booking_options` column** of the custom database table. This structural design is explicitly noted as being **inefficient, not queryable, and breaking database normalization**. This architecture means the logs **lack immutability protection** and could potentially be altered or deleted by an administrator with sufficient access without the change itself being audited [Conversation History 6]. |

The implementation of **Security & Access to Logs** requires a multi-pronged approach to address data integrity, retention, and secure export, primarily by correcting the foundational architectural flaw of storing log data as non-queryable serialized metadata.

The plan is structured around establishing a normalized audit table, configuring automated deletion, and delegating secure export functions.

### I. Data Foundation and Integrity Protection (5.4)

To ensure logs are immutable, queryable, and protected from unauthorized alteration, the current serialized metadata structure must be replaced with a dedicated, normalized database table.

1.  **Dedicated Audit Log Table:** A new custom database table (e.g., `{$wpdb->prefix}audit_log`) would be defined and created during the plugin's installation and update sequence. The setup process must be triggered by hooking into the custom action **`make_bk_action( 'wpbc_activation' )`**.
2.  **Logging Mechanism:** Logging functions would utilize **secure, prepared `$wpdb` insertions** to write immutable records directly to this new table, following the pattern used by the plugin's date and booking engines. This structure immediately fixes the architectural problem of storing meta data as a **single serialized array in the `booking_options` column**, which is explicitly noted as **inefficient and not queryable**.
3.  **Role-Based Visibility (5.1):** A dedicated administrative page, "Audit Log Viewer," would be registered using the **`WPBC_Admin_Menus`** class. This class simplifies permission management by mapping user roles to the correct WordPress capabilities, ensuring only authorized personnel can access the log. The content would be rendered using the **`wpbc_page_structure_show`** delegation hook.

### II. Automated Log Retention Policy (5.3)

Implementing a configurable log retention policy requires new configuration fields and utilizing the plugin's custom background processing system.

1.  **Configuration Settings:** New settings fields (e.g., retention period in days) would be defined within the **`WPBC_Settings_API_General`** class in `core/admin/api-settings.php`. These options would be saved using the default **'separate' strategy** (individual rows in `wp_options` table).
2.  **Scheduling Automation:** A custom recurring task (e.g., `wpbc_cron_run_retention_policy`) would be defined and scheduled using the public methods of the **`WPBC_Cron`** class. This is necessary because the plugin utilizes a **traffic-dependent pseudo-cron system** that relies on the WordPress `init` action (priority 9) for execution, rather than native WP-Cron.
3.  **Deletion Logic:** The scheduled task would retrieve the configured retention period, execute **direct `$wpdb` queries** to identify and permanently delete records older than the policy limit from the `audit_log` table, ensuring efficient database cleanup.

### III. Secure Export Delegation (5.2)

Since the plugin lacks native security controls for file export, this feature would be delegated to a companion plugin, following the established architectural pattern.

1.  **Administrative UI:** A button labeled "Export Audit Log (Secure)" would be added to the Admin UI, likely using the standardized procedural helper functions in `core/any/admin-bs-ui.php`.
2.  **Delegation Hook:** Clicking the button would trigger a specific plugin action hook (e.g., `do_action('wpbm_export_audit_secure', $params)`).
3.  **Companion Plugin Responsibility:** This hook explicitly follows the **Extensive Delegation** philosophy. The required **"Booking Manager" companion plugin** would be responsible for listening to this hook and executing the complex external logic of compiling the data and applying security measures like password protection or encryption, features the core plugin does not handle [Conversation History 4, 5].



---

### **6. Admin & Reporting Interface**

| #   | Check Item           | Description                                                                    | Implemented? |
| --- | -------------------- | ------------------------------------------------------------------------------ | ------------ |
| 6.1 | Audit dashboard      | Admin interface to view logs filtered by user, type, or date range.            | ☐            |
| 6.2 | Search & filter      | Can filter audit logs (e.g., “bookings last 7 days” or “messages by client”).  | ☐            |
| 6.3 | Export logs          | Option to export audit data to CSV or JSON for compliance review.              | ☐            |
| 6.4 | Notification trigger | Option to notify admin when suspicious changes occur (e.g., deleted bookings). | ☐            |


The plugin provides robust tools for **filtering core booking data** (like status and date ranges) and has mechanisms for **admin notifications** upon status changes, but it **does not implement a dedicated Audit Dashboard, specialized log filters, or export functionality for audit data**.

The capability to track *workflow events* exists, but the primary limitation, noted repeatedly in our conversation, is that logs are stored inefficiently as **serialized metadata**, making dedicated search and filter functionality (6.1 and 6.2) architecturally challenging [Conversation History 6].

| #   | Check Item | Implemented? |
| :--- | :--- | :--- |
| **6.1** | **Audit dashboard** | **☐** |
| **6.2** | **Search & filter** | **☒ (For Booking Data, not Audit Logs)** |
| **6.3** | **Export logs** | **☐** |
| **6.4** | **Notification trigger** | **☒** |

### Implementation Details

#### 6.1 Audit dashboard (Admin interface to view logs filtered by user, type, or date range)

**Not Implemented.** The plugin features administrative pages like the Booking Listing and Timeline View. These pages display booking status and associated metadata (where workflow logs are stored). However, the sources do not document a specific, dedicated "Audit Dashboard" designed to view or aggregate these historical workflow logs [Conversation History 3, 6].

#### 6.2 Search & filter (Can filter audit logs)

**Partially Implemented (for core booking data, not audit logs).** The plugin provides a powerful data engine to filter the core booking listing, but this system is designed for active booking records, not for querying serialized audit data.

*   **Query Engine:** The file `core/admin/wpbc-sql.php` is the "data engine" responsible for dynamically constructing complex SQL queries (`wpbc_get_sql_for_booking_listing`) based on user filters (date ranges, keywords, status, etc.).
*   **Filters:** The admin toolbar factory (`core/admin/wpbc-toolbars.php`) assembles the filters used on admin pages. The query engine is highly extensible, allowing custom conditions to be injected into the `WHERE` clause via numerous filters (e.g., `get_bklist_sql_keyword`).
*   **Limitation:** This powerful filtering relies on querying normalized data. Since workflow logs are stored in a **single serialized array in the `booking_options` column**, this data is explicitly noted as **not queryable** and **inefficient**, making specialized filtering of individual audit actions impossible with the current architecture [190, 220, 223, Conversation History 6].

#### 6.3 Export logs (Option to export audit data to CSV or JSON)

**Not Implemented.** Export functionality is confined to calendar synchronization and specialized export features, none of which cover audit logs.

*   **Delegated Export:** Advanced import/export functionality is **Extensively Delegated** to the required companion plugin, "Booking Manager".
*   **Security Deficiency:** The plugin implements **no documented option for the secure export of audit logs** to formats like CSV or JSON, nor does it support password protection or encryption for data exports [Conversation History 4, 5].

#### 6.4 Notification trigger (Option to notify admin when suspicious changes occur)

**Implemented.** The system notifies administrators of crucial workflow changes, which can include status updates indicative of suspicious activity (e.g., deleted bookings).

*   **Action Hooks:** The Developer API documents action hooks that fire upon status changes (e.g., `wpbc_deleted_booking_resources`). These hooks provide clear extension points for developers to trigger notifications.
*   **Email Notifications:** When a booking status changes (Approved, Denied, Deleted, Trash/Reject), dedicated email templates are sent via the **Email API** to the administrator(s). The Admin Notification email specifically includes shortcodes like `[moderatelink]`, `[click2approve]`, and `[click2decline]` to streamline quick action in response to new/suspicious status changes.
*   **Debugging Tool Notifications:** The plugin also includes diagnostic tools (`core/wpbc-debug.php`) that can generate and display dynamic, dismissible administrative notices for issues like severe server configuration problems (e.g., Suhosin limits), showing a built-in mechanism for critical alerts.


The implementation of the **Admin & Reporting Interface** is marked as a **4** on a scale of 1 to 10.

### Rationale for the Score

The score reflects that while the plugin successfully implements a powerful **Notification Trigger** (Check Item 6.4) and possesses an extensible filtering mechanism for core booking data, the fundamental requirements for a queryable **Audit Dashboard** (6.1), **Log Filtering** (6.2), and **Secure Export** (6.3) are either completely missing or architecturally prevented by the current data storage method [Conversation History 8].

| # | Check Item | Implementation Status (Based on Sources and History) | Why 4/10? |
| :--- | :--- | :--- | :--- |
| **6.1** | **Audit dashboard** | **Not Implemented.** | No dedicated administrative interface exists to aggregate and display workflow logs [Conversation History 8]. |
| **6.2** | **Search & filter** (Audit Logs) | **Architecturally Prevented.** | While the plugin has a query engine for filtering core booking data, audit logs are stored as a **single serialized array** in the non-queryable `booking_options` column. This structural flaw makes searching and filtering individual log events impossible, crippling the dashboard feature [Conversation History 8]. |
| **6.3** | **Export logs** (CSV/JSON) | **Not Implemented/Delegated.** | There is no documented feature for exporting audit log data for compliance review. Export features are limited to public ICS feeds or are delegated to a companion plugin ("Booking Manager"), without mention of password protection or encryption [Conversation History 8]. |
| **6.4** | **Notification trigger** | **Implemented.** | The system notifies administrators of crucial workflow changes (like status changes or deleted bookings) using **action hooks** and a granular Email API. The administrative email template specifically includes **[moderatelink]**, **[click2approve]**, and **[click2decline]** shortcodes for quick management actions. |

The score is low because the critical reporting features required for compliance (dashboard, queryable logs, and export) are functionally unavailable due to architectural limitations, regardless of the strong notification system.


The implementation of the **Admin & Reporting Interface** requires a fundamental shift in data architecture to support queryable logs, as the existing workflow logs are stored in a **single, serialized array** in the `booking_options` column, which is explicitly noted as **inefficient, not queryable, and detrimental to database normalization**.

The high-level implementation will focus on three core areas: correcting data architecture, building the queryable admin interface, and utilizing the existing robust notification system.

### I. Data Persistence: Normalized Audit Log Table (Enabling 6.1, 6.2, 6.3)

To enable robust search, filtering, and export of audit logs, a dedicated, queryable database structure is mandatory.

1.  **Custom Database Table Creation:** A new, normalized database table (e.g., `{$wpdb->prefix}audit_log`) must be created during the plugin's activation sequence, leveraging the custom lifecycle hook **`make_bk_action( 'wpbc_activation' )**.
2.  **Normalized Structure:** This table would store immutable, queryable fields such as: `log_id`, `user_id` (the actor), `timestamp` (UTC), `action_type` (CREATE, UPDATE, DELETE, FAILED\_AUTH), `resource_id`, and `details` (for additional context).
3.  **Logging Functions:** All logging actions (e.g., those triggered by events like `wpbc_track_new_booking` or `wpbc_deleted_booking_resources`) must perform **secure, direct `$wpdb` insertions** into this new table, using prepared statements for security.

### II. Audit Dashboard and Query Engine (6.1, 6.2)

A dedicated administrative interface must be built to interact with the new `audit_log` table.

1.  **Admin Page Registration (Audit Dashboard):** A new submenu page, "Audit Log," would be registered using the **`WPBC_Admin_Menus`** class. The class utilizes a capability map to enforce **Role-based visibility** (Check Item 5.1 in previous query). The page content rendering would be delegated via the custom action hook **`wpbc_page_structure_show`**.
2.  **Filter Toolbar Construction:** The "toolbar factory" file, **`core/admin/wpbc-toolbars.php`**, would be utilized to assemble a specialized toolbar for the audit log. This toolbar would use procedural helper functions from **`core/any/admin-bs-ui.php`** (e.g., `wpbc_bs_select`, `wpbc_bs_display_tab`) for consistent styling.
3.  **Log Filtering and Search (Data Engine):** The display logic will utilize the patterns established in the existing **`core/admin/wpbc-sql.php`**. The primary function, `wpbc_get_sql_for_booking_listing` (or an equivalent audit function), would be extended to dynamically construct complex SQL queries against the **`audit_log`** table based on user filters (user ID, date range, action type). This leverages the existing, highly extensible filtering system that uses **`apply_filters`** calls (e.g., `get_bklist_sql_keyword`) to inject custom `WHERE` clauses.

### III. Export and Notification Features (6.3, 6.4)

These features rely on existing system components for delivery and external delegation for security.

1.  **Notification Trigger (6.4):** This functionality is already well-supported by the existing architecture:
    *   **Hook Integration:** The system relies on documented action hooks (e.g., `wpbc_booking_approved`, `wpbc_deleted_booking_resources`). A custom function would hook into these events to check if the action is "suspicious" (e.g., deletion) and trigger an alert.
    *   **Admin Messaging:** Immediate feedback would be provided using the **debugging utility function** `wpbc_admin_show_top_notice()` (from `core/wpbc-debug.php`) to display dynamic, dismissible administrative notices.
2.  **Export Logs (6.3):** Since the core plugin does not implement secure export features, this must be handled via delegation:
    *   A dedicated export button on the "Audit Log" dashboard would trigger a specific plugin action hook (e.g., `do_action('wpbm_export_audit_data')`).
    *   This hook would require the **"Booking Manager" companion plugin** to listen and execute the complex external logic necessary to compile the query results (CSV/JSON) and apply security measures (e.g., password protection or encryption) before delivery.
---

### ✅ **Verification Levels**

| Level                           | Criteria                                                                              |
| ------------------------------- | ------------------------------------------------------------------------------------- |
| **Basic**                       | Logs booking and message changes with timestamps.                                     |
| **Good**                        | Logs all booking, reward, and messaging actions with user identity and audit view.    |
| **Excellent (Full Compliance)** | Adds secure audit database, integrity protection, retention policy, and admin alerts. |

---

Would you like me to generate this as a **CSV checklist** (so you can tick what’s implemented in your plugin and share with developers)?
