Briefing Document: Google Calendar Synchronization Feature
Overview
This briefing document analyzes the core components of the Google Calendar (GCal) synchronization feature within the WordPress Booking Calendar plugin. The synchronization is a one-way process designed to import events from a public Google Calendar into the local WordPress database as bookings, thereby blocking off dates and managing resource availability.

The functionality is split between a core engine class (WPBC_Google_Calendar) and a controller file (wpbc-gcal.php) that manages configuration, scheduling, and UI rendering.

1. Core Synchronization Engine (WPBC_Google_Calendar Class)
The WPBC_Google_Calendar class is the self-contained service responsible for executing the import process.

A. Main Themes and Process Flow
The class orchestrates a multi-step import via its central run() method, ensuring secure communication and data integrity.

Configuration: The class is configured using public setter methods (e.g., setUrl(), setResource(), set_events_from(), set_events_until()) to define the calendar ID, the local resource ID, and the time window for the import.
API Communication: It uses standard WordPress functions (wp_remote_get()) to construct and send the request to the Google Calendar API v3, utilizing an API key retrieved from plugin options (booking_gcal_api_key).
Robust Error Handling: The class performs "robust error checking on the response," checking for WP_Error objects and non-200 HTTP status codes (e.g., 404, 403) to provide actionable feedback to the administrator.
Data Transformation: It processes the JSON data, extracts event details (title, start/end time, description, location), and calls getCommaSeparatedDates() to convert Google's date/time format into the internal format used by Booking Calendar.
Deduplication: The getExistBookings_gid() method performs a direct $wpdb query to check for existing bookings that share the same Google Event ID (sync_gid), effectively preventing the creation of duplicate bookings.
Final Creation: The createNewBookingsFromEvents() method iterates through unique events and calls the critical integration point, wpbc_booking_save(), which inserts the new booking data into the database.
B. Important Facts and Quotes
The class is the "core engine for importing events from a public Google Calendar into the Booking Calendar plugin."
It handles the "entire one-way synchronization process."
Configuration parameters for the API request include the calendar ID, API key, and the time window defined by set_events_from() and set_events_until(), which can interpret "friendly terms like 'today' or 'month-start'."
Data parsing involves retrieving the "admin-configured field mapping" (booking_gcal_events_form_fields option) to correctly map Google event details to local booking form fields.
The imported events are saved as bookings, which "block off dates in the calendar. This directly affects the availability shown to visitors on the front-end."
2. Synchronization Controller and Scheduler (wpbc-gcal.php)
This file serves as the feature's primary controller, handling administrative configuration, background automation, and lifecycle management.

A. Main Themes and Functionality
The controller bridges the gap between administrator settings and the core WPBC_Google_Calendar engine.

Automated Import Scheduling: The wpbc_silent_import_all_events() function is the key to automation. It is hooked to the plugin's custom cron system, allowing for scheduled, background imports.
Configuration Delivery: This function retrieves all necessary settings (API key, maximum events, timezone) using get_bk_option() and uses them to configure the instantiated WPBC_Google_Calendar object.
Support for Multiple Resources (Paid Feature): The controller logic differentiates between free and paid plugin versions. In the paid version, it queries the wp_bookingtypes table to loop through all booking resources and runs a separate import for each resource configured with a GCal feed URL.
UI Rendering: A suite of UI helper functions (wpbc_gcal_settings_content_field_*()) renders the input fields, dropdowns, and associated JavaScript for the Booking > Settings > Sync > Google Calendar page, allowing for date selection based on relative terms (e.g., 'Start of current week').
Lifecycle Management: Dedicated functions (wpbc_sync_gcal_activate() and wpbc_sync_gcal_deactivate()) ensure that default GCal settings are added to the database upon plugin activation and cleaned up upon deactivation.
B. Important Facts and Quotes
The file "acts as the primary controller and configuration handler for the Google Calendar import feature."
wpbc_silent_import_all_events() is the function that is "executed by the plugin's cron scheduler."
The UI helper functions are responsible for rendering the form inputs on the settings page, which allow administrators to configure "how the import works."
The logic for paid versions involves querying the wp_bookingtypes table and "running a separate import for each resource that has a GCal feed URL configured."
3. Extension and Integration Points
Both files highlight the critical integration points for developers wishing to extend or modify the synchronization behavior.

A. Primary Integration Points
Integration PointDescriptionwpbc_booking_save()This global function is called by the WPBC_Google_Calendar class to finalize database insertion. It is the best place to hook into for post-import actions (e.g., logging, custom notifications) via actions like wpbc_track_new_booking.wpdev_bk_get_option FilterThis filter (used by wpbc-gcal.php when retrieving settings) allows a developer to programmatically override the GCal settings (e.g., API key, timezone, max events) used during a cron run.B. Risk and Recommendations
Direct modification of the WPBC_Google_Calendar class is considered risky due to its complexity and the potential to introduce issues with "API authentication, date conversion, or the creation of duplicate bookings."
The recommended extension method is to use the action hooks fired by the core booking creation process (wpbc_booking_save()) or to programmatically trigger an import by instantiating and configuring the WPBC_Google_Calendar class directly.