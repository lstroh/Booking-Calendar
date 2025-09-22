Plugin Analysis Summary
Files Included
The following plugin files were analyzed based on the provided source documents. Relative paths are included where strongly inferred by the context of the analysis.
1. wpbc-gcal-class.php
2. core/sync/wpbc-gcal.php (Inferred path for the controller file analyzed in wpbc-gcal.php-analysis.md)
Table of Contents
• wpbc-gcal-class.php
    ◦ Source MD file name
    ◦ Role
    ◦ Key Technical Details
    ◦ Features (Admin vs User)
    ◦ Top Extension Opportunities
    ◦ Suggested Next Files
• core/sync/wpbc-gcal.php
    ◦ Source MD file name
    ◦ Role
    ◦ Key Technical Details
    ◦ Features (Admin vs User)
    ◦ Top Extension Opportunities
    ◦ Suggested Next Files
File-by-File Summaries
wpbc-gcal-class.php
Source MD file name: wpbc-gcal-class.php-analysis.md
Role:
This file defines the WPBC_Google_Calendar class, which serves as the core engine for fetching, parsing, and creating local bookings based on events imported from a public Google Calendar.
Key Technical Details:
• API/HTTP: Uses wp_remote_get() for secure API communication with the Google Calendar API v3 and json_decode() for data parsing.
• Database: Performs direct $wpdb queries in getExistBookings_gid() to check for existing bookings using the sync_gid column, preventing duplicates.
• Integration: Delegates final database insertion to the global wpbc_booking_save() function, making this a critical integration point.
• Configuration: Configures import jobs using public setter methods like setUrl(), setResource(), and reads plugin options such as booking_gcal_api_key and booking_gcal_events_form_fields.
Features (Admin vs User):
• Admin: This is the backend engine that powers the "Import Google Calendar" function, configured on the Booking > Settings > Sync page. It also provides showImportedEvents() to display an import summary to the administrator.
• User: None directly; however, the imported events block off dates, directly affecting the availability shown on the front-end calendar.
Top Extension Opportunities:
• Programmatically instantiate the class and use its setter methods to configure and execute custom import jobs.
• Hook into the action hooks fired by the core function wpbc_booking_save() (e.g., wpbc_track_new_booking) to perform post-import actions like custom logging or notifications.
Suggested Next Files:
1. core/sync/wpbc-gcal.php (Top Priority)
2. core/timeline/flex-timeline.php
3. includes/page-resource-free/page-resource-free.php
core/sync/wpbc-gcal.php
Source MD file name: wpbc-gcal.php-analysis.md
Role:
This file acts as the primary controller for the Google Calendar import feature, managing automated scheduling (cron), handling lifecycle events (activation/deactivation), and rendering the necessary UI fields for admin settings.
Key Technical Details:
• Hooks/Cron: Defines wpbc_silent_import_all_events(), which is attached to the custom wpbc_silent_import_all_events action and triggered by the plugin's cron system.
• Configuration: Retrieves settings from the database using get_bk_option() (e.g., booking_gcal_api_key) and uses them to configure the instantiated WPBC_Google_Calendar object.
• Database/Lifecycle: Uses lifecycle functions (wpbc_sync_gcal_activate/deactivate) hooked to custom activation actions (wpbc_other_versions_activation/deactivation) to create default settings via add_bk_option() and clean up via delete_bk_option().
• Logic: Contains logic to loop through multiple booking resources from the wp_bookingtypes table if the paid version is active, running separate imports for each resource.
Features (Admin vs User):
• Admin: Provides the automated, scheduled synchronization feature. Contains UI helper functions (wpbc_gcal_settings_content_field_*()) that render the necessary form inputs and inline JavaScript on the Settings > Sync > Google Calendar page.
• User: None directly; however, its scheduled imports ensure accurate availability is shown to visitors on the front-end.
Top Extension Opportunities:
• Use the wpdev_bk_get_option filter to programmatically override configuration settings (like timezone or max events) that are read by wpbc_silent_import_all_events() during cron runs.
• Hook into post-import actions like wpbc_track_new_booking, which are triggered by the core process this controller initiates.
Suggested Next Files:
1. core/timeline/flex-timeline.php
2. includes/page-resource-free/page-resource-free.php
3. js/datepick/jquery.datepick.wpbc.9.0.js
Common Features and Patterns
The two analyzed files demonstrate several core patterns essential to the plugin's architecture:
1. Strict Separation of Concerns: The plugin separates the engine (wpbc-gcal-class.php) responsible for API communication, data parsing, and transformation, from the controller (wpbc-gcal.php) responsible for configuration, scheduling, and UI integration.
2. Reliance on Core Integration Points: Both the class and the controller rely on the shared core function wpbc_booking_save() to finalize the data persistence (creating the booking). This reliance makes the hooks associated with wpbc_booking_save() the primary points for safe extension.
3. Use of Custom DB Functions: Both files utilize wrapper functions for database interaction—the controller uses get_bk_option(), add_bk_option(), and delete_bk_option() for settings management, while the class uses direct $wpdb queries for specific efficiency tasks like checking for duplicates.
4. Admin-Focused Feature Impact: While the imported events directly affect front-end availability, neither file contains user-facing UI; all direct functionality (settings configuration, scheduling, import summary display) is aimed at the administrator.
5. Extensibility Challenges: Due to the complexity and multi-step nature of the API sync process, both analyses warn that direct modification is risky. Extension is best achieved by filtering configuration data or hooking into external, post-execution actions.
Extension Opportunities
The most effective and safest ways to extend the Google Calendar import functionality, based on the analyzed files, focus on pre-execution configuration and post-execution notification/logging:
1. Post-Import Actions via wpbc_track_new_booking: Since the import process culminates in a call to wpbc_booking_save(), the action hooks fired by this function (such as wpbc_track_new_booking) are the best place to add custom logic, like sending notifications (e.g., Slack) or logging details every time a booking is successfully imported from Google Calendar.
2. Filtering Configuration Settings: Use the wpdev_bk_get_option filter (likely found in core/wpbc-core.php) to dynamically override the Google Calendar settings (booking_gcal_api_key, booking_gcal_timezone, etc.) read by the scheduler function wpbc_silent_import_all_events().
3. Custom Programmatic Imports: A developer can instantiate the WPBC_Google_Calendar class manually, configure a unique import job using the public setter methods (setUrl(), setResource(), etc.), and run the run() method to trigger an import outside of the standard cron schedule.
Next Files to Analyze
The following files are recommended for future analysis, aggregating suggestions from both source documents, deduplicated, and prioritized.
Exact relative path
Priority
One-line reason
Which MD(s) recommended it
core/timeline/flex-timeline.php
High
Core administrative UI that reveals how booking data is queried and rendered visually in a timeline format.
wpbc-gcal-class.php-analysis.md, wpbc-gcal.php-analysis.md
includes/page-resource-free/page-resource-free.php
Medium
Responsible for managing booking resources, fundamental to understanding the plugin's core data model and how bookable items are created.
wpbc-gcal-class.php-analysis.md, wpbc-gcal.php-analysis.md
js/datepick/jquery.datepick.wpbc.9.0.js
Low
Customized core jQuery Datepick library, providing deep insight into low-level calendar UI rendering and date selection handling.
wpbc-gcal.php-analysis.md
Excluded Recommendations
The following recommendation was made by one source but is excluded from the list above because it is assumed to have been analyzed in a subsequent file upload (wpbc-gcal.php-analysis.md):
File Path
Recommended By
Reason for Exclusion
core/sync/wpbc-gcal.php
wpbc-gcal-class.php-analysis.md
This is the direct counterpart and controller for the GCal feature. It is assumed to be the file analyzed in the wpbc-gcal.php-analysis.md source.
Note: No external completed_files.txt content was provided, so the exclusion is based solely on the continuity of the analyzed MD files.
Sources
• wpbc-gcal-class.php-analysis.md
• wpbc-gcal.php-analysis.md
• completed_files.txt