# File Analysis: `core/admin/page-email-approved.php`

This document provides a detailed analysis of the `core/admin/page-email-approved.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file is a concrete implementation of the plugin's email templating system. Its primary purpose is to define and manage the "Approved" email—the notification sent to a visitor when their booking is confirmed.

Architecturally, this file is an excellent example of the plugin's object-oriented design. It defines a `WPBC_Emails_API_Approved` class that extends the abstract `WPBC_Emails_API` (from `api-emails.php`) to create the specific fields and default content for the "Approved" email. It also defines a `WPBC_Settings_Page_Email_Approved` class that builds the user interface for this template under **Booking > Settings > Emails**. Finally, it contains the `wpbc_send_email_approved()` function, which is the trigger that gathers booking data and sends the email.

## Detailed Explanation

The file has three distinct responsibilities:

1.  **Legacy Data Import**:
    -   The file contains several functions prefixed with `wpbc_import6_`. These are used for backward compatibility.
    -   `wpbc_import6_email__approved__get_fields_array_for_activation()`: This function checks if old email settings (from versions prior to the API) exist in the database (e.g., `booking_email_approval_content`). If they do, and new settings do not, it transforms the old data into the new structured format required by the `WPBC_Emails_API`. This ensures a smooth upgrade path for existing users.

2.  **Email Template Definition (`WPBC_Emails_API_Approved`)**:
    -   This class extends `WPBC_Emails_API`.
    -   **`init_settings_fields()`**: This is the core method where the specific fields for the "Approved" email template are defined. It creates fields for:
        -   Enabling/disabling the notification.
        -   Sending copies to additional admin emails.
        -   "From" / "To" names and addresses.
        -   Email `subject` and `content` (using a `wp_textarea` for the rich editor).
        -   Styling options like `template_file`, `base_color`, and `text_color`.
        -   A "Help" section that dynamically lists available shortcodes.

3.  **Admin Page and Sending Logic**:
    -   **`WPBC_Settings_Page_Email_Approved`**: This class builds the admin page. It adds the "Approved" sub-tab to the "Emails" settings page and uses its `mail_api()` instance to render the form fields within meta boxes.
    -   **`wpbc_send_email_approved()`**: This is the function called elsewhere in the plugin when a booking's status is changed to "approved." It performs the following steps:
        1.  Retrieves the relevant booking data from the database.
        2.  Instantiates `WPBC_Emails_API_Approved` to load the saved template settings.
        3.  Calls `wpbc__get_replace_shortcodes__email_approved()` to generate an array of dynamic values (like `[id]`, `[dates]`, `[cost]`, and all form field data).
        4.  Retrieves the visitor's email from the form data.
        5.  Calls the `$mail_api->send()` method, passing the recipient's email and the array of shortcode replacements to send the final, processed email.

## Features Enabled

### Admin Menu

-   This file is responsible for adding the **"Approved"** sub-tab to the **Booking > Settings > Emails** page.
-   It creates the entire settings interface for this specific email template, allowing administrators to customize its content, subject, and styling.

### User-Facing

-   This file's primary user-facing feature is the **sending of the booking approval email** to the visitor. This is a critical piece of the booking workflow, confirming the user's reservation.

## Extension Opportunities

The file itself is a concrete implementation, but it leverages the highly extensible `WPBC_Emails_API`.

-   **Adding Custom Shortcodes**: The `wpbc__get_replace_shortcodes__email_approved` function contains a powerful filter, `wpbc_replace_params_for_booking`. A developer can hook into this filter to add their own custom shortcodes that can be used in the "Approved" email template.

    ```php
    function my_custom_approved_email_shortcodes( $replace, $booking_id ) {
        // Get some custom data related to the booking
        $my_data = get_post_meta( $booking_id, 'my_custom_field', true );

        // Add a new shortcode and its value to the array
        $replace['my_shortcode'] = $my_data;

        return $replace;
    }
    add_filter( 'wpbc_replace_params_for_booking', 'my_custom_approved_email_shortcodes', 10, 2 );
    ```
    An admin could then use `[my_shortcode]` in the email template.

-   **Modifying Email Content**: As with all emails using the API, developers can use filters like `wpbc_email_api_get_content_after` or `wpbc_email_api_is_allow_send` to modify the final content or prevent the email from being sent based on custom logic.

## Next File Recommendations

We have now seen a complete, end-to-end implementation of a specific email template. To continue exploring major plugin functionality, the following unreviewed files are recommended:

1.  **`core/admin/wpbc-gutenberg.php`**: This is the top priority for understanding modern WordPress integration. It will define how the plugin provides a "Booking Form" block for the Gutenberg editor, which is the modern alternative to the classic widget.
2.  **`core/sync/wpbc-gcal.php`**: This file is responsible for the Google Calendar synchronization feature. Analyzing it is crucial for understanding how the plugin handles complex, authenticated interactions with a major third-party API.
3.  **`core/admin/wpbc-toolbars.php`**: The admin UI, particularly the Booking Listing, relies on toolbars for filtering and actions. This file likely defines those toolbars and is key to understanding the construction of the main admin pages.
