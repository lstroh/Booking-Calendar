# File Analysis: `core/wpbc-debug.php`

This document provides a detailed analysis of the `core/wpbc-debug.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file is a collection of utility functions dedicated to debugging, error handling, and performance monitoring. It is not part of the plugin's core business logic (like booking creation or calendar rendering) but serves as an essential toolkit for developers working on the plugin.

The functions within this file provide standardized ways to:
-   Print the contents of variables for inspection (`debuge`).
-   Display formatted error messages with context like file, line number, and the last database error (`debuge_error`).
-   Check server configurations for potential issues that could affect the plugin (`wpbc_check_post_key_max_number`).
-   Show dynamic, dismissible notices in the WordPress admin panel (`wpbc_admin_show_top_notice`).

It is a developer-centric file, crucial for troubleshooting and ensuring the plugin runs smoothly in different server environments.

## Detailed Explanation

The file consists of several globally-scoped functions, each wrapped in a `! function_exists()` check to prevent errors if the file is loaded more than once.

-   **`debuge()`**: A variable-dumping function for quick debugging. It accepts any number of arguments and prints their type and value, wrapped in `<pre>` tags for readability. It has a "dump and die" feature: if the last argument passed is `true`, it will stop PHP execution, which is useful for inspecting state at a specific point in the code.

-   **`debuge_error()` / `wpbc_get_debuge_error()`**: These functions generate a formatted HTML error message. The message includes the file and line number where the error occurred and, notably, attempts to retrieve the last database error from a global `$EZSQL_ERROR` variable. This indicates an integration with or legacy support for the EZSQL database library.

-   **`debuge_speed()`**: A simple performance utility that echoes the total number of database queries (`get_num_queries()`) and the script execution time (`timer_stop()`) for the current page load.

-   **`wpbc_check_post_key_max_number()`**: A specific and important diagnostic function. It checks for PHP configuration limits imposed by the **Suhosin** security extension. It iterates through all submitted `$_POST` data and checks if any key names are longer than the limits defined in `php.ini` (e.g., `suhosin.post.max_name_length`). If a limit is exceeded, it displays a detailed error message in the admin panel, guiding the site administrator on how to fix their server configuration. This is excellent defensive programming to prevent settings from failing to save due to server-specific security rules.

-   **`wpbc_admin_show_top_notice()`**: A helper function for displaying dismissible notices (info, success, warning, error) at the top of admin pages. Instead of using the standard WordPress admin notices API, it injects a small block of JavaScript that calls a client-side function (`wpbc_admin_show_message`) to display the notice. This allows for more dynamic, temporary notices. The function is also hooked into the `wpbc_admin_show_top_notice` action, allowing it to be called easily from other parts of the plugin via `do_action`.

## Features Enabled

This file is primarily for developers and administrators; it provides no direct features for front-end users.

### Admin Menu

-   This file does not add any admin menu pages.
-   It provides the underlying mechanism for displaying error messages and system notices. For example, the `wpbc_check_post_key_max_number()` function will generate a visible error on a settings page if a Suhosin configuration issue is detected.
-   The `wpbc_admin_show_top_notice` function is used by other parts of the plugin to show dynamic feedback to the administrator after performing an action.

### User-Facing

-   This file has no user-facing features. The `show_debug()` function appears intended for front-end debugging but is currently restricted to admin AJAX requests and marked with a `TODO` for refactoring.

## Extension Opportunities

-   **Calling Functions Directly**: As the functions are global, developers extending the plugin can use them for their own debugging needs (e.g., calling `debuge($my_variable, true);` to inspect data).
-   **Using Action Hooks**:
    -   `do_action( 'wpbc_admin_show_top_notice', 'My message', 'success' );` can be used to trigger a custom admin notice from another function.
    -   `do_action( 'wpbc_show_debug', $my_array );` can be used to log information to the screen if the debug listener (`wpbc_start_showing_debug`) is active.
-   **Potential Risks**: These are powerful debugging tools. They should **never** be left in production code. Functions like `debuge()` can expose sensitive server information and will break AJAX/JSON responses by outputting HTML. The "dump and die" feature (`debuge(..., true)`) will halt execution for all users.

## Next File Recommendations

This file provides insight into the plugin's development and error-handling tools. To continue understanding the core application logic, I recommend returning to the files that manage the plugin's structure and data flow.

1.  **`core/any/class-admin-menu.php`**: This file is responsible for building the plugin's admin menu. Analyzing it is the next logical step to understanding the overall structure of the backend interface.
2.  **`core/lib/wpbc-booking-new.php`**: This file most likely handles the server-side logic for creating a new booking after a form is submitted. It's a critical file for understanding the core booking lifecycle.
3.  **`includes/page-form-simple/form_simple.php`**: We've seen how forms are parsed; this file is the likely candidate for taking the parsed data and rendering the actual HTML form fields for the user.
