Perfect — let’s break down **“Ability to split a job into multiple bookings”** into testable requirements and success criteria so you’ll know exactly what to look for in the plugin implementation.

---

## **Requirements Breakdown**

### 1. Job Definition

* [ ] Client (or admin) can define a job that requires **more time than one booking slot allows** (e.g., 8 hours when slots are 4 hours each).
* [ ] Option to select **“split across multiple bookings”** during the booking flow.
* [ ] Job retains a single **Job ID** even when split into multiple booking entries.

Based on the sources, the functionality required for **Job Definition**—where a single booking encompasses multiple, potentially non-consecutive time slots—is **highly supported** by the plugin's data architecture and advanced time processing mechanisms.

I would mark the implementation of this feature as an **8 out of 10**.

The score reflects that while the core requirements for data integrity and time calculation are met, the specific front-end workflow for automatically prompting the client to select multiple split slots for a single long-duration job is not explicitly documented within the plugin's configuration or form parsing architecture.

---

### Implementation Breakdown

#### 1. Client (or admin) can define a job that requires more time than one booking slot allows.

**Status: Architecturally Supported**

The plugin is designed to handle complex booking durations and granular time slots, which is a prerequisite for defining a long job that exceeds standard single-slot limitations.

*   **Granular Time Processing:** The core logic engine for data visualization, **WPBC\_TimelineFlex** (analyzed in `wpbc-class-timeline_v2.php`), processes raw booking data and transforms it into structured arrays for rendering the timeline. This transformation logic is highly sophisticated, utilizing the "last digit of the time's 'seconds' component (:01 for check-in, :02 for check-out)" to correctly process partial-day bookings and changeovers. This proves the plugin's capacity to handle and define time slots at a granular level necessary for long-duration jobs.
*   **Availability Rules:** The plugin's **General Settings** define **Availability rules**, providing the administrative tools needed to configure the minimum and maximum duration a job can occupy.

#### 2. Job retains a single **Job ID** even when split into multiple booking entries.

**Status: Fully Implemented and Core to Architecture**

The plugin's custom database structure is designed precisely to associate multiple dates, times, and slots with a single booking record, effectively granting a single "Job ID" for a complex reservation.

*   **Database Structure:** The system stores complex bookings across its custom database tables (`{$wpdb->prefix}booking` and `{$wpdb->prefix}bookingdates`). The booking record itself acts as the single **Job ID**.
*   **Developer API Support:** The Developer API function **`wpbc_api_booking_add_new()`** is designed to accept an array of dates (`$booking_dates`) associated with a single reservation, confirming that the system tracks a single booking ID that may cover multiple, disparate date/time selections.

#### 3. Option to select **“split across multiple bookings”** during the booking flow.

**Status: Functionally Unconfirmed in UI Workflow**

While the technical capacity to handle the data is present, the sources do not explicitly confirm the existence of the specific administrative configuration setting or the resulting front-end UI flow required to automate this process for the client.

*   **Missing Workflow Evidence:** The source documents confirm the plugin has **extensive client-side jQuery logic** for managing a **dynamic UI** (e.g., showing or hiding certain settings based on user input). This capability suggests a step-by-step or wizard-like flow is technically possible. However, there is no explicit mention in the settings definitions or form parsing logic (`core/form_parser.php`) that dictates the workflow for a long-duration job to be automatically recognized and split into selectable time blocks by the system before presentation to the client.

In summary, the plugin's backend is capable of storing and processing the complex data structure of a split job under a single ID, but the explicit front-end logic for calculating the split and presenting the selection workflow to the user is not detailed in the sources.


### 2. Scheduling Logic

* [ ] System suggests the **next available slots** automatically (e.g., two consecutive days or times).
* [ ] Client can manually select different days/times for split sessions.
* [ ] Minimum/maximum time per split segment configurable (e.g., cannot split into less than 2 hours).
* [ ] Ensure system prevents **overlap or double-booking** of resources/tradesperson.

The **Scheduling Logic** feature set is **highly implemented** within the plugin's architecture, leveraging core engine components for robust availability calculation, configuration of constraints, and conflict prevention.

On a scale of 1 to 10, I would mark the implementation of this feature set as a **9 out of 10**.

The core logic for conflict checking and configuring constraints is fully supported, with high architectural confidence in the ability to handle split sessions. The sole uncertainty lies in the existence of a specific, complex algorithm that proactively *suggests* the next best available slots, rather than simply displaying open dates.

---

### Implementation Breakdown

#### 1. Ensure system prevents overlap or double-booking of resources/tradesperson.

**Status: Fully Implemented (Core Logic)**

Conflict prevention is a fundamental, fully supported feature handled by the plugin's central data engine:

*   **Database Queries:** The `core/wpbc-dates.php` file contains the primary "dates engine" and provides functions like `wpbc__sql__get_booked_dates()`, which fetches already booked dates by querying the custom database tables. This raw data is used to compute availability and detect overlaps.
*   **Consistency Check:** The Developer API (`core/wpbc-dev-api.php`) exposes `wpbc_api_is_dates_booked()`. This function ensures that any programmatic or client-side check for availability uses the same core conflict validation logic as the main booking process.
*   **Legacy Evidence:** Even the obsolete code for date checking (`core/lib/wpbc-booking-new.php`) contained a complex function (`wpbc_check_dates_intersections()`) for detecting conflicts, proving that conflict prevention is a long-standing, central architectural concern, now handled by modern capacity checking logic.

#### 2. Minimum/maximum time per split segment configurable.

**Status: Fully Implemented (Configuration and Validation)**

Constraints on booking duration are configurable through the administrative settings and enforced by core utility functions:

*   **Configuration:** The **General Settings** defined by the `WPBC_Settings_API_General` class (`core/admin/api-settings.php`) explicitly control **Availability rules**. This is the layer where administrators define minimum and maximum booking durations or time limits.
*   **Validation:** The primary dates engine (`core/wpbc-dates.php`) provides utility functions like `wpbc_check_min_max_available_times()` for validating, extracting, and converting booking times. This mechanism ensures that user input adheres to the administrative constraints.

#### 3. Client can manually select different days/times for split sessions.

**Status: Highly Implemented (Granular Time and Data Support)**

The system supports the precise time management required to define split segments:

*   **Granular Time:** The **WPBC\_TimelineFlex** class—the engine for the administrative visualization—is architecturally capable of managing precise time slots. Its data processing logic handles check-in/check-out times down to the "seconds" component of the time field (specifically, **:01 for check-in** and **:02 for check-out**) to correctly process partial-day bookings. This proves the system can define and manage discrete time segments.
*   **Multi-Date Storage:** The system is fundamentally designed to allow a single booking record (Job ID) to be associated with an array of disparate dates and times, meeting the data persistence requirement for split sessions (as confirmed in prior conversation regarding Job Definition).

#### 4. System suggests the next available slots automatically (e.g., two consecutive days or times).

**Status: Architecturally Supported (Functionality Unconfirmed)**

The system possesses the data necessary to perform this function, but the specific **automated suggestion algorithm** in the front-end UI is not explicitly confirmed:

*   **Dynamic Availability:** The system is fully equipped to calculate and display which dates are available in real-time by integrating data from the internal conflict checks and potentially external sources (e.g., Google Calendar synchronization, which blocks off dates based on imported events).
*   **Client-Side Interactivity:** The plugin relies on client-side JavaScript (`core/wpbc-js.php`) for the entire user booking process, including calendar selection and AJAX-based submission. While this foundation supports complex dynamic UI responses, the existence of an algorithm that looks ahead to proactively *suggest* the next block of 4 hours, for instance, is not detailed in the files analyzed.


### 3. Data Structure & Storage

* [ ] Each sub-booking is stored as a **linked record** under the parent job.
* [ ] Booking records include reference to parent job ID.
* [ ] Admin can view both:

  * Full job overview (total hours, all sessions).
  * Individual booking details.

Yes, the **Data Structure & Storage** feature, which supports the storage and retrieval of complex, multi-slot bookings linked under a single job ID, is **fully implemented** and forms a core part of the plugin's data architecture.

Based on the architectural analysis of the plugin's data persistence and administrative display systems, I would rate the implementation of this feature as a **10 out of 10**.

### Implementation Breakdown

#### 1. Each sub-booking is stored as a linked record under the parent job, and Booking records include reference to parent job ID.

The plugin's custom database model is designed to handle this exact requirement, where a single booking (the "parent job") can span multiple dates and times ("sub-bookings").

*   **Single Job ID:** The core architecture stores all information related to a single job under one booking ID in the custom database table, `{$wpdb->prefix}booking`. This booking record serves as the central **Job ID**.
*   **Linked Records (Multiple Dates/Slots):** The plugin is fundamentally designed to associate an **array of disparate dates and times** with a single booking ID. The database structure supports retrieving all booking dates (with time) for specific bookings using functions like `wpbc_db__get_sql_dates__in_booking__as_str()`.
*   **Granular Time Support:** The administrative **Timeline View** engine (**WPBC\_TimelineFlex**) processes raw booking data and is specifically designed to handle and display complex time slots, including check-in and check-out times, showing that the system accurately records individual time segments within a larger booking.

#### 2. Admin can view both: Full job overview (total hours, all sessions) and Individual booking details.

The administrative interface and its data engine are specifically built to retrieve and visualize this complex, linked data.

*   **Data Retrieval:** The data engine for the administrative booking listing and timeline views (`core/admin/wpbc-sql.php`) is responsible for querying and processing booking data. The function `wpbc_get_booking_listing()` executes the necessary queries and processes raw results.
*   **Retrieving Custom Details (e.g., Service IDs, Custom Descriptions):** Custom field data, which would include any service selection or custom request description related to the job, is stored as **booking meta options** in a **serialized array** within the **`booking_options` column** of the `{$wpdb->prefix}booking` table. The Developer API function `wpbc_api_get_booking_by_id()` retrieves the booking record and **unserializes** this form data into a readable array.
*   **Visualizing All Sessions (Full Job Overview):** The **Timeline View** (`core/admin/page-timeline.php`) acts as the controller that sets up filters, but delegates the heavy lifting of querying and **rendering the complex, chronological visualization** to the **WPBC\_TimelineFlex** class. This view allows the administrator to visualize all related time slots (sessions) of a job in a Gantt-chart style.
*   **Viewing Individual Booking Details:** The API allows for the direct retrieval of all data for a specific booking ID. Workflow functions manage and display the booking status (approve, pending, cancel) in the admin UI.


### 4. User Experience

* [ ] Booking form clearly explains when a job is being split (e.g., “Your 8-hour job will be booked as 2 sessions of 4 hours each”).
* [ ] Calendar view shows split bookings linked together visually (e.g., grouped or color-coded).
* [ ] Confirmation email/SMS lists **all scheduled segments** with dates/times.

Based on the sources and our conversation history, the **User Experience** criteria for handling split jobs are **partially implemented** in terms of data processing and email shortcodes, but the specific, automated **front-end notification/explanation** is not confirmed.

I would rate the implementation of this feature set as a **7 out of 10**.

The core architecture supports the complex data required for this UX, but the explicit instructional content is not guaranteed by the analyzed UI components.

---

### Implementation Breakdown

| Acceptance Criterion | Status & Supporting Evidence |
| :--- | :--- |
| **Booking form clearly explains when a job is being split.** | **Functionally Unconfirmed (Uncertainty in Automated UI)** |
| **Calendar view shows split bookings linked together visually.** | **Highly Implemented (Admin View Confirmed; Front-end Implied)** |
| **Confirmation email/SMS lists all scheduled segments.** | **Highly Implemented (Email Confirmed; SMS Unconfirmed)** |

### Detailed Analysis

#### 1. Booking form clearly explains when a job is being split.

The plugin provides robust mechanisms for displaying text, labels, and dynamic content, but the existence of a dedicated, automated message triggered by the job splitting logic is not confirmed:

*   **Custom Text/Labels:** The UI construction is managed by helper functions (e.g., `wpbc_bs_input_group()` in `core/any/admin-bs-ui.php`) which support labels and hints. Furthermore, the translation engine (`core/wpbc-translation.php`) supports the use of custom inline translation shortcodes (`[lang=xx_XX]...[/lang]`) within administrative text fields, meaning custom explanations can be added manually.
*   **Dynamic UI:** The core settings file (`core/admin/api-settings.php`) contains **"extensive client-side jQuery logic"** to manage a dynamic UI, where the visibility of settings depends on the values of others. This capability confirms that the system can display or hide messages based on user interaction (such as selecting a long duration job), but the source does not detail the specific logic that auto-calculates the split and generates the informational message ("Your 8-hour job will be booked as 2 sessions of 4 hours").

#### 2. Calendar view shows split bookings linked together visually.

The advanced data processing required for displaying linked, multi-slot bookings is confirmed for the administrative view, with the front-end shortcode output likely leveraging the same logic:

*   **Admin Timeline Visualization:** The administrative **Timeline View** is explicitly designed to handle this visual complexity. The core engine, **WPBC\_TimelineFlex**, processes raw booking data to render a Gantt-chart-like visualization. This engine uses **sophisticated logic** (checking the "seconds" component of the time field: `:01` for check-in, `:02` for check-out) to correctly display partial-day bookings and changeovers. This proves the plugin's ability to visualize individual segments (sub-bookings) under a larger Job ID.
*   **Front-End Use:** The same `WPBC_TimelineFlex` class handles the rendering for the front-end **`[bookingtimeline]` shortcode**. It is highly likely this front-end rendering uses the same CSS (which relies on Flexbox and CSS Custom Properties for theming) and logic to visually link or style the separate segments of a single job.

#### 3. Confirmation email/SMS lists all scheduled segments with dates/times.

The email system is fully capable of providing detailed date and time information for all segments:

*   **Email Confirmation (Confirmed):** The plugin's **Email API** handles notifications through dedicated classes (e.g., `WPBC_Emails_API_Approved`). The core shortcode generation function, `wpbc_get_email_help_shortcodes()`, dynamically lists available shortcodes for email templates, including `[dates]` and `[resource_title]`. The date engine (`core/wpbc_functions_dates.php`) provides helper functions like **`wpbc_get_redable_dates()`** and **`wpbc_get_redable_times()`** to convert arrays of individual dates and times into human-readable condensed ranges or lists. This confirms the system can compile and insert all necessary segment details into the confirmation email.
*   **SMS Confirmation (Unconfirmed):** As established in prior conversations, the source documents provide extensive detail on the Email API but **do not contain evidence** of native SMS functionality.



### 5. Payment & Invoicing

* [ ] Payment handled **once per job**, not separately for each booking, unless configured otherwise.
* [ ] Invoices/quotes reflect **total job cost** and note the split booking schedule.

Based on the sources analyzed, the specific features related to **Payment & Invoicing** are **not explicitly confirmed as implemented** within the core architecture reviewed, particularly the complex payment and invoicing workflow required for split jobs.

I would mark the implementation of this feature set as a **2 out of 10**.

This low score reflects the comprehensive framework for emails and settings, which *could* support quotes/invoicing manually, but the complete absence of integrated payment processing or defined invoicing mechanisms in the analyzed files.

### Implementation Breakdown

| Acceptance Criterion | Implementation Status | Supporting Evidence |
| :--- | :--- | :--- |
| **Payment handled once per job (total job cost).** | **Not Implemented (No payment logic found).** | The sources detail the AJAX handler (`core/lib/wpbc-ajax.php`) which supports front-end **cost calculation**, but there is **no evidence** in the analyzed files of a dedicated payment gateway integration, transaction handling, or a mechanism to trigger payment processing after booking submission. |
| **Invoices/quotes reflect total job cost.** | **Partially Implemented (Cost data available in email system).** | The plugin can calculate and retrieve cost data, and the Email API can display this information. The utility file `core/wpbc_functions.php` includes the function **`wpbc_get_cost_with_currency_for_user`** which dynamically formats costs and currency symbols. This function could be used to display the total job cost in an email or quote template. |
| **Invoices/quotes note the split booking schedule.** | **Highly Implemented (Date data available in email system).** | The Email API is capable of listing all scheduled segments. The core email utility (`core/wpbc-emails.php`) confirms that the function **`wpbc_get_email_help_shortcodes()`** dynamically generates a list of available shortcodes, including **`[dates]`** and **`[resource_title]`**. The date localization engine (`core/wpbc_functions_dates.php`) provides helper functions like **`wpbc_get_redable_dates()`** to convert arrays of individual dates/times (i.e., the split schedule) into human-readable strings. |

### Conclusion on Payment Logic

While the plugin is highly capable of calculating costs, storing split job data (under a single Job ID), and printing all necessary cost and schedule details in a confirmation email, the essential components required for automatic payment processing or automated invoice generation are missing from the files analyzed.

*   The AJAX controller registers dynamic actions for **cost calculation** but only for booking management (e.g., approve, delete).
*   The system includes a **Developer API** (`core/wpbc-dev-api.php`) with hooks like `wpbc_track_new_booking` and `wpbc_booking_approved`. These hooks could be used by an **external companion plugin** to trigger payment or invoicing actions, but this functionality is not native to the core plugin files reviewed.

Therefore, the invoicing *content* (cost and schedule) is supported, but the *process* of automatic payment handling and formal invoice document generation is not confirmed.

The implementation of **Payment & Invoicing** requires integrating external payment processing logic into the existing booking workflow and utilizing the plugin's highly robust Email API for document generation. This feature must handle the complexities of referencing a single total cost across multiple stored booking segments (the "Job ID").

Here is a high-level overview of the implementation, leveraging the core architectural components:

### 1. Data Calculation and Persistence Layer

The system must ensure that the total cost for the entire split job and the payment status are stored atomically under the single Job ID.

*   **Total Cost Retrieval:** The system already supports real-time cost calculation via AJAX. During final submission, the total calculated job cost will be retrieved using functions designed for cost and currency formatting, such as `wpbc_get_cost_with_currency_for_user`.
*   **Data Linking:** The payment status (e.g., "Paid," "Pending") and the final total cost will be stored as **booking meta options** using the `wpbc_save_booking_meta_option()` function. This data is efficiently stored as a **serialized array** in the **`booking_options` column** of the custom booking database table.

### 2. Payment Workflow Integration (AJAX and Hooks)

Since the plugin lacks native payment gateway functionality, the workflow must be extended using hooks to initiate external payment processing immediately upon booking creation.

*   **Payment Trigger:** Upon a successful booking submission, the system should hook into a core event, such as `wpbc_track_new_booking` or `wpbc_booking_inserted`.
*   **Action:** The function hooked into this action would retrieve the Job ID and the total cost (from Step 1) and either:
    1.  **Redirect the user** to the specific payment gateway endpoint for processing the total amount.
    2.  **Trigger a custom AJAX action** (registered via the `wpbc_ajax_action_list` filter in `core/lib/wpbc-ajax.php`) to communicate the payment data securely to a custom payment handler.
*   **Security:** Any payment action involving sensitive data must strictly adhere to the plugin’s standard security measure of **Nonce verification**.

### 3. Invoicing and Quote Generation (Email API Extension)

The plugin's robust Email API framework is the ideal tool for generating and delivering documentation that reflects the split booking schedule and total cost.

*   **Template Definition:** A new class could be created, extending the abstract **`WPBC_Emails_API`**, to define a dedicated invoice or quote template.
*   **Content Generation:** The existing **Shortcode Replacement Engine** would be used to populate the document:
    *   The total job cost is inserted using the formatted cost data.
    *   The **split booking schedule** is detailed by utilizing core shortcodes like `[dates]` and helper functions like `wpbc_get_redable_dates()`, which convert the array of booking segments into a human-readable list of sessions.
*   **Delivery:** The email is sent using the highly reliable `wpbc_wp_mail()` wrapper, ensuring correct header formatting and translation integration.

### 4. Administrative View Update

The administrative dashboard must clearly display the total job cost and the payment status.

*   **Status Display:** The administrative booking listing query logic needs to be updated. When retrieving booking details using the Developer API function `wpbc_api_get_booking_by_id()`, the stored payment status (from the unserialized `booking_options` array) would be displayed prominently alongside the single Job ID.
*   **Payment Update Hooks:** After the payment gateway confirms successful payment, an external mechanism must call a plugin function (likely via the Developer API) to update the status in the database using the same `wpbc_save_booking_meta_option()` function.

### 6. Notifications & Rescheduling

* [ ] Notifications (email/SMS) include all scheduled split bookings.
* [ ] If one booking is rescheduled, system prompts whether to adjust other linked bookings.
* [ ] Admin can cancel or edit individual sessions without losing parent job record.

The **Notifications & Rescheduling** feature set for split jobs is **highly implemented** within the plugin's architecture, leveraging its robust Email API and strong data structure for managing complex bookings.

On a scale of 1 to 10, where 10 is fully implemented as requested, I would rate this feature set an **8 out of 10**.

This score reflects the full support for comprehensive email notifications and administrative control over individual segments, with the primary deduction being the lack of explicit evidence regarding the complex transactional logic for automatically prompting synchronization across linked segments during a reschedule.

---

### Implementation Breakdown

#### 1. Notifications (email/SMS) include all scheduled split bookings.

**Status: Highly Implemented (via Email); SMS Not Supported**

The plugin’s notification system is fully equipped to retrieve and list every segment associated with a single Job ID in transactional emails.

*   **Email Support:** The core **Email API** framework uses a Shortcode Replacement Engine. The available shortcodes, such as `[dates]` and `[resource_title]`, allow dynamic content insertion. The date localization engine (`core/wpbc_functions_dates.php`) provides helper functions like `wpbc_get_redable_dates()`, which are capable of taking the array of disparate dates associated with a single booking record and converting them into a human-readable list of all scheduled segments.
*   **Email Deliverability:** The plugin uses the wrapper function `wpbc_wp_mail()` to enhance deliverability by fixing the Sender header, ensuring reliability.
*   **SMS:** The source documents provide extensive detail on the object-oriented Email API but **do not contain evidence** of a native SMS notification system.

#### 2. Admin can cancel or edit individual sessions without losing parent job record.

**Status: Highly Implemented**

The system's data model and administrative workflow inherently support the modification of individual time slots (sessions) without deleting the overarching job record (parent booking ID).

*   **Single Job ID:** The core architecture associates multiple sessions, dates, and times with a **single booking ID**. This booking ID serves as the persistent parent job record.
*   **Session Editing:** The **Developer API** (`core/wpbc-dev-api.php`) includes the critical function `wpbc_api_booking_add_new()`, which allows for editing a booking by passing a new array of dates (`$booking_dates`). This means an administrator can programmatically remove a specific session's date/time entry from the array while leaving the main Job ID and other session entries intact.
*   **Admin Workflow:** The email template sent to the administrator includes quick-action shortcodes like `[click2approve]` and `[click2decline]`. These status management features (approve, pending, delete) are handled dynamically via the AJAX controller (`core/lib/wpbc-ajax.php`), allowing administrators to manage the status of individual sessions if presented separately in the UI.

#### 3. If one booking is rescheduled, system prompts whether to adjust other linked bookings.

**Status: Functionally Unconfirmed**

The technical foundation for this feature exists, but the existence of the specific transactional logic for a synchronized rescheduling prompt is not confirmed.

*   **Underlying Data:** The system *knows* which sessions are linked because they share a single booking ID in the database.
*   **Dynamic UI Capability:** The plugin's architecture supports complex dynamic interfaces. The General Settings page uses **"extensive client-side jQuery logic"** to manage a dynamic UI (showing or hiding settings based on others). Furthermore, the AJAX handler frequently returns inline JavaScript to update the UI without a page reload, indicating the technical means to display a prompt exists.
*   **Missing Logic:** However, the sources do not explicitly detail whether the PHP engine or the AJAX handlers contain the complex, cross-session validation and synchronization logic required to: 1) detect that one segment of a split job is being rescheduled, and 2) automatically calculate new proposed times for the *other* linked segments and present that prompt to the administrator/client. This sophisticated, automated synchronization is not documented as a native feature of the core files reviewed.


---

## **Success Criteria**

✅ Client/admin can book a job longer than available slots.
✅ System supports splitting into multiple bookings automatically or manually.
✅ All sub-bookings remain linked to a parent job ID.
✅ Double-booking of tradesperson is prevented.
✅ Clients see all their split bookings in confirmation and reminders.
✅ Payment is processed once for the entire job (unless configured otherwise).
✅ Admin calendar shows linked bookings clearly.
✅ Rescheduling/cancellation workflows support split bookings correctly.
✅ Works smoothly for both short (e.g., 2 sessions) and long (e.g., 5+ sessions) jobs.

---

Do you want me to also add **technical implementation notes** (like whether this should extend WordPress’s `post_parent` relationship or use custom booking tables), so it’s easier to guide a developer?
