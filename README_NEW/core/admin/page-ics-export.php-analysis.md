# File Analysis: `core/admin/page-ics-export.php`

This document provides a detailed analysis of the `core/admin/page-ics-export.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file is responsible for creating the admin page located at **Booking > Settings > Sync > Export**. Its purpose is to provide the user interface for configuring and generating `.ics` iCalendar feed URLs. These URLs allow users and administrators to export bookings from the website and subscribe to them in external calendar applications like Google Calendar, Apple Calendar, or Outlook.

Architecturally, the file demonstrates a clear separation between the Free and Premium versions of the plugin. For the Free version, it provides a basic, self-contained UI for a single export feed. For Premium versions, it acts as a bridge, displaying a prominent notice if the required "Booking Manager" companion plugin is missing and delegating the full functionality to that plugin when it's active.

## Detailed Explanation

The file's logic is primarily contained within the `WPBC_Page_SettingsExportFeeds` class, which extends the standard `WPBC_Page_Structure` for creating admin pages.

-   **Admin Page & Tab Creation**:
    -   The `tabs()` method registers a top-level "Sync" tab on the settings page and a sub-tab titled "Export - .ics". This creates the navigation path to this specific settings panel.

-   **Dependency Checks**:
    -   The `content()` method performs several crucial checks before rendering the main UI:
        1.  It verifies if the `mbstring` PHP extension is enabled using `function_exists( 'mb_detect_encoding' )`, as it's required for this feature.
        2.  It checks for the existence and minimum version (`2.1`) of a companion plugin, "Booking Manager," using `wpbc_get_wpbm_version()`. If the companion plugin is not active or outdated, it displays a prominent error message with a link to install or update it. This highlights a strong dependency for the full feature set.

-   **Conditional UI Rendering**:
    -   **Free Version**: If the premium version is not active (`! class_exists('wpdev_bk_personal')`), it calls the local function `wpbc_export_ics_feed__table()`. This function renders a simple settings table with a single text field where the user can define the URL path for their `.ics` feed (e.g., `/ics/default.ics`).
    -   **Premium Version**: If a premium version is active, it calls `wpbc_export_feeds__show_table()`. This function is not defined in this file and is expected to be provided by the "Booking Manager" plugin. It would render a more advanced interface, likely a table of all booking resources, each with its own unique `.ics` export URL.

-   **Data Handling**:
    -   The `update()` method handles saving the settings on form submission.
    -   **Free Version**: It calls `wpbc_export_ics_feed__update()`, which sanitizes the user-provided URL path using `sanitize_file_name` and saves it to the `booking_resource_export_ics_url` option in the database.
    -   **Premium Version**: It calls `wpbc_export_feeds__update()`, again delegating the save logic to the companion plugin.

## Features Enabled

### Admin Menu

-   This file creates the **Export - .ics** settings page, located under the **Booking > Settings > Sync** tab.
-   It provides the UI for setting the `.ics` feed URL(s).

### User-Facing

-   The primary user-facing feature enabled by this file is the **`.ics` feed itself**. While not a visible element on the website, the generated URL (e.g., `https://example.com/ics/default.ics`) is a public endpoint that external calendar applications can use to fetch booking data, allowing users to subscribe to their bookings.

## Extension Opportunities

-   **Limited Direct Hooks**: This specific file has very few direct extension points, as most of the advanced logic is delegated. The form submission logic does not apply any filters before saving the single URL in the free version.
-   **Extending via Companion Plugin**: The clear architectural pattern is that advanced export functionality is handled by the "Booking Manager" plugin. Therefore, to extend the export feature (e.g., to add custom data to the `.ics` feed or create new types of feeds), a developer would need to find and use the hooks within that separate plugin.
-   **Potential Risks**: Directly modifying this file to change the `.ics` generation logic would be ineffective for premium users, as the logic is not located here. For free users, any modifications would be overwritten on plugin updates.

## Next File Recommendations

The analysis of this file clearly shows a paired import/export functionality for `.ics` feeds and a more advanced sync feature for Google Calendar. The next logical steps are to explore these related features.

1.  **`core/admin/page-ics-import.php`**: This is the direct counterpart to the export page. Analyzing it will complete the picture of the `.ics` sync functionality by showing how the plugin is configured to fetch and import events from external `.ics` feed URLs.
2.  **`core/admin/page-ics-general.php`**: This file likely contains general settings that apply to both ICS import and export, such as caching rules or refresh rates, providing more context for the overall feature.
3.  **`core/sync/wpbc-gcal.php`**: Now that we've seen the file-based `.ics` sync, analyzing this file is the next logical step. It will reveal how the plugin handles a more complex, API-based synchronization with Google Calendar, offering a valuable comparison of different sync strategies.
