Great — let’s break down **“Right to opt out of marketing but still receive operational updates”** into clear **checklist items** you can use to verify plugin implementation.

---

## **Requirements Breakdown**

### 1. Client Opt-Out Options

* [ ] Clients have a **clear way to opt out of marketing communications** (e.g., checkbox in booking form, profile settings, or unsubscribe link in emails).
* [ ] Opt-out applies **only to marketing messages** (promotions, newsletters, referral campaigns).
* [ ] Opt-out does **not affect operational updates** (booking confirmations, reminders, status updates).


Based on the sources and our conversation history, the requested Client Opt-Out Options are **not explicitly implemented** as a dedicated, client-facing feature for managing marketing communications, although the architectural components exist to easily classify and control all outgoing transactional emails.

Here is a detailed breakdown of the implementation status:

### Client Opt-Out Options Implementation Status

| Feature | Implementation Status (Based on Sources) | Supporting Architectural Evidence |
| :--- | :--- | :--- |
| Clients have a **clear way to opt out of marketing communications** (e.g., checkbox in booking form, profile settings, or unsubscribe link in emails). | **No** | The sources do not describe any dedicated mechanism, UI element, setting, or logic that allows a client (visitor) to manage or opt out of any messaging, whether marketing or operational. The primary transactional emails are sent automatically based on status changes (e.g., Approval, Deletion). |
| Opt-out applies **only to marketing messages** (promotions, newsletters, referral campaigns). | **Partial/Architecturally Capable** | The plugin's current email architecture supports only **transactional notifications** tied to the booking lifecycle (e.g., Approved, Pending/Denied, Deleted). These are inherently operational. The plugin **does not mention** or include any native support for "marketing messages" (promotions, newsletters). |
| Opt-out does **not affect operational updates** (booking confirmations, reminders, status updates). | **Yes (Operational Controls Implemented)** | The architecture provides granular control over **operational updates** at the administrative level, but not the client level: |
| | | *   **Granular Control:** Each transactional email notification (which serves as an operational update) is defined by a dedicated class (e.g., `WPBC_Emails_API_Approved`) that specifies fields for **enabling or disabling that specific notification**. This allows the administrator to control which operational updates are sent. |
| | | *   **Global Control Filter:** The abstract `WPBC_Emails_API` class includes the filter **`wpbc_email_api_is_allow_send`**, which acts as a global "kill switch" that can be used programmatically to prevent any email from being sent. This filter could be leveraged to enforce opt-out rules, ensuring operational messages, if enabled, are still sent. |

### Conclusion on Client Opt-Out

While the plugin's sophisticated Email API allows the administrator to control *which* operational updates are sent, and the system's focus on transactional updates inherently aligns with operational needs, there is **no architecture** documented that allows the client to influence this delivery, nor does the system support marketing communications that would require an opt-out.

***

### Numerical Marking (1-10 Scale)

| Feature | Rating (1-10) | Justification (Based on Architectural Evidence) |
| :--- | :--- | :--- |
| Clients have a **clear way to opt out of marketing communications**. | **1** | No evidence of client-facing preference settings, forms, or unsubscribe links. |
| Opt-out applies **only to marketing messages**. | **5** | The plugin does not send marketing messages, fulfilling this criterion by omission. Its email system is purely transactional/operational. |
| Opt-out does **not affect operational updates**. | **7** | The separation between different types of transactional emails (operational updates) is robustly enforced by dedicated, individually controllable email classes (e.g., `WPBC_Emails_API_Approved`) and the global `wpbc_email_api_is_allow_send` filter. This separation provides the architectural capability to protect operational messages, even if a client opt-out system were added. |

This implementation plan details how to introduce clear client opt-out mechanisms for marketing communications while ensuring **operational notifications** remain unaffected, leveraging the plugin’s established **Settings API, Email API**, and **Data Abstraction Layers**.

### I. Data Model and User Preference Persistence

The core challenge is capturing and storing the client's marketing preference, distinguishing it from booking-specific metadata.

1.  **Form Integration (Opt-Out UI):**
    *   The booking form definition uses a text-based, shortcode-like syntax, processed by **`core/form_parser.php`**. This parser would need minor extension to recognize a new field type, such as `[marketing_opt_in]`, or an existing field (like a checkbox) would be repurposed and tagged as the opt-in flag.
    *   The **`wpbc_parse_form()`** function uses regular expressions to deconstruct the shortcode syntax into a structured array for rendering.
2.  **Data Storage:** The client's opt-out status must be saved. While custom booking data is often saved via **`wpbc_save_booking_meta_option()`** to the `booking_options` column, a global, persistent opt-out status (if the client is a registered user, or associated with their email) is better stored as a separate option or user meta.
    *   If stored per booking, the status is saved using `wpbc_save_booking_meta_option()`.
    *   If stored globally (for a repeat client or user), a separate WordPress option would be used, managed via the wrappers **`update_bk_option()`** and **`get_bk_option()`** defined in **`core/wpbc-core.php`**.

### II. Message Classification and Enforcement (Operational Integrity)

This is the most critical step, ensuring that the opt-out preference is applied only to marketing content while preserving **operational updates** (booking confirmations, status updates).

1.  **Message Classification:** All existing email templates (e.g., **`WPBC_Emails_API_Approved`**, **`WPBC_Emails_API_Deleted`**) are inherently **operational notifications**, as they are strictly tied to a booking status lifecycle change. Any new promotional content (marketing) must be implemented using a distinct, new class (e.g., `WPBC_Emails_API_Marketing`).
2.  **Enforcement Hook (The Global Kill Switch):** The global filter **`wpbc_email_api_is_allow_send`** (defined in **`WPBC_Emails_API`** in **`core/any/api-emails.php`**) must be used to enforce the opt-out.
    *   A custom function would hook into `wpbc_email_api_is_allow_send`.
    *   **Operational Check:** If the email being dispatched is one of the existing operational classes (e.g., Approval, Denial), the function always returns `true`, allowing the email to send, regardless of the client's opt-out preference.
    *   **Marketing Check:** If the email is the new Marketing type, the function retrieves the client's stored opt-out status (via `get_bk_option()` or user meta). If the client has opted out, the filter returns `false`, blocking the delivery of the marketing message.

### III. Unsubscribe Mechanism (Email Link)

To provide a clear way for clients to opt out from communications delivered via email, a dynamic, secure unsubscribe link is required.

1.  **Custom Shortcode Generation:** The Email API uses a robust **Shortcode Replacement Engine** to populate content dynamically. A new shortcode, such as `[marketing_unsubscribe_link]`, would be registered. The function generating this shortcode must retrieve the client's unique identifier and a **security nonce** to protect the subsequent action.
2.  **AJAX Handler for Opt-Out:**
    *   The link generated by the shortcode would point to a new administrative AJAX action (e.g., `wpbc_ajax_CLIENT_OPT_OUT`).
    *   This action must be securely registered using the extensibility filter **`wpbc_ajax_action_list`**.
    *   The server-side handler must first perform **nonce verification** (though typically nonces are checked on admin-facing AJAX, a secure token is needed here for public access) and validate the client ID.
    *   Upon successful validation, the handler would update the client's permanent opt-out preference in the database using **`update_bk_option()`**.
  



---

### 2. Message Categorisation

* [ ] Plugin distinguishes between **operational messages** and **marketing messages** in configuration.
* [ ] Operational = confirmations, reminders, cancellations, auto-updates.
* [ ] Marketing = newsletters, loyalty/referral promotions, special offers.
* [ ] Each message template is tagged as **“operational” or “marketing.”**

Based on the sources, the plugin utilizes a sophisticated **Email API** that manages notifications based on the booking lifecycle. This system inherently treats its communications as operational updates but **does not formally implement a distinct configuration layer to categorize messages explicitly as "operational" versus "marketing."**

Here is a breakdown of the implementation status:

### Message Categorisation Implementation Status

| Feature | Implementation Status (Based on Sources) | Supporting Architectural Evidence |
| :--- | :--- | :--- |
| Plugin distinguishes between **operational messages** and **marketing messages** in configuration. | **No, but distinction is architecturally implied.** | The configuration system, managed by dedicated classes extending the abstract **`WPBC_Emails_API`**, only supports notifications tied to the core transactional workflow. There are **no documented files, classes, or settings fields** that define, manage, or send general "marketing messages" (newsletters, promotions, referral campaigns). |
| Operational = confirmations, reminders, cancellations, auto-updates. | **Yes (Operational updates are fully supported).** | All defined email types are **transactional** (operational). These updates are triggered by changes in booking status (e.g., approval, denial, deletion). Examples include: |
| | | *   **Confirmations:** `WPBC_Emails_API_Approved` (sent upon approval). |
| | | *   **Cancellations:** `WPBC_Emails_API_Deleted` and `WPBC_Emails_API_Trash` (sent upon deletion or denial). |
| | | *   **Admin Notifications:** `WPBC_Emails_API_NewAdmin` (includes quick action links like `[click2approve]`). |
| Marketing = newsletters, loyalty/referral promotions, special offers. | **No** | The plugin’s email system is designed purely for **transactional/operational purposes**. The sources provide no information or architectural structure for creating or managing generalized marketing content. |
| Each message template is tagged as **“operational” or “marketing.”** | **Partial (Tagging is absent, but control exists).** | The templates themselves do not carry an explicit "operational" or "marketing" tag in configuration. However, the plugin provides the ultimate control mechanism to enforce separation: the filter **`wpbc_email_api_is_allow_send`**. This filter acts as a **global "kill switch"** that can programmatically block any email from being sent. Since all native templates are operational, this filter could be used to enforce rules on any *hypothetical* marketing email class added externally, ensuring operational integrity. |

### Numerical Marking (1-10 Scale)

| Feature | Rating (1-10) | Justification (Based on Architectural Evidence) |
| :--- | :--- | :--- |
| Plugin distinguishes between operational messages and marketing messages in configuration. | **5** | The plugin supports a comprehensive set of **operational templates**, meaning half of the classification is inherently covered. It lacks any architecture for marketing messages, fulfilling the separation criterion by omission. |
| Operational = confirmations, reminders, cancellations, auto-updates. | **10** | This functionality is **fully implemented and central** to the plugin's notification system, with dedicated, customizable template classes for all core lifecycle events (Approved, Deleted, Pending, etc.). |
| Marketing = newsletters, loyalty/referral promotions, special offers. | **1** | The plugin contains no structural or functional support for marketing communications. |
| Each message template is tagged as “operational” or “marketing.” | **6** | No explicit tagging exists in the configuration. However, the advanced architectural control provided by the **`wpbc_email_api_is_allow_send`** filter means the system is fully capable of applying classification rules (tags) programmatically upon execution, scoring higher than a simple "Not Implemented." |


The implementation of **Message Categorisation** would involve leveraging the plugin’s object-oriented **Email API Pattern** and its dedicated filter system to formally tag and enforce separation between transactional/operational messages and any future marketing communications.

### I. Formalizing Message Classification (Configuration Layer)

The existing architecture provides a highly extensible framework where each email template is managed by a dedicated class extending the abstract `WPBC_Emails_API`.

1.  **Adding a Classification Tag:** Each concrete email class (e.g., `WPBC_Emails_API_Approved`, `WPBC_Emails_API_Deleted`) would be extended to include a new field, `message_type`, within its setting blueprint, which is defined by the `init_settings_fields()` method.
2.  **Tagging Existing Templates:** All current notification classes (Approved, Pending, Denied, New Admin/Visitor) would be explicitly tagged as **"operational"**.
3.  **Introducing Marketing Templates:** A new concrete email class (e.g., `WPBC_Emails_API_Marketing`) would be created by extending the `WPBC_Emails_API`. This class would define the subject, content, and fields necessary for marketing campaigns and be tagged as **"marketing"**. It would also require a new administrative sub-tab under Booking > Settings > Emails to manage its content.

### II. Runtime Enforcement and Segregation

The distinction between operational and marketing tags is enforced immediately before the email is sent, leveraging the most powerful extension point in the Email API.

1.  **The Global Dispatch Filter:** The abstract `WPBC_Emails_API` class defines the critical filter **`wpbc_email_api_is_allow_send`**. This filter acts as a "kill switch" that programmatically prevents any email from being dispatched.
2.  **Enforcing Opt-Out Rules:** A custom function would hook into the `wpbc_email_api_is_allow_send` filter.
    *   **Operational Check:** The function would check the `message_type` tag of the email being processed. If the tag is **"operational,"** the filter automatically returns `true`, ensuring critical updates (confirmations, status changes) are always delivered, regardless of the client's marketing opt-out status.
    *   **Marketing Check:** If the tag is **"marketing,"** the function would retrieve the recipient's saved opt-out preference from the database. If the client has opted out, the filter returns `false`, preventing the marketing email from being sent.

### III. Data Management and Template Customization

This classification system relies entirely on the plugin's existing robust systems for configuration and content population.

1.  **Configuration Persistence:** The new `message_type` setting, along with the customized subject and body, would be stored and validated using the `WPBC_Settings_API` framework, which ensures consistent data integrity and saving.
2.  **Dynamic Content:** Both operational and marketing templates would utilize the **Shortcode Replacement Engine** built into the `WPBC_Emails_API`. This allows administrators to customize content using placeholders like `[booking_id]`, `[dates]`, and `[resource_title]`, ensuring dynamic data is included in either type of communication.

3.  
---

### 3. Admin Controls

* [ ] Admin can configure marketing campaigns separately from operational notifications.
* [ ] Admin can **see client consent status** (opted in/out of marketing).
* [ ] Marketing tools (Mailchimp, SendGrid, etc.) integrate with **subscription preferences** automatically.
* [ ] Opt-out does not delete client data but updates **consent flag in DB**.

Based on the sources and our conversation history, the requested "Admin Controls" related to message categorization, client consent, and marketing tool integration are **not implemented** in the plugin's core architecture. The system is designed strictly around **operational/transactional** messaging.

Here is a detailed breakdown of the implementation status:

### Admin Controls Implementation Status

| Feature | Implementation Status (Based on Sources) | Supporting Architectural Evidence |
| :--- | :--- | :--- |
| Admin can configure marketing campaigns separately from operational notifications. | **No** | The plugin's email system is exclusively designed for **transactional/operational** notifications tied to the booking lifecycle (e.g., **Approved, Deleted, New Admin/Visitor, Pending/Denied, Trash/Reject**). There are **no files, classes, or settings** documented for defining or configuring generic "marketing campaigns" such as newsletters or promotions. All existing email templates are inherent operational updates. |
| Admin can **see client consent status** (opted in/out of marketing). | **No** | There is **no evidence** of a system for storing or displaying a client's "consent status" or "opt-in/out" flag for marketing. The plugin supports storing custom data as a **serialized array** in the `booking_options` column via **`wpbc_save_booking_meta_option()`**, but there is no confirmed dedicated field or interface for displaying a standardized consent status. |
| Marketing tools (Mailchimp, SendGrid, etc.) integrate with **subscription preferences** automatically. | **No** | The plugin exhibits advanced synchronization with external calendar services (ICS feeds, Google Calendar API), but there is **no mention** of integration with dedicated marketing automation or CRM tools like Mailchimp or SendGrid for subscription management. External integration points are typically for triggering side-effects upon booking events (like `wpbc_track_new_booking`), but this is not confirmed for subscription preferences. |
| Opt-out does not delete client data but updates **consent flag in DB**. | **Architecturally Capable (But Feature Absent)** | The system provides robust functions for **data abstraction** and management that could support a consent flag: |
| | | *   **Update Flag:** The foundation for updating a flag is supported via wrappers like **`update_bk_option()`** for global settings or **`wpbc_save_booking_meta_option()`** for booking-specific data. |
| | | *   **Non-Deletion:** The plugin's architecture manages data deletion explicitly. For instance, data is only deleted upon plugin deactivation if the administrator has enabled the **`booking_is_delete_if_deactive`** option. Thus, a specific "consent flag" would be updated, not deleted, unless explicitly triggered by an administrative action like **`wpbc_ajax_DELETE_APPROVE()`**. |
| | | However, since the client opt-out feature itself is not implemented, the existence of this specific database flag is **unconfirmed**. |

***

### Numerical Marking (1-10 Scale)

| Feature | Rating (1-10) | Justification (Based on Architectural Evidence) |
| :--- | :--- | :--- |
| Admin can configure marketing campaigns separately from operational notifications. | **1** | The plugin supports *only* operational notifications (Approved, Deleted, etc.) and lacks any architectural support for defining or managing marketing campaigns. |
| Admin can **see client consent status** (opted in/out of marketing). | **1** | There is no dedicated UI or data structure documented to capture or display client consent status for marketing. |
| Marketing tools (Mailchimp, SendGrid, etc.) integrate with **subscription preferences** automatically. | **1** | Integration is explicitly supported for calendar sync (Google Calendar, ICS feeds), but there is no evidence of native integration or API support for third-party marketing or subscription management tools. |
| Opt-out does not delete client data but updates **consent flag in DB**. | **3** | The plugin has the necessary architectural components (DB access via `$wpdb`, meta option storage, and update wrappers) to implement this feature easily. However, since the core client opt-out functionality is missing, the flag itself is not implemented. |


The implementation of Admin Controls for message categorization, client consent, and marketing integration requires extending the plugin's existing object-oriented frameworks, particularly the **Email API** and the **Custom Settings API**.

This high-level overview details the steps necessary to separate marketing from operational communications and manage client consent across the plugin's architecture.

### I. Message Categorization and Configuration

The plugin's existing emails are inherently transactional/operational. Formal separation requires explicitly tagging these templates and defining separate configuration paths for marketing campaigns.

1.  **Formal Message Tagging:** The abstract class **`WPBC_Emails_API`** defines the structure for all notification templates.
    *   Each existing operational class (e.g., `WPBC_Emails_API_Approved`, `WPBC_Emails_API_Deleted`) must be updated to include an explicit **`message_type`** property (set to "Operational") within its settings definition, implemented via the **`init_settings_fields()`** method.
2.  **Marketing Configuration Separation:** A new concrete class, `WPBC_Emails_API_Marketing`, must be created that extends **`WPBC_Emails_API`** and is explicitly tagged as **"Marketing."**
    *   This new class manages templates for newsletters or promotions, allowing administrators to configure them separately from operational updates.
    *   The corresponding settings page would be registered using the standard admin menu delegation hooks, such as **`wpbc_define_nav_tabs`**, to appear as a new sub-tab under Booking > Settings > Emails.

### II. Client Consent Management and Persistence

This phase focuses on capturing, storing, displaying, and enforcing the client's marketing opt-out status via a **consent flag**.

1.  **Consent Flag Storage:** The opt-out status must be stored in the database.
    *   If tied to a specific booking request (e.g., checkbox on the form), the status is stored as a **serialized array** in the **`booking_options`** column of the custom booking table via **`wpbc_save_booking_meta_option()`**.
    *   For global client opt-out (user profile), a new option is stored using the settings wrappers **`update_bk_option()`** and **`get_bk_option()`**. The requirement that opt-out updates a flag, rather than deleting data, aligns with the plugin's existing data integrity philosophy, where data deletion is an explicit, configurable process (`booking_is_delete_if_deactive`).
2.  **Admin Visibility of Consent:** To allow administrators to **see client consent status**, the stored consent flag must be retrieved and displayed on the administrative booking details view, utilizing existing UI helpers found in **`core/any/admin-bs-ui.php`**.
3.  **Opt-Out Enforcement (Operational Integrity):** The filter **`wpbc_email_api_is_allow_send`** (the global "kill switch") is the point of enforcement.
    *   A custom function hooks into this filter and checks the template's **`message_type`** tag.
    *   If "Operational," the message is allowed to send, ensuring **operational updates** (confirmations, status changes) are never blocked.
    *   If "Marketing," the function checks the client's consent flag. If opted out, the filter returns `false`, preventing dispatch.

### III. External Marketing Integration

Integrating with third-party marketing tools (e.g., Mailchimp) requires a new API service triggered by a change in the client's consent status.

1.  **Marketing API Engine:** A new, dedicated API service class (e.g., `WPBC_Mailchimp_API`) would be created. This class would handle secure communication with the external tool's API (e.g., Mailchimp's subscription endpoint).
    *   API communication would follow the pattern of the Google Calendar synchronization, using secure WordPress functions like **`wp_remote_get()`** or **`wp_remote_post()`** for external HTTP requests.
2.  **Triggering Synchronization:** A custom internal action hook (e.g., `wpbc_client_consent_updated`) would be defined and triggered immediately after the consent flag is updated in the database.
    *   This custom hook listener would instantiate the `WPBC_Mailchimp_API` class and execute a method to automatically push the client's email and their new subscription preference to the external marketing tool, ensuring preferences are kept synchronized.
  
3.  
---

### 4. Compliance & Transparency

* [ ] Unsubscribe links in all marketing emails are **clear and functional**.
* [ ] Consent status and changes are **logged with timestamp** (audit trail).
* [ ] Privacy notice (in booking form or client portal) explains the difference between operational vs marketing messages.
* [ ] If SMS/WhatsApp marketing is used, plugin supports **STOP/UNSUBSCRIBE commands**.

Based on the sources and our conversation history, the requested options for **Compliance & Transparency**, particularly concerning marketing opt-out, consent logging, and multi-channel support, are **not implemented** in the plugin's core architecture. The system is designed purely for transactional, operational emails.

Here is a detailed breakdown of the implementation status:

### Compliance & Transparency Implementation Status

| Feature | Implementation Status (Based on Sources) | Justification (Based on Architectural Evidence) |
| :--- | :--- | :--- |
| Unsubscribe links in all marketing emails are **clear and functional**. | **No** | The sources do not indicate that the plugin generates or sends marketing emails; all native communications are **transactional/operational** (e.g., **Approved, Deleted, Pending/Denied**). Furthermore, there is **no evidence** of any mechanism for generating a client-facing unsubscribe link or handling the resulting opt-out action. |
| Consent status and changes are **logged with timestamp** (audit trail). | **No** | The architecture lacks a dedicated **Conversation History** or generalized **logging system** for communication events. The focus of logging functions, where implied, is typically on booking status, debug information, or database errors (via `$EZSQL_ERROR`). The system also relies on storing booking metadata in a **serialized array** in the `booking_options` column, which is ill-suited for queryable audit trails or message logging. |
| Privacy notice (in booking form or client portal) explains the difference between operational vs marketing messages. | **No** | The plugin does **not formally classify** messages as "operational" versus "marketing" in its configuration. All existing templates are tied to the operational booking workflow. There is also **no documentation** of a client portal or any mandated privacy notice content in the settings or form editor. |
| If SMS/WhatsApp marketing is used, plugin supports **STOP/UNSUBSCRIBE commands**. | **No** | The plugin's primary and only fully supported communication channel is **Email**. There is **no documented integration** with external APIs for SMS (like Twilio) or WhatsApp. Without support for these channels, the corresponding compliance commands (STOP/UNSUBSCRIBE) cannot be implemented. |

***

### Numerical Marking (1-10 Scale)

| Feature | Rating (1-10) | Justification (Based on Architectural Evidence) |
| :--- | :--- | :--- |
| Unsubscribe links in all marketing emails are **clear and functional**. | **1** | No marketing emails are sent, and no unsubscribe link generation or opt-out handling mechanism is documented in the core Email API. |
| Consent status and changes are **logged with timestamp** (audit trail). | **1** | The architecture lacks a dedicated messaging log/audit table. The current metadata storage format (**serialized array in `booking_options`**) is unsuitable for efficient chronological audit trails. |
| Privacy notice (in booking form or client portal) explains the difference between operational vs marketing messages. | **1** | There is no formal configuration for message categorization and no mandated privacy notice content or client portal documented in the source material. |
| If SMS/WhatsApp marketing is used, plugin supports **STOP/UNSUBSCRIBE commands**. | **1** | There is no architectural support or integration with SMS or WhatsApp APIs to require this functionality. |


The implementation of Compliance and Transparency requires establishing a secure, persistent audit trail and enforcing client consent across all communication channels, leveraging the plugin’s dedicated architectural frameworks.

Here is a high-level overview of the implementation strategy:

### I. Audit Trail and Logging System

Since the plugin lacks a dedicated message history or logging system, a new persistent structure is required to capture consent changes and message delivery attempts with necessary timestamps.

1.  **Dedicated Database Table (Audit Trail):** A new, queryable database table (e.g., `wp_message_log`) would be created to record every communication attempt and every consent change. This table must be configured for creation during plugin activation by hooking into the custom action **`make_bk_action( 'wpbc_activation' )`**.
2.  **Centralized Logging Function:** A new function would be introduced to intercept the dispatch of all messages (Operational and Marketing). This function logs the communication attempt, the `message_type` (Operational/Marketing), the recipient, the timestamp, and the initial `delivery_status` before the mail wrapper calls `wp_mail()`.
3.  **Failure Logging:** The existing mail wrapper function, **`wpbc_wp_mail()`**, must be enhanced to log the final outcome. Any non-successful return from the underlying `wp_mail()` call would update the message log entry with a "Send Failure" status. Similarly, external API handlers (for SMS/WhatsApp) would log specific delivery failure codes received from the third-party gateway.

### II. Unsubscribe Links and Consent Enforcement

This phase introduces client-side opt-out mechanisms and enforces those preferences using the plugin's core "kill switch."

1.  **Unsubscribe Link Generation:** A new shortcode (e.g., `[marketing_unsubscribe_link]`) would be registered within the plugin's **Shortcode Replacement Engine**. This shortcode dynamically generates a URL that includes a unique identifier for the client and a security token/nonce.
2.  **AJAX Opt-Out Handler:** A new AJAX action (e.g., `wpbc_ajax_OPT_OUT_MARKETING`) would be registered using the **`wpbc_ajax_action_list` filter**. The handler must enforce **nonce verification** and, upon validation, update the client’s **consent flag** in the database (stored as a plugin option via **`update_bk_option()`** or user meta). The change in consent status must be logged immediately to the new audit trail (Step I).
3.  **Enforcing Opt-Out:** The abstract **`WPBC_Emails_API`** defines the global filter **`wpbc_email_api_is_allow_send`**. A custom function would hook into this filter and check the `message_type` tag. If the message is **Marketing**, the function checks the client's consent flag before allowing dispatch. If the message is **Operational** (confirmations, reminders, status updates), the filter must always return `true`, ensuring critical updates are never blocked.

### III. Compliance and Transparency Controls

New configuration fields are required to manage privacy text and multi-channel compliance.

1.  **Privacy Notice Configuration:** New text area fields would be defined within the **`WPBC_Settings_API_General`** class blueprint, using the `init_settings_fields()` method. This setting stores the text for the privacy notice that administrators must include in their booking forms or client portals.
2.  **Localization of Compliance Text:** The privacy notice field would leverage the plugin's existing custom translation system, using the **`wpbc_lang()`** function to parse and display embedded translations (e.g., `[lang=xx_XX]`) that explain the crucial distinction between operational and marketing messages.
3.  **SMS/WhatsApp STOP/UNSUBSCRIBE:** For integrated messaging channels (SMS/WhatsApp), the corresponding API handlers must be built to receive webhooks for inbound messages. The system must implement logic to detect **STOP** or **UNSUBSCRIBE** commands from the client, and automatically trigger the same logic used in the AJAX opt-out handler (Step II) to update the client’s consent flag and log the action to the audit trail (Step I).
---

## **Success Criteria Checklist**

✅ Clients can opt out of marketing without losing booking confirmations/reminders.
✅ Plugin separates marketing vs operational templates.
✅ Admin can check client consent status in dashboard/CRM.
✅ Marketing integrations respect opt-out automatically.
✅ Opt-out preferences logged with timestamp.
✅ Unsubscribe links/buttons work in every marketing message.
✅ Privacy notice explains operational vs marketing categories.

---

👉 Do you want me to **also draft test cases** (e.g., “Opt out → still get booking confirmation, not promo email”) so you can run them against the plugin to validate this feature?
