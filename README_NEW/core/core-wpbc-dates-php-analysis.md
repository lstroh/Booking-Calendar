# File Analysis: core/wpbc-dates.php

## High-Level Overview

- **Summary:**  
  `core/wpbc-dates.php` is a core utility file that provides all date and time handling logic for the Booking Calendar plugin. It is responsible for parsing, formatting, converting, comparing, and validating booking dates and times, as well as calculating availability and interfacing with the database for booking-related date queries.
- **Role in Architecture:**  
  This file acts as the central "dates engine" for the plugin. It is used throughout booking creation/editing, calendar rendering, availability checking, form parsing, and admin workflows. By providing a consistent API for date management, it ensures accurate and flexible handling of bookings in both frontend and backend contexts.

---

## Detailed Explanation

### Key Functions, Classes, Hooks

- **Date Parsing and Conversion:**
  - `wpbc_get_dates_in_diff_formats($str_dates__dd_mm_yyyy, $booking_type, $booking_form_data)`
    - Converts date strings from various formats to arrays, adds time info, and sorts dates.
  - `wpbc_get_comma_seprated_dates_from_to_day($from, $to)`
    - Expands a date range into a comma-separated list of dates.
  - `wpbc_get_dates_array_from_start_end_days($start, $end)`
    - Returns an array of dates between two boundaries.
  - `wpbc_get_sorted_days_array($booking_days)`
    - Normalizes, sorts, and formats a date list for DB operations.
- **Time Handling:**
  - `wpbc_check_min_max_available_times($time, $min_time, $max_time)`
    - Validates that a time falls within allowed bounds.
  - `wpbc_get_times_in_form($booking_form_data, $booking_type)`
    - Extracts start/end/duration times from booking forms.
  - `wpbc_get_time_in_24_hours_format($time_str)`
    - Converts AM/PM times to 24-hour format.
  - `wpbc_time_slot_in_format`, `wpbc_time_in_format`
    - Formats time slots for display.
- **Date Comparison and Info:**
  - `wpbc_get_difference_in_days($day1, $day2)`
    - Calculates interval in days.
  - `wpbc_is_today_date($some_day)`, `wpbc_is_tomorrow_date($some_day)`, `wpbc_is_date_in_past($some_day)`
    - Day comparisons for UI logic or validations.
- **Database Interactions:**
  - `wpbc_db__get_sql_dates__in_booking__as_str($booking_id_str)`
    - Gets all booking dates (with time) for one or more bookings.
  - `wpbc__sql__get_booked_dates($params)`
    - Retrieves booked dates, with options for filtering by approval status, resource, and exclusions.
  - `wpbc__sql__get_season_availability($params)`
    - Computes season-based availability for a resource, integrating with season filters and calendar options.
- **Date Formatting:**
  - `wpbc_get_date_in_correct_format($dt, $date_format, $time_format)`
    - Returns formatted date/time for display, respecting plugin and WordPress settings.
  - `wpbc_get_short_dates_formated_to_show($dates, ...)`
    - Renders booking dates as HTML for admin panels.
- **Helpers for Range/Comma Separated Lists:**
  - `wpbc_get_dates_arr__from_dates_range($params)`, `wpbc_get_dates_arr__from_dates_comma_separated($params)`
    - Normalize and sort dates from various input formats.

#### Example: Expanding a Date Range
```php
wpbc_get_comma_seprated_dates_from_to_day('06.04.2015', '08.04.2015');
// returns '06.04.2015, 07.04.2015, 08.04.2015'
```

### Interaction with WordPress Core APIs or Database

- **Database Usage:**
  - Uses `$wpdb` for direct queries to custom tables (`booking`, `bookingdates`) to fetch, sort, and filter booking dates and statuses.
  - All queries are prepared for security.
- **WordPress APIs:**
  - Uses option APIs (`get_bk_option`), date/time functions (`gmdate`, `date_i18n`, `mysql2date`), and array utilities (`wp_parse_args`).
- **No direct registration of hooks, menus, or UI elements.**
  - This file is intended for backend logic and is called by other modules (admin, frontend, AJAX handlers).

---

## Features Enabled

### Admin Menu

- **Indirectly:**  
  - Provides all date and time data for booking listings, filters, and calendar displays in the admin.
  - Supports HTML rendering of dates for admin tables and meta boxes.
  - Enables backend availability calculation for resources and seasons (affects settings and resource management pages).

### User-Facing

- **Indirectly:**  
  - All calendar rendering, date selection, and availability checking on the frontend relies on these functions.
  - Ensures accurate date formatting and validation for booking forms and shortcodes.
  - Supports dynamic calculation of available dates based on seasons, approval status, and resource filters (affects what users see as "available" or "booked").

---

## Extension Opportunities

- **Recommended Safe Extension Points:**
  - Add new date parsing or formatting functions for custom calendar needs.
  - Use or extend existing functions for new booking fields or custom workflows.
  - Hook into availability calculation by filtering/enhancing `wpbc__sql__get_season_availability` or integrating with season filters.
  - Add new filters or actions to allow plugins/themes to modify the output, especially around calendar rendering or availability.

- **Suggested Improvements or Modularization:**
  - Refactor repetitive date parsing logic into reusable utilities.
  - Add PHP type hints and more robust error handling for invalid dates/times.
  - Consider using WP’s metadata API for booking meta instead of raw SQL.
  - Separate database-dependent logic from pure PHP helpers for easier testing.

- **Potential Risks:**
  - Direct `$wpdb` queries are prone to errors and require careful sanitization.
  - Changes may affect core booking logic—thorough regression testing is vital.
  - Heavy reliance on date string formats; changes must preserve backwards compatibility with stored booking data.

---

## Next File Recommendations

1. **includes/page-bookings/bookings__actions.php**  
   - *Reason:* Implements booking status transitions (approve, pending, cancel), uses date logic from this file.
2. **core/wpbc-functions.php**  
   - *Reason:* Contains broader utility functions for booking, UI, and admin logic, likely calls date functions for workflow and interface.
3. **core/wpbc-core.php**  
   - *Reason:* Houses the core hooks and options system which date logic may interact with or be extended from.

*These files collectively drive the booking workflow, admin UI, and extensibility of the plugin, leveraging the date engine provided by `core/wpbc-dates.php`.*