# File Analysis: `js/wpbc_times.js`

## High-Level Overview

This file is a client-side JavaScript library responsible for advanced time-slot validation and for displaying informational hints within the calendar date cells. It works in conjunction with the main calendar script (`jquery.datepick.wpbc.9.0.js`) and the form submission script (`client.js`). Its primary role is to ensure that if a user selects a specific time range in the booking form, that range is actually available for the selected dates, providing real-time validation before the form is submitted via AJAX.

Architecturally, it acts as a validation and UI-enhancement layer. It reads booking data (including already booked time slots) from the global `_wpbc` JavaScript object and uses this data to perform complex checks. It also enhances the calendar's UI by providing functions to display at-a-glance information, like cost or availability counts, directly within the date cells.

## Detailed Explanation

The file consists of several globally-scoped functions that can be grouped into two main categories: UI display and time validation.

### UI Display Functions

-   **`wpbc_show_date_info_top( param_calendar_id, my_thisDateTime )`**: This function is designed to be a callback for the datepicker. It calculates and displays dots (`&centerdot;`) at the top of a date cell to indicate partially booked time slots. It checks the booked `merged_seconds` for each child resource on a given day and displays a number of dots corresponding to the maximum number of separate bookings in any single resource.

-   **`wpbc_show_date_info_bottom( param_calendar_id, my_thisDateTime )`**: This is another datepicker callback for displaying information at the bottom of a date cell. It aggregates hints from two other functions:
    -   `wpbc_show_day_cost_in_date_bottom()`: (If defined elsewhere) Displays the cost for the day.
    -   `wpbc_get_in_date_availability_hint()`: Displays the number of available items for that day (e.g., "5 available"), retrieving this hint from the main `_wpbc` data object.

### Time Validation Logic

-   **`wpbc_is_time_field_in_booking_form( resource_id, form_elements )`**: A utility function that scans the booking form to detect if any time-related fields (`rangetime`, `durationtime`, `starttime`, `endtime`) are present. This is used by other functions to determine if time validation logic needs to be executed.

-   **`wpbc_is_this_time_selection_not_available( resource_id, form_elements )`**: This is the core validation function, likely called before form submission. It orchestrates the entire time validation process:
    1.  It parses the booking form to find the user's selected start time, end time, or duration.
    2.  If a duration is provided instead of an end time, it calculates the end time.
    3.  It then calls helper functions (`checkTimeInside` or `checkRecurentTimeInside`) to validate the calculated time range against the booked slots for the selected dates.
    4.  If the validation fails, it uses `wpbc_front_end__show_message__warning_under_element()` to display a specific error message (e.g., "This time is not available") directly underneath the problematic time field.

-   **`checkTimeInsideProcess( mytime, is_start_time, bk_type, my_dates_str )`**: This is the low-level validation engine. It retrieves the booked time slots for a given day from the `_wpbc` data object, compares the user's selected time against these booked intervals, and returns `true` or `false` based on whether the time is available. Its logic is complex, designed to handle start times, end times, and intervals that span across midnight.

-   **`isTimeTodayGone( myTime, sort_date_array )`**: A helper function that checks if a selected time on the current day has already passed, preventing users from booking in the past.

## Features Enabled

This file is exclusively for the front-end and provides no admin panel features.

### User-Facing

-   **In-Cell Calendar Hints**: It provides the logic for displaying dynamic information directly within the calendar date cells, such as dots for partially booked days, daily costs, and the number of available slots. This gives users at-a-glance information without needing to click.
-   **Client-Side Time Validation**: It performs instant, client-side validation of selected time slots, start/end times, or duration. This prevents users from submitting a form with an unavailable time, improving the user experience and reducing server-side errors.
-   **Contextual Error Messages**: When a time validation fails, it displays a targeted error message directly below the specific form field that is causing the issue, making it easy for the user to correct their selection.

## Extension Opportunities

-   **Reusing Validation Functions**: The validation functions like `wpbc_is_this_time_selection_not_available` are global. A developer could call this function from a custom script to trigger validation on a different event (e.g., as soon as a user changes a time field, instead of waiting for form submission).

-   **Filtering Data Server-Side**: The script's logic is entirely dependent on the data provided in the global `_wpbc` object. The most robust way to influence this script's behavior is to use PHP filters on the server-side to modify the booking data (e.g., the `booked_time_slots` array) before it is passed to the client. This would allow a developer to add custom unavailable times or modify existing ones.

-   **Customizing Messages**: The error messages are retrieved from the `_wpbc` object via `_wpbc.get_message()`. This means they can be translated or customized using the plugin's standard localization mechanisms.

## Next File Recommendations

Now that we understand the client-side logic for time selection, the next logical steps are to analyze the CSS that styles this interface and to explore other un-analyzed client-side scripts.

1.  **`css/wpbc_time-selector.css`**: This is the stylesheet that almost certainly styles the time-slot selection interface that appears when a user clicks on a date. It is the direct visual counterpart to the logic in this file.
2.  **`js/wpbc_tinymce_btn.js`**: This file is responsible for integrating the plugin with the Classic WordPress editor (TinyMCE). It will show how the "Add Booking Calendar" button and its associated modal for generating shortcodes are created, which is important for understanding backward compatibility.
3.  **`js/user-data-saver.js`**: This un-analyzed file could be related to the booking form. It might handle client-side persistence of user-entered data to prevent loss on page reloads, which is an important usability feature.
