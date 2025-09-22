Frequently Asked Questions about the Google Calendar Synchronization Feature
1. What is the primary function of the WPBC_Google_Calendar class and how does it relate to booking synchronization?
The WPBC_Google_Calendar class serves as the core engine for the one-way synchronization process, importing events from a public Google Calendar into the local Booking Calendar plugin. Its primary function is to encapsulate all the necessary logic for the import, which includes:

API Communication: Constructing the request URL (using the calendar ID and API key) and using wp_remote_get() to securely fetch event data from the Google Calendar API v3.
Configuration: Allowing external methods to define key parameters like the calendar URL, the target local resource ID, and the time window for the import.
Data Processing: Parsing the returned JSON data, translating Google's date/time formats into the internal format used by the Booking Calendar, and mapping event details (title, description, location) to the correct booking form fields.
Database Integration: Checking the local database for existing events to prevent duplicates and calling the core wpbc_booking_save() function to create new bookings for non-existent events, effectively blocking off dates in the local calendar.
2. How are the settings for the Google Calendar import managed and executed?
The settings are managed through a two-file system:

Configuration & UI: The wpbc-gcal.php file contains functions that render the settings fields (dropdowns for relative dates, text inputs) on the Booking > Settings > Sync > Google Calendar admin page, allowing administrators to configure the import parameters (e.g., API key, maximum events, time window).
Execution: The same wpbc-gcal.php file defines the wpbc_silent_import_all_events() function. This function is triggered by a custom cron job, retrieves the configured settings from the database, instantiates the WPBC_Google_Calendar class, configures it using its public setter methods, and initiates the import process by calling the run() method.
3. How does the plugin handle time windows, date formatting, and duplicate prevention during the import process?
The import class handles these critical aspects systematically:

Time Windows: Configuration methods like set_events_from() and set_events_until() define the period for which events are fetched. They can interpret 'friendly terms' (like 'today' or 'month-start') and translate them into the specific timestamps required by the Google API.
Date Formatting: The class uses a helper method (getCommaSeparatedDates()) to convert the date/time format returned by Google into the specific internal format required by the Booking Calendar plugin before a booking is created.
Duplicate Prevention: Before creating any new bookings, the getExistBookings_gid() method performs a direct database query against the local wp_booking table. It checks if the Google Event IDs (sync_gid) from the current API feed already exist locally, ensuring that the same event is not imported multiple times.
4. What is the difference in import handling between single-feed (free) and multi-resource (paid) plugin versions?
The wpbc_silent_import_all_events() function in the controller file handles this version difference:

Free Version: If the plugin is not identified as a paid version, the function runs a single import job based on a globally configured GCal feed URL.
Paid Version: If it is a paid version, the function queries the wp_bookingtypes table to retrieve all defined booking resources. It then iterates through these resources, running a separate, distinct import job for each resource that has its own unique Google Calendar feed URL configured.
5. What are the key lifecycle actions associated with the Google Calendar sync feature?
The lifecycle of the Google Calendar sync feature is managed by activation and deactivation hooks found in the controller file (wpbc-gcal.php):

Activation (wpbc_sync_gcal_activate()): When the plugin is activated, this function uses add_bk_option() to create and set default values for all necessary Google Calendar synchronization settings (e.g., API key, event mapping) in the WordPress options table.
Deactivation (wpbc_sync_gcal_deactivate()): When the plugin is deactivated, this function performs cleanup by calling delete_bk_option() to remove all Google Calendar-related settings from the database.
6. How does the imported Google Calendar data affect the user-facing part of the Booking Calendar?
Although the core import files (wpbc-gcal-class.php and wpbc-gcal.php) do not contain any direct user-facing features, the data they process directly impacts the front-end availability:

Every successfully imported Google event is saved as a new booking in the local database.
These new bookings block off the corresponding dates and times in the calendar interface shown to website visitors.
The synchronization process thus ensures that the front-end availability accurately reflects the scheduling status from the external Google Calendar.
7. How can developers safely extend or customize the Google Calendar import process?
Direct modification of the core import class is risky due to complexity and potential issues with authentication or duplicate creation. The safest methods for extension involve leveraging the existing integration points:

Programmatic Triggering: Developers can instantiate the WPBC_Google_Calendar class externally, configure it using the setter methods, and manually call the run() method to execute a custom import job.
Post-Import Actions: The best extension point is hooking into the action fired by the core booking saving function (wpbc_booking_save()), such as the wpbc_track_new_booking action. This allows a developer to perform custom actions (e.g., sending a notification, logging data) immediately after a new booking has been successfully imported from Google Calendar.
Filtering Settings: For automated imports, developers can use the wpdev_bk_get_option filter to programmatically override the setting values (like API key or timezone) that the wpbc_silent_import_all_events() cron function reads from the database.
8. What mechanisms are in place for error handling during the API communication?
The WPBC_Google_Calendar class includes robust error handling within its main run() method to ensure reliable API communication:

It checks the response from the Google Calendar API call (made via wp_remote_get()) for standard WordPress error objects (WP_Error).
It also validates the HTTP status code of the response. If the code is not 200 (OK), it registers an error, providing user-friendly messages for common issues like 404 Not Found (incorrect calendar ID) or 403 Forbidden (API key restrictions). This helps administrators diagnose problems with their synchronization setup.