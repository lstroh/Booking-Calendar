# File Analysis: `core/wpbc-dev-api.php`

This document provides a detailed analysis of the `core/wpbc-dev-api.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file is the official **Developer API** for the Booking Calendar plugin. Its purpose is to provide a stable, documented set of functions for third-party developers to interact with the plugin programmatically. It acts as an abstraction layer, allowing other plugins, themes, or custom scripts to create, edit, and check bookings without needing to know the complex internal data structures, database schema, or AJAX endpoints.

The key functions in this file allow for:
-   Programmatically adding or editing bookings.
-   Checking if a specific date or time range is available for a booking resource.
-   Retrieving booking data.

The file also serves as excellent inline documentation, listing and explaining numerous action and filter hooks that developers can use to extend the plugin's functionality.

## Detailed Explanation

The file provides several key public-facing functions:

-   **`wpbc_api_booking_add_new( $booking_dates, $booking_data, $resource_id, $params )`**
    This is the most important function in the API. It allows for the creation or editing of a booking.
    -   **Abstraction:** It takes developer-friendly input: an array of dates and a simple associative array for form data. Internally, it translates this into the plugin's complex, custom-serialized form string (`type^name^value~...`) before calling the core `wpbc_booking_save()` function. This is a critical feature, as it shields the developer from the internal data format.
    -   **Functionality:** It can be used to create a new booking or, by passing a `booking_id` in the `$params` array, to edit an existing one.
    -   **Error Handling:** It returns the new `booking_id` on success or a standard `WP_Error` object on failure, which is a WordPress best practice.

-   **`wpbc_api_is_dates_booked( $booking_dates, $resource_id, $params )`**
    This function checks if a given set of dates/times is available for a specific resource.
    -   **Consistency:** It works by calling the internal `wpbc__where_to_save_booking()` function, which is the same availability-checking engine used by the actual booking creation process. This ensures that the API check will return the same result as a real booking attempt.
    -   **Usage:** It returns `true` if the dates are already booked (unavailable) and `false` if they are available.

-   **Other API Functions:**
    -   `wpbc_api_get_bookings_arr()`: A **deprecated** function for retrieving a list of bookings. It's a wrapper around the internal `wpbc_get_bookings_objects()` function.
    -   `wpbc_api_get_booking_by_id()`: Retrieves all data for a specific booking ID and helpfully unserializes the form data into a readable array.
    -   `wpbc_get_form_fields_free()`: A helper to get the field names and labels from the simple booking form in the free version.

-   **Hook Documentation & Examples:**
    A large portion of the file is dedicated to documenting action hooks like `wpbc_track_new_booking`, `wpbc_booking_approved`, and `wpbc_deleted_booking_resources`. It provides clear examples of how a developer can use these hooks to trigger custom code when specific events happen in the plugin. It also includes a commented-out but functional example (`wpbc_resource_created__add_other_params`) showing how to use an action to programmatically configure a new booking resource right after it's created.

## Features Enabled

This file does not enable any direct features in the user interface. Its sole purpose is to enable **third-party integrations**.

-   **Admin Menu:** None.
-   **User-Facing:** None.

This API is the key to allowing other systems to connect to Booking Calendar. For example, a developer could use it to:
-   Create a custom front-end booking form that submits data to Booking Calendar.
-   Sync bookings from an external CRM or property management system.
-   Build a custom reporting dashboard that pulls booking data.

## Extension Opportunities

This entire file is an extension point by design.

-   **Programmatic Booking:** The primary use is to call `wpbc_api_booking_add_new()` from your own plugin or theme's `functions.php` to create bookings automatically.

    ```php
    // Example of creating a new booking via the API
    $booking_details = array(
        'dates'       => array( '2027-08-24', '2027-08-25' ),
        'data'        => array(
            'name'       => 'John Smith',
            'email'      => array( 'value' => 'john.smith@example.com', 'type' => 'email' ),
        ),
        'resource_id' => 1
    );
    $new_booking_id = wpbc_api_booking_add_new(
        $booking_details['dates'],
        $booking_details['data'],
        $booking_details['resource_id']
    );

    if ( is_wp_error( $new_booking_id ) ) {
        // Handle the error
        $error_message = $new_booking_id->get_error_message();
    } else {
        // Booking created successfully
    }
    ```

-   **Event-Driven Logic:** Use the documented action hooks (e.g., `wpbc_booking_approved`, `wpbc_track_new_booking`) to trigger your own functionality when events occur within the plugin.

## Next File Recommendations

This API file provides a high-level interface to the plugin's core functions. To understand what happens "under the hood," we should investigate the files that this API calls.

1.  **`core/any/class-admin-menu.php`**: This file is still a top priority as it defines the entire admin menu structure, which is a fundamental part of the plugin's architecture that we have not yet explored.
2.  **`core/lib/wpbc-booking-new.php`**: While the API calls `wpbc_booking_save()`, this file is still highly relevant. It likely contains the logic for creating bookings manually from the admin panel (e.g., from the "Add Booking" page), which may follow a different path than the front-end AJAX creation.
3.  **`core/wpbc-emails.php`**: The API provides an option to suppress emails (`is_send_emeils`). Analyzing this file will reveal how those emails are constructed, what data they contain, and how they are sent, which is a critical part of the booking workflow.
