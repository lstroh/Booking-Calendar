# File Analysis: `js/user-data-saver.js`

## High-Level Overview

This file provides a generic, reusable, and declarative system for saving user-specific data to the WordPress database via AJAX. It is not tied to a specific feature (like booking creation) but acts as a utility that can be used by any component, such as a "Save View" button, that needs to persist user preferences.

Architecturally, it's a powerful client-side utility that allows developers to implement "save" functionality purely through HTML `data-*` attributes, without writing any custom JavaScript. The script reads its entire configuration from the element that triggers it, serializes the specified data, and sends it to a standard AJAX endpoint for processing.

## Detailed Explanation

The file's logic is contained within a single main function, `wpbc_save_custom_user_data_from_element(el)`, which is designed to be called from an `onclick` event on an HTML element.

### Declarative, Attribute-Driven Configuration

The function's behavior is controlled entirely by `data-*` attributes on the HTML element (`el`) that is passed to it.

-   `data-wpbc-u-save-user-id`: The ID of the user for whom to save the data.
-   `data-wpbc-u-save-nonce`: The security nonce for the AJAX action.
-   `data-wpbc-u-save-action`: The nonce action name.
-   `data-wpbc-u-save-name`: The name of the user meta option where the data will be stored (e.g., `booking_custom_add_booking_calendar_options`).
-   `data-wpbc-u-save-fields`: A comma-separated string of CSS selectors pointing to the form fields whose values should be saved.
-   `data-wpbc-u-save-value`: An alternative for saving a single, direct value instead of form fields.
-   `data-wpbc-u-save-value-json`: An alternative for saving a complex value from a JSON string.
-   `data-wpbc-u-save-callback`: The name of a global JavaScript function to be executed upon a successful response.

### Data Serialization Logic

The function has a clear order of precedence for what data it serializes:
1.  It first looks for `data-wpbc-u-save-value-json`. If found, it parses the JSON and serializes it.
2.  If not, it looks for `data-wpbc-u-save-value` and serializes that single value.
3.  If neither is present, it falls back to `data-wpbc-u-save-fields`. It splits the string of selectors, finds each form field on the page, gathers its value, and serializes the resulting key-value pairs.

### AJAX Request

-   The function sends a `POST` request to the AJAX URL defined in the global `WPBC_UserDataSaver.ajax_url` object (which is localized from PHP).
-   The `action` for the request is also retrieved from `WPBC_UserDataSaver.action`.
-   It sends the `user_id`, `nonce`, `data_name` (the meta key), and the serialized `data_value` to the server.
-   The server-side handler for this action is expected to use `update_user_meta()` to save the data.

### Extensibility via Custom Events

The script provides two custom jQuery events for hooking into the save process:
-   `wpbc:userdata:beforeSave`: Fires just before the AJAX request is sent. It passes the triggering element and the serialized data, allowing a developer to perform actions like disabling the button.
-   `wpbc:userdata:afterSave`: Fires after the AJAX request completes, passing the entire server response. This can be used for custom logging, showing success messages, or re-enabling UI elements.

## Features Enabled

This file is a utility and does not enable a single, specific feature. Instead, it is the engine that powers any "save user preference" feature in the admin panel.

### Admin Menu

-   This script provides the functionality for the "Save" and "Reset" buttons found on toolbars, such as the "Calendar View" toolbar on the **Bookings > Add New** page. It allows the plugin to remember user-specific UI preferences (like the number of months to show in a calendar) across sessions by saving them to the `wp_usermeta` table.

### User-Facing

-   This file is intended for the admin panel and has no user-facing features.

## Extension Opportunities

-   **Declarative Use (Primary Method)**: The intended way to use this script is declaratively. A developer can add a button to any admin page with the necessary `data-wpbc-u-save-*` attributes to save any combination of form fields to any user meta option, all without writing a single line of new JavaScript.

-   **Event-Based Hooks (Advanced)**: The `wpbc:userdata:beforeSave` and `wpbc:userdata:afterSave` events are the correct, update-safe way to add custom client-side logic to the save process. The commented-out examples in the file provide a clear template for how to use them.

    ```javascript
    // Example of hooking into the 'afterSave' event
    jQuery( document ).on( 'wpbc:userdata:afterSave.myLogger', function ( e, response ) {
        if ( response.success ) {
            alert( 'Your custom settings were saved!' );
        } 
    } );
    ```

## Next File Recommendations

This analysis clarifies how user-specific data is saved on the client side. The next logical steps are to investigate the server-side counterpart and other un-analyzed core features.

1.  **Server-Side AJAX Handler**: The next step would be to find the PHP file that registers the AJAX action specified in the `WPBC_UserDataSaver.action` variable and contains the `update_user_meta` logic. A search for `WPBC_UserDataSaver` in the PHP files would likely reveal this. Since I cannot search, I will prioritize other files.
2.  **`includes/page-settings-form-options/page-settings-form-options.php`**: This file is a high priority. It is responsible for the admin settings page where users configure the booking form fields themselves. Understanding this is key to learning how the plugin allows for form customization.
3.  **`js/wpbc_phone_validator.js`**: This file provides a specific piece of form functionality: phone number validation. It's a good, self-contained example of how the plugin handles input validation and enhances the user experience on specific fields within the booking form.
