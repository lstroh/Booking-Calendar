# File Analysis: `includes/page-availability/availability__page.php`

This document provides a detailed analysis of the `includes/page-availability/availability__page.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file is the controller for the **Booking > Availability** admin page. It exemplifies a modern, AJAX-driven approach to building a WordPress admin interface. The file's primary responsibility is to render the page's "shell"—including the main structure, toolbar, and navigation tabs—and then to kick off a JavaScript-driven process that loads the main content—the availability calendar—asynchronously.

The page is composed of two classes:
1.  `WPBC_Page_AJX_Availability`: Renders the main page, the toolbar, and the AJAX-loading container for the availability calendar.
2.  `WPBC_Page_Availability_General`: A clever UI addition that adds a "General Availability" tab to this page, which is actually a direct link to the relevant section on the main plugin Settings page.

## Detailed Explanation

The file's architecture is designed to create a fast and responsive user experience.

-   **AJAX-First Architecture**: The core architectural pattern here is "AJAX-first." The `content()` method of the main class does not contain the logic for displaying the availability calendar. Instead, it calls the `ajx_availability_container__show()` method, which does two things:
    1.  Renders a simple `<div>` with a "Loading..." message.
    2.  Outputs an inline `<script>` block that initializes a JavaScript object (`wpbc_ajx_availability`) and immediately tells it to fetch the calendar content via an AJAX call.
    This means the user sees the page structure instantly, while the heavy data lifting happens in the background.

-   **State Management**: The `get_cleaned_params__saved_requestvalue_default()` method is crucial for user experience. It uses a custom `WPBC_AJX__REQUEST` class to intelligently retrieve the user's current view settings (like the selected resource, month, etc.). It prioritizes in this order:
    1.  Previously saved settings for that user from the `usermeta` table.
    2.  Parameters from the current URL (`$_REQUEST`).
    3.  Default values.
    This ensures that when an admin returns to the page, their previous filter and view settings are preserved.

-   **UI Composition**: The page's navigation is composed from multiple sources:
    -   The main "Availability" tab in the WordPress side menu is defined in the `tabs()` method of `WPBC_Page_AJX_Availability`.
    -   Secondary tabs at the top of the content area ("Set Availability", "Options") are added via the `wpbc_toolbar_toolbar_tabs()` method.
    -   A "General Availability" tab is added by the separate `WPBC_Page_Availability_General` class, which cleverly links to a different admin page (`wpbc-settings`) to group related functionality together.

-   **JavaScript Initialization**: The link between the PHP backend and the JavaScript front-end is the `wpbc_ajx_availability__send_request_with_params()` function call. The PHP `content()` method gathers all the initial state parameters (resource ID, dates, view mode), encodes them as a JSON object, and passes them to this JavaScript function to initiate the first AJAX request.

## Features Enabled

### Admin Menu

-   **Availability Page**: This file creates the **Booking > Availability** page, which is the central interface for managing the availability of booking resources.
-   **AJAX-Powered UI**: It provides the foundation for a dynamic, app-like interface where the calendar and availability data are loaded and updated without full page reloads.
-   **Persistent Filters**: It enables user-specific saving of filter and view preferences, so the interface remembers the user's last state.

### User-Facing

-   This file has no direct user-facing features. However, the availability rules configured on this page are the data source that the front-end calendar uses to determine which dates to show as available or unavailable to visitors.

## Extension Opportunities

The file uses action hooks to allow for the injection of custom content and tabs.

-   **`wpbc_toolbar_top_tabs_insert`**: A developer can use this action to add their own tabs to the horizontal toolbar on this page.
-   **`wpbc_hook_settings_page_header` / `wpbc_hook_settings_page_footer`**: These actions allow for adding content before or after the main content area of the page.
-   **JavaScript Interaction**: An advanced developer could potentially interact with the global `wpbc_ajx_availability` JavaScript object to listen for custom events (if any are fired) or call its methods to manipulate the view, although this is not a formally documented API.

## Next File Recommendations

This file is the "shell" for the Availability page. The next logical step is to analyze the file that contains the actual server-side logic for handling the AJAX requests.

1.  **`includes/page-availability/availability__class.php`**: This is the highest priority. The current file references `WPBC_AJX__Availability` and its JavaScript object `wpbc_ajx_availability`. This class file is almost guaranteed to contain the WordPress AJAX action hook (`wp_ajax_WPBC_AJX_AVAILABILITY`) and the core server-side logic for processing requests and returning the calendar data.
2.  **`core/any/api-emails.php`**: This remains a key un-analyzed file. It likely defines the core API for managing email templates, which is a fundamental part of the plugin's notification system.
3.  **`core/admin/wpbc-gutenberg.php`**: To understand how the plugin integrates with the modern WordPress editor, analyzing this file will show how the booking form block is created and managed.
