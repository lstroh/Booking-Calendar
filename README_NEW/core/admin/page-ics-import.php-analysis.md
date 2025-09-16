# File Analysis: `core/admin/page-ics-import.php`

This document provides a detailed analysis of the `core/admin/page-ics-import.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file is responsible for creating the admin page located at **Booking > Settings > Sync > Import**. Its purpose is to provide a user interface for manually importing bookings from an external `.ics` iCalendar feed. This allows administrators to sync availability from other platforms like Airbnb, VRBO, or any calendar that provides an `.ics` URL.

Architecturally, this file acts as a UI and integration layer. It builds the settings page and the import toolbar, but it crucially **delegates the core import logic to a required companion plugin, "Booking Manager."** The file's primary role is to provide the user interface, gather the necessary parameters (the `.ics` feed URL and the target booking resource), and then trigger an action hook that the companion plugin listens for.

## Detailed Explanation

The file's logic is primarily contained within the `WPBC_Page_SettingsImportFeeds` class and the AJAX handler functions.

-   **`WPBC_Page_SettingsImportFeeds` Class**: This class extends `WPBC_Page_Structure` to build the admin page.
    -   **`tabs()`**: This method registers the "Import - .ics" sub-tab under the main "Sync" tab in the plugin's settings, creating the navigation to this page.
    -   **`content()`**: This is the main rendering method. It performs a critical dependency check:
        -   It verifies that the "Booking Manager" companion plugin (version 2.1 or newer) is active using `wpbc_get_wpbm_version()`.
        -   If the companion plugin is missing or outdated, it displays a prominent notice and instructions on how to install it.
        -   If the companion plugin is active, it renders the import interface by calling `show_toolbar_import_fields()`.

-   **`show_toolbar_import_fields()` Method**: This function renders the actual import UI, which consists of:
    1.  A dropdown menu to select the target booking resource (calendar) for the import.
    2.  A text input field for the user to paste the URL of the `.ics` feed.
    3.  An "Upload / Select" button that integrates with the WordPress media library (via the Booking Manager plugin) to allow uploading an `.ics` file directly.
    4.  An "Import" button that triggers the AJAX process.

-   **AJAX Workflow**:
    -   **`wpbc_ics_import_ajax_js()`**: This function is called on the page to output the necessary JavaScript. It sets up a jQuery click handler on the "Import" button.
    -   When clicked, the JavaScript gathers the `.ics` URL, the selected resource ID, and a security nonce. It then sends an AJAX `POST` request to WordPress with the action `WPBC_IMPORT_ICS_URL`.
    -   **`wpbc_ajax_WPBC_IMPORT_ICS_URL()`**: This is the PHP function that handles the AJAX request.
        -   It first verifies the nonce for security.
        -   **Delegation via Hook**: Instead of containing the complex logic for parsing the `.ics` file, it simply packages the parameters into an array and executes a custom action hook: `do_action( 'wpbm_ics_import_start', ... )`.
        -   This is the key architectural point: the actual import functionality is handled by a function in the "Booking Manager" plugin that is listening for the `wpbm_ics_import_start` action.

## Features Enabled

### Admin Menu

-   This file creates the **Import - .ics** settings page, located under the **Booking > Settings > Sync** tab.
-   It provides the UI for manually triggering an import of bookings from an `.ics` feed URL or an uploaded file.

### User-Facing

-   This file has no direct user-facing features. However, the events it imports will block off dates in the calendar, directly affecting the availability that is shown to visitors on the front-end.

## Extension Opportunities

-   **Primary Extension Point**: The main point for extension is the `wpbm_ics_import_start` action hook. A developer could hook into this action (with a priority before or after the Booking Manager's hook) to perform custom actions. For example, one could log all import attempts to a separate file or send an email notification after an import is completed.
-   **Programmatic Import**: A developer could trigger an import programmatically by using `wp_remote_post` to send a request to `admin-ajax.php` with the `WPBC_IMPORT_ICS_URL` action and the required parameters. However, a more direct approach would be to find the function that hooks into `wpbm_ics_import_start` within the Booking Manager plugin and call it directly if possible.
-   **Limitations**: Because the core logic resides in a separate, required plugin, there are no filters within this file to modify the import behavior itself (e.g., to change how dates are parsed). Such modifications would need to be done by hooking into the Booking Manager plugin.

## Next File Recommendations

We have now analyzed the UI for both `.ics` import and export. The next logical steps are to investigate the plugin's other major synchronization feature and to explore its integration with the modern WordPress editor.

1.  **`core/sync/wpbc-gcal.php`**: **Top Priority.** This file is the direct counterpart to the `.ics` sync system. It will contain the more advanced logic for API-based, two-way synchronization with Google Calendar, which is a core feature of the premium versions.
2.  **`core/admin/wpbc-gutenberg.php`**: This remains a key un-analyzed file. Understanding how the plugin provides a "Booking Form" block for the modern Gutenberg editor is essential for a complete architectural overview.
3.  **`core/admin/page-timeline.php`**: The booking "Timeline" is a core data visualization feature for administrators. Analyzing this file will reveal how booking data is queried and rendered in a timeline format.
