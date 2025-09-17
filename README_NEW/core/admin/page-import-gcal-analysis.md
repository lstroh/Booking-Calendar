# File Analysis: `core/admin/page-import-gcal.php`

This document provides a detailed analysis of the `core/admin/page-import-gcal.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file is responsible for creating and managing the admin settings page for importing events from Google Calendar. It allows an administrator to configure API keys, set up automatic import schedules, and define default parameters for fetching events.

Architecturally, it integrates into the plugin's admin UI framework by defining a new settings tab under "Booking > Settings > Sync". It uses a custom Settings API class to define the form fields and handles the saving of these settings. A key function of this file is to schedule a recurring WordPress cron job (`wp_cron`) to automate the import process based on the administrator's configuration.

## Detailed Explanation

The file's logic is split between two main classes and a filter function.

-   **`class WPBC_Page_SettingsImportGCal extends WPBC_Page_Structure`**: This class builds the admin page itself.
    -   `__construct()`: Hooks into the `wpbc_menu_created` action to register the settings page.
    -   `tabs()`: Defines the page's location in the admin menu, placing it under a "Sync" tab with the title "Import Google Calendar".
    -   `content()`: Renders the HTML for the settings page, including the form, meta-boxes for different setting groups (General, Auto-import, Defaults), and help text. It also handles the form submission by calling the `update()` method.
    -   `update()`: Processes the form submission. It calls `wpbc_import_gcal__update()` (if it exists, likely in a premium version file) and then uses the associated Settings API class to validate and save the submitted options to the database.
    -   `enqueue_js()`: Adds inline JavaScript to the page for dynamic UI behavior, such as showing or hiding fields based on the values of other settings (e.g., toggling date offset fields).

-   **`class WPBC_API_SettingsImportGCal extends WPBC_Settings_API`**: This class defines the specific settings fields for the GCal import page.
    -   `init_settings_fields()`: This method populates a `$fields` array with the definitions for all settings. This includes fields for the Google API Key, auto-import activation (checkbox), import frequency (select dropdown), and parameters for retrieving events (max number, start/end dates, and offsets). It separates fields into logical groups like `general`, `auto_import`, and `default_settings`.

-   **`function wpbc_fields_after_saving_to_db__import_gcal(...)`**: This function is hooked into the `wpbc_fields_after_saving_to_db` filter.
    -   Its primary role is to manage the WordPress cron schedule. After the settings are saved, it checks if "Activate auto import" is enabled.
    -   If enabled, it uses `WPBC()->cron->update()` to schedule or update a recurring cron event named `wpbc_import_gcal`. The event is configured to trigger the `wpbc_silent_import_all_events` action at the frequency selected by the user.
    -   If disabled, it calls `WPBC()->cron->delete('wpbc_import_gcal')` to remove the scheduled cron job.

## Features Enabled

### Admin Menu

-   **Settings Page**: This file adds a new settings page accessible at **Booking > Settings > Sync > Import Google Calendar**.
-   **Configuration Options**: It provides the UI for configuring:
    -   Google API Key.
    -   Automatic import activation and frequency.
    -   Default query parameters for fetching events (e.g., start date, end date, max number of events).
-   **Resource-Specific Feeds**: For premium versions (`class_exists('wpdev_bk_personal')`), it includes a section to manage Google Calendar feeds for individual booking resources, handled by the `wpbc_import_gcal__show_table()` function.

### User-Facing

This file does **not** implement any direct user-facing features. It is purely for administrative configuration. The data it imports (Google Calendar events) is intended to be used by other parts of the plugin to block off dates in front-end calendars, but this file is not responsible for that display logic.

## Extension Opportunities

-   **Safe Extension Points**:
    -   `wpbc_fields_before_saving_to_db__import_gcal` filter: You can use this filter to modify or validate the settings data just before it is saved to the database. This is the safest way to inject custom logic into the saving process.
    -   `do_action( 'wpbc_hook_settings_page_header', 'import_gcal_settings' )`: You can hook into this action to add custom notices or content at the top of the settings page.
    -   `do_action( 'wpbc_hook_settings_page_footer', 'import_gcal_settings' )`: Similarly, you can add content to the bottom of the page.

-   **Suggested Modifications**:
    -   **Adding a New Setting**: To add a new setting, you would ideally need a filter for the `$this->fields` array within `WPBC_API_SettingsImportGCal::init_settings_fields()`. Since one doesn't exist, the next best approach is to hook into `wpbc_hook_settings_page_footer` to render your new form field and then use the `wpbc_fields_before_saving_to_db__import_gcal` filter to intercept and save your custom field's `$_POST` data.
    -   **Customizing Import Logic**: The actual import is triggered by the `wpbc_silent_import_all_events` action within the cron job. To customize what happens during an import, you would need to find where that action is defined (likely in `core/sync/wpbc-gcal.php`) and see if it has its own internal hooks, or remove the default action and add your own.

-   **Potential Risks**:
    -   Directly modifying the cron update/delete logic in `wpbc_fields_after_saving_to_db__import_gcal` could break the auto-import feature.
    -   Incorrectly validating or sanitizing data via the filters could lead to invalid settings being saved, causing errors during the API import process.

## Next File Recommendations

1.  **`core/sync/wpbc-gcal.php`**: This file is the most logical next step. It likely contains the core logic for the Google Calendar import process, including the `wpbc_silent_import_all_events` action and the `wpbc_import_gcal__show_table()` / `wpbc_import_gcal__update()` functions. It's the direct next step in understanding the GCal sync feature.
2.  **`core/sync/wpbc-gcal-class.php`**: This file probably defines the class used to interact with the Google Calendar API. It would reveal how the plugin fetches, parses, and handles the event data from Google.
3.  **`core/admin/wpbc-toolbars.php`**: This file seems responsible for creating the admin toolbars seen on various pages. Analyzing it would provide a better understanding of the overall admin UI construction and how to add or modify UI elements consistently within the plugin's framework.
