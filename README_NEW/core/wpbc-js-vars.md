# File Analysis: core/wpbc-js-vars.php

## High-Level Overview
This file acts as the primary data bridge between the server-side PHP environment and the client-side JavaScript. Its sole purpose is to gather a wide range of data—such as configuration settings, translated text, and timezone information—and inject it into the page as a JavaScript object. 

Instead of using the standard `wp_localize_script` function in a simple way, this file implements a more robust and custom solution. It generates a block of inline JavaScript that populates a global `_wpbc` object. This script is then wrapped in a sophisticated "bootstrap" loader that ensures it runs correctly even on sites with aggressive caching or deferred JavaScript loading, preventing common race condition errors.

## Detailed Explanation

### Data Aggregation: `wpbc_get_localized_js_vars()`
This function is the heart of the file. It builds a large string of JavaScript code that populates the global `_wpbc` object with data. It does this by making a series of calls to custom methods on that object:

- **`_wpbc.set_other_param('key', 'value')`**: This is used to pass configuration settings and environmental data. Key data points include:
  - **Time & Locale**: The current locale, GMT time, and local time, allowing the calendar to be timezone-aware.
  - **Calendar Settings**: Core settings from the database, such as `booking_start_day_weeek`, `booking_max_monthes_in_calendar`, unavailable weekdays, and the complex rules for day-selection modes (single, multiple, range).
  - **URLs & Version**: The plugin URL and current version number.

- **`_wpbc.set_message('key', 'value')`**: This is used to pass all translatable strings needed for client-side messages. This includes form validation errors, confirmation dialogs, and processing messages.

```javascript
// Example of the generated JavaScript string
_wpbc.set_other_param( 'locale_active', 'en_US' ); 
_wpbc.set_other_param( 'calendars__first_day', '1' ); 
_wpbc.set_message( 'message_check_required', "This field is required" ); 
_wpbc.set_message( 'message_processing', "Processing" );
```

### Script Injection: `wpbc_localize_js_vars()`
This function is hooked into the `wpbc_enqueue_js_files` action (with a priority of 51) and is responsible for injecting the JavaScript into the page.

1.  **`wp_add_inline_script( ..., 'before' )`**: It first adds the `wpbc_url_ajax` variable before the main `wpbc_all.js` script. This is critical, as it ensures the AJAX URL is available immediately.
2.  **Bootstrap Loader**: It then wraps the main data payload (from `wpbc_get_localized_js_vars`) inside a sophisticated bootstrap script. This loader is designed to be highly resilient:
    - It checks if the `window._wpbc` object is already available. If so, it runs the data injection immediately.
    - If not, it starts polling every 50ms to wait for the main `wpbc_all.js` script to load and create the `_wpbc` object.
    - It also listens to a wide range of browser and custom events (`DOMContentLoaded`, `load`, `click`, `scroll`, `wpbc-ready`) to trigger the data injection as early as possible.
    - This robust mechanism ensures that the data is correctly loaded regardless of whether other plugins are deferring or delaying JavaScript execution.

## Features Enabled
This file is a foundational component that enables almost all client-side functionality:

- **Dynamic Configuration**: It makes the client-side scripts aware of the backend settings, allowing the calendar and forms to be configured dynamically.
- **Internationalization (i18n)**: It provides all the necessary translated strings for JavaScript-driven alerts, messages, and UI text.
- **Timezone Awareness**: It gives the calendar the data it needs to correctly handle different timezones.
- **AJAX Capability**: It provides the client-side scripts with the AJAX endpoint URL (`admin-ajax.php`).
- **Compatibility & Resilience**: The bootstrap loader ensures the plugin's JavaScript works reliably even in complex environments with caching and script optimization.

## Extension Opportunities

- **`wpbc_js_vars` Filter**: The `$wpbc_js_vars` array is passed through this filter before being converted to JavaScript. This allows a developer to easily add, modify, or remove variables from the main `wpbc_vars` object.

  ```php
  function add_my_vars_to_wpbc( $wpbc_vars ) {
      $wpbc_vars['my_custom_var'] = 'my_value';
      return $wpbc_vars;
  }
  add_filter( 'wpbc_js_vars', 'add_my_vars_to_wpbc' );
  ```

- **`wpbc_define_js_vars` Action**: This action hook runs just before the main localization script. A developer can use it to call `wp_localize_script` themselves, creating a separate, custom data object for their own scripts if they prefer not to modify the main `wpbc_vars` object.

## Next File Recommendations
We have now fully dissected the plugin's backend architecture and the data bridge to the frontend. The clear next step is to analyze the client-side JavaScript that consumes all this data.

1.  **`js/client.js`**: **Top Priority.** This is the main JavaScript file for the front-end. It will use the `_wpbc` object to configure the calendar, handle date selections, validate the booking form, and submit the final booking via AJAX.
2.  **`core/lib/wpbc-booking-new.php`**: When a visitor submits a booking, an AJAX request is made. This file likely contains the server-side PHP logic for receiving this request, validating the submitted data, and creating the new booking in the database.
3.  **`wpdev-booking.php`**: It would be beneficial to quickly review the main plugin file again to see how the `[booking]` shortcode is registered. This shortcode is the initial entry point that renders the HTML structure for the booking form, which `js/client.js` then brings to life.
