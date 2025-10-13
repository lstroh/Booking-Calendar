Great choice — let’s break down **“Security → Integration Security → OAuth authentication for external calendars”** into clear, checkable items so you can verify whether a plugin is handling OAuth securely with **Google Calendar API / Microsoft Outlook API**.

---

## 🔍 Breakdown Into Checkable Items

### 1. **OAuth Flow Implementation**

* [ ] Does the plugin use **OAuth 2.0 authorization flow** (redirecting user to Google/Microsoft for login/consent)?
* [ ] Are **client ID and secret** managed securely (not hardcoded in plugin code)?
* [ ] Is the **redirect URI** correctly registered with Google/Microsoft (no open redirects)?


Based on the sources, the plugin's architecture for the existing Google Calendar synchronization feature relies on an **API Key** model, and there is **no evidence** that a complete OAuth 2.0 flow is implemented. The sources focus heavily on `core/sync/wpbc-gcal.php`, which manages the settings and scheduling for this API Key-based synchronization.

Here is the implementation status for the requested OAuth Flow options:

### OAuth Flow Implementation Status

#### 1. Does the plugin use OAuth 2.0 authorization flow (redirecting user to Google/Microsoft for login/consent)?

**No.** The plugin does **not** implement an OAuth 2.0 authorization flow [Triage].

*   The implementation for Google Calendar Import relies on the administrator configuring a **Google API Key** on the settings page (`core/admin/page-import-gcal.php`).
*   The synchronization engine retrieves this API key from the plugin options (`booking_gcal_api_key`) and uses it to authenticate the request sent via `wp_remote_get()` to the Google Calendar API v3.
*   There is no documentation of a redirect, login, or consent screen for Google, Microsoft, or any other external API integration [Triage].

#### 2. Are client ID and secret managed securely (not hardcoded in plugin code)?

**Not Applicable (due to API Key usage).** However, the persistent credential (the API Key) is **not hardcoded**, but **lacks documented encryption** for storage.

*   **Credential Type:** Since OAuth 2.0 is not used, the concepts of Client ID and Secret are not relevant to the documented synchronization feature [Triage].
*   **Storage Location:** The actual credential used, the **Google API Key**, is stored as a configuration setting in the WordPress database. Lifecycle management functions, such as `wpbc_sync_gcal_activate()`, use `add_bk_option()` to initialize this setting (`booking_gcal_api_key`).
*   **Not Hardcoded:** This credential is configured by the administrator and stored in the database as a **"separate row in the wp_options table"** (the 'separate' strategy).
*   **Security:** The sources **do not mention any mechanism for encrypting** the API key or any other setting stored in the `wp_options` table [2, 190, Triage].

#### 3. Is the redirect URI correctly registered with Google/Microsoft (no open redirects)?

**Not Applicable.**

*   Since the plugin relies on a static **API Key** entered manually by the administrator, there is no OAuth flow and, consequently, no **redirect URI** or callback required for authentication [Triage].

***

### Score Assessment (1–10)

The implementation of the requested **OAuth Flow Implementation** receives a score of **1 / 10**.

**Rationale:** The core requirement of using an OAuth 2.0 flow is fundamentally unmet, as the plugin relies on a persistent API Key. Although the API Key is not hardcoded, it fails to meet the underlying security criteria for token management because the key is stored in the database without documented encryption [2, Triage].


The implementation of a secure **OAuth 2.0 flow** requires modifying the existing components responsible for configuration UI, data persistence, and background automation. This high-level overview details how the core architecture of the plugin would be adapted, focusing on the Google Calendar synchronization feature (managed by `core/sync/wpbc-gcal.php`).

### 1. Refactor UI and Initiate OAuth Authorization Flow

The administrative configuration file must be overhauled to initiate the external authorization process, replacing the manual API Key entry.

*   **Settings Controller Refactoring:** The file `core/admin/page-import-gcal.php` must be updated. Instead of collecting the persistent Google API Key, it will gather the **OAuth Client ID** and **Client Secret** [Triage].
*   **Authorization Trigger:** A button or link must be added to the UI, which, when clicked, initiates the OAuth 2.0 flow. This action redirects the administrator to the external service (e.g., Google) for login and consent. This request must strictly define the minimum necessary **scopes** (e.g., read-only access) [Triage].
*   **UI Helpers:** Existing UI helper functions (`wpbc_gcal_settings_content_field_*()`) would be updated to manage the rendering and saving of the new Client ID and Secret fields, utilizing the **WPBC\_Settings\_API** framework.

### 2. Implement and Secure the Redirect URI Handler

A custom endpoint is required to receive the authorization code and exchange it for functional tokens.

*   **Redirect URI Registration:** The plugin must define a specific URL registered with the external service as the **Redirect URI**. This URL would likely hook into WordPress's core AJAX system (`core/lib/wpbc-ajax.php`) via a custom `wp_ajax_` action.
*   **Token Exchange Logic:** The AJAX handler at this endpoint receives the authorization code and immediately uses the stored Client ID and **Client Secret** to request the initial **Access Token** (short-lived) and **Refresh Token** (long-lived) from the external service.
*   **Security:** This handler, like all sensitive admin AJAX functions, must enforce **Nonce verification** and immediately pass the tokens to the secure storage layer (Step 3).

### 3. Implementing Secure Storage and Encryption

Since the sources indicate that the current option storage (the 'separate' strategy in the `wp_options` table) lacks explicit encryption, a critical security layer must be introduced for token management.

*   **Encryption Layer:** An encryption utility must be implemented and integrated into the plugin's data abstraction wrappers defined in `core/wpbc-core.php`.
*   **Secure Persistence:** The wrappers **`update_bk_option()`** and **`get_bk_option()`** must be modified to automatically encrypt the sensitive data (the **Refresh Token**, **Access Token**, and **Client Secret**) before saving them to the database, and decrypt them upon retrieval.
*   **Storage Location:** The encrypted tokens replace the static `booking_gcal_api_key` option in the database, ensuring credentials are not stored in plain text.

### 4. Automated Token Rotation and Synchronization

The core synchronization engine must be refactored to use the dynamic token, ensuring that the background task runs reliably without manual intervention.

*   **Engine Refactoring:** The `WPBC_Google_Calendar` class is updated to utilize the Access Token (retrieved via the encrypted `get_bk_option`) in the request headers for external communication via **`wp_remote_get()`**.
*   **Token Expiration Handling:** The `run()` method must integrate logic to handle authentication failures (e.g., HTTP 401 Unauthorized, detectable via the existing robust error checking).
*   **Silent Renewal:** If a token expires, the engine automatically uses the encrypted **Refresh Token** and **Client Secret** to obtain a new Access Token. The newly acquired token is then securely saved using the encrypted **`update_bk_option()`** wrapper (Step 3).
*   **Cron Reliability:** This entire process ensures that the scheduled background execution, triggered by the traffic-dependent **pseudo-cron system** via `wpbc_silent_import_all_events()`, remains functional even when access tokens expire, fulfilling the requirement for seamless **token rotation** [Triage].



---

### 2. **Token Handling**

* [ ] Are **access tokens** stored securely (encrypted in WordPress DB, not plain text)?
* [ ] Are **refresh tokens** used for long-term access (so user doesn’t need to log in daily)?
* [ ] Does the plugin implement **token expiry handling** (refresh before expiry, re-authenticate when invalid)?
* [ ] Can tokens be **revoked manually by admin** (disconnect account)?


Based on the sources, the plugin's architecture for the documented Google Calendar synchronization feature relies on a persistent **Google API Key** rather than the OAuth 2.0 framework (Access Tokens and Refresh Tokens).

Therefore, the specific **Token Handling** options requested are generally **not implemented** in the manner described. The implementation status of the plugin's current persistent credential model is detailed below:

### Token Handling Implementation Status

#### 1. Are access tokens stored securely (encrypted in WordPress DB, not plain text)?

**Not implemented as requested.**

*   The plugin does not use short-lived **access tokens** for synchronization, as it relies on a static **Google API Key** (`booking_gcal_api_key`).
*   This persistent API key is configured by the administrator and stored in the database using the plugin's data abstraction wrappers (`add_bk_option`, `get_bk_option`).
*   The general settings of the plugin are saved using the 'separate' strategy, meaning they are stored as a **separate row in the standard WordPress `wp_options` table**.
*   The sources **do not mention any mechanism for encrypting** the API key or any other setting stored in the `wp_options` table.

#### 2. Are refresh tokens used for long-term access (so user doesn’t need to log in daily)?

**No.**

*   The concept of **refresh tokens** is **not applicable** because the plugin utilizes a persistent API Key rather than an OAuth 2.0 flow [Triage].
*   The system achieves automated long-term access by relying on the non-expiring nature of the API Key combined with the plugin's custom **pseudo-cron system** (`WPBC_Cron`). The function `wpbc_silent_import_all_events()` is executed by this cron scheduler in the background, ensuring recurring synchronization jobs run without manual administrator intervention.

#### 3. Does the plugin implement token expiry handling (refresh before expiry, re-authenticate when invalid)?

**No.**

*   Since the system does not use expiring tokens, **token expiry handling is not implemented** [Triage].
*   However, the core engine (`WPBC_Google_Calendar` class) does perform **robust error checking** using the `wp_remote_get()` function. It checks for non-200 HTTP status codes, such as **403 Forbidden**. This mechanism allows the administrator to be notified if the persistent API key is invalid or revoked externally, but this is API failure handling, not automated token refreshing.

#### 4. Can tokens be revoked manually by admin (disconnect account)?

**Partial Implementation (Revocation of the persistent key is possible).**

*   While tokens cannot be revoked, the administrator **can disconnect the account** by removing the persistent API Key from the synchronization settings page [Triage].
*   The plugin includes lifecycle functions dedicated to cleaning up credentials. For instance, `wpbc_sync_gcal_deactivate()` calls `delete_bk_option()` to remove all Google Calendar settings from the database upon plugin deactivation. This effectively removes the stored key, blocking external access.

Based on the detailed architectural analysis of the plugin's synchronization features, the implementation of the requested **Token Handling** criteria scores low due to fundamental architectural choices.

The overall score is: **3 / 10**

### Rationale

The scoring reflects that the required secure token management architecture (OAuth 2.0, encryption, rotation) is absent, replaced by a static API Key system.

| Criterion | Implementation Status | Rationale & Citation |
| :--- | :--- | :--- |
| **Access tokens stored securely (encrypted, not plain text)?** | **Not Implemented (Fails Encryption)** | The plugin does not use Access Tokens. The persistent **Google API Key** (`booking_gcal_api_key`) is stored in the database as a **"separate row in the wp_options table"** (the 'separate' strategy). The sources **do not mention any mechanism for encrypting** this sensitive key [190, Triage]. |
| **Refresh tokens used for long-term access?** | **Not Implemented (N/A)** | The plugin relies on a non-expiring **API Key** for long-term access, not short-lived Access Tokens or **Refresh Tokens** [Triage]. Background automation is handled by the custom **pseudo-cron system** (`WPBC_Cron`), bypassing the need for manual re-logins but failing to implement the requested token rotation mechanism. |
| **Token expiry handling (refresh before expiry)?** | **Not Implemented (N/A)** | Token expiry handling is **not implemented** as the system uses a persistent API Key [Triage]. Error handling is in place to catch API request failures (like HTTP 403 Forbidden) if the key is revoked externally, but this is API failure handling, not token expiry management. |
| **Tokens be revoked manually by admin (disconnect account)?** | **Partially Implemented** | While OAuth tokens cannot be revoked, the administrator **can disconnect the integration** by manually deleting the persistent API Key from the settings page. Lifecycle functions, such as `wpbc_sync_gcal_deactivate()`, are implemented to **remove these settings** upon plugin deactivation [151, Triage]. |


The implementation of robust **Token Handling** requires transitioning the synchronization feature from the current persistent API Key model to a secure OAuth 2.0 flow, integrating encryption for storage, and automating the token lifecycle using the plugin's custom architectural components.

Here is a high-level overview of how these features would be implemented:

### 1. Secure Token Storage and Encryption

The plugin's existing data abstraction layer must be fortified to ensure sensitive tokens and secrets are never stored in plain text, addressing the requirement that access tokens be stored securely (encrypted) [Triage].

*   **Encryption Utility:** New utility functions for encryption and decryption must be introduced into the foundational file, likely **`core/wpbc-core.php`**, which governs internal plugin settings management.
*   **Data Abstraction Wrappers:** The custom settings functions—**`update_bk_option()`** and **`get_bk_option()`**—must be modified. These wrappers will automatically encrypt the sensitive data (the OAuth Client Secret and the Refresh Token) before they are saved into the `wp_options` table and automatically decrypt them upon retrieval.
*   **Token Persistence:** The encrypted Refresh Token and Access Token (and the Client Secret) will replace the static Google API Key (`booking_gcal_api_key`) currently stored in the options table [144, 163, Triage].

### 2. OAuth Flow and Refresh Token Acquisition

The configuration UI must be refactored to initiate the OAuth 2.0 flow to acquire the necessary short-lived Access Tokens and long-lived Refresh Tokens for long-term access [Triage].

*   **UI Update:** The settings page (`core/admin/page-import-gcal.php`) will be updated to collect the Client ID and Client Secret and provide an "Authorize" button to initiate the external service redirect.
*   **Redirect Handler:** A secure administrative AJAX endpoint must be registered (via the custom AJAX router in `core/lib/wpbc-ajax.php`) to handle the callback from the external service. This handler receives the authorization code and exchanges it for the initial Access Token and Refresh Token.
*   **Security:** This new AJAX handler must enforce **Nonce verification** and pass the received tokens directly to the secure, encrypted storage system (Step 1).

### 3. Implementing Token Expiry Handling and Rotation

The core synchronization engine, the **`WPBC_Google_Calendar`** class, must be updated to manage the dynamic token lifecycle, using refresh tokens for long-term access and handling expiry automatically [144, 147, Triage].

*   **Token Usage:** The `WPBC_Google_Calendar::run()` method must be modified to retrieve the Access Token securely (via the decryption wrapper) and include it in the request headers when calling the Google API via `wp_remote_get()`.
*   **Expiry Detection:** The synchronization engine already performs robust error checking for non-200 HTTP status codes. This logic is extended to specifically detect authentication failure codes (e.g., HTTP 401 Unauthorized), which indicates an expired Access Token.
*   **Silent Renewal:** Upon detection of an expired token, the engine will automatically execute a silent background request using the stored **Refresh Token** and **Client Secret** to acquire a new Access Token. This new token is then saved using the encrypted `update_bk_option()` wrapper (Step 1).
*   **Automation:** Since the automated sync function, **`wpbc_silent_import_all_events()`**, is executed by the custom **pseudo-cron system** (`WPBC_Cron`), this token rotation process runs silently and automatically during scheduled background imports, ensuring no downtime or repeated re-logins are required.

### 4. Manual Token Revocation (Disconnect Account)

The administrative control must be updated to not only delete local data but also revoke access externally.

*   **Admin UI:** A "Revoke Access" control (button) is added to the sync settings page [Triage].
*   **Secure Revocation Handler:** This control triggers a new, Nonce-verified AJAX action (registered via the `wpbc_ajax_action_list` filter in `core/lib/wpbc-ajax.php`).
*   **Two-Step Cleanup:** The handler will first call the external service's API endpoint (e.g., Google's) to formally invalidate the Refresh Token. Upon success, it performs local cleanup by calling **`delete_bk_option()`** wrappers to remove the encrypted Refresh Token, Access Token, and Client Secret from the database.


---

### 3. **Scope & Permissions**

* [ ] Does the plugin request **only required scopes** (e.g., `https://www.googleapis.com/auth/calendar.events` rather than full `calendar` access)?
* [ ] For Outlook, does it use minimal Graph API scopes (e.g., `Calendars.ReadWrite` not `User.ReadWrite.All`)?
* [ ] Is the scope request **visible to the user/admin** during consent?


The sources indicate that the plugin does not implement an OAuth 2.0 flow for its core Google Calendar synchronization feature, relying instead on a static API key model. Therefore, the implementation of dynamic **Scope & Permissions** management is not present in the documented architecture.

### Scope & Permissions Implementation Status

#### Does the plugin request **only required scopes** (e.g., `https://www.googleapis.com/auth/calendar.events` rather than full `calendar` access)?

**No.** The plugin does not request scopes because it relies on a persistent **Google API Key** (`booking_gcal_api_key`) configured by the administrator on the settings page (`core/admin/page-import-gcal.php`).

*   The key is used to construct and send the request via `wp_remote_get()` to the Google Calendar API v3.
*   When using a static API key, permissions are governed by how the key was configured within the external service's ecosystem (e.g., the Google Developer Console), not by dynamic scope requests initiated by the plugin during runtime.

#### For Outlook, does it use minimal Graph API scopes (e.g., `Calendars.ReadWrite` not `User.ReadWrite.All`)?

**Not Applicable/Not Documented.**

*   The sources analyze synchronization support only for **ICS Feeds** (Import/Export) and **Google Calendar Import**.
*   There is no information in the sources regarding any integration or API communication with external services such as Outlook, Microsoft, Zoho, or HubSpot [Triage].

#### Is the scope request **visible to the user/admin** during consent?

**No.** Since the plugin uses the persistent **API Key** for synchronization, the core architectural requirement—the OAuth 2.0 consent screen—is bypassed. Consequently, there is no mechanism for the plugin to display or manage the visibility of granted scopes during the connection process [Triage].


Based on the sources, the overall score for the implementation of **Scope & Permissions** is **1 / 10**.

### Rationale Breakdown

The extremely low score is due to the plugin's foundational architecture for Google Calendar synchronization relying entirely on a static, persistent **Google API Key** rather than the dynamic, revocable OAuth 2.0 framework required to manage scopes and privileges.

| Criterion | Implementation Status | Rationale |
| :--- | :--- | :--- |
| **Does the plugin request only required scopes?** | **Not Implemented.** | The plugin uses a persistent **Google API Key** (`booking_gcal_api_key`) configured by the administrator. This architecture bypasses the use of OAuth 2.0 scopes (like `calendar.readonly`) entirely, as key permissions are set outside the plugin within the external Google Console [Triage in previous responses]. |
| **For Outlook, does it use minimal Graph API scopes?** | **Not Applicable / Not Documented.** | The sources do not document any synchronization or API communication with Outlook, Zoho, or HubSpot; only Google Calendar Import and ICS feeds are mentioned. |
| **Is the scope request visible to the user/admin during consent?** | **Not Implemented.** | Since the plugin requires manual entry of a static API Key and does not initiate an OAuth 2.0 redirect flow, there is no consent screen or mechanism to display the requested scopes to the administrator during the connection process. |


The implementation of **Scope & Permissions** requires a fundamental shift to the **OAuth 2.0 authorization flow**, moving away from the current Google API Key model [Triage]. This ensures the plugin requests and receives only the minimum necessary permissions (Least Privilege) and provides clear controls for the administrator.

This plan integrates the new OAuth logic into the plugin’s established settings framework and UI controllers, particularly leveraging the file responsible for Google Calendar configuration (`core/sync/wpbc-gcal.php`).

### Phase 1: Enforcing Minimum Scope and Visible Consent

The goal is to strictly define the minimum scope required for synchronization and make the request visible during the connection process.

*   **Scope Definition (Least Privilege):** The core synchronization logic (managed by the `WPBC_Google_Calendar` class or its controllers) must be updated to request **only the minimum required scopes** for the intended functionality, which is a **one-way import**. This means strictly defining a read-only scope (e.g., `calendar.readonly`).
*   **UI Initiation and Redirect:** The administrative settings page (`core/admin/page-import-gcal.php`), managed by the class `WPBC_Page_SettingsImportGCal`, must be refactored. The field for the static API Key is replaced with input fields for the **OAuth Client ID/Secret** and an **Authorization Button**.
*   **Visible Consent:** Clicking the Authorization Button triggers a server-side redirect that includes the required minimal scope in the request URL. This ensures the external service's consent screen displays the exact access being requested (e.g., "View your calendars") [Triage].

### Phase 2: Secure Scope Persistence and Review

The system must securely store the credentials and the granted scope, and allow the administrator to review the status.

*   **Token & Scope Storage:** Upon successful authorization, the callback handler must exchange the authorization code for the Access Token and Refresh Token. These tokens, along with the **granted scope** metadata, must be saved using the plugin's data abstraction wrappers (`update_bk_option`). Critically, this storage process must integrate an **encryption layer** to ensure the tokens and client secret are never saved in plain text in the database [Triage].
*   **Administrative Review:** The synchronization settings page (`page-import-gcal.php`) must utilize its UI helper functions (`wpbc_gcal_settings_content_field_*`) to prominently display the currently active access permissions, confirming to the administrator the exact **scopes granted** (e.g., "Read-only access confirmed").

### Phase 3: Implementing Scope Revocation Control

The ability to manually disconnect and revoke the granted scope ensures the administrator maintains control over the third-party integration.

*   **Revocation UI:** A clear **"Revoke Access"** control (button) must be added to the settings screen in `page-import-gcal.php`.
*   **Secure AJAX Handler:** The revocation control triggers an administrative AJAX request routed through `core/lib/wpbc-ajax.php`. This handler must be secured using **Nonce verification** (via `wpbc_check_nonce_in_admin_panel()` or equivalent custom wrapper) to prevent CSRF attacks.
*   **Two-Step Revocation:** The server-side revocation logic must execute two steps to fulfill the request:
    1.  **External Revocation:** Call the external service's API endpoint (e.g., Google's) to formally revoke the granted scope, invalidating the Refresh Token and Access Token at the source.
    2.  **Local Cleanup:** Upon confirmation of external revocation, use the secure `delete_bk_option()` wrappers to permanently remove the encrypted tokens, client secret, and scope metadata from the local database.
 
*   
---

### 4. **API Communication Security**

* [ ] Are all API calls made over **HTTPS**?
* [ ] Are OAuth credentials (client secret, tokens) **never exposed in front-end code or logs**?
* [ ] Does the plugin validate **API responses** (avoid injection/spoofing)?


Based on the sources, the implementation status of **API Communication Security** for the documented synchronization feature (Google Calendar Import) is mixed. The plugin utilizes secure transport and performs robust validation, but its architecture (reliance on an API key) makes part of the OAuth credential security requirement inapplicable.

Here is the implementation status for each criterion, focusing on the communication flow managed by the `WPBC_Google_Calendar` class.

### 1. Are all API calls made over **HTTPS**?

**Yes (Implemented, or Strongly Implied).**

The core synchronization engine, the `WPBC_Google_Calendar` class, communicates with the Google Calendar API v3 using the WordPress function **`wp_remote_get()`**. This function is described as the **"standard, secure, and reliable way in WordPress to handle external HTTP requests"**. The secure and reliable nature of external API communication implies the use of HTTPS (TLS) for encrypted transport.

### 2. Are OAuth credentials (client secret, tokens) **never exposed in front-end code or logs**?

**Partially Implemented (Credential is not client-side, but lacks secure logging).**

*   **Front-end Exposure:** The current synchronization architecture relies on a persistent **Google API Key** (`booking_gcal_api_key`), not OAuth tokens. This key is retrieved from plugin options and used exclusively in the server-side `run()` method of the `WPBC_Google_Calendar` class to construct the request via `wp_remote_get()`. There is no indication in the sources that this key is passed to client-side scripts (JS/HTML).
*   **Logging Security:** While there is a developer-centric debugging utility, **`debuge_error()`**, and functions for providing "actionable feedback to the administrator" upon failure, the sources **do not confirm** that sensitive credentials (like the API Key) are securely redacted or prevented from leaking into these logs or error messages [Triage]. Secure logging is required to meet the "never exposed in logs" criterion [Triage].

### 3. Does the plugin validate **API responses** (avoid injection/spoofing)?

**Yes (Implemented).**

The plugin incorporates multiple validation and sanitation steps, ensuring the data integrity of the synchronization process:

*   **Robust Error Checking:** The `WPBC_Google_Calendar` class performs **"robust error checking on the response"** immediately after the `wp_remote_get()` call. This includes checking for standard WordPress error objects (`WP_Error`) and non-200 HTTP status codes (such as 403 Forbidden or 404 Not Found). If an error is detected, processing halts.
*   **Data Transformation and Mapping:** The synchronization engine processes the fetched JSON data, applying the **"admin-configured field mapping"** (using the `booking_gcal_events_form_fields` option) to extract and map event details (title, start/end time, description, location) to the local booking form fields. This structured mapping acts as a data integrity layer, preventing raw or unexpected data structures from being injected into the booking form fields.
*   **Deduplication as Validation:** Before saving data, the `getExistBookings_gid()` method performs a direct `$wpdb` query to check for duplicate Google Event IDs (`sync_gid`) in the local database, ensuring only unique, valid events are created via `wpbc_booking_save()`.

***

## Score Assessment (1–10)

The implementation of **API Communication Security** scores a **7 / 10**.

This score reflects strong implementation of secure transport (HTTPS/TLS via `wp_remote_get()`) and excellent validation of response integrity, but the lack of documented security measures against logging the API key prevents a perfect score.



---

### 5. **Error & Recovery Handling**

* [ ] If OAuth login fails (expired token, revoked access), does the plugin **prompt admin to reconnect**?
* [ ] Are OAuth errors **logged securely** (without exposing sensitive info)?
* [ ] Does the system **fall back gracefully** (bookings remain intact even if sync fails)?


Based on the analysis of the plugin's architecture, particularly its reliance on a **Google API Key** for synchronization rather than the OAuth 2.0 flow, the options related to OAuth failure recovery are **not implemented** in the manner requested.

However, the third criterion regarding graceful fallback is fully implemented for API failures.

### Error & Recovery Handling Implementation Status

#### If OAuth login fails (expired token, revoked access), does the plugin prompt admin to reconnect?

**Not Implemented / Not Applicable.**

*   The plugin utilizes a persistent **Google API Key** (`booking_gcal_api_key`) for its synchronization feature, bypassing the use of expiring OAuth tokens and the login/consent mechanism [144, 163, Triage in previous responses].
*   Therefore, the system **cannot detect an OAuth failure** or prompt the administrator to "reconnect" via an OAuth flow, as that flow is not established [Triage in previous responses].
*   While the system does provide **"actionable feedback to the administrator"** immediately upon detecting an API failure (like a 403 Forbidden status), the sources **do not confirm** that the plugin's custom cron system includes logic to **track repeated API Key failures** and escalate them to a persistent administrative notice that prompts corrective action [Triage in previous responses].

#### Are OAuth errors logged securely (without exposing sensitive info)?

**Cannot be confirmed as implemented.**

*   Since OAuth is not used, the logging of OAuth errors is inapplicable. However, the requirement for secure logging of *any* sensitive credential (the Google API Key) upon API failure cannot be confirmed.
*   The plugin includes **debugging utility functions** like `debuge_error()` and provides detailed failure feedback.
*   The sources **do not contain any documentation** detailing whether these administrative error reports or logs redact, hash, or otherwise secure the persistent **Google API Key** when reporting an API communication failure [Triage in previous responses].

#### Does the system fall back gracefully (bookings remain intact even if sync fails)?

**Yes, this is implemented.**

The synchronization engine is designed with robust error handling and data integrity checks to ensure that local booking data is not corrupted or overwritten if the external API call fails.

*   **Robust Error Checking:** The core synchronization class, `WPBC_Google_Calendar`, uses **`wp_remote_get()`** for API communication and immediately performs **"robust error checking on the response"** for `WP_Error` objects and non-200 HTTP status codes.
*   **Halt Execution:** If an error is detected early (e.g., 403 Forbidden or 404 Not Found), the class halts its execution.
*   **Data Integrity:** The final database insertion is handled by calling the core **`wpbc_booking_save()`** function. This function is only executed after successful data parsing, conversion, and a critical **deduplication check** (`getExistBookings_gid()`) has confirmed the event is new and unique. By halting execution before this saving process begins, the system ensures data remains intact and corruption is avoided.

The overall implementation of **Error & Recovery Handling** for the Google Calendar synchronization feature receives a score of: **4 / 10**.

### Rationale Breakdown

The score reflects a system that is robust against data corruption during a failure but lacks the secure and automated recovery and escalation mechanisms required for long-running integrations, primarily due to its reliance on a static API Key instead of an OAuth 2.0 flow.

| Criterion | Implementation Status | Rationale |
| :--- | :--- | :--- |
| **If OAuth login fails, does the plugin prompt admin to reconnect?** | **Not Implemented / Not Applicable.** | The plugin uses a persistent **Google API Key**, not expiring OAuth tokens. While the system detects single API failures (e.g., 403 Forbidden), there is no documented architectural component to **track repeated failures** by the custom cron system (`WPBC_Cron`) and escalate them into a persistent, actionable administrative warning (like those managed by the `WPBC_Notices` class) [Triage in previous responses]. |
| **Are OAuth errors logged securely (without exposing sensitive info)?** | **Cannot be Confirmed.** | This is not applicable as OAuth is not used. However, the requirement applies to the persistent **Google API Key** (`booking_gcal_api_key`). The system provides **"actionable feedback to the administrator"** upon errors, but the sources **do not document** that this feedback (or any internal logs generated by utilities like `debuge_error()`) securely redact or hide the sensitive API key upon failure [Triage in previous responses]. |
| **Does the system fall back gracefully (bookings remain intact even if sync fails)?** | **Fully Implemented (Yes).** | The plugin features **robust error checking** within the `WPBC_Google_Calendar::run()` method, checking for `WP_Error` objects and non-200 HTTP status codes immediately after the API call. If a failure is detected, execution is halted, preventing the core database saving function (`wpbc_booking_save()`) from being called, ensuring **data integrity**. Furthermore, data saving is protected by a **deduplication check** (`getExistBookings_gid()`) that prevents partial or duplicate imports. |



This implementation plan details how to establish secure and robust **Error & Recovery Handling** by tracking consecutive synchronization failures, securely logging errors, and prompting administrators for corrective action. This plan assumes the prior implementation of an **OAuth 2.0 Token Management** system (including encrypted storage of Refresh Tokens) to handle the concept of token expiry.

The implementation focuses on modifying the existing synchronization controller, `core/sync/wpbc-gcal.php`, and leveraging the administrative notice framework (`WPBC_Notices`).

### 1. Implementing Secure Error Logging (Redaction)

The first step ensures that persistent credentials (the Google API Key or OAuth Client Secret/Tokens) are never leaked into actionable error feedback or system logs.

*   **Credential Redaction:** The error reporting mechanism must be updated to retrieve and redact sensitive settings.
    *   Before displaying or processing any API error message (e.g., a 403 Forbidden status) detected by the **`WPBC_Google_Calendar`** class, the code must retrieve the sensitive credential (e.g., `booking_gcal_api_key` or encrypted tokens) using the data abstraction wrapper (`get_bk_option`).
    *   The raw credential string is then replaced with a secure placeholder (e.g., `[REDACTED]`) within the error message before it is sent to the administrator.
*   **Safe Display:** This sanitized error message is then used by the plugin's debugging utilities (like `debuge_error()`) or displayed as **"actionable feedback to the administrator"**.

### 2. Failure Tracking and Automated Escalation

This step introduces persistence for tracking failures during background synchronization runs, which are executed by the custom pseudo-cron system (`WPBC_Cron`).

*   **Failure Counter Persistence:** A new configuration option (e.g., `gcal_consecutive_fail_count`) must be introduced and stored using the secure settings wrappers (`update_bk_option`).
*   **Controller Logic Modification:** The main background execution function, **`wpbc_silent_import_all_events()`**, is the central point of control:
    *   **On Success:** If the `WPBC_Google_Calendar::run()` method completes successfully, the `gcal_consecutive_fail_count` is **reset to zero**.
    *   **On Failure:** If `run()` fails to connect, authenticate, or retrieve data (e.g., API key is invalid or OAuth refresh failed), the function records the error and **increments the `gcal_consecutive_fail_count`**.
*   **Persistent Notice Trigger:** If the `gcal_consecutive_fail_count` exceeds a predefined threshold (e.g., 3 consecutive failures), the controller triggers a new persistent warning using the dedicated **`WPBC_Notices` class**.
    *   The notice must be hooked into custom plugin actions (e.g., `wpbc_hook_*_page_header`) to ensure it appears consistently at the top of plugin administration pages.

### 3. Implementing Reconnection Prompt (Recovery)

The persistent notice must guide the administrator to the necessary recovery steps, addressing the scenario where token rotation has failed or access was manually revoked.

*   **Prompt Content:** The persistent notice text should inform the administrator that scheduled synchronization has failed repeatedly (e.g., due to an expired OAuth token or API key invalidation).
*   **Actionable Link:** The notice must provide a direct link to the synchronization settings page (managed by **`WPBC_Page_SettingsImportGCal`**), instructing the administrator to **"Reconnect"** or **"Revoke and Reauthorize"** the external account.
*   **Graceful Fallback Confirmation:** The system implicitly adheres to the "fall back gracefully" requirement because the synchronization engine employs **robust error checking** and halts execution before calling `wpbc_booking_save()` if communication fails, thus preventing data corruption.
*   **Notice Dismissal:** The administrative notice should be designed to be automatically removed from the system only when a subsequent execution of **`wpbc_silent_import_all_events()`** succeeds and resets the failure count, confirming that the recovery action (reconnecting) was successful. The notice should also leverage the `WPBC_Notices` system's ability for a user to permanently dismiss warnings.


---

### 6. **Admin Experience & Control**

* [ ] Is there a **secure settings page** in WP admin to connect/disconnect Google/Outlook accounts?
* [ ] Can admins **see which accounts are connected** and when last sync happened?
* [ ] Can admins **revoke access** from within WP without editing DB?


Based on the provided sources and our conversation history, the implementation status of the requested **Admin Experience & Control** options is assessed below, focusing on the documented **Google Calendar Import** feature.

### Admin Experience & Control

#### 1. Is there a secure settings page in WP admin to connect/disconnect Google/Outlook accounts?

**Partially Implemented (Settings page exists, but connection method is not secure OAuth 2.0).**

*   **Settings Page:** A dedicated settings page exists in the WordPress admin for managing synchronization. The file **`core/sync/wpbc-gcal.php`** is the controller that handles the configuration for the Google Calendar import feature, accessible under Booking > Settings > Sync. This page utilizes UI helper functions like `wpbc_gcal_settings_content_field_*()` to render input fields.
*   **Connection Method:** The connection method documented in the sources is **not secure OAuth 2.0**. The administrator is required to configure the **Google API Key** manually. This static API Key system is not equivalent to the modern security practice of redirecting users for OAuth login/consent, which would prevent the raw copy-paste of secrets [Triage in previous responses].

#### 2. Can admins see which accounts are connected and when last sync happened?

**Partially Implemented (Sync status is available, but account connection status is implied, not explicitly detailed for OAuth).**

*   **Connection:** The source indicates that the settings page manages the **Google API Key** and its activation status.
*   **Last Sync/Status:** The plugin implements a custom **pseudo-cron system** via the `WPBC_Cron` class. This system is used for scheduling and executing the import via the `wpbc_silent_import_all_events()` function. The sources indicate that the cron file, `core/lib/wpbc-cron.php`, contains features to **display the status of scheduled tasks** on admin pages, which implies the admin can view when the synchronization last occurred. Additionally, after a manual import, the `WPBC_Google_Calendar` class can display an HTML table summarizing the newly created bookings via its `showImportedEvents()` method, providing immediate feedback on a sync operation.

#### 3. Can admins revoke access from within WP without editing DB?

**Yes (Revoking the persistent API Key is possible via the UI).**

*   **Disconnection via UI:** Administrators can "disconnect" the integration by removing the **Google API Key** from the input field on the synchronization settings page and saving the changes [Triage in previous responses].
*   **Database Cleanup:** The controller file, `core/sync/wpbc-gcal.php`, defines a dedicated lifecycle function, **`wpbc_sync_gcal_deactivate()`**, which is responsible for system cleanup. This function uses **`delete_bk_option()`** to **remove all Google Calendar-related settings** from the database upon plugin deactivation. While this is primarily for deactivation cleanup, it confirms the presence of functions that delete the stored credentials without requiring manual DB editing. The UI typically provides a way for admins to clear the API key field and save, which achieves the same result for the active integration.

***

### Score Assessment (1–10)

Based on the existing API Key architecture, the implementation of **Admin Experience & Control** receives a score of **6 / 10**.

The score is driven up by the fact that the required settings UI exists, status is trackable via cron tools, and manual disconnection is possible. However, the connection method fails the "secure setup screen" requirement by not using OAuth 2.0, which drags the score down significantly.

| Criterion | Implementation Status | Score Impact |
| :--- | :--- | :--- |
| **Secure settings page (connect/disconnect)** | **Partially Implemented (UI exists, but connection is API Key, not OAuth)** | **Medium.** The required page (`page-import-gcal.php`) and lifecycle functions exist. However, the setup is **not secure** as it requires the manual entry of the persistent **Google API Key**. |
| **Admins can see accounts/last sync** | **Partially Implemented (Status trackable)** | **High.** The controller manages scheduling via cron, and the cron utility files contain tools to display the status of scheduled tasks. Manual sync success is reported via `showImportedEvents()`. |
| **Can admins revoke access without editing DB?** | **Implemented (Revoke key via UI).** | **High.** Admins can manually clear the API Key field in the UI, and the plugin has built-in functions (`wpbc_sync_gcal_deactivate` calling `delete_bk_option`) to remove settings from the database without manual SQL editing. |



---

### ✅ Success Criteria

* OAuth 2.0 flow is implemented using **secure redirects + minimal scopes**.
* Access & refresh tokens are **stored securely and refreshed automatically**.
* Admins can **connect, disconnect, and monitor** external calendar accounts.
* Plugin handles **errors and token expiry** without breaking booking system.
* No OAuth secrets or tokens are **leaked in logs, code, or front-end**.

---

👉 Do you want me to also prepare a **step-by-step testing checklist** (like “Connect Google Calendar → inspect DB for token storage → check scopes requested → revoke token and see behavior”) so you can hands-on validate if a plugin truly follows this?
