# File Analysis: core/wpbc-js.php

## High-Level Overview
This file is the master controller for all JavaScript within the Booking Calendar plugin. Its core responsibility is to manage the loading of JavaScript files and to pass data from the PHP backend to the JavaScript frontend. It defines the `WPBC_JS` class, which hooks into WordPress's `wp_enqueue_scripts` and `admin_enqueue_scripts` to conditionally load the necessary scripts for either the front-end or the admin panel.

A key function of this file is the localization of scripts, which makes server-side data (like translation strings, AJAX nonces, and settings) available to the client-side JavaScript code. It also includes important functionality for preventing common issues, such as removing conflicting scripts from other plugins and ensuring its own scripts are not loaded with `async` or `defer` attributes, which could break functionality.

## Detailed Explanation
The file's logic is organized into the `WPBC_JS` class and several helper functions.

- **`WPBC_JS::enqueue( $where_to_load )`**: This is the main orchestrator method. It is hooked into the appropriate WordPress action and executes a series of other functions:
  1.  `wpbc_js_load_vars()`: This function (and the corresponding `do_action( 'wpbc_define_js_vars' )` hook) is responsible for localizing scripts. It passes a PHP array of data to a JavaScript object (likely `wpbc_vars`), making it accessible in `.js` files.
  2.  `wpbc_js_load_libs()`: Enqueues common libraries, including jQuery and, for the admin panel, jQuery UI, color picker, and Thickbox.
  3.  `wpbc_js_load_files()`: This function enqueues all the plugin-specific and third-party JavaScript files.

- **`wpbc_js_load_files( $where_to_load )`**: This function contains the core `wp_enqueue_script` calls:
  - **Core & Third-Party**: It loads `wpbc_all.js` (a primary script), `popper.js` and `tippy.js` for tooltips, `jquery.datepick.wpbc.9.0.js` for the calendar, and other libraries like `imask.js` and `chosen.js`.
  - **Conditional Loading**: It uses `if ( 'admin' === $where_to_load )` to load `admin.js` and other admin-specific scripts only in the backend. Conversely, it loads `client.js` and related scripts only on the front-end or on specific admin pages that require them (like the "Add New Booking" page).

- **`WPBC_JS::filter_script_loader_tag( $tag, $handle, $src )`**: This is a critical technical function. It hooks into the `script_loader_tag` filter to programmatically remove `async` and `defer` attributes from the `<script>` tags of jQuery and all of the plugin's own scripts. This is a defensive measure to guarantee that these scripts are loaded and executed in a predictable order, preventing race conditions and "jQuery is not defined" errors.

- **`wpbc_remove_js_conflicts()`**: Similar to its CSS counterpart, this function explicitly calls `wp_deregister_script` to remove specific `.js` files from other plugins that are known to cause conflicts.

- **`wpbc_is_load_css_js_on_client_page( $is_load_scripts )`**: This function, hooked to a filter, implements the "Load JS/CSS only on specific pages" feature. It checks if the current page URL is in the user-defined list of pages where scripts should be loaded, preventing assets from loading unnecessarily on other front-end pages.

## Features Enabled
This file is the engine for all client-side interactivity in the plugin.

- **Front-End Interactivity**: It enables the entire booking process for the user, including clicking on the calendar to select dates, validating form fields, and submitting the booking via AJAX.
- **Admin Panel Dynamics**: It powers the interactive elements of the admin panel, such as the show/hide logic on the settings pages, the functionality of filters and action buttons on the booking listing page, and AJAX-based operations like approving or deleting bookings without a full page reload.
- **Data Bridge**: Through script localization, it acts as a bridge, providing front-end scripts with essential backend data like security nonces for AJAX, translated text, and configuration options.

## Extension Opportunities
- **`do_action( 'wpbc_enqueue_js_files', $where_to_load )`**: This is the designated hook for developers to safely enqueue their own custom JavaScript files. Scripts added here will load after the plugin's core scripts, ensuring dependencies like jQuery are available.

- **`do_action( 'wpbc_define_js_vars', $where_to_load )`**: This hook is the proper way to add custom data to the JavaScript environment. A developer can hook a function here that calls `wp_localize_script` to add their own settings, URLs, or other data to the `wpbc_vars` object or a custom object.

  ```php
  function my_custom_wpbc_js_vars() {
      wp_localize_script( 'wpbc-admin-main', 'my_vars', array(
          'my_setting' => 'some_value'
      ) );
  }
  add_action( 'wpbc_define_js_vars', 'my_custom_wpbc_js_vars' );
  ```

## Next File Recommendations
We have now seen how the JavaScript files are enqueued and how data is passed to them. The next logical step is to analyze the actual content of these JavaScript files to understand the client-side application logic.

1.  **`js/admin.js`**: **Top Priority.** This is the main JavaScript file for the entire admin panel. It will contain the jQuery code that handles UI interactions, AJAX requests for managing bookings, and the dynamic behavior of the settings pages.
2.  **`js/client.js`**: This is the primary script for the front-end. It controls the user-facing calendar, handles date selection logic, performs form validation, and manages the AJAX submission of the booking form.
3.  **`core/wpbc-js-vars.php`**: This file is highly likely to contain the main `wp_localize_script` call that defines the `wpbc_vars` object. Analyzing it will show us exactly what data is being passed from PHP to JavaScript, which is crucial for understanding the client-side scripts.
