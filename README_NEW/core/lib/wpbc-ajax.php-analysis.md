# File Analysis: core/lib/wpbc-ajax.php

## High-Level Overview
This file is the central AJAX request router for the plugin. It contains the server-side PHP functions that handle client-side requests for actions like approving a booking, deleting it, or moving it to the trash. It acts as the bridge between the interactive admin panel (driven by `js/admin.js`) and the WordPress database.

The file defines a series of functions, each hooked to a specific WordPress AJAX action (`wp_ajax_{action}`). When an AJAX request is received, the corresponding function is executed. The standard workflow for each function is to: 1) verify security (nonce), 2) process the request (e.g., update a database record), 3) perform side-effects (like sending an email), and 4) `echo` a JavaScript response back to the client to update the UI.

## Detailed Explanation

### AJAX Action Registration
The file uses a centralized and extensible method to register its AJAX endpoints. At the bottom of the file, an array `$actions_list` maps action names to their context (`admin`, `client`, or `both`). The code then iterates through this array and uses `add_action()` to hook the handler functions to `wp_ajax_*` (for logged-in users) and `wp_ajax_nopriv_*` (for visitors).

```php
// At the end of the file
$actions_list = array(
    'UPDATE_APPROVE' => 'admin',
    'TRASH_RESTORE'  => 'admin',
    'DELETE_APPROVE' => 'admin',
    'BOOKING_SEARCH' => 'both'
    // ... and so on
);

$actions_list = apply_filters( 'wpbc_ajax_action_list', $actions_list );

foreach ($actions_list as $action_name => $action_where) {
    // ... logic to add wp_ajax_ hooks ...
}
```

### Core Handler Functions
The file defines the functions that execute the core booking management logic.

- **`wpbc_ajax_UPDATE_APPROVE()`**: This function handles approving or pending a booking.
  - It first checks for a valid nonce using `wpbc_check_nonce_in_admin_panel()`.
  - It retrieves the booking ID(s) and the new status from the `$_POST` array.
  - It executes a direct, prepared `$wpdb->query()` to update the `approved` status in the `{$wpdb->prefix}bookingdates` table.
  - It then calls helper functions to send notification emails (`wpbc_send_email_approved` or `wpbc_send_email_deny`).
  - Finally, it `echo`es a `<script>` block back to the browser. This script contains calls to the JavaScript functions in `admin.js` (e.g., `set_booking_row_approved(123);`), which then update the UI.

- **`wpbc_ajax_TRASH_RESTORE()`**: Handles moving bookings to or from the trash. It follows the same pattern: check nonce, get data, run a direct `UPDATE` query on the `{$wpdb->prefix}booking` table to set the `trash` flag, and echo a JavaScript response (`set_booking_row_trash(...)` or `set_booking_row_restore(...)`).

- **`wpbc_ajax_DELETE_APPROVE()`**: Handles permanent deletion. It checks the nonce, gets the booking ID(s), and runs direct `DELETE` queries against both the `bookingdates` and `booking` tables. It then echoes a JavaScript response to remove the booking from the UI.

### Security
Every admin-facing AJAX function begins with a call to `wpbc_check_nonce_in_admin_panel()`, which is a wrapper for WordPress's nonce verification system. This is a critical security measure to prevent Cross-Site Request Forgery (CSRF) attacks.

## Features Enabled
This file provides the server-side power for all the real-time, interactive features in the admin panel, including:
- Approving, pending, trashing, and deleting bookings without requiring a page refresh.
- Saving user interface preferences, like the open/closed state of meta boxes (`wpbc_ajax_USER_SAVE_WINDOW_STATE`).
- The server-side component of the front-end availability search.

## Extension Opportunities
This file provides several excellent, well-defined extension points.

- **`wpbc_ajax_action_list` filter**: This filter allows a developer to register their own custom AJAX actions within the plugin's ecosystem. This is the ideal way to add a new AJAX endpoint.

  ```php
  function my_custom_ajax_actions( $actions_list ) {
      $actions_list['MY_CUSTOM_ACTION'] = 'admin';
      return $actions_list;
  }
  add_filter( 'wpbc_ajax_action_list', 'my_custom_ajax_actions' );
  
  // Then, define the handler function and hook it
  function wpbc_ajax_MY_CUSTOM_ACTION() { /* ... */ }
  add_action( 'wp_ajax_MY_CUSTOM_ACTION', 'wpbc_ajax_MY_CUSTOM_ACTION' );
  ```

- **Action Hooks**: The handler functions contain specific `do_action` calls at key moments, such as `do_action( 'wpbc_booking_approved', ... )`, `do_action( 'wpbc_booking_trash', ... )`, and `do_action( 'wpbc_booking_delete', ... )`. These are perfect for triggering side-effects. For example, you could hook into `wpbc_booking_approved` to sync newly approved bookings with an external CRM or calendar service.

## Next File Recommendations
We have now followed a complete AJAX request-response cycle. To complete our understanding of the plugin's data flow and architecture, we should look at these final key files.

1.  **`core/wpbc-js-vars.php`**: **Top Priority.** This is the last piece of the AJAX puzzle. It will define how the `wpbc_url_ajax` variable and security nonces are generated in PHP and passed to the client-side JavaScript, making these AJAX calls possible.
2.  **`js/client.js`**: We have a very clear picture of the admin-side functionality. It is now time to analyze the main JavaScript file for the front-end, which controls the entire booking and calendar experience for the visitor.
3.  **`core/wpbc-sql.php`**: The AJAX handlers in this file perform direct database queries, but other parts of the plugin likely use a more abstracted method. This file probably contains a set of functions for more complex and reusable database queries, serving as a data access layer.
