# File Analysis: `core/admin/wpbc-toolbars.php`

This document provides a detailed analysis of the `core/admin/wpbc-toolbars.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file is a comprehensive UI library dedicated to constructing the various toolbars found across the plugin's admin pages. It contains the functions responsible for rendering the rows of buttons, filters, and navigation elements that appear at the top of pages like "Bookings," "Timeline," and "Add New Booking."

Architecturally, this file acts as a "toolbar factory." It uses the lower-level UI component functions (from `admin-bs-ui.php` and `flex_ui_elements.php`) to assemble complex, page-specific toolbars. It encapsulates the logic for what buttons to show, what filters to offer, and how to handle user interactions like saving view preferences via AJAX.

## Detailed Explanation

The file is composed of several large functions, each responsible for building a specific toolbar, along with smaller helper functions for individual buttons and assets.

-   **`wpbc_add_new_booking_toolbar()`**: This is a major function that builds the entire toolbar for the "Add New Booking" page.
    -   It creates a tabbed interface with "Options" and "Calendar View" tabs.
    -   The **Options** tab includes controls for selecting the booking resource, the booking form, and a toggle to allow creating bookings in the past.
    -   The **Calendar View** tab is a powerful feature that allows the admin to customize the appearance of the calendar on this page. It includes dropdowns for setting the number of visible months, the number of months per row, and the calendar's width/height.
    -   Crucially, it includes "Save" and "Reset" buttons that use an AJAX call (`wpbc_save_custom_user_data`) to save these view preferences as user meta (`booking_custom_add_booking_calendar_options`), providing a personalized experience for each administrator.

-   **`wpbc_toolbar_btn__timeline_navigation()`**: This function builds the dynamic navigation toolbar for the Timeline page.
    -   It contains complex conditional logic to generate the correct "Previous" and "Next" buttons based on the current view mode (e.g., day, week, month, or year).
    -   It assembles a large array of parameters defining a button group and passes it to `wpbc_bs_button_group()` for rendering.
    -   A key feature is the "Custom" date dropdown, which contains a datepicker, allowing the user to jump directly to any specific date in the timeline.

-   **Individual UI Element Functions (`wpbc_toolbar_btn__*`)**: The file is populated with many smaller, reusable functions for rendering specific toolbar components.
    -   `wpbc_toolbar_is_send_emails_btn()`: Renders a toggle switch that controls whether email notifications are sent for the action being performed.
    -   `wpbc_toolbar_search_by_id__top_form()`: Creates the search form (for ID or Title) that appears at the top right of pages like "Resources" or "Settings." It cleverly handles being a real form or a "pseudo" form that populates a hidden form elsewhere on the page.
    -   `wpbc_toolbar_btn__add_new_booking()`: Renders the final "Add booking" submit button, which has an `onclick` attribute that triggers the main `mybooking_submit` JavaScript function.

-   **Asset Loading**:
    -   The file includes helper functions like `wpbc_js_for_bookings_page()` and `wpbc_datepicker_js()` to ensure that necessary JavaScript and CSS for components like the datepicker are loaded on the pages where these toolbars are used.

## Features Enabled

### Admin Menu

-   This file does not add any new admin pages. It is the engine that builds the interactive toolbars *within* those pages, defining a significant portion of the admin user experience. It is responsible for:
    -   The filter and action toolbars on the Booking Listing and Timeline pages.
    -   The resource selection and calendar customization options on the "Add New Booking" page.
    -   The top-right search forms on various settings pages.

### User-Facing

-   This file has **no user-facing features** and is used exclusively for building the admin panel interface.

## Extension Opportunities

This file is primarily a rendering library and was not designed with many formal extension points like actions or filters.

-   **Adding New Toolbar Elements**: The most straightforward way to add a new button or filter to a toolbar is to find where the main toolbar function (e.g., `wpbc_add_new_booking_toolbar()`) is called in the page template and then add a call to your own custom rendering function before or after it.
-   **Modifying Existing Elements**: Modifying an existing toolbar is difficult without direct code changes. For example, the options in the dropdowns are hardcoded within their respective functions. To make this more extensible, the plugin would need to add filters to the arrays that define the UI elements before they are rendered.
-   **Potential Risks**: Because the file directly outputs HTML and JavaScript `onclick` events, any modifications must be made carefully to avoid creating broken HTML or JavaScript errors.

## Next File Recommendations

The analysis of this file provides a clear picture of how the admin UI is constructed. The most logical next steps are to investigate the remaining core features that have not yet been fully explored.

1.  **`js/wpbc-gutenberg.js`**: **Top Priority.** We have now analyzed the PHP registration for the Gutenberg block. This JavaScript file contains the actual client-side React code that defines the block's editor interface and functionality. Analyzing it is essential to fully understand the plugin's integration with the modern WordPress editor.
2.  **`core/timeline/flex-timeline.php`**: We have seen the page and toolbar for the timeline view. This file contains the `WPBC_TimelineFlex` class, which is the engine that actually queries the data and renders the timeline.
3.  **`core/sync/wpbc-gcal.php`**: This is the last major un-analyzed feature. This file will explain the complex process of how the plugin synchronizes booking data with Google Calendar.
