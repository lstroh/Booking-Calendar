# File Analysis: `core/sync/wpbc-gcal.php`

## High-Level Overview

This file acts as the primary controller and configuration handler for the Google Calendar import feature. It bridges the gap between the admin settings UI, the automated background processing (cron), and the core import engine (`WPBC_Google_Calendar` class).

Its main purposes are:
1.  **Automated Import**: It defines the `wpbc_silent_import_all_events()` function, which is hooked into the plugin's custom cron system to perform scheduled, background imports from Google Calendar.
2.  **Settings UI Rendering**: It contains the helper functions that render the actual form fields (dropdowns, text inputs) on the Google Calendar settings page.
3.  **Lifecycle Management**: It hooks into the plugin's activation and deactivation processes to add and remove the necessary database options for the GCal sync feature.

## Detailed Explanation

The file is composed of several procedural functions that serve distinct roles.

-   **`wpbc_silent_import_all_events()`**: This is the function executed by the plugin's cron scheduler.
    -   **Hooking**: It is attached to the custom `wpbc_silent_import_all_events` action, which is triggered by the cron class (`WPBC_Cron`).
    -   **Instantiation**: It creates a new instance of the `WPBC_Google_Calendar` class.
    -   **Configuration**: It retrieves all the relevant settings from the database using `get_bk_option()` (e.g., `booking_gcal_api_key`, `booking_gcal_events_max`, `booking_gcal_timezone`) and uses the setter methods of the `WPBC_Google_Calendar` object to configure the import job.
    -   **Execution**: It checks if the plugin is a paid version. If not, it runs a single import based on the global GCal feed URL. If it is a paid version, it queries the `wp_bookingtypes` table to get all booking resources and loops through them, running a separate import for each resource that has a GCal feed URL configured.

-   **`wpbc_gcal_settings_content_field_*()` functions**: This is a suite of UI helper functions (e.g., `wpbc_gcal_settings_content_field_from`, `wpbc_gcal_settings_content_field_until`).
    -   **Purpose**: Each function is responsible for rendering the HTML and JavaScript for a specific group of fields on the **Settings > Sync > Google Calendar** page.
    -   **Functionality**: They generate the dropdowns for selecting relative dates (e.g., 'Today', 'Start of current week'), the text inputs for date or offset values, and the inline JavaScript required to show/hide the correct fields based on the user's selection.

-   **`wpbc_sync_gcal_activate()` and `wpbc_sync_gcal_deactivate()`**: These are lifecycle functions.
    -   `wpbc_sync_gcal_activate()` is hooked to the custom `wpbc_other_versions_activation` action. It uses `add_bk_option()` to create and set default values for all Google Calendar settings in the `wp_options` table when the plugin is first activated.
    -   `wpbc_sync_gcal_deactivate()` is hooked to `wpbc_other_versions_deactivation`. It cleans up the database by calling `delete_bk_option()` for all GCal settings when the plugin is deactivated.

## Features Enabled

### Admin Menu

-   **Automated Synchronization**: This file provides the function that powers the automatic, scheduled import of Google Calendar events, a key feature of the sync functionality.
-   **Settings Page Fields**: The `wpbc_gcal_settings_content_field_*()` functions render the actual form inputs on the **Booking > Settings > Sync > Google Calendar** page, allowing administrators to configure how the import works.

### User-Facing

-   This file has no direct user-facing features. However, the events it imports block off dates in the calendar, which directly affects the availability that is shown to visitors on the front-end.

## Extension Opportunities

This file acts as a controller and has limited direct extension points. The most effective way to interact with its functionality is by filtering the data it uses or hooking into actions that it triggers indirectly.

-   **Filtering Settings**: A developer can use the `wpdev_bk_get_option` filter (from `core/wpbc-core.php`) to programmatically override the settings that `wpbc_silent_import_all_events()` reads. For example, one could force a specific timezone or a different maximum number of events during the cron run.

-   **Post-Import Actions**: The `wpbc_booking_save()` function, which is called by the `WPBC_Google_Calendar` class, fires its own action hooks like `wpbc_track_new_booking`. This is the best place to hook in to perform custom actions (like logging or sending notifications) on newly imported bookings.

-   **Potential Risks**: Directly unhooking `wpbc_silent_import_all_events` and replacing it with a custom function is possible but risky, as any changes to the underlying `WPBC_Google_Calendar` class in future plugin updates could break the custom implementation.

## Next File Recommendations

This file completes the analysis of the Google Calendar import feature. The next logical steps are to investigate the remaining major un-analyzed features of the plugin.

1.  **`core/timeline/flex-timeline.php`** — The booking "Timeline" is a core data visualization feature for administrators. Analyzing this file will reveal how booking data is queried and rendered in a chronological, visual format.
2.  **`includes/page-resource-free/page-resource-free.php`** — This file (or one in its directory) is responsible for managing booking resources, which is a fundamental concept in the plugin. Understanding how these "bookable" items are created and managed is key to understanding the plugin's core data model.
3.  **`js/datepick/jquery.datepick.wpbc.9.0.js`** — We have analyzed the high-level client-side logic, but this file is the core jQuery Datepick library that has been customized for the plugin. Analyzing it would provide a deep understanding of how the calendar UI is rendered and how date selection and styling are handled at a low level.
