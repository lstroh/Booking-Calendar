Great — let’s break down **“Parallel booking support (e.g., consultation during long job)”** into **requirements** and **success criteria** so you’ll have a checklist to confirm implementation in the plugin.

---

## **Requirements Breakdown**

### 1. Core Functionality

* [ ] System allows **more than one booking to occur at overlapping times** if business rules permit.
* [ ] Admin can configure **which services/resources allow parallel booking** (e.g., “consultation” can overlap with “main job”, but two main jobs cannot).
* [ ] Booking form supports selecting a **secondary/parallel service** linked to the primary booking.
* [ ] Each booking (primary and parallel) is stored as a **separate record** but linked together under a job ID or group.

Based on the detailed architectural analysis of the plugin's core components, settings, and data flows, **the specific combination of "parallel booking" features listed in your query cannot be definitively confirmed as implemented or documented in the analyzed files.**

However, the sources reveal strong architectural foundations in place regarding resource management and capacity checks that would be necessary for such functionality:

### 1. System allows more than one booking to occur at overlapping times (if rules permit).

The sources indicate that the plugin's core logic focuses on **preventing overlaps** and managing availability, although the older conflict checking system has been explicitly superseded by a newer, more robust capacity management structure.

*   The plugin contains a **legacy function `wpbc_check_dates_intersections()`** which was responsible for checking date conflicts in an older workflow, but this file (`core/lib/wpbc-booking-new.php`) is marked as **deprecated or obsolete**.
*   The current, active booking creation workflow is noted as having been moved to files like `includes/page-bookings/bookings__actions.php` and `includes/_capacity/create_booking.php`, indicating a shift toward a modern **capacity checking** system.
*   The system uses sophisticated internal logic to handle specific check-in and check-out times, often using the last digit of the time's 'seconds' component to signify a check-in (:01) or check-out (:02). This high level of granularity is necessary for managing capacity, but the sources **do not explicitly state** that this new capacity system is configured to *allow* overlaps or parallel booking based on rules.

### 2. Admin can configure which services/resources allow parallel booking.

The plugin clearly supports the concept of **multiple bookable resources** (analogous to services), which is central to configuring availability rules.

*   The plugin relies on the concept of **Booking Resources**, and the file `includes/page-resource-free/page-resource-free.php` is repeatedly identified as a **Top Priority** for future analysis, as it dictates the plugin's **fundamental data model for managing bookable items**.
*   The Google Calendar synchronization feature confirms that the system handles **multiple resources**, specifically querying the **`wp_bookingtypes` table** in paid versions to run a separate import job for each resource configured with a unique GCal feed URL.
*   Administrative settings control availability using functions like `wpbc__sql__get_season_availability()`.
*   **However, the sources do not contain documentation or configuration files that define a setting which explicitly allows toggling "parallel booking" or "overlap permission" on a per-resource basis.**

### 3. Booking form supports selecting a secondary/parallel service linked to the primary booking.

While the plugin uses a comprehensive system for building forms, there is **no mention** of specific functionality for selecting a "secondary/parallel service" linked to a primary one.

*   The administrative form is built using a custom, shortcode-like syntax parsed by `core/form_parser.php`.
*   The form parser uses regular expressions to process fields and values. A limitation noted in the analysis is that the list of supported shortcode types is **hardcoded** into the regex, which would make adding custom, interconnected "secondary service" fields difficult without modifying core files.

### 4. Each booking is stored as a separate record but linked together under a job ID or group.

The plugin uses custom database tables and fields for data storage, but the existence of a specific linking field is not confirmed.

*   Bookings are stored in custom database tables, typically `{$wpdb->prefix}booking`.
*   The plugin stores **custom metadata** (like custom form field values) for a booking as a single, serialized array within the **`booking_options` column** of the booking table. This meta structure could potentially be used to store a custom linking ID.
*   The system uses the `sync_gid` column to store the Google Event ID for synchronization purposes.
*   **However, the sources do not mention a generic `job ID` or `group` column/field** used specifically to link multiple concurrently generated *local* bookings (primary and parallel services) together under a unified job or group identifier.

Based on the comprehensive architectural review provided in the sources, and acknowledging that the features required are highly specific and customized (parallel bookings, overlapping rules, grouping), I would mark the implementation level as **2 out of 10**.

This very low score reflects that while the plugin contains the fundamental architectural building blocks (like managing multiple resources and storing custom data), the specific logic, database structure, and administrative controls necessary to implement and configure "parallel bookings" are **not documented, mentioned, or evidenced** in the analyzed files. The documented architecture is primarily geared toward single-resource availability and **conflict prevention**.

Here is a breakdown of the specific requirements versus the documented architecture:

| Feature Requirement | Implementation Status Based on Sources | Score Rationale |
| :--- | :--- | :--- |
| **1. Allows overlapping times** (if rules permit) | **Not implemented/documented.** | The core documented goal of the plugin's logic (in files like the obsolete `core/lib/wpbc-booking-new.php`) is to check for and **prevent date conflicts**. The modern logic is deferred to capacity checking in other files, but no documentation confirms the capacity system is configured to *allow* exceptions or overlapping based on rules. |
| **2. Admin can configure which services/resources allow parallel booking** | **No documented settings.** | The system uses **Booking Resources** (analogous to services), and paid versions query the `wp_bookingtypes` table to handle multiple resources for synchronization. However, none of the documented settings files (e.g., `core/admin/api-settings.php`) contain a field or configuration setting that defines or toggles "overlap permission" or "parallel booking rules" on a per-resource basis. |
| **3. Booking form supports selecting a secondary/parallel service** | **No evidence, faces architectural limitation.** | The front-end booking form is rendered by parsing a custom, shortcode-like syntax via `core/form_parser.php`. The analysis explicitly notes that the list of supported shortcode types is **hardcoded** into the regular expression in the parser, making the creation of a complex, linked "secondary service" field difficult without modifying core files. |
| **4. Separate record linked by job ID or group** | **No internal grouping ID documented.** | Bookings are stored in custom database tables, typically `{$wpdb->prefix}booking`. The system stores **custom metadata** (from forms) as a single, serialized array in the **`booking_options` column**. Google Calendar imports use the `sync_gid` column to link to external events. However, **no column or field** is mentioned specifically for linking concurrently created *local* bookings together under a unified "job ID" or "group" identifier. |

This implementation requires modification across four key architectural areas of the plugin: **Configuration, UI/Form Handling, Core Availability Logic,** and **Database Persistence**.

Based on the existing plugin architecture, here is a high-level overview of how the parallel booking option could be implemented, drawing on the observed design patterns, data flows, and extension points:

### 1. Configuration: Defining Parallel Resources and Rules

To allow the administrator to define which services or resources permit parallel bookings, the custom configuration framework must be extended.

*   **Settings Implementation:** The administrator controls are centralized in the custom settings framework, specifically defined in the `WPBC_Settings_API_General` class (`core/admin/api-settings.php`). A new setting would need to be introduced either as a global rule (a new option defining which booking types can overlap) or, ideally, attached directly to the Booking Resources defined in the `wp_bookingtypes` table (which is queried in paid versions for complex features like GCal sync).
*   **Settings Persistence and Interception:** If a simple global setting is used, it would be added to the settings array via the `init_settings_fields()` method. The custom option wrapper functions, such as `get_bk_option()` (defined in `core/wpbc-core.php`), allow intercepting or modifying configuration values via the **`wpdev_bk_get_option`** filter. This filter would be used to retrieve the "parallel permission" status before checking availability.

### 2. UI and Form Handling: Selecting the Secondary Service

To enable selecting a secondary/parallel service, the front-end form must support a new input field.

*   **Form Rendering:** The plugin uses `core/form_parser.php` to transform a custom shortcode-like text configuration into a structured PHP array for rendering. However, the parser's list of recognized shortcode types is hardcoded into its regular expression, which makes adding entirely new field types difficult.
*   **Implementation Strategy:** Instead of relying on the hardcoded form parser, a developer would use the plugin's documented extension filters to **inject custom HTML** for the secondary service selection field. The filter **`wpdev_booking_form`** (defined in `core/lib/wpdev-booking-class.php`) is the designated extension point for modifying the rendered booking form output. This injected HTML would handle selecting the parallel resource ID and pass it to the submission logic.
*   **Client-Side Data:** The `core/wpbc-js-vars.php` file acts as the primary data bridge, injecting necessary configuration and data into the global `_wpbc` JavaScript object. The configuration data for the parallel resource selection (e.g., current resource availability status) would be passed to the JavaScript via the **`wpbc_js_vars`** filter.

### 3. Core Logic: Overriding Availability Checks

The key architectural challenge is overriding the core logic that currently aims to *prevent* intersections (superseded legacy logic existed in `core/lib/wpbc-booking-new.php`) and instead allow overlap based on the configured rules.

*   **Data Access Abstraction:** Availability is checked using complex database queries defined in files like `core/wpbc-dates.php`. Functions such as **`wpbc__sql__get_booked_dates()`** or **`wpbc__sql__get_season_availability()`** retrieve booked dates by querying the custom `booking` and `bookingdates` database tables.
*   **Implementation Strategy (Logic Override):** The most effective method is to modify the SQL query that checks for booked dates. Although the core SQL functions are contained in utility files, the primary data retrieval function, **`wpbc_get_bookings_objects()`** (used by the Timeline feature to fetch data), is designed to be filterable. A developer would need to find the appropriate filter point (likely associated with the `core/admin/wpbc-sql.php` data engine) to inject a custom condition into the `WHERE` clause of the query. This condition would allow the system to ignore conflicts for bookings associated with resources specifically marked as "parallel-allowed" by the administrator.

### 4. Data Persistence: Storing and Linking Records

Since no dedicated `job ID` or `group` column is documented in the custom database tables, the linking mechanism must utilize the plugin's existing metadata storage structure.

*   **Booking Creation:** The saving process is abstracted via the developer API function **`wpbc_api_booking_add_new()`** (in `core/wpbc-dev-api.php`) or the internal core save function, which handles conversion of inputs to the plugin's complex internal format. The parallel service feature would require this core saving logic to be triggered multiple times (once for the primary service, once for the secondary service).
*   **Custom Linking ID:** A unique Job ID would be generated during submission. This ID would then be stored in the custom metadata field **`booking_options`** for *both* resulting booking records. The function **`wpbc_save_booking_meta_option()`** (in `core/wpbc-core.php`) provides the safe, intended way to save custom data to this serialized column without altering the database schema.
*   **Post-Creation Workflow:** A key point for triggering the creation of the parallel booking is the post-creation action hook. The official Developer API documents the action hook **`wpbc_track_new_booking`**, which is triggered when a new booking is inserted (including those inserted via the GCal synchronization process). A developer would hook into this action to check if the primary booking included a parallel service request and, if so, initiate the second booking record with the shared Job ID.
*   **Database Querying:** Retrieving linked parallel bookings later would rely on querying the serialized data in the `booking_options` column for the Job ID, which, while possible, is noted as **inefficient** and breaks database normalization.


### 2. Scheduling Rules

* [ ] Admin can define **max number of parallel bookings** per resource/tradesperson.
* [ ] Parallel bookings **cannot conflict** if they require the same exclusive resource (e.g., one tradesperson).
* [ ] If parallel service uses a different resource (e.g., admin staff for consultation), system checks that person’s availability separately.
* [ ] Booking duration logic supports overlap — e.g., a 1-hour consultation inside an 8-hour main job.

Based on the sources and our previous conversation, **none of the specific scheduling rules required for complex "parallel booking" functionality are explicitly documented or confirmed as implemented.**

The plugin's architecture is focused on standard resource management and conflict **prevention**, and while it has the necessary components to build such a system, the specialized logic is not evidenced in the analyzed files.

Here is a breakdown of the required scheduling rules against the plugin's documented architecture:

### 1. Admin can define max number of parallel bookings per resource/tradesperson.

This functionality, often referred to as resource **capacity management**, is implied as the intended direction of the newer architecture, but the specific configuration setting is not defined in the sources.

*   The plugin contains configuration settings for **Availability rules** defined in the `WPBC_Settings_API_General` class found in `core/admin/api-settings.php`. However, the sources do not mention a specific setting field that defines the "max number of parallel bookings" or capacity limits.
*   The system includes a dedicated date engine in `core/wpbc-dates.php` that uses **SQL functions** to retrieve booked dates and calculate **season availability** (`wpbc__sql__get_season_availability()`). This capacity-checking architecture supersedes an older, obsolete function for checking dates intersections (`wpbc_check_dates_intersections()`).
*   The analysis consistently recommends investigating the file **`includes/page-resource-free/page-resource-free.php`** as crucial for understanding the plugin’s fundamental data model for managing **Booking Resources**. This file would likely contain the configuration required for capacity settings, but it has not yet been analyzed.

### 2. Parallel bookings cannot conflict if they require the same exclusive resource.

The core logic documented in the source is designed to **prevent** conflicts generally, but there is **no evidence** that it can selectively allow conflicts (parallel bookings) for some resource rules while enforcing exclusivity for others.

*   The plugin has robust database querying capabilities in `core/wpbc-dates.php` for fetching booked dates (`wpbc__sql__get_booked_dates()`) and calculating availability based on rules.
*   The core resource management system is based on **Booking Resources**, which are queryable and managed in paid versions via the `wp_bookingtypes` table, allowing the system to run separate synchronization imports for each resource. This structure is a necessary foundation for resource exclusivity rules, but the rules themselves are not documented.
*   The system's deprecated conflict checker relied on complex and unconventional string manipulation, using the last digit of the time's seconds component (:01 or :02) to mark start or end times. While complex time handling exists, the sources do not show how modern capacity checks implement flexible exclusivity rules.

### 3. If parallel service uses a different resource, system checks that person’s availability separately.

The system is designed to handle **multiple resources**, making it architecturally capable of checking separate availability, but the requirement that a single booking submission triggers simultaneous, separate availability checks is not documented.

*   The core architecture supports **Multi-User (MU) support** and resource ownership via utility functions in `core/wpbc_functions.php` (e.g., `wpbc_mu_set_environment_for_owner_of_resource`). This allows the system to change context based on which "tradesperson" owns the resource.
*   The **Developer API** (`core/wpbc-dev-api.php`) includes a function, `wpbc_api_is_dates_booked()`, which checks if a given set of dates/times is available for a resource. This function could be leveraged internally by a parallel booking feature to check multiple resources separately, but the source does not indicate that the core form submission workflow currently calls this function multiple times for a single submission.

### 4. Booking duration logic supports overlap.

The system's sophisticated date and time engine handles complex time manipulation, making overlap technically possible, but the logic is primarily focused on strict time boundaries.

*   The timeline rendering engine relies on the method `wpbc_get_dates_and_times_for_timeline()` within the `WPBC_TimelineFlex` class to process raw booking data into structured time slot arrays. This logic uses the specific time marker (:01 for check-in, :02 for check-out) to handle partial-day and changeover bookings precisely.
*   The date utilities in `core/wpbc-dates.php` and `core/wpbc_functions_dates.php` handle parsing, validation, and conversion of dates and times.
*   While this complexity allows the system to know exactly when a booking begins and ends, **there is no documented function or setting that would instruct the system to permit an intentional overlap** (like scheduling a 1-hour event *within* an 8-hour event) without triggering a conflict based on resource capacity rules.

  Based on the detailed architectural analysis of the plugin's configuration, core logic, and scheduling systems, the implementation level for these specific **Scheduling Rules** features is still considered **very low**.

I would mark the implementation level as **2 out of 10**.

This score is given because the plugin's architecture is strongly built around preventing conflicts and ensuring exclusivity, which runs counter to the requirement for conditional *parallelism*. While the necessary foundational components (resource data models, availability engines) are present, the specific custom settings and logic needed to define and enforce complex parallel scheduling rules are not documented in the analyzed files.

Here is a breakdown of the specific requirements against the documented architecture:

| Feature Requirement | Implementation Status Based on Sources | Score Rationale |
| :--- | :--- | :--- |
| **1. Admin can define max number of parallel bookings per resource/tradesperson (Capacity).** | **No documented configuration setting.** | The plugin architecture utilizes an Availability rules system and a modern capacity checking structure. Functions like `wpbc__sql__get_season_availability()` handle capacity calculations. However, there is no evidence of a dedicated setting field, likely in `core/admin/api-settings.php` or a resource management file (e.g., `includes/page-resource-free/page-resource-free.php`), that allows an administrator to input a **numerical maximum** for overlapping bookings per resource. |
| **2. Parallel bookings cannot conflict if they require the same exclusive resource.** | **Core logic prevents conflict by default.** | The plugin contains functions whose primary purpose is to retrieve booked dates (`wpbc__sql__get_booked_dates()`) and manage multi-user (MU) environments (via functions in `core/wpbc_functions.php`), implying resource ownership. However, the system lacks the crucial *conditional override* logic that would allow "Service A" to overlap with "Service B" but strictly block two instances of "Service A" at the same time. The presence of the obsolete conflict checker (`wpbc_check_dates_intersections()` in `core/lib/wpbc-booking-new.php`) confirms the historical focus on conflict *prevention*, not conditional allowance. |
| **3. If parallel service uses a different resource, system checks that person’s availability separately.** | **Capability exists, but the integrated workflow is missing.** | The core plugin architecture supports multiple bookable resources. The Developer API provides the function `wpbc_api_is_dates_booked()`, allowing external code to check availability for a specific resource programmatically. However, since the necessary front-end form fields to select a parallel resource are missing (as noted in our previous conversation), the core form submission process cannot trigger simultaneous, separate checks for multiple distinct resources. |
| **4. Booking duration logic supports overlap (e.g., a 1-hour consultation inside an 8-hour main job).** | **Complex time handling is present, but permissive rules are absent.** | The logic engine uses complex date and time handling, including sophisticated logic to process raw booking data into structured time arrays (utilizing the seconds markers :01/:02 for check-in/out). While this precision exists, the sources do not document any logic that would instruct the system to permit one booking to be nested *within* another without triggering a conflict based on standard availability rules. This would require the conditional exclusivity settings (Feature 2), which are not evidenced. |

The implementation of the specialized "Scheduling Rules" for parallel bookings requires coordinated modifications across the plugin's Settings API, data querying engine, front-end submission process, and booking persistence layer.

This implementation relies heavily on utilizing the plugin's custom hook systems (defined in **`core/wpbc-core.php`**) and leveraging the abstraction points provided by the **Developer API** (**`core/wpbc-dev-api.php`**).

Here is a high-level overview of the implementation strategy:

---

### 1. Configuration: Defining Capacity and Conditional Overlap Rules

The system must be extended to allow administrators to define the maximum number of parallel bookings allowed for a resource and to configure which resources are considered "exclusive."

*   **Resource Capacity Setting:** The administrative configuration is defined in the **`WPBC_Settings_API_General`** class in **`core/admin/api-settings.php`**. New fields (e.g., "Max Parallel Bookings" field for numerical capacity, and a toggle for "Exclusive Resource") must be defined within the `init_settings_fields()` method. These settings should ideally be attached to the **Booking Resources** themselves, which are fundamental to the data model.
*   **Settings Persistence:** The new resource configuration data would be saved via the custom settings framework. The filter **`wpbc_settings_validate_fields_before_saving`** or **`wpdev_bk_get_option`** is the designated extension point to read, modify, or inject custom logic before configuration data is saved or retrieved.

### 2. Submission & Persistence: Creating Linked, Parallel Records

The challenge here is taking a single user submission for a "primary job" and "parallel service" and translating it into two separate, linked database entries.

*   **Front-End Selection:** The analysis noted that **`core/form_parser.php`** has a hardcoded list of supported shortcode types. To add a selection field for the "secondary/parallel service," a developer would need to use the **`wpdev_booking_form`** filter (likely found within the main **`wpdev_booking`** class logic) to **inject custom HTML/JavaScript** for selecting the parallel resource ID.
*   **Dual Booking Creation:** Upon submission, the server-side logic must instantiate two distinct booking processes: one for the primary resource and one for the parallel resource. This logic should utilize the Developer API function **`wpbc_api_booking_add_new()`** to create each booking. This function abstracts the internal complexities and ultimately delegates to the core saving function, **`wpbc_booking_save()`**.
*   **Data Linking (Job ID):** A unique Job ID or Group ID must be generated. This identifier would be stored in the custom metadata column **`booking_options`** for *both* resulting booking records, utilizing the safe persistence function **`wpbc_save_booking_meta_option()`**.
*   **Post-Creation Workflow:** The core save process fires the action hook **`wpbc_track_new_booking`**. A developer would hook into this action to trigger follow-up logic, such as sending two distinct sets of confirmation emails (via **`core/wpbc-emails.php`**) or syncing the two related bookings with external systems.

### 3. Core Availability Logic: Selective Conflict Checking

This is the most critical step, requiring modification of the core availability engine to conditionally allow overlapping bookings based on resource rules while enforcing exclusivity where required.

*   **Data Abstraction Layer:** The timeline and list views retrieve data via the global function **`wpbc_get_bookings_objects()`**, which in turn relies on the query building engine in **`core/admin/wpbc-sql.php`**.
*   **Conditional SQL Override:** The `wpbc_get_sql_for_booking_listing()` function in the SQL engine is explicitly noted as being highly extensible via **"numerous filters"**. A developer would use an appropriate filter (e.g., one related to the WHERE clause, such as `get_bklist_sql_keyword`) to **inject logic that modifies the core availability query**.
*   **Logic Implementation:** This injected SQL modification would conditionally exclude a booking from capacity calculations if:
    1.  The resource is configured to allow parallel bookings (Capacity > 1).
    2.  The overlap is between different types of resources (e.g., an exclusive "tradesperson" resource and a parallel "admin staff" resource, requiring separate availability checks).

### 4. Advanced Scheduling: Duration and Timezone Management

The system’s existing date/time engine provides the necessary precision to manage complex overlap scenarios.

*   **Accurate Time Handling:** Functions in **`core/wpbc-dates.php`** and **`core/wpbc_functions_dates.php`** are already highly sophisticated, using timezone offsets and specific second markers (:01 for check-in, :02 for check-out) to manage complex time boundaries, which is essential for ensuring a 1-hour consultation can be correctly scheduled *inside* an 8-hour main job without conflict.
*   **Capacity Enforcement:** The final check must ensure that even if overlaps are allowed, the configured *max capacity* is not exceeded for any shared, non-exclusive resources, utilizing the capacity information defined in the resource settings (Step 1).




### 3. User Experience

* [ ] Client-facing form makes it clear when a **parallel service** is being booked (e.g., “Book consultation during your installation”).
* [ ] Confirmation screen and emails list **both the main booking and the parallel session** with their times.
* [ ] Calendar view shows overlapping bookings visually (e.g., stacked events, side-by-side).


Based on the architectural review of the plugin's User Experience (UX) components, form processing, email system, and calendar visualization, the specific options required for clearly displaying and managing **"parallel service" bookings are not implemented.**

The sources describe a robust system designed for single-session bookings, notifications, and calendar visualization, but they lack evidence of the specialized functionality necessary to recognize, process, and display two separate, linked sessions as a unified parallel booking event.

Here is an analysis of each point based on the provided sources:

### 1. Client-facing form makes it clear when a parallel service is being booked.

While the plugin features a robust form rendering engine, the form parsing system is noted as having a limitation that prevents the easy introduction of complex, linked fields required for parallel booking selection.

*   The core file **`core/form_parser.php`** is responsible for transforming the administrator-defined shortcode-like configuration string into a structured PHP array for rendering.
*   A key limitation noted in the analysis is that the parser's list of **supported shortcode types is hardcoded into the regular expression**. This means that implementing a specialized field that explicitly links a "secondary/parallel service" to a primary one would be difficult and likely require modification of the core parser logic.
*   The system allows administrators to define custom form fields (using **`wpbc_parse_form`**) and values, but there is no documented mechanism to make the form dynamically understand when a complex "parallel service" is being selected.

### 2. Confirmation screen and emails list both the main booking and the parallel session with their times.

The email system is highly customizable through shortcodes, but there is no indication that the underlying booking data includes the necessary structure to display two separate, linked bookings as a single confirmation message.

*   The email system is handled by dedicated classes (e.g., `WPBC_Emails_API_Approved`) and managed by **`core/wpbc-emails.php`**.
*   Emails are customized using dynamic **shortcodes** (e.g., `[booking_id]`, `[dates]`, `[resource_title]`) listed by **`wpbc_get_email_help_shortcodes()`**. The **`wpbc_replace_params_for_booking` filter** is available to inject custom shortcodes.
*   If a parallel service were implemented (which requires two separate records linked by a common Job ID, as discussed in our history), the standard shortcodes would only retrieve information related to the **single booking record** currently being processed (the primary or the parallel one). There is no documented shortcode (like `[parallel_session_dates]`) or inherent email template logic that understands how to query the database for a linked secondary booking and display its specific details alongside the primary session.

### 3. Calendar view shows overlapping bookings visually (e.g., stacked events, side-by-side).

The administrative and front-end calendar visualizations are focused on representing resource availability and capacity, which currently prioritizes conflict *prevention* rather than conditional overlapping.

*   The administrative **Timeline View** is the most sophisticated visual component, rendering a Gantt-chart-like representation of bookings using the **`WPBC_TimelineFlex`** class.
*   The timeline's logic is highly detailed in handling specific check-in and check-out times, utilizing the last digit of the time's seconds component (:01 or :02) to correctly process partial-day bookings and changeovers.
*   However, this visualization engine, powered by **`wpbc_class-timeline_v2.php`**, is based on transforming raw booking data into structured arrays that accurately show capacity usage. Since the underlying scheduling rules are designed to prevent conflicts (as discussed in the prior response), the system would likely view overlapping events as conflicts rather than separate, stackable events, unless the data processing logic in **`wpbc_get_dates_and_times_for_timeline()`** were specifically modified to allow and render concurrent, non-conflicting bookings.

---
### **Implementation Score**

Based on the lack of explicit UI components, email templating logic, or calendar rendering features to support a linked parallel session, I would mark the implementation level as **2 out of 10**.

This score is consistent with previous ratings, reflecting that while the plugin has the architectural framework (custom forms, email templates, timeline view) needed for a booking system, it lacks the specialized configuration, data linking, and display logic required to handle and present **linked parallel sessions** to the client.

The implementation of the User Experience features for parallel bookings requires extending the plugin's form handling, email templating, and administrative visualization components. This strategy relies heavily on custom hooks, the centralized data abstraction layers (like option wrappers and meta functions), and the specialized rendering systems established in the plugin's architecture.

Here is a high-level overview of the implementation for the User Experience features:

### 1. Client-Facing Form Clarity (Selecting the Parallel Service)

To ensure the client-facing form clearly presents the option to book a parallel service, the front-end rendering mechanism must be extended, leveraging the plugin’s UI framework.

*   **Custom UI Injection:** Since the form parser (**`core/form_parser.php`**) is noted as having hardcoded shortcode types, limiting the easy introduction of new field types, the custom service selector must be injected via a hook during the form rendering process (likely a filter hook exposed by the central controller in a file like `core/lib/wpdev-booking-class.php`).
*   **Aesthetics and Styling:** The injected HTML should utilize helper functions from the UI component library (**`core/any/admin-bs-ui.php`**) to maintain the plugin's consistent, Bootstrap-style presentation.
*   **Data Submission:** The selection for the secondary resource ID must be passed to the server via the main AJAX request. This request is routed through the central AJAX controller (**`core/lib/wpbc-ajax.php`**) to ensure security via nonce verification.

### 2. Confirmation Screen and Emails Listing Both Sessions

To display details for both the primary and parallel sessions in a single confirmation email or screen, the existing shortcode replacement engine must be customized to query the linked booking records.

*   **Data Linking:** The implementation assumes that both booking records (primary and parallel) are linked by a common "Job ID," which is stored as a serialized meta-option in the **`booking_options`** column of the custom booking table via **`wpbc_save_booking_meta_option()`**.
*   **Custom Shortcode Implementation:** The Email API pattern relies on filters for dynamic content. A developer would define a new, custom email shortcode (e.g., `[parallel_session_details]`) and hook into the **`wpbc_replace_params_for_booking`** filter.
*   **Retrieval and Formatting:** The custom shortcode logic must:
    1.  Use functions like **`wpbc_get_booking_meta_option()`** to retrieve the Job ID from the current booking record being processed.
    2.  Query the database to find the linked parallel booking record(s) using this Job ID.
    3.  Format the dates and times for the parallel session using the localization utility functions in **`core/wpbc_functions_dates.php`** (e.g., **`wpbc_datetime_localized()`**) to ensure accurate display based on the site’s configured locale and timezone offset.
*   **Email Sending:** The final, customized content is processed and dispatched by the core email hub (**`core/wpbc-emails.php`**), which wraps the sending in **`wpbc_wp_mail()`** to ensure proper deliverability and header configuration.

### 3. Calendar View Showing Overlapping Bookings

The visualization of overlapping parallel bookings requires modifying the complex rendering engine used by the administrative timeline view.

*   **Core Rendering Engine:** The administrative Timeline View is managed by the **`WPBC_TimelineFlex`** class, which is responsible for transforming raw booking data into a structured Gantt-chart-like grid. The class handles complex time slot processing, including using the "seconds" component (:01 for check-in, :02 for check-out) for granular time accuracy.
*   **Data Flow Strategy:** The **`WPBC_TimelineFlex`** class fetches data by calling the global data-access function **`wpbc_get_bookings_objects()`**. This data flow must be configured to pass both the primary and parallel records to the rendering engine. Direct modification of the complex PHP logic is discouraged due to risk.
*   **Visual Customization (Stacking):** The visual stacking of parallel events is achieved by leveraging the modern CSS architecture:
    *   The timeline utilizes **CSS Custom Properties** (variables) defined in **`timeline_v2.1.css`**. The safest method is to override these variables in a custom stylesheet, perhaps defining a unique color for parallel bookings.
    *   The layout relies on **CSS Flexbox** for alignment. Custom CSS must be written to apply specific Flexbox or positioning rules to ensure that linked, parallel bookings are displayed either stacked or side-by-side without visually conflicting with the resource’s capacity display.



### 4. Payment & Invoicing

* [ ] Payment can be handled either:

  * As a **single combined invoice** for main + parallel service, or
  * As **separate line items** configurable by admin.
* [ ] Total cost clearly reflects parallel services (no double-charging).

Based on the architectural files analyzed in the sources, there is **no explicit implementation or documented feature related to handling, combining, or invoicing costs for complex "parallel service" bookings.**

The core files detail infrastructure necessary for calculating **a single booking's cost**, managing emails (which could include pricing data), and extending functionality for third-party systems, but they do not describe the specific payment logic requested.

Here is a breakdown of the existing architecture in relation to your query:

### 1. Payment can be handled either as a single combined invoice or as separate line items.

The sources **do not document** any invoicing or cart system that combines or itemizes costs for multiple, linked bookings (which would be required for "main + parallel service").

*   **Cost Calculation:** The AJAX infrastructure handles dynamic cost calculation on the front-end for *a single booking request*. This functionality is crucial for displaying the cost before submission.
*   **Data Structure:** The plugin stores custom booking data, likely including the final cost and associated form fields, as a single serialized array in the **`booking_options`** column of the custom database table. While this column could theoretically hold line item data, the sources do not provide functions for retrieving or generating structured invoices from this data.
*   **Invoicing System:** There is **no mention** of a dedicated invoicing system, invoice generation functions, or logic to manage combined vs. separate invoicing based on administrative configuration.

### 2. Total cost clearly reflects parallel services (no double-charging).

While the cost system itself is designed to handle calculations, the complexity of combining costs from **two separate, linked records** (the primary booking and the parallel service booking, as established in our previous conversation) is not addressed in the documented architecture.

*   **Booking Creation:** We know that a parallel booking implementation would require creating **two separate booking records** linked by a custom "Job ID". The current workflow functions for booking creation (such as `wpbc_api_booking_add_new()`) and availability checks are designed for a single booking instance.
*   **Data Aggregation:** To create a combined invoice or ensure no double-charging, the system would need custom logic to query the database, identify all bookings sharing the same Job ID (by querying the serialized `booking_options` column), aggregate their individual costs, and then present a single, corrected total. **This aggregation logic is not present** in the administrative data retrieval engine (`core/admin/wpbc-sql.php`) or the core utility files.

### 3. Email Confirmation for Invoicing

The system has robust email functionality, but the ability of the email shortcode system to pull and combine cost data from multiple, linked records is highly unlikely without custom development.

*   **Email System:** The email hub (**`core/wpbc-emails.php`**) handles validation, formatting, and delivery. Email templates rely on dynamic shortcodes (e.g., `[booking_id]`, `[dates]`, `[resource_title]`) listed by `wpbc_get_email_help_shortcodes()`.
*   **Shortcode Limitation:** To list a combined total, a custom shortcode would be required to perform the complex, cross-record database query and cost calculation mentioned above, then return a final, single price string. The standard shortcode replacement process is primarily designed to retrieve data relevant to the single booking ID being processed.

---
### **Implementation Score**

Based on the lack of any documented feature for multi-record cost aggregation, itemized invoicing, or configured cost handling for linked parallel services, the implementation level is marked as **1 out of 10**.

While the foundation for cost calculation on a single booking exists (via AJAX) and custom data storage is possible (via `booking_options`), the highly specific logic for combining, itemizing, and invoicing the costs of multiple, linked bookings is entirely absent.

The implementation of Payment & Invoicing for parallel bookings requires extending the plugin's configuration, database interaction, and email templating systems to handle cost aggregation across multiple, linked booking records.

This strategy relies heavily on the plugin's use of **custom metadata** and its **internal hook system** to perform complex cost calculation outside of the normal, single-booking workflow.

Here is a high-level overview of the implementation:

### 1. Configuration: Defining Invoice Presentation

The administrator must be able to configure how the customer views the cost (combined vs. itemized).

*   **Settings Extension:** A new setting would be added to the plugin's custom Settings API, which is primarily managed by the **`WPBC_Settings_API_General`** class (defined in `core/admin/api-settings.php`). This setting would define the preferred invoice style (e.g., 'combined total' or 'itemized line items').
*   **Data Retrieval Hook:** This setting's value should be retrieved programmatically using the `get_bk_option()` wrapper function, allowing developers to intercept and modify the setting via the **`wpdev_bk_get_option`** filter.

### 2. Data Persistence: Tracking Individual Costs and Linking Records

Since parallel services result in **separate booking records**, the system must ensure the records are trackable and contain their respective costs.

*   **Job ID Storage:** When the primary and parallel bookings are created (likely using the Developer API function `wpbc_api_booking_add_new()`), a unique **Job ID** must be generated. This ID would be stored in the custom metadata column **`booking_options`** for *each* booking record, using the safe method **`wpbc_save_booking_meta_option()`**.
*   **Cost Segregation:** Each record would store the individual calculated cost for that specific service (e.g., Primary Job cost, Parallel Service cost) within its main database fields, or as part of the serialized `booking_options` array.

### 3. Core Logic: Cost Aggregation and Total Calculation

To ensure **no double-charging** and to generate the final invoice, a new function is required to query and aggregate costs based on the Job ID.

*   **Custom Aggregation Function:** A new helper function must be developed that takes the unique Job ID as input. This function would perform a direct **SQL query** (using the global `$wpdb` object, as seen in `core/wpbc-core.php` and `core/wpbc-dates.php`) against the custom booking table.
*   **Querying Serialized Data:** The query must search the serialized data within the **`booking_options`** column for all records matching the shared Job ID.
*   **Total Calculation:** The function would retrieve the cost from each linked record and calculate the **single combined total**. This central function would be called whenever a final price needs to be displayed or sent to the user.

### 4. Presentation: Dynamic Invoice and Email Content

The final step involves customizing the transactional emails and confirmation screens to display the correct combined or itemized total based on the admin setting.

*   **Email Custom Shortcode:** The plugin's robust Email API relies on dynamic shortcodes (like `[booking_id]`, `[resource_title]`). A new shortcode (e.g., `[combined_invoice_total]`) would be registered via the **`wpbc_replace_params_for_booking`** filter.
*   **Conditional Display:** The logic behind this custom shortcode would:
    1.  Retrieve the Job ID from the current primary booking record.
    2.  Call the new Cost Aggregation function (Step 3) using the Job ID.
    3.  Check the admin setting (Step 1). If set to 'combined,' display the single total. If set to 'itemized,' list the components pulled from the linked records and display the sum.
*   **Deliverability:** All transactional emails using this new content would be processed by the system in `core/wpbc-emails.php`, which wraps `wp_mail()` in a custom function (`wpbc_wp_mail`) to ensure correct header setup for improved deliverability.

### 5. Admin Controls

* [ ] Admin can see linked parallel bookings in dashboard/job view.
* [ ] Admin can reschedule/cancel a parallel booking without affecting the main booking (and vice versa).
* [ ] Reports and exports treat parallel bookings as distinct but linked entities.

Based on the detailed architectural review and the constraints discussed regarding the lack of native "parallel booking" functionality in the current plugin structure, **none of the specific administrative controls for linked parallel bookings are implemented or evidenced in the analyzed files.**

The existing administrative systems are designed to manage independent booking records based on standard filters and single identifiers, rather than custom grouping or complex linkage.

### 1. Admin can see linked parallel bookings in dashboard/job view.

This functionality is not implemented, as the administrative viewing tools rely on standard status and resource filters, not custom grouping identifiers.

*   **Dashboard View:** The administrative dashboard widget provides "at-a-glance" statistics by querying the database via `wpbc_db_dashboard_get_bookings_count_arr()`. This function provides simple counts for single bookings (new, pending, approved) and does not contain logic to recognize, group, or display linked job records.
*   **Booking Listing/Job View:** The core data engine for the Booking Listing (`core/admin/wpbc-sql.php`) constructs complex SQL queries. While the core query function, `wpbc_get_sql_for_booking_listing()`, is highly extensible through "numerous filters" (e.g., `get_bklist_sql_keyword`), there is **no documented feature** that utilizes this extensibility to automatically search the serialized custom data column (`booking_options`) for a shared Job ID to group bookings visually in the main administrative view.
*   **Timeline View:** The Timeline View visualizes bookings chronologically using the `WPBC_TimelineFlex` class. This view is focused on showing capacity and availability by resource over time, and does not have built-in logic to group two co-occurring events based on a custom internal Job ID.

### 2. Admin can reschedule/cancel a parallel booking without affecting the main booking (and vice versa).

The system architecture handles status changes based on a single booking ID, meaning it lacks the custom logic required to understand the relationship between linked bookings during an action.

*   **Single-ID Focus:** Booking status changes (approve, trash, delete) are processed dynamically via AJAX requests routed through the central controller, `core/lib/wpbc-ajax.php`. Functions like `wpbc_ajax_UPDATE_APPROVE()` execute direct database queries to modify the status of the **single booking ID** provided in the request.
*   **Missing Relational Logic:** There is **no documented logic** within the AJAX handlers or the core status management functions (found in `core/wpbc_functions.php`) that checks for a linked Job ID (stored in the serialized `booking_options` column) before executing a cancellation or reschedule.
*   **Status Action Hooks:** The system does fire action hooks upon status changes (e.g., `wpbc_booking_approved`, `wpbc_booking_delete`), but these hooks are primarily intended for triggering external side-effects (like CRM synchronization), not for managing internal peer-to-peer booking relationships within the core plugin logic.

### 3. Reports and exports treat parallel bookings as distinct but linked entities.

There is no evidence of cost aggregation or entity linking in the documented export and reporting architecture.

*   **Data Aggregation Limitation:** The core architectural challenge is that custom data (like the hypothetical Job ID used for linking) is stored as a **single serialized array in the `booking_options` column**. This makes querying or filtering based on that meta field highly inefficient and complex. Standard data reporting relies on direct, efficient queries.
*   **Export Delegation:** Advanced export features (such as ICS feed management) are delegated to the "Booking Manager" companion plugin via hooks like `do_action( 'wpbm_ics_import_start', ... )`. Since the core plugin does not define the structure for combined entities, it is not possible to confirm that the delegated reporting feature can treat parallel bookings as a single linked unit.

Based on the comprehensive architectural review provided in the sources and our previous discussion, **none of the specific administrative controls for linked parallel bookings are implemented or evidenced.**

I would mark the implementation level as **2 out of 10**.

This low score reflects the fact that while the plugin provides the foundational systems for data persistence and administrative actions, the specialized logic required to recognize, manage, and coordinate these actions across *two distinct, linked booking records* is entirely absent. Furthermore, the plugin's architectural choices create structural challenges for implementing this feature:

### Justification Breakdown

| Feature Requirement | Implementation Status Based on Sources | Score Rationale |
| :--- | :--- | :--- |
| **1. Admin can see linked parallel bookings in dashboard/job view.** | **Not implemented.** | The administrative **Dashboard Widget** queries for simple, categorized counts (new, pending, approved) and does not group records. The Booking Listing relies on dynamic SQL queries constructed by **`core/admin/wpbc-sql.php`**. Because the proposed linking "Job ID" would be stored in the **`booking_options`** column as a **serialized array**, querying the database natively to find all records sharing that Job ID is highly **inefficient and breaks database normalization**. Therefore, the administrative views cannot natively group them. |
| **2. Admin can reschedule/cancel a parallel booking without affecting the main booking (and vice versa).** | **Not implemented.** | The plugin's architecture dictates that all booking status changes (approve, trash, delete) are handled via the central AJAX router (**`core/lib/wpbc-ajax.php`**). Functions like `wpbc_ajax_DELETE_APPROVE()` execute direct database queries to modify the status of the **single booking ID** provided in the request. There is **no documented logic** that would check for a linked "Job ID" or apply conditional processing required to maintain the independent status of the peer booking during a cancellation or reschedule event. |
| **3. Reports and exports treat parallel bookings as distinct but linked entities.** | **Not implemented.** | Advanced export functionality (e.g., ICS feeds) is explicitly delegated to a companion plugin, **"Booking Manager,"** via hooks such as `do_action( 'wpbm_ics_import_start', ... )`. Since the core plugin lacks the necessary data aggregation logic to query and combine records based on the serialized metadata, it cannot pass consolidated, linked entity data to its internal reporting tools or to the external companion plugin for export. |

This implementation plan outlines the necessary high-level modifications across the plugin's architecture—specifically data storage, administrative data retrieval, AJAX handling, and reporting—to introduce the requested administrative controls for linked parallel bookings.

The architecture relies heavily on storing a unique identifier (Job ID) in the custom booking metadata, which is stored as a serialized array in the database.

---

### 1. Data Persistence: Establishing the Linking ID

The foundation of all administrative controls is ensuring that the primary booking and the parallel service booking (which are separate database entries) are permanently linked.

*   **Job ID Generation and Storage:** During the single booking submission process, a unique Job ID must be generated. This ID is then stored as a custom meta-option within the **`booking_options`** column of the custom `{$wpdb->prefix}booking` table for *each* resulting booking record.
*   **API Utilization:** This storage must utilize the secure helper function **`wpbc_save_booking_meta_option()`** (located in `core/wpbc-core.php`), which handles serialization and direct raw SQL interaction via the global `$wpdb` object.

### 2. Admin Controls: Visualization and Grouping (Dashboard/Job View)

To allow the administrator to see linked parallel bookings in the dashboard/job view, the plugin's core data querying and rendering engines must be extended.

*   **Query Modification:** The Booking Listing and Timeline pages retrieve data using the data engine functions in `core/admin/wpbc-sql.php`, specifically leveraging **`wpbc_get_sql_for_booking_listing()`**. The most viable extension is to use the existing **"numerous filters"** (e.g., `get_bklist_sql_keyword`) provided within this function to inject a custom condition into the SQL query. This condition must search the serialized data within the `booking_options` column for the Job ID, allowing all linked bookings to be retrieved together.
*   **Timeline Rendering:** The administrative Timeline View is rendered by the **`WPBC_TimelineFlex`** class. After the data is retrieved (via `wpbc_get_bookings_objects()`), the class’s logic (specifically the complex processing method **`wpbc_get_dates_and_times_for_timeline()`**) must be enhanced to recognize the Job ID and apply specific visual styling (e.g., grouping, stacking, or a shared visual marker) to show the two co-occurring events as part of a single job.
*   **Admin UI Injection:** Custom elements (like a "View Job" link using the Job ID) can be injected into the administrative listing using hooks associated with the toolbar factory (**`core/admin/wpbc-toolbars.php`**).

### 3. Admin Controls: Independent Action Logic (Reschedule/Cancel)

The ability to reschedule or cancel a parallel booking without affecting the main booking requires overriding the default single-ID action workflow.

*   **AJAX Interception:** Administrative actions (approving, deleting, trashing) are processed via AJAX in **`core/lib/wpbc-ajax.php`**. Specific functions like `wpbc_ajax_DELETE_APPROVE()` and `wpbc_ajax_UPDATE_APPROVE()` must be modified or hooked into immediately before they execute their database operation.
*   **Relational Logic Check:** Before executing the status change (which uses direct prepared `$wpdb` queries), the logic must check the target booking ID's **`booking_options`** metadata using **`wpbc_get_booking_meta_option()`** to retrieve the Job ID.
*   **Execution:** If a Job ID exists, the system must confirm that the database update (cancellation, status change, date update) is strictly limited to the **single `booking_id`** provided in the initial AJAX request. This prevents the default workflow from cascading the action to all other bookings associated with the shared resource/time, thereby achieving independent control.
*   **Notification Management:** After the status change, core email functions in `core/wpbc-emails.php` are triggered. The email logic must be filtered to ensure that a cancellation email is only sent for the service that was actually modified, not the parallel service that remains active.

### 4. Admin Controls: Linked Reporting and Export

To treat parallel bookings as distinct but linked entities in reports and exports, the system needs an aggregation mechanism.

*   **Data Aggregation Function:** A custom function must be created to aggregate data by Job ID. This function would rely on querying the serialized **`booking_options`** column to find all linked bookings (a noted risk due to poor query performance) and combining their details.
*   **Developer API Leverage:** The Developer API (**`core/wpbc-dev-api.php`**) provides high-level functions like **`wpbc_api_get_booking_by_id()`**, which is critical because it automatically **unserializes** the custom form data into a readable array. The custom aggregation function would utilize this to pull data from all linked records.
*   **Export Integration:** Since advanced export features are delegated to companion plugins (like "Booking Manager") via hooks such as **`do_action( 'wpbm_ics_import_start', ... )`**, the custom aggregation function must be hooked into the data retrieval pipeline *before* the data is passed to the export delegate, ensuring the reports receive the unified, itemized job view.



### 6. Notifications

* [ ] Notifications (email/SMS) include both main and parallel bookings.
* [ ] Rescheduling prompts system to **check conflicts** with parallel bookings.
* [ ] Clients and staff receive clear details on both bookings.

Based on the sources and our previous conversation, **the specific notification features required for complex "parallel bookings" are not implemented.** The current notification system is designed to handle communications for **a single, individual booking record**, not two separate, linked sessions.

Here is a breakdown of the specific requirements against the plugin's documented architecture:

### 1. Notifications (email/SMS) include both main and parallel bookings.

The sources confirm a robust Email API structure, but they do not provide evidence that this system can aggregate data from multiple, linked booking records (which we established would be necessary for parallel services).

*   **Email Structure:** The email system uses a consistent, object-oriented pattern where each email type (Approved, Deleted, New Admin, etc.) is handled by a dedicated class (e.g., `WPBC_Emails_API_Approved`).
*   **Dynamic Content:** Emails rely on a **shortcode replacement engine** to dynamically populate content using placeholders like `[booking_id]`, `[dates]`, and `[resource_title]`.
*   **Limitation:** The replacement engine uses the filter **`wpbc_replace_params_for_booking`** to generate dynamic content. If the core logic only sends an email per booking ID, the standard shortcodes will only retrieve data relevant to that **single booking record**. The plugin does not document any shortcode or logic designed to query the database for a linked secondary booking record (via a hypothetical "Job ID" stored in serialized metadata) and combine its details with the primary booking in one message.

### 2. Rescheduling prompts system to check conflicts with parallel bookings.

The architectural focus on scheduling and availability is centered on **conflict prevention** for a single requested action, lacking the necessary conditional logic to manage multiple linked bookings simultaneously.

*   **Availability Check:** The Developer API provides the function **`wpbc_api_is_dates_booked()`**, which checks if a given set of dates/times is available for a resource by calling the internal booking engine. This function is intended for checking availability for *one transaction*.
*   **Core Workflow:** Rescheduling or status changes in the admin panel are handled via **AJAX requests** that target a **single booking ID**. The logic in the AJAX handler (e.g., `core/lib/wpbc-ajax.php`) executes **direct, prepared $wpdb queries** to modify the status of that single record.
*   **Missing Parallel Logic:** Since the system is designed to act upon a single ID and its core purpose is generally to **prevent** conflicts, there is no documented logic within the AJAX handlers or the core date engine (`core/wpbc-dates.php`) that would force a simultaneous availability check against the linked parallel booking or conditionally enforce conflict rules based on resource type during a reschedule action.

### 3. Clients and staff receive clear details on both bookings.

As established above, the architecture treats each booking as a distinct entity for communication purposes, meaning the display of combined, clear details is not supported by default.

*   **Staff Notifications:** The administrator notification email (managed by `core/admin/page-email-new-admin.php`) is designed to streamline workflow for a single new booking. It includes unique, single-booking management shortcodes such as **`[moderatelink]`**, **`[click2approve]`**, and **`[click2decline]`**. This focus on single-booking action confirms the notification is geared toward a single ID, not a linked job consisting of two records.
*   **Client Notifications:** Emails to the visitor use dynamic shortcodes. The system includes robust **localization features** for date and time display via functions like **`wpbc_datetime_localized()`**, ensuring accurate timezones and formats for the customer. However, this accuracy applies only to the dates/times retrieved for the single booking record associated with the email being sent.

In summary, the plugin provides a strong foundation for managing and communicating status changes for single bookings, but the specialized features required for **aggregating data, checking dependent conflicts, and displaying details for two separate, linked parallel bookings** are not documented.

Based on the sources and the detailed architectural analysis of the plugin's notification, scheduling, and data handling systems, the specific features related to handling and coordinating notifications for linked "parallel bookings" are **not implemented.**

I would mark the implementation level as **2 out of 10**.

This score is consistent with the low implementation rating for the related features (such as configuration and administrative controls), reflecting that the plugin's architecture is strongly optimized for handling and communicating changes for **independent, single booking records**, not aggregated, linked jobs.

Here is a breakdown of the requirements against the plugin's documented architecture:

### 1. Notifications (email/SMS) include both main and parallel bookings.

This feature requires the email system to perform a complex query to retrieve data from two separate database records (the primary booking and the linked parallel service) and combine that information into one notification.

*   **Email Architecture:** The plugin uses a robust, object-oriented Email API structure, where each email type (Approved, New Admin, etc.) is managed by a dedicated class (e.g., `WPBC_Emails_API_Approved`, `WPBC_Emails_API_NewAdmin`).
*   **Content Limitation:** Email content is generated using a **shortcode replacement engine**. The filter **`wpbc_replace_params_for_booking`** is the primary hook for injecting custom content. However, the standard shortcodes listed by `wpbc_get_email_help_shortcodes()` are designed to retrieve data relevant only to the **single booking ID** currently being processed. There is no documented internal mechanism that queries the database for a linked secondary booking ID (via metadata) and aggregates its details (times, services) into the primary booking's confirmation message.

### 2. Rescheduling prompts system to check conflicts with parallel bookings.

This requires the administrative action workflow to recognize the relationship between bookings and enforce conditional availability rules during an update.

*   **Action Workflow:** Administrative actions like cancelling or rescheduling are handled dynamically via the central AJAX controller (`core/lib/wpbc-ajax.php`). These handlers execute **direct, prepared $wpdb queries** to update the status of the **single booking ID** sent in the request.
*   **Missing Relational Check:** The sources confirm the existence of scheduling engine functions (like the Developer API's `wpbc_api_is_dates_booked()`) but do not indicate that the AJAX status update workflow includes logic to:
    1.  Check the booking's metadata for a linked parallel session ID.
    2.  If found, run a **simultaneous conflict check** using scheduling rules (e.g., ensuring a shared resource remains available for the linked parallel booking).
*   The current system focuses on **single-transaction integrity**, not multi-transaction dependency checking.

### 3. Clients and staff receive clear details on both bookings.

This is fundamentally constrained by the single-record nature of the notification system (Feature 1).

*   **Staff Workflow:** The administrator notification (`WPBC_Emails_API_NewAdmin`) uses admin-specific shortcodes like **`[moderatelink]`**, **`[click2approve]`**, and **`[click2decline]`**. These links are tied directly to the single `booking_id`, reinforcing that the notification and subsequent management workflow treat the booking as an isolated entity.
*   **Client Localization:** While the plugin uses sophisticated localization logic in `core/wpbc_functions_dates.php` (including `wpbc_datetime_localized()`) to ensure dates and times are correctly formatted with timezones and locale, this formatting only applies to the data retrieved from the single booking record associated with the email being sent. The capacity to retrieve and present combined details from two separate records is absent.

This implementation requires deep modification of the plugin's notification framework and its sensitive AJAX processing layer. The core challenge is making a system designed to manage and communicate based on a **single booking ID** recognize and aggregate data from **two separate, linked records** (primary and parallel).

Here is a high-level overview of the implementation, drawing on the observed architectural patterns:

### 1. Notification Content Enhancement: Listing Both Sessions

To include details for both the primary and parallel bookings in a single notification, the plugin’s dedicated Email API and shortcode engine must be extended.

*   **Custom Shortcode Implementation:** The Email API uses dynamic content generated via shortcodes. A new, custom shortcode (e.g., `[parallel_job_details]`) must be registered by hooking into the **`wpbc_replace_params_for_booking`** filter. This filter provides the official method for injecting custom shortcode logic into all email templates (e.g., those defined by classes like `WPBC_Emails_API_NewAdmin` or `WPBC_Emails_API_Approved`).
*   **Data Aggregation Logic:** The custom shortcode function would execute the following steps:
    1.  Retrieve the current booking ID being processed by the email function.
    2.  Use the established booking meta function, **`wpbc_get_booking_meta_option()`**, to safely retrieve the **serialized Job ID** from the current booking record.
    3.  Perform a **custom database query** (using the global `$wpdb` object, as seen in `core/wpbc-core.php`) to find all other booking records that share this Job ID in their `booking_options` column.
    4.  Format the retrieved data for both the primary and parallel sessions. This step must utilize the plugin's specialized localization function, **`wpbc_datetime_localized()`** (in `core/wpbc_functions_dates.php`), to ensure the times and dates are accurately translated and respect the site's configured timezone offset and locale.
    5.  Return the combined, formatted text string for insertion into the email template.
*   **Administrative Messages:** Helper functions in **`core/wpbc_functions.php`** (like `wpbc_show_message`) or the dynamic notice system in **`core/wpbc-debug.php`** (via `wpbc_admin_show_top_notice`) can be used to display clear administrative feedback detailing both sessions after creation or status change.

### 2. Relational Logic: Reschedule Conflict Checking

Rescheduling or modifying a primary booking must trigger a conflict check against its linked parallel session. This requires intervening in the highly secure AJAX workflow.

*   **AJAX Interception:** All sensitive status and data updates (including reschedule requests) are handled dynamically via the central AJAX router, **`core/lib/wpbc-ajax.php`**. This file enforces security using **nonce verification** and executes direct `$wpdb` queries.
*   **Pre-Execution Hook:** A filter (e.g., a hook fired before status is updated, related to actions like `wpbc_ajax_UPDATE_APPROVE()`) must be used to intercept the reschedule parameters (new dates/times) before the core database update occurs. The filter **`wpbc_ajax_action_list`** could potentially be leveraged if the core action logic is cleanly separated, allowing developers to inject validation before the update proceeds.
*   **Conditional Availability Check:** The injected logic would perform the following dependency check:
    1.  Retrieve the Job ID from the booking being rescheduled (via serialized metadata lookup).
    2.  Determine the **resource ID** and **proposed new dates/times** for the primary booking.
    3.  Call the Developer API function **`wpbc_api_is_dates_booked()`** to check availability for the parallel booking's resource, ensuring the proposed reschedule does not conflict with the *parallel* session's scheduling rules (e.g., capacity limits or exclusivity constraints).
    4.  If a conflict is detected, the process must be halted, and an error message must be returned to the client-side JavaScript, which updates the UI (similar to how `core/lib/wpbc-ajax.php` echoes script blocks back to the browser for UI updates).

### 3. Event-Driven Workflow: Post-Action Notifications

The implementation must ensure that once the status of either the primary or parallel booking changes, the corresponding staff and client notifications are accurate and relate to the correct linked job.

*   **Core Event Hook:** The Developer API exposes numerous action hooks that fire when a booking status changes (e.g., **`wpbc_track_new_booking`**, **`wpbc_booking_approved`**). These hooks are triggered by the internal save and status functions.
*   **Conditional Notification:** Functions hooked to these post-status actions would check the `booking_options` field for the "Job ID."
    *   If a **cancellation** or **deletion** occurs, the system should check if the administrator intended to cancel *only* the single record provided in the AJAX request, or the entire linked job. If only one is canceled, the notification must clearly reference the shared Job ID but confirm that the other parallel session remains active.
    *   The email sending process, managed by wrapper function **`wpbc_wp_mail()`** (in `core/wpbc-emails.php`), must be utilized to dispatch the customized, single-record notifications.
---

## **Success Criteria**

✅ Admin can configure which services support parallel booking.
✅ Client can add a parallel service during or after booking the main job.
✅ Parallel bookings are stored as linked but distinct records.
✅ Scheduling rules prevent conflicts when services share resources.
✅ Calendar view displays overlapping bookings clearly.
✅ Confirmation and reminder messages list both main and parallel bookings.
✅ Payment/invoices show both services accurately.
✅ Admin can manage parallel bookings independently (reschedule, cancel, report).
✅ Works for different scenarios:

* Consultation during job.
* Overlapping jobs by different staff.
* Add-on services (e.g., inspection, delivery).

---

Would you like me to also **map possible technical approaches** (e.g., custom booking table with a `parent_job_id` and `booking_type` column, or extending WP post relationships) so you have developer-facing guidance too?

