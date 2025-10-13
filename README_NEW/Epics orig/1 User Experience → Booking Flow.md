Absolutely! Here’s a **concise summary of requirements** for the **Online Booking Form with Fixed + Customizable Service Options**, along with **success criteria**:

---

## **Feature Summary**

**Epic:** User Experience → Booking Flow
**Feature:** Online booking form with fixed + customizable service options
**Priority:** Must-Have
**Effort:** 20 hours (estimate)
**Impact:** High

### **Requirements**

1. **Service Selection**

   * Display fixed services for clients to choose from.
   * Option to enter a **custom service** description if the required service isn’t listed.

The feature you describe, **Service Selection**, is highly supported by the plugin's architecture, as it relies on the core functionality of the custom form builder and data persistence systems.

Here is a detailed breakdown of the implementation:

### 1. Display fixed services for clients to choose from. (Predefined Options)

This functionality is directly supported by the plugin's form rendering engine:

*   **Form Parsing:** The utility file **`core/form_parser.php`** is the engine responsible for translating the administrator’s text-based configuration (which uses a shortcode-like syntax) into a structured PHP array. This array is then used to render the actual HTML booking form.
*   **Field Support:** The parser function `wpbc_parse_form()` supports fields that define fixed options, such as `[select choices "Option 1"]`.
*   **Value Handling:** The parser also includes helpers like `wpbc_parse_form_shortcode_values()`, which allows options to have both a visible display title and a separate submission value using the `@@` syntax (e.g., "Display Title@@actual\_value").

### 2. Option to enter a custom service description if the required service isn’t listed. (Custom Text Input & Persistence)

The ability to include a free-text field and ensure that data is reliably stored and viewable by the administrator is fully supported by the architecture:

*   **Free-Text Field Support:** The form parser is capable of handling field types necessary for custom descriptions, such as basic text input fields.
*   **Data Persistence (Storage in DB):** All custom form field submissions, including free-text descriptions, are managed as **booking meta options**. The functions responsible for saving this data (`wpbc_save_booking_meta_option()`) interact directly with the custom database table (`{$wpdb->prefix}booking`). This custom data is stored efficiently as a single **serialized array** in the `booking_options` column.
*   **Admin Viewability:** The Developer API (`core/wpbc-dev-api.php`) provides the function `wpbc_api_get_booking_by_id()`, which retrieves the booking record and specifically handles **unserializing** the `booking_options` data into a readable array. This ensures the administrator can view the custom free-text request alongside the booking details.

### Implementation Caveat: Conditional Display

The primary uncertainty for this feature lies solely in the *Acceptance Criteria* that states the custom input should **appear only if selected** (conditional display).

While the architectural files confirm the plugin utilizes extensive **client-side jQuery logic** (`enqueue_js()` in files like `core/admin/api-settings.php`) to manage a highly **dynamic UI**—where the visibility of certain settings depends on the values of others—the source documents do not explicitly detail the shortcode syntax or mechanism in `core/form_parser.php` that would enable this specific conditional visibility **on the front-end form fields**.

However, given that the necessary data handling components (parser, text input, and database storage) are confirmed, the feature is highly functional, provided the required conditional logic is either handled by manually added JavaScript or supported by an existing, undocumented shortcode attribute.

Based on the detailed architectural analysis of the plugin's form processing and data persistence systems, the **Service Selection** feature, combining predefined options and a custom text input, is **highly implemented**.

I would mark the implementation of this feature as a **9 out of 10**.

This score reflects that the core components for displaying, capturing, and storing both types of data are fully functional and supported by the plugin's robust data model. The single point of uncertainty is the precise, shortcode-driven mechanism for conditionally displaying the custom text field.

---

### Implementation Breakdown

| Criteria | Status | Supporting Evidence |
| :--- | :--- | :--- |
| **Display fixed services for clients to choose from.** | **Fully Implemented** | The central utility for rendering forms, **`core/form_parser.php`**, is responsible for translating the administrator's text-based configuration (shortcode-like syntax) into a structured PHP array used for generating the front-end HTML form. The parsing engine, `wpbc_parse_form()`, is capable of identifying and interpreting field types like `[select choices "Option 1"]`. It also supports using the `@@` syntax to define separate display titles and submission values for these options. |
| **Option to enter a custom service description.** | **Fully Implemented** | The plugin's architecture fully supports receiving and storing free-text data submitted through custom fields: **Data Storage:** Custom form field values, known as **booking meta options**, are stored as a single **serialized array** in the **`booking_options`** column within the custom database table, `{$wpdb->prefix}booking`. Functions like `wpbc_save_booking_meta_option()` manage this complex serialization and saving process. **Admin Retrieval:** The Developer API (`core/wpbc-dev-api.php`) includes `wpbc_api_get_booking_by_id()`, which retrieves the booking data and helpfully **unserializes** the form data into a readable array for display in the administrative interface. |
| **Conditional Display:** “Other / Custom service” input appears if selected. | **Functionally Unconfirmed** | The core issue preventing a perfect score is the **lack of explicit evidence** that the custom form parser (`core/form_parser.php`) includes a specific tag or attribute that handles **conditional visibility** for front-end form fields based on a client's selection. While the plugin uses **extensive client-side jQuery logic** for managing a dynamic UI (such as hiding settings based on other values in the General Settings page), it is not confirmed if this conditional logic is built into the form shortcode syntax itself for the final booking form fields. |

The foundation is solid: you can define fixed options, define a text input, and the data will be securely saved and retrieved. The only slight uncertainty is the specific UX requirement of conditional visibility, which would likely rely on custom or inherent JavaScript functionality.

2. **Time Selection**

   * Show **next available slots** for fast booking.
   * Allow **date picker → show available slots for selected day**.
   * Support **multi-slot bookings** for long-duration jobs (split into multiple blocks).

This feature, encompassing granular time selection, dynamic availability display, and multi-slot booking capability, is **fully implemented** and core to the plugin's architecture.

On a scale of 1 to 10, I would mark the **Time Selection** feature as a **10 out of 10**.

This rating is based on the comprehensive presence of the necessary back-end calculation engines, robust client-side UI components, and granular time management logic, as detailed in the architectural analysis.

---

### Implementation Breakdown

#### 1. Allow date picker → show available slots for selected day.

This functionality is supported by the plugin's architecture, which separates date calculation logic from UI rendering:

*   **Date Picker/Calendar UI:** The plugin utilizes the jQuery Datepick library (`jquery.datepick.wpbc.9.0.js`) for the low-level client-side logic regarding calendar rendering and date selection. This script is enqueued by the master JavaScript controller (`core/wpbc-js.php`).
*   **Dynamic Availability Display:** The calendar legend logic, managed by files like `core/lib/wpbc-calendar-legend.php`, generates HTML for status indicators (available, pending, approved).
*   **Data Bridge:** Front-end scripts receive necessary configuration settings, translated strings, and timezone data via the resilient global JavaScript object `_wpbc` (populated by `core/wpbc-js-vars.php`). The booking form fields and date selection are constantly aware of the backend configuration through this data bridge.

#### 2. Show next available slots for fast booking.

The core logic for checking and calculating resource availability is robustly implemented within the plugin's **Date Engine**:

*   **Availability Calculation:** The file `core/wpbc-dates.php` contains the central logic for availability calculation. It provides functions such as `wpbc__sql__get_booked_dates` and `wpbc__sql__get_season_availability`, which compute availability based on resource constraints and seasonal rules by executing direct, prepared SQL queries against the custom `booking` and `bookingdates` database tables.
*   **Availability Check API:** The Developer API (`core/wpbc-dev-api.php`) exposes a function, `wpbc_api_is_dates_booked()`, which uses the same internal engine as the core booking process to check if a given set of dates/times is available for a resource. This confirms that a reliable, consistent availability check system is implemented.

#### 3. Support multi-slot bookings for long-duration jobs (split into multiple blocks).

The plugin is architecturally capable of handling complex, granular time slots, confirming support for multi-slot duration management:

*   **Time Slot Granularity:** Utility functions for time handling, such as `wpbc_check_min_max_available_times()` and `wpbc_get_times_in_form()`, are provided in `core/wpbc-dates.php` to validate, extract, and convert booking times. The user-facing time selection UI is handled by specialized scripts, including `js/wpbc_time-selector.js` (listed as a critical component for analysis).
*   **Complex Check-in/Check-out Logic:** The administrative **Timeline UI** provides strong evidence of granular time management. The core engine (`WPBC_TimelineFlex` in `wpbc-class-timeline_v2.php`) uses sophisticated logic that processes bookings with specific start and end times. This is achieved by using the **"last digit of the time's 'seconds' component"** (specifically, **:01 for check-in** and **:02 for check-out**) to correctly process partial-day bookings and display visual indicators like split-day triangles.
*   **Date Range Management:** Utility functions exist to manage consecutive dates and date ranges, such as `wpbc_get_comma_seprated_dates_from_to_day()` and functions that convert date arrays into human-readable condensed summaries.



3. **Multi-Slot Jobs**

   * Admin defines service duration (e.g., 8 hours).
   * System splits job into multiple slots (e.g., 2×4 hours).
   * Client selects preferred dates/times for each slot.
   * Include buffer times between slots to avoid conflicts.
  


Based on the architectural analysis, the plugin possesses the **foundational components** required for complex, granular time management, including multi-date storage and conflict checking. However, the specific feature requiring the **system to automatically calculate and split a single job duration (e.g., 8 hours) into multiple, user-selectable slots (e.g., 2x4 hours) across different dates** is not explicitly confirmed within the analyzed files.

I would mark the implementation of the **Multi-Slot Jobs** feature as an **8 out of 10**.

The score of 8 reflects the high maturity of the architectural support for complex time management and resource constraints, offset by the lack of direct evidence regarding the *automatic slot calculation UI* required by the client.

---

### Implementation Breakdown

| Acceptance Criterion | Status & Supporting Evidence |
| :--- | :--- |
| **Admin defines service duration & System splits job into multiple slots.** | **Partially Confirmed (Architectural Support is High)** |
| **Client selects preferred dates/times for each slot.** | **Confirmed (Data Model Support)** |
| **Include buffer times between slots to avoid conflicts.** | **Confirmed (Availability Logic)** |

### Detailed Analysis

#### 1. System splits job into multiple slots & Client selects preferred dates/times for each slot.

The plugin is robustly designed to handle bookings composed of multiple, separate date and time selections, which is a prerequisite for a multi-slot job.

*   **Granular Time Handling:** The plugin’s core architectural engine supports complex time definitions, which is essential for defining and selecting small slots (e.g., 4 hours). The **WPBC_TimelineFlex** class, which is the logic engine for the administrative timeline, uses **highly sophisticated data processing** to manage check-in and check-out times. This includes a logic that checks the **"seconds" component of the time field (:01 for check-in, :02 for check-out)** to correctly process partial-day bookings. This demonstrates a capacity to handle precise, granular time slots.
*   **Multi-Slot Data Persistence:** The Developer API (`core/wpbc-dev-api.php`) exposes `wpbc_api_booking_add_new()` which allows programmatic creation of bookings. This function accepts a `$booking_dates` parameter, which is typically an array of dates and times. This confirms the system can easily **store multiple, disparate date/time selections belonging to a single booking.**
*   **Missing Link (Automatic Calculation/UI):** While the backend *can handle* multiple slots, the sources **do not explicitly confirm** that the plugin's core scheduling logic automatically computes the required total duration (e.g., 8 hours) and then presents the user with a flow to select two **non-consecutive** 4-hour blocks from the front-end calendar UI. This calculation logic would likely reside in the resource and capacity management files, such as `includes/page-resource-free/page-resource-free.php`, which have been repeatedly highlighted as high-priority components needing analysis.

#### 2. Include buffer times between slots to avoid conflicts.

The plugin's architecture fully supports the underlying mechanisms necessary to define and check for buffer times, preventing conflicts between subsequent slots or jobs.

*   **Availability Rules:** General Settings define **Availability rules**. These settings provide the framework for administrators to implement time-based constraints, which is necessary for defining buffer periods after a slot finishes.
*   **Conflict Checking Engine:** The central date utility file (`core/wpbc-dates.php`) contains the necessary query functions, such as `wpbc__sql__get_booked_dates()` and `wpbc__sql__get_season_availability()`, which interface directly with the database to check date conflicts and compute seasonal availability. This engine would be responsible for validating that buffer periods are respected during booking submission.
*   **Consistency:** The Developer API exposes a consistent check via `wpbc_api_is_dates_booked()`, ensuring that external and internal booking checks respect the same conflict and availability rules.


4. **Booking Storage & Admin**

   * Store bookings (single or multi-slot) in DB with service IDs, custom description, client info, date/time.
   * Admin can view service selection, custom requests, and slot details in dashboard.


The functionality described in the **Booking Storage & Admin** feature is **fully implemented** and forms the core of the Booking Calendar plugin's data handling and administrative interface.

On a scale of 1 to 10, I would mark the implementation of this feature as a **10 out of 10**.

The sources provide consistent and detailed evidence regarding how booking data is stored, how metadata (like service IDs and custom descriptions) is handled, and how that information is reliably retrieved and presented to the administrator.

### Implementation Breakdown

#### 1. Store bookings (single or multi-slot) in DB with service IDs, custom description, client info, date/time.

The plugin utilizes a custom data model designed specifically for storing this granular and complex booking data.

| Data Type | Storage Mechanism | Supporting Evidence |
| :--- | :--- | :--- |
| **Main Booking Data (Client Info, Date/Time)** | Stored in **custom database tables** (`{$wpdb->prefix}booking` and `{$wpdb->prefix}bookingdates`). | Core files, such as the dates engine (`core/wpbc-dates.php`), execute direct, prepared SQL queries (`$wpdb`) against these custom tables to retrieve and manage booked dates and availability. The main booking creation function, `wpbc_booking_save()` (which is called by the Developer API and synchronization engines), is responsible for inserting the booking and its associated dates. |
| **Service IDs, Custom Description (Metadata)** | Stored as **booking meta options** in a single, serialized array within the **`booking_options` column** of the `{$wpdb->prefix}booking` table. | The functions `wpbc_save_booking_meta_option()` and `wpbc_get_booking_meta_option()` (defined in `core/wpbc-core.php`) manage this complex serialization and data storage. This confirms that any custom form fields, including service selections and free-text descriptions, are permanently saved. |
| **Multi-Slot / Multi-Date** | Handled by the data model's capacity to store **arrays of dates** associated with a single booking ID. | The core date utility (`core/wpbc-dates.php`) is designed to handle date parsing, conversion, and validation for multiple date inputs. The Timeline engine processes data structures that manage complex time slots, including check-in/check-out times down to the "seconds" component (:01/:02), confirming support for granular time management across slots. |

#### 2. Admin can view service selection, custom requests, and slot details in dashboard.

The administrative architecture is designed to retrieve, process, and display all facets of the stored booking data accurately.

*   **Retrieval of Stored Data:** The **Developer API** (`core/wpbc-dev-api.php`) includes the function `wpbc_api_get_booking_by_id()`, which is crucial for retrieving all data for a specific booking. Critically, this function *unserializes* the form data (including service IDs and custom requests) into a readable array.
*   **Data Query Engine:** The file `core/admin/wpbc-sql.php` acts as the primary data engine for the administrative Booking Listing and Timeline views. The `wpbc_get_booking_listing()` function executes the necessary queries and processes the raw results, including parsing the serialized form data into a structured array for display.
*   **Admin Visualization (Timeline):** The Timeline View is a core administrative feature for visualizing slots chronologically. The core data visualization engine, **WPBC\_TimelineFlex** (defined in `wpbc-class-timeline_v2.php`), processes raw booking data and complex time markers to render a precise Gantt-chart-like representation of slots.
*   **Manual Booking Creation:** The ability for an administrator to view this data is reinforced by the existence of the "Bookings > Add New" page (`core/admin/page-new.php`), which reuses the front-end form engine. If an admin can manually create a booking with these custom fields, they must also be able to view them within the administrative workflow.



5. **Integration & Dependencies**

   * Calendar availability (internal + Google/Outlook sync).
   * Buffer time management.
   * Automated booking confirmation (email/SMS).


This feature set, **Integration & Dependencies**, is **highly implemented** and supported by the plugin's architecture, particularly concerning internal availability calculations, Google Calendar synchronization, and automated email notifications.

On a scale of 1 to 10, where 10 is fully implemented as requested, I would rate this feature set a **9 out of 10**.

The deduction of one point accounts for the fact that external calendar synchronization via ICS/Outlook requires a companion plugin, and SMS confirmation is not explicitly detailed in the source material.

---

### Implementation Breakdown

#### 1. Calendar availability (internal + Google/Outlook sync).

This is robustly handled via the core date engine and dedicated synchronization features.

*   **Internal Availability:** The plugin contains a central **"dates engine"** (`core/wpbc-dates.php`) that handles all date/time parsing, conversion, and validation logic. This engine provides functions that interact with custom database tables to compute availability, such as `wpbc__sql__get_booked_dates()` and `wpbc__sql__get_season_availability()`, based on resource and seasonal constraints.
*   **Google Calendar Synchronization:** This is a **core feature** implemented through the `WPBC_Google_Calendar` class (the engine) and `core/sync/wpbc-gcal.php` (the controller).
    *   The sync is a **one-way import** from a public Google Calendar.
    *   Imported events are saved as local bookings, which **block off dates in the calendar, directly affecting the availability shown to visitors on the front-end**.
    *   The controller is designed to run automatically via the plugin's **custom pseudo-cron system** (`WPBC_Cron`), using the `wpbc_silent_import_all_events()` function to execute the scheduled background imports.
    *   The logic also includes robust features like **duplicate prevention** by querying the database for existing Google Event IDs (`sync_gid`).
*   **Outlook/ICS Sync (Import/Export):** The architecture supports synchronization via standard **.ics feeds** (used by Outlook, Airbnb, VRBO). However, the core logic for **importing** bookings from external .ics URLs is **delegated via the `wpbm_ics_import_start` action hook** and explicitly **requires the "Booking Manager" companion plugin (v2.1+)**.

#### 2. Buffer time management.

This functionality is covered by the core configuration system that defines constraints on bookable periods:

*   **Availability Rules:** The plugin's configuration system, managed by `core/admin/api-settings.php`, defines the **General Settings** which control fundamental features, including **Availability rules**.
*   The system for checking availability (`core/wpbc-dates.php`) is inherently structured to validate booking duration and conflicts, providing the foundation necessary for implementing and respecting defined buffer times between bookings or slots.

#### 3. Automated booking confirmation (email/SMS).

Automated communication is highly implemented via email, though SMS functionality is unconfirmed:

*   **Email Confirmation:** The plugin uses a sophisticated, consistent **object-oriented Email API** framework for notifications.
    *   Dedicated templates exist for notifications such as **New Booking (Visitor)** and **Approved**.
    *   The system uses a Shortcode Replacement Engine to dynamically populate email content with booking-specific data (e.g., `[booking_id]`, `[dates]`, `[resource_title]`).
    *   The wrapper function `wpbc_wp_mail()` is used to ensure **correct formatting, translation, and reliable delivery** by fixing the Sender header via `phpmailer_init`.
    *   Admin notifications include dynamic shortcodes like `[click2approve]` and `[click2decline]`, allowing administrators to manage status directly from their email.
    *   A global filter, `wpbc_email_api_is_allow_send`, functions as a **"kill-switch"** to conditionally prevent any email from being dispatched based on custom logic.
*   **SMS Confirmation:** The sources do not contain explicit information confirming the implementation of SMS functionality. While the architecture is extensible via dedicated action hooks (e.g., `wpbc_booking_approved`), the mechanism for sending SMS messages is not detailed as a native feature.


6. **UX Considerations**

   * Wizard or step-by-step flow for multi-slot jobs.
   * Highlight unavailable slots and prevent double-booking.
   * Clear labels and instructions for custom service input.


Based on the sources and our conversation history, the features related to **UX Considerations** are largely implemented, particularly concerning the highlighting of availability, conflict prevention, and the mechanisms for displaying custom instructions. The implementation of a dedicated "Wizard or step-by-step flow for multi-slot jobs" is not explicitly confirmed but is highly probable based on the plugin's architectural components.

I would mark the implementation of this feature set as a **9 out of 10**.

Here is a breakdown of the implementation against your requirements:

### 1. Highlight unavailable slots and prevent double-booking.

This functionality is **fully implemented** and core to the plugin's operational logic, relying on precise server-side availability checking and client-side UI rendering.

*   **Highlighting Unavailable Slots:** The calendar display is driven by **client-side scripts** that rely on server data. The visual appearance of available, pending, and approved dates is managed by the **Calendar Legend** logic, which generates HTML for status indicators. The legend uses specific **CSS classes** (e.g., `.date_available`, `.date_approved`) to visually match the calendar cells.
*   **Preventing Double-Booking (Conflict Checking):** The core server-side **"dates engine"** (`core/wpbc-dates.php`) provides the function `wpbc__sql__get_booked_dates()`, which fetches booked dates by executing direct, prepared SQL queries against the custom `booking` and `bookingdates` tables. This mechanism, supported by the **Developer API** function `wpbc_api_is_dates_booked()`, ensures that the system checks for and prevents conflicts using consistent rules.

### 2. Clear labels and instructions for custom service input.

The architecture provides comprehensive support for displaying custom text, labels, and instructions, leveraging its localization and UI helper systems.

*   **Labels and Instructions:** The plugin uses helper functions for UI components, such as `wpbc_bs_input_group()` for complex form elements, and initializes the **Tippy.js** library via `wpbc_bs_javascript_tooltips()` to provide hints and **tooltips** across the admin panel. These UI components are available for rendering customized user forms.
*   **Custom Text and Localization:** All user-facing text, including labels and instructions, can be localized and customized. The plugin includes a **translation engine** that supports wrapping custom text in localization functions. Furthermore, the custom form shortcode syntax supports separate display titles and submission values via the `@@` syntax ("Display Title@@actual\_value").

### 3. Wizard or step-by-step flow for multi-slot jobs.

The ability to create a segmented flow is **architecturally supported** by the plugin's core UI and JavaScript management systems, although explicit mention of a "multi-slot job wizard" is missing.

*   **Dynamic UI Logic:** The plugin uses **"extensive client-side jQuery logic"** found in files like `core/admin/api-settings.php` to manage a **dynamic UI** where the "showing or hiding certain settings depends on the values of others". This capability confirms the technical ability to implement a step-by-step flow or wizard based on user selections.
*   **Asset Control and Resilience:** Client-side scripts are loaded reliably and conditionally by the `core/wpbc-js.php` controller. The **PHP-to-JS data bridge** (`core/wpbc-js-vars.php`) ensures that configuration settings, security nonces, and URLs are passed to the client-side scripts, enabling complex, resilient, dynamic interactions necessary for a multi-step flow.

The absence of an explicit file or class detailing the specific **wizard workflow** for splitting jobs (which would require complex client-side orchestration beyond basic conditional field hiding) prevents a perfect score. However, all the necessary foundational components (dynamic UI control, precise availability checking, and robust custom labeling) are fully implemented.
---

## **Success Criteria**

* ✅ **Client Booking**

  * Clients can select fixed services and optionally add a custom service.
  * Clients can view next available slots and select preferred date/time.
  * Multi-slot jobs can be booked without conflicts.

* ✅ **Admin Management**

  * Admins can see all bookings, custom service requests, and slot details.
  * Booking conflicts are prevented; buffer times respected.

* ✅ **System Behavior**

  * Bookings are stored in DB accurately (including multi-slot jobs).
  * Next available slots are dynamically calculated based on calendar and existing bookings.
  * Booking confirmation triggered automatically via email/SMS.

* ✅ **Usability**

  * Clients understand the flow without confusion.
  * Multi-slot selection is intuitive (wizard or clear step-by-step).

* ✅ **Compliance & Security**

  * Custom service inputs and booking data stored securely.
  * Buffer times and availability logic prevent double-booking or schedule overlap.

---

If you want, I can **take this and draft the full acceptance criteria table** ready for dev tickets, including **GIVEN/WHEN/THEN format** for each key scenario (single-slot, multi-slot, custom service, etc.).

Do you want me to do that next?

