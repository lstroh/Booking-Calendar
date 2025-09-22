# File Analysis: `includes/page-availability/availability__resource.php`

This file serves as a data-access layer for the Availability feature. It provides a set of utility functions to retrieve availability information—specifically, dates marked as "unavailable"—for one or more booking resources from the `wp_booking_dates_props` database table.

## High-Level Overview

This file acts as a specialized API for querying resource availability. It does not render any UI itself but provides the crucial data that other parts of the plugin need to function. For example, when the admin-side availability calendar or the front-end booking calendar is rendered, they call functions from this file to determine which dates should be marked as unavailable based on rules set by the administrator.

The core of this file is the `wpbc_availability__get_dates_status__sql()` function, which dynamically builds and executes a prepared SQL query. The other functions in the file are convenient wrappers around this core function, tailored for specific use cases like getting unavailable dates for a single resource or for an array of resources.

## Detailed Explanation

The file contains three main procedural functions.

-   **`wpbc_availability__get_dates_status__sql( $params )`**:
    -   **Purpose**: This is the central query engine of the file. It retrieves date properties from the `wp_booking_dates_props` table based on a flexible set of parameters.
    -   **Database Interaction**: It constructs a prepared `SELECT` statement using the global `$wpdb` object. The `WHERE` clause is built dynamically based on the `$params` array, allowing for filtering by `resource_id`, `prop_name`, `prop_value`, and a date range.
    -   **Caching**: It includes a robust caching layer. Before executing the query, it checks for a cached result using `wpbc_cache__get()`. After a successful query, it saves the result to the cache with `wpbc_cache__save()`. This significantly improves performance by avoiding repeated database queries for the same availability data.
    -   **Sanitization**: It uses a helper function, `wpbc_sanitize_params_in_arr()`, to validate and sanitize all incoming parameters, ensuring data integrity and preventing SQL injection.

-   **`wpbc_resource__get_unavailable_dates( $resource_id = 1 )`**:
    -   **Purpose**: A simple wrapper function to get all future unavailable dates for a single booking resource.
    -   **Functionality**: It calls `wpbc_availability__get_dates_status__sql()` with a hardcoded query for `prop_value = 'unavailable'` and `calendar_date >= CURDATE()`. It then processes the results to return a simple, flat array of date strings (e.g., `['2023-12-25', '2024-01-01']`).

-   **`wpbc_for_resources_arr__get_unavailable_dates( $resource_id_arr, $search_dates = 'CURDATE' )`**:
    -   **Purpose**: An efficient function for getting unavailable dates for multiple resources at once.
    -   **Functionality**: It takes an array of resource IDs, implodes them into a comma-separated string for use in an `IN ()` clause, and makes a single call to `wpbc_availability__get_dates_status__sql()`. It then formats the results into an associative array where each key is a resource ID and the value is an array of its unavailable dates. This is an efficient design that avoids making multiple database queries in a loop.

## Features Enabled

This file is a backend data provider and does not directly enable any visible features. It is a dependency for other components.

### Admin Menu

-   This file has no direct effect on the admin menu.
-   It provides the data that the **Booking > Availability** page uses to render the calendar, showing which dates have been manually marked as unavailable.

### User-Facing

-   This file has no direct user-facing features.
-   It is a critical dependency for the front-end calendar. The data it returns is used to disable or style dates that are unavailable, directly impacting what dates a visitor can select for booking.

## Extension Opportunities

-   **Reusing Functions**: The functions in this file are globally accessible and can be used by third-party developers. For example, if you were building a custom report on resource availability, you could call `wpbc_resource__get_unavailable_dates()` to get the necessary data.

-   **Limitations**: The core query function, `wpbc_availability__get_dates_status__sql()`, does not contain any WordPress filters (`apply_filters`). This means a developer cannot easily modify the SQL query to add custom availability logic (e.g., "make all Fridays in May unavailable for resource 5") without modifying the core file, which is not update-safe.

-   **Suggested Improvements**: To make this data layer more extensible, the developers could add filters to `wpbc_availability__get_dates_status__sql()`:
    -   A filter on the `$sql['where']` string would allow developers to add custom conditions to the query.
    -   A filter on the final `$bookings_sql_obj` result set would allow developers to add, remove, or modify availability data before it is returned and cached.

## Next File Recommendations

Now that we understand the data-access layer for resource availability, the next logical step is to see how "resources" themselves are defined and managed.

1.  **`includes/page-resource-free/page-resource-free.php`**: This is the top priority. It's the most likely file to contain the admin UI for creating, editing, and managing booking resources. Understanding this is fundamental to the plugin's data model.
2.  **`includes/page-availability/_out/availability_page.js`**: We've analyzed the PHP controller and the data-access layer for the Availability page. This JavaScript file is the missing piece that will show how the front-end of that admin page is built and how it interacts with the AJAX endpoint.
3.  **`js/user-data-saver.js`**: This file has been recommended before and remains a good candidate. It could be related to how user-specific data (like the last-viewed resource) is saved, which is relevant to the state-management features seen in the Availability page controller.
