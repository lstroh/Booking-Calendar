# File Analysis: `includes/page-availability/availability__class.php`

This file defines the `WPBC_AJX__Availability` class, which serves as the server-side engine for the AJAX-powered **Booking > Availability** page. It handles fetching calendar data, processing availability updates, and managing the user's view-state preferences.

## High-Level Overview

This class is the backend controller for the dynamic availability calendar in the admin panel. When an administrator navigates to the Availability page, this class's JavaScript counterpart sends an AJAX request. This class receives that request, processes it, queries the database for all relevant availability data (including bookings, seasonal rules, and custom availability), and sends a complete data package back to the client in JSON format.

It also handles actions, suchas when an admin selects dates and marks them as "Available" or "Unavailable." It processes these requests and updates the custom `wp_booking_dates_props` table accordingly. Architecturally, it's a modern, AJAX-driven component that decouples the page's UI from its data, enabling a fast, app-like user experience.

## Detailed Explanation

The class's functionality is centered around its role as an AJAX endpoint.

-   **Asset Loading (`init_load_css_js_tpl`, `js_load_files`)**:
    -   The class first ensures that its specific JavaScript (`availability_page.js`) and CSS (`availability_page.css`) are loaded, but only on the `page=wpbc-availability` admin page.
    -   It also enqueues several other core scripts like `wpbc_all.js` and `client.js`, indicating that the availability calendar reuses components from the main front-end calendar.

-   **Request Parameter Management (`request_rules_structure`)**:
    -   This static method defines a strict set of rules for all expected AJAX parameters (e.g., `resource_id`, `dates_selection`, `calendar__view__visible_months`).
    -   For each parameter, it specifies a validation type (`d` for digit, `s` for string, or an array of allowed values) and a default value.
    -   This structure is used by a `WPBC_AJX__REQUEST` helper class to sanitize all incoming data and manage user-specific settings, which are saved to the database to preserve the user's view state between visits.

-   **AJAX Endpoint (`ajax_WPBC_AJX_AVAILABILITY`)**:
    -   **Hook**: This method is hooked to the `wp_ajax_WPBC_AJX_AVAILABILITY` action.
    -   **Security**: It performs a nonce check (`check_ajax_referer`) to secure the endpoint.
    -   **Action Handling**: It checks for a `do_action` parameter in the request to perform specific tasks:
        -   `'set_availability'`: Calls `wpbc_availability__update_dates_status__sql()` to update or insert rows in the `wp_booking_dates_props` table, marking dates as available or unavailable for a specific resource.
        -   `'erase_availability'`: Calls `wpbc_availability__delete_dates_status__sql()` to remove all availability rules for a resource.
    -   **Data Aggregation**: It gathers all data needed to render the calendar by calling several lower-level functions:
        -   `wpbc__sql__get_booked_dates()`: Fetches approved and pending booking dates.
        -   `wpbc__sql__get_season_availability()`: Gets dates blocked by season filters.
        -   `wpbc_resource__get_unavailable_dates()`: Gets dates blocked by general resource settings (e.g., unavailable weekdays).
    -   **JSON Response**: It packages all the fetched data, booking resources, popover hint text, and action messages into a single array and sends it back to the client using `wp_send_json()`.

-   **JavaScript Templating (`hook__page_footer_tmpl`)**:
    -   This method injects several Underscore.js templates into the page footer.
    -   Templates like `tmpl-wpbc_ajx_availability_main_page_content` and `tmpl-wpbc_ajx_widget_available_unavailable` define the HTML structure for the page's main layout and widgets.
    -   The client-side JavaScript uses these templates along with the data from the AJAX response to dynamically build and render the page content.

## Features Enabled

### Admin Menu

-   This class provides the entire server-side logic for the **Booking > Availability** page.
-   It powers the dynamic, AJAX-driven availability calendar where administrators can view and set date availability.
-   It handles the logic for the "Apply availability" widget, allowing admins to mark selected dates as available or unavailable.
-   It provides the data for the calendar legend and the resource selection dropdown in the page's toolbar.

### User-Facing

-   This file has no direct user-facing features. However, the availability rules configured here are the primary data source that the front-end calendar uses to determine which dates to show as available or unavailable to visitors.

## Extension Opportunities

-   **Data Filtering**: The most effective way to extend this functionality is to filter the data *before* it gets to this class. The data-fetching functions it calls (e.g., `wpbc__sql__get_booked_dates`, `wpbc__sql__get_season_availability`) are the ideal places to hook in. If those functions have filters on their SQL queries or return values, a developer could add custom availability rules or modify existing ones.
-   **JavaScript Interaction**: The client-side `availability_page.js` script will receive the large JSON object from this class. A developer could potentially listen for custom JavaScript events fired by that script after it has rendered the calendar to make further DOM modifications.
-   **Potential Risks**: The client-side script expects a very specific JSON data structure in the AJAX response. Any custom code that modifies this data structure could easily break the client-side rendering of the calendar. Direct modification of this class is not recommended.

## Next File Recommendations

The analysis of the Availability page's PHP controller and AJAX handler is now complete. The next logical step is to examine the client-side script that drives the UI.

1.  **`includes/page-availability/_out/availability_page.js`**: This is the JavaScript file that is enqueued by this class. It will contain the client-side logic that sends the AJAX request, receives the JSON response, and uses the Underscore.js templates to render the calendar and widgets. This is the top priority.
2.  **`includes/page-resource-free/page-resource-free.php`**: The availability page is centered around booking resources. This file is the most likely place to find the UI and logic for creating and managing those resources, which is a fundamental concept for the plugin. It's not on the completed list.
3.  **`js/user-data-saver.js`**: This file has been recommended before and is still a good candidate. It might be related to how user preferences for the availability page (and other pages) are saved. It's not on the completed list.
