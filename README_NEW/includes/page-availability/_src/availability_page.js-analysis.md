# File Analysis: `includes/page-availability/_src/availability_page.js`

## High-Level Overview

This file is the client-side JavaScript controller for the entire **Booking > Availability** admin page. It is responsible for making the page a dynamic, single-page application. It manages the page's state, sends AJAX requests to the PHP backend to fetch data and apply changes, and then uses the returned JSON data to dynamically render the entire user interface—including the calendar and widgets—using Underscore.js templates.

Architecturally, this script is the direct counterpart to the `WPBC_AJX__Availability` PHP class. It handles all user interactivity on the page, creating a fast, app-like experience where the calendar and its data can be manipulated without full page reloads. As a file in the `_src` directory, it is the human-readable source code that is then prepared for production use (in this case, copied to the `_out` directory).

## Detailed Explanation

The script's logic is centered around a global `wpbc_ajx_availability` object and a series of functions that handle the AJAX request/response cycle and UI rendering.

-   **State Management (`wpbc_ajx_availability` object)**:
    -   The script initializes a global object that acts as a namespace and data store for the page.
    -   It holds security parameters (nonce, user ID), and the current filter/view parameters (like `resource_id`, `calendar__view__visible_months`).
    -   Functions like `wpbc_ajx_availability.search_set_param()` and `wpbc_ajx_availability.search_get_param()` provide a clean API for other parts of the script to interact with this state.

-   **AJAX Workflow**:
    1.  The page's initial PHP loads a shell with a "Loading..." message and an inline script that calls `wpbc_ajx_availability__send_request_with_params()` to kick off the first data load.
    2.  This triggers `wpbc_ajx_availability__ajax_request()`, which displays a spinner and sends a `POST` request to WordPress's AJAX handler (`wpbc_url_ajax`).
    3.  The request `action` is set to `WPBC_AJX_AVAILABILITY`, which matches the server-side hook in `availability__class.php`. All current filter parameters are sent in the request payload.
    4.  The `success` callback of the AJAX request receives a large JSON object from the server. If the request is successful, it calls `wpbc_ajx_availability__page_content__show()` to render the page.

-   **Dynamic Rendering with Underscore.js**:
    -   `wpbc_ajx_availability__page_content__show()` is the main rendering function. It takes the JSON data from the AJAX response.
    -   It uses `wp.template('wpbc_ajx_availability_main_page_content')` to get the compiled Underscore.js template (which was injected into the page footer by the PHP class).
    -   It then renders this template with the AJAX data and injects the resulting HTML into the main container, replacing the "Loading..." message.
    -   Finally, it calls `wpbc_ajx_availability__calendar__show()` to initialize the calendar itself.

-   **Calendar Initialization (`wpbc_show_inline_booking_calendar`)**:
    -   This function initializes the `jQuery.datepick` plugin on the calendar container.
    -   **`beforeShowDay`**: This is the most critical callback. It calls `wpbc__inline_booking_calendar__apply_css_to_days`, which contains the complex logic to determine the state of each date (available, pending, approved, season unavailable, etc.) and returns the appropriate CSS classes. This is how the calendar gets its colors and styles.
    -   **`onSelect`**: This callback fires when a user selects dates. It updates a hidden `<textarea>` with the selected date string and then calls `wpbc__inline_booking_calendar__on_days_select` to update the help text in the toolbar, guiding the user on the next action.
    -   **`onHover`**: This callback is used to highlight a range of dates when a user is making a range selection.

-   **Event-Driven Hooks**: The script makes good use of custom jQuery events for extensibility and to coordinate actions.
    -   `jQuery(...).trigger('wpbc_page_content_loaded', ...)`: Fired after the main content is rendered. The script itself listens to this to trigger a "blinking" animation on the help text.
    -   `jQuery('body').on('wpbc_datepick_inline_calendar_loaded', ...)`: Fired by the datepicker library after it has been initialized. This script hooks into this event to attach tooltips to the newly rendered calendar cells.

## Features Enabled

This file is exclusively for the admin panel and provides no user-facing features.

### Admin Menu

-   This script provides all the client-side logic that makes the **Booking > Availability** page functional and interactive. It enables:
    -   Asynchronous loading and rendering of the entire page content.
    -   Dynamic display of the availability calendar based on selected resources and filters.
    -   Interactive date selection (single, multiple, and range) in the calendar.
    -   Real-time updates to the UI (like the help text in the toolbar) based on user actions.
    -   The ability to apply "Available" or "Unavailable" status to dates via an AJAX call without a page reload.

### User-Facing

-   This file has no effect on the user-facing side of the website.

## Extension Opportunities

-   **Custom jQuery Events**: The script fires several custom events that serve as excellent, update-safe extension points.
    -   A developer could listen for `wpbc_page_content_loaded` to execute custom JavaScript after the page has been rendered. This could be used to add new UI elements or modify existing ones.
    -   Listening for `wpbc_datepick_inline_calendar_loaded` or `wpbc_datepick_inline_calendar_refresh` allows for safe interaction with the calendar cells after they are drawn by the datepicker library.

-   **Modifying AJAX Parameters**: Before an AJAX call is made, a developer could potentially hook into the UI elements' `click` events with a higher priority to modify the parameters stored in the `wpbc_ajx_availability` object before `wpbc_ajx_availability__send_request_with_params` is called.

-   **Potential Risks**: While this is the source file, direct modification is still discouraged in favor of using the provided hooks, as this is more maintainable and less prone to breaking with future plugin updates. The logic is complex and tightly coupled with the server-side JSON response; any custom code that interacts with it must respect the expected data structure to avoid breaking the page.

## Next File Recommendations

The analysis of the Availability page feature is now complete. The next logical steps are to investigate other core concepts of the plugin that have not yet been reviewed.

1.  **`js/wpbc_times.js`** — This file is the likely JavaScript counterpart to the time-slot selection feature. Analyzing it is crucial for understanding how time-based bookings are handled on the front-end, including how available slots are displayed and how user selections are processed.
2.  **`includes/page-settings-form-options/page-settings-form-options.php`** — This file appears to be responsible for the admin settings page where users configure the booking form fields. Understanding this file is key to learning how the plugin allows for form customization and how those settings are saved and used.
3.  **`css/skins/multidays.css`** — We have analyzed the base calendar CSS. Looking at a specific skin file like this one will reveal how the plugin's theming system works, applying custom colors and styles for different calendar appearances, such as for multi-day selections.
