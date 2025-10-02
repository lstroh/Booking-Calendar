Got it — let’s break down **“Security → Integration Security → Secure APIs for calendar and CRM integrations”** into clear, checkable items so you can verify whether a WordPress plugin is handling API security correctly.

---

## 🔍 Breakdown Into Checkable Items

### 1. **Authentication & Authorization**

* [ ] Does the plugin use **OAuth 2.0** (not basic auth or API keys in plain text) for Google/Outlook/CRM connections?
* [ ] Are **access tokens** stored securely (e.g., in the WordPress options table with encryption, not in plain DB fields)?
* [ ] Are **refresh tokens** handled properly to avoid repeated re-logins?


Based on the analysis of the provided architectural sources, here is a response regarding the implementation of the requested authentication and authorization features in the plugin:

The sources provide specific details only for the **Google Calendar synchronization** feature, which relies on **API keys** rather than the OAuth 2.0 framework for external communication. There is no information provided in the sources regarding dedicated API connections or authentication for Outlook or CRM systems.

### Authentication & Authorization Implementation

#### Does the plugin use OAuth 2.0 (not basic auth or API keys in plain text) for Google/Outlook/CRM connections?

*   The plugin's architecture for **Google Calendar Import** is explicitly described as using a **Google API Key**.
*   This API key is configured by the administrator on the settings page (`page-import-gcal.php`). The synchronization engine retrieves the API key from the plugin options (`booking_gcal_api_key`).
*   The sources **do not mention the use of OAuth 2.0** for any external connection, including Google Calendar, Outlook, or CRM [Triage].

#### Are access tokens stored securely (e.g., in the WordPress options table with encryption, not in plain DB fields)?

*   Since the documented method relies on a persistent **API Key** rather than an OAuth flow, the concept of a short-lived access token is not applicable based on the existing documentation.
*   The general settings of the plugin, including options like the Google API Key (`booking_gcal_api_key`), are stored as a **separate row in the standard WordPress `wp_options` table** (referred to as the 'separate' strategy).
*   The sources **do not mention any mechanism for encrypting** the API key or any other setting stored in the `wp_options` table [Triage].

#### Are refresh tokens handled properly to avoid repeated re-logins?

*   The sources **do not mention refresh tokens** [Triage].
*   Refresh tokens are a component of the OAuth 2.0 standard, which is designed to prevent re-logging when an access token expires. Given that the documented synchronization uses a persistent **API Key**, an access token/refresh token workflow is not used for this feature.
*   The recurring task for importing Google Calendar data is managed by the plugin's custom **pseudo-cron system** via the `wpbc_silent_import_all_events()` function, which executes the job in the background using the configured API key. Repeated manual "re-logins" are avoided through this automated scheduling process.

Based on the detailed architectural analysis of the plugin's synchronization features, the implementation of the requested Authentication & Authorization criteria scores extremely low.

The overall score is: **1 / 10**

### Rationale

The requested score is **1** because the core security architecture demanded in the checklist (OAuth 2.0, access tokens, refresh tokens) is not implemented for the documented external connection (Google Calendar) [Triage]. Instead, the system relies on less secure alternatives, and there is no information about Outlook or CRM connections.

Here is the breakdown against the checklist:

| Feature Requested | Implementation Found in Sources | Status (Score Impact) |
| :--- | :--- | :--- |
| **Use OAuth 2.0** | The **Google Calendar Import** feature uses a persistent **Google API Key** configured by the administrator (`booking_gcal_api_key`). There is **no mention of OAuth 2.0** for Google, Outlook, or CRM integrations [Triage]. | **Not Implemented.** |
| **Access tokens stored securely (with encryption)** | Since OAuth 2.0 is not used, short-lived access tokens are not relevant. The persistent API key is stored as a **separate row in the WordPress `wp_options` table**. The sources **do not mention any mechanism for encrypting** this key or any other setting [Triage]. | **Not Implemented (Lacks Encryption).** |
| **Refresh tokens handled properly** | Refresh tokens are only used in an OAuth flow. This system uses a persistent API Key. Automated synchronization is managed by a custom **pseudo-cron system** (`WPBC_Cron`) via the `wpbc_silent_import_all_events()` function, which runs automatically and thus avoids manual re-logins, but does **not implement the required token handling** [Triage]. | **Not Implemented (Irrelevant due to API key).** |

The core functionality (Google Calendar synchronization) is operational but relies on a persistent **API Key** for communication, which is explicitly noted as *not* being the requested authentication mechanism (OAuth 2.0). Furthermore, the lack of documented encryption for this persistent key stored in the database fails the security requirement for token storage [2, Triage].


The implementation of OAuth 2.0, secure token storage, and proper refresh token handling requires a fundamental shift from the current persistent API Key model by leveraging the plugin's existing object-oriented architecture and extending its data management and synchronization components.

Here is a high-level overview of how these features would be implemented using the documented plugin architecture:

### 1. Refactoring Configuration & UI (Switching to OAuth Flow)

The administrator settings page (`core/admin/page-import-gcal.php`) must be updated to initiate the OAuth flow instead of collecting a static API key.

*   **Settings Fields:** The plugin uses its custom framework based on the **WPBC_Settings_API** to define configuration options. New settings fields would be introduced for the **OAuth Client ID** and **Client Secret**, which are necessary to initiate the OAuth process. These credentials must themselves be treated as sensitive data and managed by the secure storage system (see Step 2).
*   **Authorization Interface:** The UI would replace the API Key input with a button or link directing the administrator through the external service's (e.g., Google's) OAuth process. The plugin would receive the necessary tokens via a registered callback URL.

### 2. Implementing Secure Token Storage and Encryption

Since the existing system stores settings (including the static API Key) as unencrypted, separate rows in the standard `wp_options` table, a critical security layer must be introduced to securely store the dynamic **Access Tokens** and **Refresh Tokens**.

*   **Encryption Layer:** An encryption utility must be implemented to encrypt all sensitive credentials (Client Secret, Access Token, and Refresh Token) before they are written to the database.
*   **Data Abstraction:** This encryption logic would be integrated into the existing data abstraction layer defined in **`core/wpbc-core.php`**. Specifically, the wrapper functions **`update_bk_option`** and **`get_bk_option`** would be modified to automatically encrypt data before saving and decrypt data immediately upon retrieval.
*   **Token Persistence:** The encrypted **Access Token** (short-lived) and **Refresh Token** (long-lived) would be stored in the database options, replacing the currently stored API Key (`booking_gcal_api_key`) [144, Triage].

### 3. Handling Access and Refresh Tokens (The Core Synchronization Engine)

The core logic for API interaction, currently defined in the **`WPBC_Google_Calendar`** class, must be refactored to manage the token lifecycle.

*   **Token Retrieval:** The `WPBC_Google_Calendar::run()` method, which orchestrates the synchronization, would retrieve the Access Token from the database via the new, secure `get_bk_option` wrapper.
*   **Access Token Usage:** The token would be included in the header of the API request sent via **`wp_remote_get()`**.
*   **Refresh Token Logic:** If the API response results in an authentication error (e.g., HTTP 401 Unauthorized), the `WPBC_Google_Calendar` class would automatically initiate the refresh process:
    1.  It retrieves the encrypted Refresh Token from the database.
    2.  It uses the Refresh Token and the Client Secret/ID to request a new Access Token from the external service.
    3.  It securely saves the newly acquired Access Token (and potentially a new Refresh Token) back to the database using the encrypted `update_bk_option` function (Step 2).
    4.  It automatically retries the original API synchronization process using the new, valid Access Token.

### 4. Integration with Background Automation

The automation system must handle the token refresh without requiring administrative intervention, a core requirement to avoid repeated re-logins [Triage].

*   **Cron Execution:** The background synchronization is triggered by the **`wpbc_silent_import_all_events()`** function, which is scheduled using the plugin's custom **WPBC_Cron** system.
*   **Silent Refresh:** Since `wpbc_silent_import_all_events()` instantiates and runs the `WPBC_Google_Calendar` object, the automatic Refresh Token logic implemented in Step 3 ensures that if the Access Token expires during a scheduled cron run, the token is silently refreshed in the background, and the import proceeds successfully. This architecture removes the traffic dependency associated with repeated re-logins, as the token management is automated within the scheduled task.

This structure allows the plugin to fulfill the OAuth 2.0 requirements by integrating the new security flow directly into its existing foundational components (Settings API, Data Abstraction Layer, Synchronization Engine, and Cron Controller).



---

### 2. **API Communication Security**

* [ ] Are all API requests made over **HTTPS (TLS 1.2 or higher)**?
* [ ] Are requests **signed or validated** (e.g., with HMAC, JWT, or OAuth signatures)?
* [ ] Are responses from APIs **validated** to prevent injection or spoofing?

The sources focus on the Google Calendar synchronization feature as the primary example of external API communication. Based on the documented architecture of the `WPBC_Google_Calendar` class and its controller (`core/sync/wpbc-gcal.php`), here is the assessment of the requested API Communication Security options:

### API Communication Security

#### Are all API requests made over **HTTPS (TLS 1.2 or higher)**?

The implementation uses the standard WordPress function `wp_remote_get()` to communicate with the **Google Calendar API v3**. This is described as the standard, **secure, and reliable way** in WordPress to handle external HTTP requests.

While the use of `wp_remote_get()` for a major service like Google strongly implies the request is made over HTTPS, the sources **do not explicitly confirm** that HTTPS is enforced or specify the minimum TLS version (1.2 or higher) [Triage, 163]. The communication layer handles the request construction, API Key inclusion, and time window parameters.

#### Are requests **signed or validated** (e.g., with HMAC, JWT, or OAuth signatures)?

**No.** The external requests are authorized using a persistent **Google API Key**, which the administrator configures on the settings page (`booking_gcal_api_key`).

*   The plugin's architecture for Google Calendar integration relies on supplying this API key directly in the request.
*   The sources contain no information indicating the use of advanced signing mechanisms such as **HMAC, JWT, or OAuth signatures** for external API communication [Triage]. (It is worth noting that for *internal* administration, the plugin's core AJAX router uses **Nonce verification** to secure sensitive functions against CSRF, but this applies to internal actions, not external API requests).

#### Are responses from APIs **validated** to prevent injection or spoofing?

**Yes, partial validation and robust error handling are implemented.** The `WPBC_Google_Calendar` class performs checks and data sanitization on the response before processing it into a booking:

1.  **Error and Status Code Checking:** The class performs **robust error checking** immediately after receiving the response (via `wp_remote_get()`). This includes checking for standard WordPress `WP_Error` objects and ensuring the HTTP status code is **200 (OK)**. Non-200 status codes (e.g., 404 Not Found or 403 Forbidden) are caught to provide actionable feedback to the administrator.
2.  **Data Parsing and Transformation:** The successful response (which is JSON data) is parsed using `json_decode()`. The engine extracts specific details (title, description, location) and converts the date/time format using internal methods (`getCommaSeparatedDates()`).
3.  **Field Mapping:** The extracted data is then mapped to the local booking form fields using an **admin-configured field mapping** (stored in the `booking_gcal_events_form_fields` option). This structured transformation mitigates injection risks by expecting and processing data according to predefined formats and fields.
4.  **Deduplication Check:** Before a booking is created, the system executes a direct `$wpdb` query using `getExistBookings_gid()` to check the local `wp_booking` table for existing bookings that share the unique **Google Event ID (`sync_gid`)**, effectively preventing the injection of duplicate records.
5.  **Final Saving:** The transformed data is then sent to the core `wpbc_booking_save()` function for final database insertion.

Based on the detailed architectural analysis of the Google Calendar synchronization feature, the overall security implementation of API communication receives a score of **4 / 10**.

The low score reflects the fundamental lack of modern request signing protocols (like HMAC or OAuth signatures) and unconfirmed TLS version requirements, despite the strength of the response validation mechanism.

### Rationale Breakdown

| Security Criterion | Implementation Status | Score Impact |
| :--- | :--- | :--- |
| **HTTPS (TLS 1.2 or higher)** | **Implied, but Unconfirmed.** The system uses `wp_remote_get()` for communication with the Google Calendar API v3. This is called the standard, secure, and reliable method in WordPress. While communication with Google APIs almost certainly uses HTTPS, the sources do not explicitly confirm HTTPS enforcement or specify the minimum required TLS version (1.2 or higher) [Triage]. | **Partial.** |
| **Requests signed or validated (HMAC, JWT, or OAuth signatures)** | **Not Implemented.** The requests are authenticated by supplying a persistent **Google API Key** (`booking_gcal_api_key`) directly in the request. The sources provide no information indicating the use of advanced signing mechanisms such as HMAC, JWT, or OAuth signatures to validate the request integrity or source [Triage]. | **Critical Failure.** |
| **Responses from APIs validated to prevent injection or spoofing** | **Fully Implemented.** The plugin employs robust checks and structured processing to ensure data integrity:
    *   **Error Checking:** The `WPBC_Google_Calendar` class performs robust checks for `WP_Error` objects and explicitly verifies that the HTTP status code is **200 (OK)**. Non-200 codes (like 403 Forbidden or 404 Not Found) are caught for admin feedback.
    *   **Data Structure:** Successful JSON responses are parsed (`json_decode()`), and specific event details are extracted and mapped to local booking fields using an **admin-configured field mapping** (`booking_gcal_events_form_fields`).
    *   **Deduplication/Injection Prevention:** Before saving, the system executes a direct `$wpdb` query (via `getExistBookings_gid()`) to check for the unique **Google Event ID (`sync_gid`)** in the local database, which prevents the creation of duplicate or spoofed records. | **Strong Implementation.** |

The score of 4 is assigned because while the system performs excellent data validation (Criterion 3 is fulfilled), the lack of modern request signing (Criterion 2) represents a significant security weakness for external communication, particularly when using a persistent API key model.


The implementation of the requested features—**OAuth 2.0**, **secure token storage**, and **enhanced API communication security**—requires a major architectural shift, moving away from the current persistent Google API Key model and integrating new layers of security directly into the plugin's existing object-oriented framework.

This high-level implementation overview focuses on refactoring the current Google Calendar synchronization feature (managed by `core/sync/wpbc-gcal.php` and the `WPBC_Google_Calendar` class) and leveraging the plugin's foundational utilities.

---

### Phase 1: Authentication Engine Shift (OAuth 2.0)

The goal is to replace the reliance on the static `booking_gcal_api_key` with a dynamic, token-based authorization flow.

#### 1. UI and Settings Refactoring (`page-import-gcal.php`)

*   The administrative settings page for Google Calendar synchronization (inferred path: `core/admin/page-import-gcal.php`) must be updated.
*   The input field for the Google API Key is replaced with fields for the **OAuth Client ID** and **Client Secret**, and an **Authorization Button**.
*   Clicking the button initiates the external OAuth flow, sending the administrator through the external service's authorization screen. A redirect URI registered with the service handles the callback, receiving the initial **Access Token** and **Refresh Token** [Triage].
*   The controller functions within `wpbc-gcal.php` must be updated to handle these new credential types during activation and saving, utilizing existing UI helper functions like `wpbc_gcal_settings_content_field_*()`.

#### 2. Secure Token Persistence (Encryption Layer)

To satisfy the requirement for securely stored access tokens and refresh tokens [Triage], an encryption layer must be added to the data abstraction system.

*   **Encryption Utility:** A robust encryption/decryption utility (e.g., using PHP encryption functions) must be implemented within a utility file, such as `core/wpbc-core.php`, which houses the data abstraction wrappers.
*   **Wrapper Modification:** The custom functions for saving and retrieving global settings—**`update_bk_option()`** and **`get_bk_option()`**—must be modified. These wrappers would automatically **encrypt sensitive data (Access Token, Refresh Token, Client Secret)** before saving to the WordPress options table, and automatically decrypt them upon retrieval.
*   **Token Storage:** The encrypted tokens and client credentials replace the previously stored API Key in the database options, providing a mechanism for persistent, secure storage.

### Phase 2: Secure Communication and Automation

The goal is to integrate the tokens into the synchronization engine and automate the token lifecycle.

#### 3. Request Signing and Security Enforcement (HTTPS/Token Usage)

The reliance on bearer tokens for API access automatically fulfills the requirement for requests to be signed or validated (using OAuth signatures) [Triage].

*   **HTTPS Enforcement:** Although `wp_remote_get()` is used for secure communication, the implementation must **explicitly verify that API requests use `https://`** to meet the TLS requirement. This verification can be integrated into the request construction method within the `WPBC_Google_Calendar` class.
*   **Token Integration:** The `WPBC_Google_Calendar` class must be refactored to retrieve the decrypted **Access Token** via `get_bk_option()` and include it in the `Authorization: Bearer` header of every API request, replacing the API Key supplied in the request parameters.

#### 4. Automatic Token Management and Background Refresh

This step ensures **refresh tokens are handled properly** to avoid repeated re-logins [Triage].

*   **Engine Update (`WPBC_Google_Calendar`):** The main execution method, **`run()`**, is updated to include logic for handling expired Access Tokens (typically indicated by a 401 Unauthorized status code).
*   **Silent Refresh:** If a token expires, the class will use the stored **Refresh Token** and **Client Secret** to perform a silent background API request to acquire a new Access Token.
*   **Update Storage:** The new tokens must then be securely saved back to the database using the **encrypted** `update_bk_option()` function (Phase 1, Step 2). The synchronization attempt is then retried seamlessly.
*   **Cron Integration:** Since synchronization is managed by the traffic-dependent pseudo-cron system (`WPBC_Cron` executing `wpbc_silent_import_all_events()`), the token refresh logic runs automatically during scheduled background imports, ensuring the system remains connected without administrative intervention.

#### 5. Enhanced Response Validation

The existing response validation system, already noted for its robustness, should be formally integrated into the security architecture.

*   **Error Checking:** Formalize the existing robust checks for `WP_Error` objects and non-200 HTTP status codes (like 403 Forbidden or 404 Not Found) within `WPBC_Google_Calendar::run()` to provide clear diagnostic feedback.
*   **Data Integrity:** Utilize the existing mechanism for **data parsing, transformation, and field mapping** (using the `booking_gcal_events_form_fields` option). This structured transformation mitigates injection risks by strictly mapping extracted JSON data to the local booking form fields.
*   **Deduplication as Validation:** Rely on the established **`getExistBookings_gid()`** method, which uses a direct `$wpdb` query to check for the unique Google Event ID (`sync_gid`) before saving, preventing spoofed or duplicate entries from entering the `wp_booking` table.


---

### 3. **Scope & Least Privilege**

* [ ] Does the plugin request only the **minimum required scopes** from Google, Outlook, Zoho, HubSpot?
  (e.g., `calendar.readonly` instead of full account access).
* [ ] Can admins **review or revoke scopes** granted?

Based on the analysis of the provided sources, the plugin's architecture does not contain sufficient information to confirm the implementation of the requested **Scope & Least Privilege** options.

The only external API synchronization detailed in the sources is for **Google Calendar Import**, which relies on a simple **Google API Key** rather than OAuth 2.0.

### Scope & Least Privilege Implementation

#### Does the plugin request only the minimum required scopes from Google, Outlook, Zoho, HubSpot?

The plugin uses a persistent **Google API Key** configured by the administrator (`booking_gcal_api_key`) to communicate with the **Google Calendar API v3**.

*   When using a basic API Key, the concept of **OAuth 2.0 Scopes** (like `calendar.readonly`) is **not applicable** because the key grants access based on its configuration within the Google Console, not on a dynamic user consent screen [Triage].
*   The function that handles the communication, `WPBC_Google_Calendar::run()`, constructs the request URL and utilizes this API key.
*   The sources **do not mention any API connection or integration** for Outlook, Zoho, or HubSpot [Triage].
*   Therefore, the requirement regarding *requesting minimum required scopes* **is not met** because the fundamental authentication mechanism (API Key) does not employ scopes in the manner requested [Triage].

#### Can admins review or revoke scopes granted?

Since the mechanism uses a persistent **API Key** rather than an OAuth flow that grants revocable scopes, this option is generally **not applicable** in the current documented architecture.

*   **Reviewing:** The administrator can view and manage the API Key itself on the synchronization settings page (`page-import-gcal.php`). However, this key is a single credential, not a dynamically granted scope. The scope is implicitly defined by the API key permissions set within the external Google ecosystem.
*   **Revoking:** The administrator can **delete the API Key** from the plugin's database options. Lifecycle functions exist (like `wpbc_sync_gcal_deactivate()`) to clean up settings, including the API key, upon plugin deactivation. While removing the API Key blocks access, this is not equivalent to revoking a limited OAuth scope granted by an external user.

The sources do not provide any information about a dedicated UI element or process for reviewing or revoking OAuth scopes, as no OAuth flow is documented [Triage].

***

### Score Assessment (1–10)

The implementation of the requested security feature receives a score of **1 / 10**.

**Rationale:** The score is extremely low because the plugin relies on a basic, persistent API Key model, which fundamentally bypasses the sophisticated security framework (OAuth 2.0 scopes, dynamic consent, and granular revocation) required to fulfill the concept of "Least Privilege" as requested [Triage]. While the admin can remove the API key to block access, this does not represent the implementation of scope management.


The implementation of **Scope & Least Privilege** requires building upon the shift to OAuth 2.0 (as previously outlined for Authentication). This involves ensuring the plugin requests only the minimum necessary permissions (scopes) from external services and providing the administrator with a clear mechanism to review and revoke that access.

Here is a high-level overview of how this would be implemented using the documented plugin architecture:

### 1. Define and Enforce Minimum Required Scopes (Least Privilege)

The current Google Calendar synchronization is defined as a **"one-way process"** to **import** events. This mandates a read-only scope for maximum security.

*   **Scope Definition:** The synchronization engine (the `WPBC_Google_Calendar` class) would be updated to strictly define and request the minimal scope necessary for importing events (e.g., `calendar.readonly`).
*   **Authorization Request:** The configuration controller (`core/sync/wpbc-gcal.php`), which renders the setup UI, would ensure that the OAuth flow initiated by the administrator requests **only** this minimal scope, adhering to the principle of least privilege.

### 2. Modify the Authorization Flow

The administrative process must integrate the scope request during the consent phase.

*   **UI Initiation (`page-import-gcal.php`):** The synchronization settings page, which currently accepts an API Key, would be modified to trigger the OAuth redirection. This redirection URL must include the necessary parameters, specifically the defined read-only scope.
*   **Token Retrieval:** Upon successful authentication, the callback handler would receive the **Access Token** and **Refresh Token**. These tokens implicitly confirm that the administrator consented only to the requested minimum scope.

### 3. Implement Secure Scope Persistence

The system must securely store the tokens obtained during the OAuth flow.

*   **Secure Storage (Prerequisite):** The sensitive **Refresh Token** (required for automated background imports) and the scope metadata must be stored using the custom data abstraction layer (wrappers like `update_bk_option`) and secured with an **encryption layer** to prevent plain-text storage [190, 2, Triage].

### 4. Implement Scope Review and Revocation UI

The administrative interface needs a dedicated section to display the current access status and provide the revocation control.

*   **Status Display:** The synchronization settings page (`core/sync/wpbc-gcal.php`) would use its UI helper functions (e.g., `wpbc_gcal_settings_content_field_*()`) to render fields that display:
    *   Confirmation that the required scope (e.g., "Read-only access to Google Calendar") was granted.
    *   The expiration status of the Access Token and the last time the Refresh Token was used.
*   **Revocation Button:** A dedicated administrative control (button) would be added, labeled "Revoke Access," that triggers an AJAX request specifically for deauthorization.

### 5. Backend Revocation and Cleanup Logic

The core task of revocation must be handled securely via an administrative AJAX endpoint.

*   **AJAX Handler Registration:** A new AJAX endpoint (e.g., `wpbc_ajax_revoke_gcal`) would be registered within the plugin's central AJAX router (`core/lib/wpbc-ajax.php`). This endpoint must be protected by **Nonce verification** to prevent CSRF attacks.
*   **Token Revocation:** The handler logic would perform two critical steps:
    1.  **External Revocation:** It would use the stored Refresh Token or Access Token to call the external service's (e.g., Google's) API endpoint responsible for revoking access. This invalidates the tokens at the source.
    2.  **Local Cleanup:** Upon confirmation of external revocation, the function would call the secure `delete_bk_option()` wrappers to permanently remove the stored Refresh Token, Access Token, and any scope-related metadata from the local `wp_options` table.

This process ensures that administrators can fully review and revoke the granted access, satisfying the **Scope & Least Privilege** requirement, and integrating token management directly into the existing UI and AJAX framework.



---

### 4. **Token & Credential Management**

* [ ] Are API keys, tokens, or secrets **never exposed in the client-side code** (JS, HTML)?
* [ ] Are credentials stored in **WordPress secure storage** and not hardcoded?
* [ ] Is there a **token rotation mechanism** (refresh tokens automatically renewed without downtime)?


Based on the architectural analysis of the plugin, particularly concerning the Google Calendar synchronization feature, the implementation status of **Token & Credential Management** is mixed. The system uses a less secure credential type (API Key) and lacks explicit encryption for storage, but handles usage securely on the server-side.

Here is the implementation status for each criterion:

### 1. Are API keys, tokens, or secrets never exposed in the client-side code (JS, HTML)?

**Likely Implemented (No Exposure)**

The plugin uses a **Google API Key** for authentication with the Google Calendar API.

*   This key (`booking_gcal_api_key`) is configured and managed entirely within the administrative backend.
*   The actual communication with the external API is a server-side process, orchestrated by the PHP class `WPBC_Google_Calendar`.
*   The key is retrieved from plugin options and is used by the `run()` method to construct and execute the request via the server-side WordPress function **`wp_remote_get()`**.
*   While the plugin uses a data bridge to pass global variables to JavaScript, the sources do not indicate that this specific sensitive key is included in the client-side data payload, suggesting it remains server-side only for API interaction.

### 2. Are credentials stored in WordPress secure storage and not hardcoded?

**Partially Implemented (Not Hardcoded, but Storage Lacks Encryption)**

The credential is not hardcoded, but the documented storage method does not include encryption, failing the "secure storage" requirement.

*   **Not Hardcoded:** The Google API key is configured by the administrator on the settings page and stored in the database as a setting.
*   **Storage Security:** The plugin utilizes a custom data abstraction layer, and general settings are configured to save each option as a **"separate row in the wp_options table"** (the 'separate' strategy).
*   The sources **do not mention any mechanism for encrypting** the Google API Key or any other sensitive settings stored in the `wp_options` table [2, Triage]. Storing sensitive, persistent credentials like API keys unencrypted in the standard options table is generally not considered "secure storage."

### 3. Is there a token rotation mechanism (refresh tokens automatically renewed without downtime)?

**Not Implemented**

The mechanism requested (refresh tokens) is a feature of the OAuth 2.0 standard, which the plugin does not use for Google Calendar synchronization.

*   The plugin relies on a persistent **Google API Key** rather than short-lived access tokens and refresh tokens [144, Triage].
*   The automated synchronization process itself is handled by the plugin's **custom pseudo-cron system** (`WPBC_Cron`), which executes the `wpbc_silent_import_all_events()` function in the background. This automation avoids the need for manual re-login, but it does so by relying on the non-expiring nature of the API Key, not by implementing a token rotation workflow.

***

## Score Assessment (1–10)

The overall implementation of **Token & Credential Management** scores a **3 / 10**.

This score reflects that while the key is not hardcoded and is likely kept off the client side, the fundamental requirement for using dynamic tokens (token rotation) is bypassed, and the sensitive credential (API Key) is stored without documented encryption, resulting in a significant security lapse regarding persistence.


This implementation plan details the necessary architectural changes to achieve secure **Token & Credential Management** by replacing the current static API Key model with the required OAuth 2.0 flow, integrating encryption, and implementing automated token rotation.

The implementation focuses on modifying the existing data abstraction layer, the synchronization controller, and the core communication engine.

### 1. Implementing Secure Credential Storage (Encryption)

To address the requirement that credentials be stored in secure storage (not plain text in the options table), an encryption layer must be integrated directly into the plugin's data abstraction architecture.

*   **Encryption Utility:** A robust encryption/decryption function must be introduced within the plugin's utilities (e.g., a new file or integrated into `core/wpbc-core.php`).
*   **Data Abstraction Layer Modification:** The wrapper functions used for settings persistence—**`update_bk_option`** and **`get_bk_option`**—must be updated.
    *   **`update_bk_option`**: When storing sensitive credentials (specifically the **OAuth Client Secret** and the long-lived **Refresh Token**), this function will automatically encrypt the data before saving it as a separate row in the WordPress `wp_options` table (which is the current persistence strategy).
    *   **`get_bk_option`**: When retrieving these tokens, this function will automatically decrypt the data before passing it to the synchronization engine.

This mechanism ensures that the persistent security tokens are never stored in plain text, fulfilling the secure storage requirement.

### 2. Implementing Token Rotation Mechanism

Token rotation requires shifting the architecture to use short-lived **Access Tokens** managed by long-lived **Refresh Tokens**. This rotation must be handled automatically during scheduled synchronization runs [Triage].

*   **Engine Refactoring (`WPBC_Google_Calendar`):** The core synchronization class must be refactored to manage the OAuth token lifecycle.
    *   **Token Usage:** The `WPBC_Google_Calendar::run()` method must use the currently valid Access Token (retrieved via the secure `get_bk_option` wrapper) to authenticate API requests.
    *   **Expiration Handling:** If an API request fails due to an expired Access Token (e.g., HTTP 401 Unauthorized, detectable via the existing robust error checking), the system must trigger a silent refresh routine.
*   **Refresh Routine:** The refresh routine must automatically:
    1.  Retrieve the encrypted **Refresh Token** and **Client Secret** from the database.
    2.  Send a request to the external service's authorization endpoint to exchange the Refresh Token for a new Access Token (and potentially a new Refresh Token).
    3.  Securely save the new tokens back to the database using the encrypted **`update_bk_option`** function (Step 1).
    4.  Retry the original synchronization API call using the newly acquired Access Token.
*   **Cron Integration:** The plugin's existing **custom pseudo-cron system** (`WPBC_Cron`), which executes the `wpbc_silent_import_all_events()` function, will trigger this logic automatically during background imports. Since the refresh mechanism is built into the `WPBC_Google_Calendar` engine, it performs the token renewal without requiring administrative intervention or causing downtime for the scheduled task.

### 3. Preventing Client-Side Credential Exposure

The plugin must ensure that credentials remain strictly server-side, upholding the existing architectural practice of server-side API calls.

*   **Server-Side Execution:** Authentication (using `wp_remote_get()`) and all token management logic must remain entirely within server-side PHP classes (`WPBC_Google_Calendar`).
*   **Data Bridge Verification:** The file **`core/wpbc-js-vars.php`** is the data bridge responsible for passing configuration settings and translated strings to the global JavaScript object `_wpbc`. Developers must verify that no token, API key, or client secret is ever included in the data localized via this mechanism or any related action hooks (like `wpbc_js_vars` filter).
*   **AJAX Security:** While AJAX requests handle dynamic booking actions, only temporary Nonces are used to secure communication, not long-lived tokens. This separation must be strictly maintained; the Access Token is never passed to the client-side for use in front-end API calls.



---

### 5. **Error & Failure Handling**

* [ ] Does the plugin handle **API request failures gracefully** (e.g., no data corruption in bookings if sync fails)?
* [ ] Are errors logged securely (without leaking sensitive tokens/keys in logs)?
* [ ] Are admins notified if integrations **fail repeatedly** (e.g., expired token)?


Based on the analysis of the plugin's architecture, particularly concerning the Google Calendar synchronization feature and administrative handling mechanisms, here is the status of the requested options:

### Error & Failure Handling

#### Does the plugin handle API request failures gracefully (e.g., no data corruption in bookings if sync fails)?

**Yes, this is implemented.**

The synchronization engine, defined by the `WPBC_Google_Calendar` class, implements **robust error checking** before proceeding with data processing or database interaction.

*   **Early Failure Check:** The `run()` method uses the standard WordPress function `wp_remote_get()` to communicate with the Google Calendar API v3 and immediately checks the response for standard WordPress error objects (`WP_Error`) and non-200 HTTP status codes.
*   **Halted Processing:** If an error occurs (e.g., 403 Forbidden or 404 Not Found), the class halts execution, preventing further processing.
*   **Data Integrity:** The final database insertion is delegated to the core `wpbc_booking_save()` function and is only called for successfully parsed events that have also passed a **deduplication check** (`getExistBookings_gid()`). By stopping early, the system ensures that **no partial or corrupted data** reaches the local `wp_booking` tables.

#### Are errors logged securely (without leaking sensitive tokens/keys in logs)?

**Cannot be confirmed based on the sources.**

The plugin architecture contains robust error-reporting tools and mechanisms for providing feedback to the administrator, but the sources do not confirm the security details of how API failures are logged:

*   **Error Reporting:** The system does provide **"actionable feedback to the administrator"** upon failure (e.g., reporting a 403 Forbidden error due to an API key restriction). Furthermore, debugging utilities exist, such as the `debuge_error()` function, which generates formatted HTML error messages.
*   **Security Unknown:** The sources **do not explicitly detail** whether the synchronization engine logs the failure to a server-side file, what information (such as the Google API Key, which is stored in the database option `booking_gcal_api_key`) is included in the logged error, or how any error logs are secured [Triage].

#### Are admins notified if integrations fail repeatedly (e.g., expired token)?

**Cannot be confirmed as implemented.**

While the plugin has mechanisms for both error handling and persistent administrative notices, there is **no explicit architectural component documented to track and escalate repeated synchronization failures** to a persistent warning state:

*   **Single Failure Report:** A single cron failure will be caught by the robust error handling, and actionable feedback will be provided.
*   **Background Cron System:** The synchronization is executed in the background by the traffic-dependent **custom pseudo-cron system** (`WPBC_Cron` executing `wpbc_silent_import_all_events()`). There is no documented logic within this cron controller to monitor, count, or log consecutive failures of the synchronization task.
*   **Persistent Notices:** The plugin features a dedicated `WPBC_Notices` class for displaying **persistent, dismissible admin warnings** on plugin pages (e.g., warning about downgrading from paid to free version). However, there is **no evidence** that API synchronization failures integrate with this specific persistent notification system [Triage].


Based on the analysis of the plugin's architecture for the Google Calendar synchronization feature, the implementation of the requested **Error & Failure Handling** features scores moderately low.

The overall score is: **4 / 10**

### Rationale Breakdown

The score reflects a strong implementation of data integrity checks (Criterion 1) but a failure to address the crucial security and escalation requirements for background synchronization jobs (Criteria 2 and 3).

| Criterion | Implementation Status | Score Impact |
| :--- | :--- | :--- |
| **Handle API request failures gracefully (no data corruption)** | **Strongly Implemented.** The `WPBC_Google_Calendar` class performs robust error checking for standard WordPress `WP_Error` objects and non-200 HTTP status codes immediately after the API call. If the sync fails, processing is halted, and the booking creation function (`wpbc_booking_save()`) is never called, preventing data corruption. | **High.** |
| **Errors logged securely (without leaking sensitive tokens/keys)** | **Cannot be Confirmed.** The system provides "actionable feedback to the administrator" upon errors (e.g., reporting a 403 Forbidden error). However, the sources **do not contain any documentation** detailing whether this feedback (or any internal server logs) redacts or secures the persistent Google API Key (`booking_gcal_api_key`) or other sensitive configuration data [Triage]. | **Low (Security Risk Unconfirmed).** |
| **Admins notified if integrations fail repeatedly (e.g., expired token)** | **Not Implemented.** The synchronization task runs in the background using the custom **pseudo-cron system** (`WPBC_Cron` executing `wpbc_silent_import_all_events()`). While the plugin has a dedicated class for persistent admin warnings (`WPBC_Notices`), there is **no documented architectural component** to monitor, count, or escalate *consecutive* synchronization failures to a persistent, dismissible notice for the administrator [Triage]. | **Low (Lacks Escalation Logic).** |

The score of 4 reflects that while the system protects the database during a single, immediate failure, it is functionally lacking in secure monitoring and notification required for long-running, background API tasks.


This implementation plan details the necessary architectural changes to achieve secure and robust **Error & Failure Handling** by leveraging the plugin's existing synchronization, configuration, and administrative notice systems.

### 1. Enhanced Secure Error Logging (Goal 2 Implementation)

The priority is ensuring that sensitive data, such as the Google API Key or any future OAuth tokens (if implemented), are **never leaked** into logs or error messages presented to the administrator.

*   **Sanitization in the Engine (`WPBC_Google_Calendar`):** The synchronization engine (`WPBC_Google_Calendar` class) is where API communication and error detection occur. The `run()` method already performs robust error checking for `WP_Error` objects and non-200 HTTP status codes.
    *   This method must be modified to **filter the output** of any error message or diagnostic feedback generated.
    *   **Redaction Logic:** Before reporting a failure, the code must retrieve the sensitive credentials (e.g., the `booking_gcal_api_key`) via `get_bk_option()` and redact or hash them within the error message string.
*   **Safe Display:** When generating actionable feedback (using utilities from `core/wpbc-debug.php` like `debuge_error()`), the sanitized error message is used, ensuring that the administrator receives context (e.g., "403 Forbidden: Check API Key Status") without seeing the actual key.

### 2. Implementing Repeated Failure Tracking

A new mechanism is required to track consecutive synchronization failures, moving beyond handling a single error to escalating persistent integration problems.

*   **Persistence of Failure Count:** A new option (e.g., `booking_gcal_fail_count`) must be introduced and managed via the plugin's data abstraction wrappers, such as `update_bk_option()`. This option will track the number of consecutive synchronization failures.
*   **Controller Logic Modification:** The main background execution function, **`wpbc_silent_import_all_events()`** (defined in `core/sync/wpbc-gcal.php` and triggered by the **`WPBC_Cron`** system):
    *   **Success:** If the `WPBC_Google_Calendar::run()` method executes successfully, the `booking_gcal_fail_count` is **reset to zero** using `update_bk_option()`.
    *   **Failure:** If `run()` encounters a non-recoverable error (e.g., 403 Forbidden due to revoked API key), the `booking_gcal_fail_count` is **incremented**.

### 3. Admin Notification and Escalation (Goal 3 Implementation)

This ensures the administrator is persistently warned when the scheduled background synchronization job is chronically failing.

*   **Integration with Persistent Notices:** The plugin already utilizes the **`WPBC_Notices` class** for displaying persistent, dismissible admin warnings.
    *   If the `booking_gcal_fail_count` exceeds a predefined threshold (e.g., three consecutive cron failures), the controller logic triggers a **persistent notice** using the `WPBC_Notices` system.
*   **Custom Notice Registration:** A new persistent notice ID (e.g., `gcal_sync_failed_repeatedly`) would be registered with the `WPBC_Notices` class.
*   **Display Logic:** This notice would display consistently at the top of plugin pages (hooked via custom actions like `wpbc_hook_*_page_header`), informing the administrator that the Google Calendar synchronization is currently disabled due to repeated failures and linking them directly to the synchronization settings page (`page-import-gcal.php`) for troubleshooting.
*   **Resolution:** The notice would be automatically dismissed (removed from the `WPBC_Notices` system) when the `booking_gcal_fail_count` is successfully reset to zero following a successful execution of `wpbc_silent_import_all_events()`.


---

### 6. **Data Protection During API Sync**

* [ ] Is booking/customer data **encrypted during transfer** to CRM/Calendar APIs?
* [ ] Are sync payloads **limited** to necessary fields (no oversharing of PII)?
* [ ] Can admins configure which data fields are sent to external systems?


Based on the sources, the following is an assessment of the implementation of **Data Protection During API Sync** for the documented synchronization feature, which is the **Google Calendar Import**.

It is important to note that the documented synchronization process is **one-way: importing events from Google Calendar to the plugin**. There is no information in the sources regarding the *transferring* or *sending* of local booking/customer data (PII) from the plugin to external Calendar or CRM APIs [Triage]. Therefore, the security analysis is focused on the reverse data flow (import) and the administrative controls around data fields.

### Data Protection During API Sync (Google Calendar Import)

#### Is booking/customer data encrypted during transfer to CRM/Calendar APIs?

**Yes, the data transfer is secured via HTTPS/TLS.**

*   The communication used by the core synchronization engine (`WPBC_Google_Calendar` class) relies on the standard WordPress function **`wp_remote_get()`** to fetch event data from the Google Calendar API v3.
*   The use of `wp_remote_get()` for communication with a major third-party API like Google is described as the **standard, secure, and reliable way** in WordPress to handle external HTTP requests. This strongly implies that the requests are made over **HTTPS/TLS** (Transport Layer Security), securing the transfer of the API key and fetched event data.

#### Are sync payloads limited to necessary fields (no oversharing of PII)?

**Not applicable in the documented direction (Import).**

*   Since the synchronization is a **one-way process designed to import events** from a *public* Google Calendar into the plugin, the plugin is *receiving* external data (event details like title, description, location) rather than *sending* local PII (Personally Identifiable Information).
*   The plugin processes and filters the data it receives, extracting specific event details from the Google JSON feed.

#### Can admins configure which data fields are sent to external systems?

**Yes, partial control is implemented for the data mapping used during the *import* process.**

*   Admins can configure which Google event fields map to which local booking fields. This control is defined on the synchronization settings page (`page-ics-general.php` and referenced in `page-import-gcal.php`).
*   The configuration is stored in the database option **`booking_gcal_events_form_fields`**.
*   The configuration determines how the synchronization engine processes the fetched data, applying the **"admin-configured field mapping"** to correctly map Google event details (title, description, location) to the corresponding local booking form fields.

While this configuration pertains to inbound data mapping rather than outbound data sharing, it demonstrates a mechanism for administrative control over which data fields are processed and stored locally during synchronization.

***

### Score Assessment (1–10)

The implementation of the requested security features scores a **7 / 10**.

The high score reflects that the one implemented security measure (secure transfer via implied HTTPS) is strong, and a key administrative control (field mapping) is present, despite the question being architecturally challenging given the one-way nature of the feature.

| Criterion | Implementation Status | Rationale |
| :--- | :--- | :--- |
| **Encrypted during transfer** | **Implemented (Implied)** | Communication uses `wp_remote_get()` to Google Calendar API v3, confirming secure HTTP transport (HTTPS) for the request/response cycle. |
| **Payload limited (no oversharing)** | **Not Applicable (One-way sync)** | The process imports external data; it does not share local customer PII. |
| **Admins configure fields** | **Implemented (for Import)** | Admins configure the **field mapping** between Google event details (title, description) and local booking form fields using the `booking_gcal_events_form_fields` option. |



---

### 7. **Admin Controls**

* [ ] Is there a **secure setup screen** where admins connect integrations (no raw copy-paste of secrets)?
* [ ] Can admins **disconnect integrations** at any time and revoke tokens?
* [ ] Is there an **audit log** of API calls (who/what data was synced, when)?

Based on the sources detailing the architecture of the Google Calendar synchronization feature, here is an assessment of the implementation status for the requested Admin Controls:

### Admin Controls Implementation Status

#### 1. Is there a secure setup screen where admins connect integrations (no raw copy-paste of secrets)?

**No, this is not implemented.** The documented integration method relies on the administrator manually entering a persistent credential, not a secure token-based flow.

*   The Google Calendar import configuration page (`core/admin/page-import-gcal.php`) allows the administrator to configure the **Google API Key**.
*   The synchronization engine retrieves this API key from the database option `booking_gcal_api_key`.
*   UI helper functions (`wpbc_gcal_settings_content_field_*()`) render the necessary fields on the settings page for the administrator to input these settings.
*   The architecture does **not use an OAuth 2.0 flow** that would shield the administrator from having to handle raw secrets; instead, it requires the manual entry (copy-paste) of a persistent API key.

#### 2. Can admins disconnect integrations at any time and revoke tokens?

**Partially Implemented (Disconnection is possible, but token revocation is not applicable).**

*   **Disconnection:** Administrators can disconnect the integration by removing the **API Key** from the settings page. Furthermore, dedicated lifecycle management functions exist:
    *   `wpbc_sync_gcal_deactivate()` is triggered upon plugin deactivation and uses `delete_bk_option()` to **remove all Google Calendar-related settings** from the database.
*   **Revoking Tokens:** The requested mechanism of revoking tokens is **not applicable** because the plugin uses a persistent **API Key** for communication with Google Calendar API v3, rather than short-lived access tokens and refresh tokens associated with OAuth 2.0. Therefore, there are no tokens to revoke.

#### 3. Is there an audit log of API calls (who/what data was synced, when)?

**No, a persistent audit log is not documented as implemented.**

*   **Error Reporting:** The system provides **robust error checking**. If an API request fails (e.g., non-200 HTTP status code like 403 Forbidden), the synchronization engine halts execution and provides **"actionable feedback to the administrator"**. Debug utilities are available (like `debuge_error()` in `core/wpbc-debug.php`) to generate formatted error messages.
*   **Success Reporting:** After a manual import, the `WPBC_Google_Calendar` class contains a `showImportedEvents()` method that generates an HTML table to display a **summary of the newly created bookings**.
*   **Logging:** While hooks exist (e.g., `wpbc_track_new_booking`) that developers could use to implement logging, the sources **do not document the existence of a dedicated, persistent, and auditable log** that records the details (who, what data, when) of every automated API call and sync event.

Based on the detailed architectural analysis of the plugin's synchronization features and administrative handling, the implementation of the requested **Admin Controls** scores low against the defined security standards.

The overall score is: **3 / 10**

### Rationale Breakdown

The score reflects the fact that while administrators can manually disconnect the feature, the system fails the modern security requirement for the setup screen and lacks the necessary persistent audit logging for background tasks.

| Criterion | Implementation Status | Rationale |
| :--- | :--- | :--- |
| **Secure setup screen (no raw copy-paste of secrets)** | **Not Implemented.** | The Google Calendar import configuration page (`core/admin/page-import-gcal.php`) requires the administrator to manually configure the **Google API Key**. This is a raw copy-paste of a persistent credential and does **not** utilize a secure, automated OAuth 2.0 flow designed to shield the user from handling raw secrets [Triage]. |
| **Admins can disconnect integrations at any time and revoke tokens** | **Partially Implemented.** | **Disconnection** is possible: Administrators can delete the API Key from the settings page, which blocks external access. Furthermore, dedicated lifecycle functions like `wpbc_sync_gcal_deactivate()` are designed to clean up and remove all Google Calendar settings (`delete_bk_option()`) from the database upon plugin deactivation. However, the key concept of **revoking tokens** (refresh/access tokens) is **not applicable** because the plugin uses a persistent API Key, not an OAuth 2.0 flow [Triage]. |
| **Is there an audit log of API calls (who/what data was synced, when)?** | **Not Implemented.** | The sources document robust error checking and immediate feedback mechanisms. For manual imports, the `WPBC_Google_Calendar` class provides the `showImportedEvents()` method, which generates an HTML table summary of *newly created bookings* for the administrator. However, there is **no documented dedicated, persistent audit log** that tracks every automated API call and synchronization event executed by the custom pseudo-cron scheduler (`wpbc_silent_import_all_events()`) [146, 157, Triage]. |


The implementation of enhanced **Admin Controls** requires transforming the Google Calendar synchronization architecture from a static API Key model into a secure, auditable OAuth 2.0 system. This involves leveraging the plugin's core components: the modular page controllers, the custom settings wrappers, the AJAX router, and the event-driven extension points.

Here is a high-level overview of the implementation plan:

### 1. Secure Setup Screen Implementation (Replacing API Key with OAuth 2.0)

The first step is to replace the manual "copy-paste" entry of the persistent API Key with an administrative flow that requests and stores revocable tokens securely [Triage].

*   **UI Modification:** The settings page controller (`core/admin/page-import-gcal.php`) must be refactored. Instead of an input field for `booking_gcal_api_key`, the page will display configuration fields for the **OAuth Client ID** and **Client Secret**, along with a button to initiate the external service's authorization flow. Helper functions (`wpbc_gcal_settings_content_field_*()`) must be updated to manage these new inputs.
*   **Secure Credential Storage:** The successful OAuth callback returns sensitive **Access Tokens** and **Refresh Tokens**. These must be stored securely. The system must introduce an **encryption layer** integrated into the core data abstraction functions, such as the wrappers **`update_bk_option`** and **`get_bk_option`** (defined in `core/wpbc-core.php`), to ensure tokens and secrets are encrypted before being saved to the `wp_options` table.
*   **Token Management:** The synchronization engine must use the encrypted Refresh Token to silently acquire new Access Tokens when needed, ensuring the system remains operational without requiring the admin to re-authenticate (token rotation) [Triage].

### 2. Implementing Integration Disconnection and Token Revocation

The capability for administrators to explicitly disconnect and revoke granted access is crucial. This is handled via a secure administrative action.

*   **UI Integration:** A "Revoke Access" button is added to the synchronization settings page (`page-import-gcal.php`).
*   **Secure AJAX Handler:** The button triggers an AJAX request routed through the central AJAX controller (`core/lib/wpbc-ajax.php`). A new action handler must be registered using the **`wpbc_ajax_action_list` filter** and must strictly enforce **Nonce verification** (using `wpbc_check_nonce_in_admin_panel()` or similar wrappers) to prevent CSRF attacks.
*   **Revocation Logic:** The AJAX handler executes the two-step revocation process:
    1.  **External Revocation:** Call the external API's (e.g., Google's) token revocation endpoint, passing the Refresh Token to invalidate the access on the external service's side.
    2.  **Local Cleanup:** Call the secure database abstraction function **`delete_bk_option()`** to permanently remove all encrypted OAuth tokens and secrets from the local database.

### 3. Implementing a Persistent Audit Log of API Calls

A new, centralized system is needed to log synchronization events (successes, failures, and data processed) without leaking sensitive information.

*   **Log Data Structure:** A new database table or a dedicated options field (if logs are compressed) must be established to store audit records (timestamp, resource ID, action type, status, summary of synced data).
*   **Logging Hooks (Execution):** The logic for recording successful imports must be attached to the plugin's internal workflow.
    *   **Success Logging:** Since the core import class culminates in calling **`wpbc_booking_save()`**, the most robust method is hooking into the post-save event, such as the **`wpbc_track_new_booking`** action, to record the successful creation of a booking imported from the API.
    *   **Failure Logging:** The existing **robust error checking** implemented in the synchronization engine (`WPBC_Google_Calendar::run()`) must be extended to securely record non-200 HTTP status codes (e.g., 403 Forbidden) and API errors to the new audit log, ensuring any potentially sensitive data is redacted from the log entry.
*   **Admin Display:** A new administrative page or sub-tab must be created and registered using the central Admin Menu framework (managed by the `Booking_Calendar` singleton and utilizing the content delegation hooks like `wpbc_page_structure_show`). This page will query the log data and display it to the administrator, using UI helpers like those found in `core/any/admin-bs-ui.php`.


---

### ✅ Success Criteria

* Plugin uses **OAuth 2.0** with secure token storage.
* All calendar/CRM API calls are **HTTPS only, signed, and scoped minimally**.
* Tokens and secrets are **encrypted, not exposed**, and rotated as needed.
* Failed API calls **don’t corrupt bookings** and are **logged securely**.
* Admins have **visibility and control** over integrations (connect, revoke, audit).

---

Do you want me to also create a **hands-on testing checklist** (e.g., “Inspect DB for stored API keys,” “Check if plugin requests excessive scopes,” “Block HTTPS to see if requests still go through”)? That way you can directly verify a plugin’s security posture.
