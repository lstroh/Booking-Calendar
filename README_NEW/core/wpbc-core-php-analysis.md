# File Analysis: core/wpbc-core.php

## High-Level Overview

- **Role in Plugin:**  
  `core/wpbc-core.php` is a foundational utility file for the Booking Calendar plugin. It provides an internal hooks system (actions and filters, similar to WordPress core), option management wrappers, and booking meta option (custom fields) handlers.  
- **Connection to Architecture:**  
  It’s loaded early via `includes/wpbc-include.php`, making its functions available to all plugin modules. This file allows the plugin to maintain its own extensibility (actions/filters), abstract option storage, and handle per-booking metadata independently of WordPress’s built-in hooks.

---

## Detailed Explanation

### Key Functions, Classes, Hooks

- **Custom Actions and Filters:**
  - `add_bk_filter`, `apply_bk_filter`, `remove_bk_filter`
    - Allow modules to register, apply, and remove custom filter callbacks.
  - `add_bk_action`, `make_bk_action`, `remove_bk_action`
    - Similar for actions—lets modules trigger and listen for custom plugin events.
  - Example:
    ```php
    add_bk_filter('booking_status', function($status) { return $status === 'pending' ? 'Awaiting' : $status; });
    $status = apply_bk_filter('booking_status', 'pending');
    ```
- **Option Management Wrappers:**
  - `get_bk_option`, `update_bk_option`, `delete_bk_option`, `add_bk_option`
    - Wrap WordPress option functions (`get_option`, etc.), but allow interception via custom filters.
    - This enables overriding or extending option logic without modifying core code.
- **Booking Meta Option Handlers:**
  - `wpbc_save_booking_meta_option($booking_id, $option_arr)`
    - Saves/updates serialized meta data in the `booking_options` field in the custom DB table.
  - `wpbc_get_booking_meta_option($booking_id, $option_name = false)`
    - Retrieves meta options for a booking row, or a single option by name.

### Interaction with WordPress Core APIs / Database

- **Custom Tables:**  
  - Directly interacts with a custom table (`{$wpdb->prefix}booking`), storing and retrieving metadata in the `booking_options` column.
- **WordPress API Wrappers:**  
  - All option management goes through wrappers that can be filtered.
  - Uses `$wpdb` for raw SQL queries, with prepared statements for security.
- **No direct hooks to admin menu or frontend registration.**  
  - This file provides utility functions for other components to use, rather than registering menus, shortcodes, or blocks itself.

#### Example: Saving Booking Meta Option
```php
wpbc_save_booking_meta_option(123, ['locale' => 'de_DE', 'paid' => '100.00']);
```

---

## Features Enabled

### Admin Menu

- **Indirectly:**  
  - No direct registration of admin pages, notices, or widgets.
  - Underpins admin features by powering option and booking meta management for settings and booking details.

### User-Facing

- **Indirectly:**  
  - No direct registration of shortcodes, blocks, REST endpoints, or scripts.
  - Enables user-facing features (e.g., custom booking fields, translations, dynamic options) by providing the backend infrastructure.

---

## Extension Opportunities

- **Safe Extension Points:**
  - Use `add_bk_filter` and `add_bk_action` to hook into plugin events or data flows.
    ```php
    add_bk_action('booking_created', 'my_custom_function');
    ```
  - Override option logic using filters:
    ```php
    add_bk_filter('wpdev_bk_get_option', function($result, $option, $default) {
      if ($option === 'my_special_setting') return 'custom';
      return $result;
    });
    ```
- **Improvements/Modularization:**
  - Consider refactoring to use WordPress’s core hooks where possible for better compatibility.
  - Add error handling/logging for database operations.
  - Use strict typing where possible to reduce runtime errors.
- **Potential Risks:**
  - Direct use of `$wpdb` for meta options bypasses WP’s metadata API (could be less portable, harder to debug).
  - Overuse of global variables (`$wpdev_bk_action`, `$wpdev_bk_filter`) could lead to naming conflicts or unexpected side effects.
  - If the custom hooks system is not well documented, third-party developers may misuse or overlook it.

---

## Next File Recommendations

1. **core/wpbc-functions.php**  
   - *Reason:* Contains additional utility functions, possibly for bookings, dates, and plugin logic—builds on the primitives from this core file.
2. **includes/page-bookings/bookings__actions.php**  
   - *Reason:* Implements key booking actions (approve, deny, delete, etc.), likely uses meta option and hook functions from `wpbc-core.php`.
3. **core/wpbc-dates.php**  
   - *Reason:* Manages date and time logic for bookings, essential for understanding how availability, scheduling, and capacity are implemented.

*These files will reveal how the utility functions and extensibility points in `wpbc-core.php` are leveraged throughout the plugin’s admin and frontend features.*