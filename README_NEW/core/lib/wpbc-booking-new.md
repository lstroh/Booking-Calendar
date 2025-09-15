# File Analysis: `core/lib/wpbc-booking-new.php`

This document provides a detailed analysis of the `core/lib/wpbc-booking-new.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file appears to be a **deprecated or obsolete** component of the plugin. While its name and original purpose were related to creating new bookings, prominent `TODO` comments at the top of the file explicitly state that the new booking creation workflow has been moved to other files, specifically `bookings__actions.php`.

The file now contains only a single, complex legacy function, `wpbc_check_dates_intersections()`, which was designed to check for date and time conflicts. Another `TODO` comment suggests that even this function has been superseded by a newer one (`where_to_save`).

Therefore, this file should be considered an artifact of a previous architecture. It does not appear to be actively used in the current booking creation process and likely remains for backward compatibility or as a remnant of incomplete refactoring.

## Detailed Explanation

The file's entire content is the function `wpbc_check_dates_intersections()`.

-   **`wpbc_check_dates_intersections( $dates_for_check, $dates_exist )`**:
    -   **Purpose**: This function was designed to determine if a new set of booking dates (`$dates_for_check`) conflicts with an existing set of booked dates (`$dates_exist`).
    -   **Methodology**: It operates on arrays of date/time strings. It uses a custom and unconventional system where the last digit of the time's second component indicates a start or end time (e.g., `...:01` for a start time, `...:02` for an end time). The function manually adds or subtracts seconds from timestamps to create small gaps, then sorts the combined list of dates to check for invalid sequences (e.g., two "start time" markers in a row), which would indicate an overlap.
    -   **Complexity**: The logic is complex, difficult to follow, and relies on manual date string manipulation and `strtotime()` calls, which is generally less reliable than using modern `DateTime` objects.

-   **Evidence of Deprecation**:
    The most critical information in this file are the comments left by the developers:
    ```php
    //TODO: 2023-10-03 14:49  the new workflow have to be in bookings__actions.php  in line 1190
    ```
    This comment directly points to the new location for the booking creation workflow.
    ```php
    // TODO:  re-update this function: wpbc_check_dates_intersections. we need to  call the where_to_save function
    ```
    This comment confirms that `wpbc_check_dates_intersections` itself is outdated and should be replaced by a different function, which previous analysis has shown is located in `includes/_capacity/create_booking.php`.

## Features Enabled

Based on the evidence of deprecation, this file enables **no active features** in the current version of the plugin.

### Admin Menu
-   This file has no effect on the WordPress Admin Menu.

### User-Facing
-   This file has no effect on the user-facing side of the website. The actual booking creation is handled by more modern files.

## Extension Opportunities

-   **Recommendations**: It is strongly recommended **not to use, extend, or interact with this file in any way**. The logic is obsolete, and any attempt to use it would bypass the plugin's current, more robust availability and capacity checking systems.
-   **Potential Risks**: Using the `wpbc_check_dates_intersections()` function for any custom development would likely lead to incorrect availability checks, allowing double bookings, as it does not account for the plugin's modern capacity management, seasonal rates, and other advanced rules.

For any functionality related to creating or validating new bookings, developers should instead investigate the functions and hooks within `includes/_capacity/create_booking.php` and `includes/page-bookings/bookings__actions.php`.

## Next File Recommendations

Given that this file is a dead end, we should proceed by analyzing other core, unreviewed components of the plugin to better understand its key features.

1.  **`core/any/api-emails.php`**: This file has been consistently recommended as the location of the core Email API. Understanding how email templates are defined, parsed, and sent is crucial for grasping the plugin's notification workflow.
2.  **`core/admin/wpbc-gutenberg.php`**: To understand how the plugin integrates with the modern WordPress Block Editor (Gutenberg), this file is essential. It will show how the "Booking Form" block is registered, its attributes, and how it's rendered.
3.  **`core/sync/wpbc-gcal.php`**: This file likely contains the logic for the Google Calendar synchronization feature. Analyzing it would provide significant insight into how the plugin handles communication with external, third-party APIs.
