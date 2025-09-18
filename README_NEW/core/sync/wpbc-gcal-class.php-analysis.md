# File Analysis: `core/sync/wpbc-gcal-class.php`

## High-Level Overview

This file defines the `WPBC_Google_Calendar` class, which is the core engine for importing events from a public Google Calendar into the Booking Calendar plugin. It handles the entire one-way synchronization process, from fetching data from the Google Calendar API to creating corresponding bookings in the local WordPress database.

Architecturally, this class is a self-contained service that is instantiated and configured by other parts of the plugin (likely `core/sync/wpbc-gcal.php`). It encapsulates all the logic for API communication, data transformation, and error handling related to the Google Calendar import feature.

## Detailed Explanation

The class orchestrates the import process through its main `run()` method, supported by several configuration and helper methods.

-   **Configuration Methods**: The class has a series of public setter methods to configure an import job:
    -   `setUrl()`: Sets the Google Calendar ID.
    -   `setResource()`: Sets the local Booking Calendar resource ID into which events will be imported.
    -   `set_events_from()` & `set_events_until()`: Define the time window for the import. These methods can translate friendly terms like 'today' or 'month-start' into the required timestamps.
    -   `set_events_max()`: Sets the maximum number of events to fetch.

-   **`run()` method**: This is the main execution method that performs the import.
    1.  **API Request**: It constructs the request URL for the Google Calendar API v3. It uses the configured calendar ID, an API key retrieved from the plugin's options (`booking_gcal_api_key`), and the time window parameters.
    2.  **HTTP Communication**: It uses the standard WordPress function `wp_remote_get()` to make the API call to Google. This is a secure and reliable way to handle external HTTP requests.
    3.  **Error Handling**: It performs robust error checking on the response. It checks for `WP_Error` objects and non-200 HTTP status codes (e.g., 404 Not Found, 403 Forbidden), providing user-friendly error messages for common issues.
    4.  **Data Parsing**: On a successful response, it uses `json_decode()` to parse the JSON data returned by the Google API.
    5.  **Event Processing**: It iterates through each event (`items`) in the Google Calendar feed. For each event, it:
        -   Extracts key details: title, description, location, start time, and end time.
        -   Calls `getCommaSeparatedDates()` to convert Google's date/time format into the internal format used by Booking Calendar.
        -   Retrieves the admin-configured field mapping (from the `booking_gcal_events_form_fields` option) to correctly assign the event title, description, and location to the appropriate booking form fields.
        -   Assembles a `$submit_array` containing all the data required to create a new booking.

-   **`getExistBookings_gid()` method**: Before creating new bookings, this method performs a direct `$wpdb` query on the local `wp_booking` table. It checks which of the Google event IDs (`sync_gid`) from the current feed have already been imported to prevent creating duplicate bookings.

-   **`createNewBookingsFromEvents()` method**: This method iterates through the fetched events and, for each event that does not already exist locally, it calls the global `wpbc_booking_save()` function. This is a critical integration point, as it delegates the final step of database insertion to the plugin's core booking creation engine.

-   **`showImportedEvents()` method**: A UI helper function that generates an HTML table to display a summary of the newly imported events to the administrator after a manual import.

## Features Enabled

This file provides the backend logic for a major plugin feature but does not register any UI elements itself.

### Admin Menu

-   This file has no direct effect on the admin menu.
-   It is the engine that powers the "Import Google Calendar" functionality, which is configured on the **Booking > Settings > Sync** page.

### User-Facing

-   This file has no direct user-facing features.
-   However, the events it imports are saved as bookings, which block off dates in the calendar. This directly affects the availability shown to visitors on the front-end.

## Extension Opportunities

The class is largely a self-contained process with limited internal hooks, making direct modification difficult. The best extension points are before or after this class is executed.

-   **Safe Extension Method**: A developer could programmatically trigger an import by instantiating this class, calling the setter methods to configure a custom import job, and then executing the `run()` method.

-   **Post-Import Actions**: The most practical way to extend this feature is to use the action hooks fired by the `wpbc_booking_save()` function, which this class calls. For example, a developer could hook into `wpbc_track_new_booking` to perform a custom action (like sending a Slack notification) every time a new booking is successfully imported from Google Calendar.

-   **Potential Risks**: The class is a complex, multi-step process. Modifying it directly could lead to issues with API authentication, date conversion, or the creation of duplicate bookings. The use of direct database queries in `getExistBookings_gid()` means any changes there must be done with care to avoid SQL errors or security issues.

## Next File Recommendations

This file is the core class for GCal imports. The next logical step is to analyze the file that uses this class to expose the feature to administrators.

1.  **`core/sync/wpbc-gcal.php`** — **Top Priority.** This is the direct counterpart to the class file. It will contain the code that instantiates `WPBC_Google_Calendar`, sets its parameters based on user settings from the admin page, and calls the `run()` method. It will also likely contain the cron job registration for automatic imports.
2.  **`core/timeline/flex-timeline.php`** — The booking "Timeline" is a core administrative UI. This file will show how booking data is queried and rendered in a visual timeline format, providing insight into the plugin's data visualization techniques.
3.  **`includes/page-resource-free/page-resource-free.php`** — This file (or one in its directory) is responsible for managing booking resources, which is a fundamental concept in the plugin. Analyzing it is key to understanding the plugin's core data model.
