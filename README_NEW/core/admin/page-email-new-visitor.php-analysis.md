# File Analysis: `core/admin/page-email-new-visitor.php`

This document provides a detailed analysis of the `core/admin/page-email-new-visitor.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file defines the implementation for the "New Booking" email template that is sent to the **visitor** immediately after they submit a booking. This is the initial confirmation email that the user receives, acknowledging their request.

Architecturally, this file is the final primary example of the plugin's consistent, object-oriented Email API pattern. It extends the abstract `WPBC_Emails_API` class to define the unique fields and default content for the visitor notification. It also creates the corresponding admin interface under **Booking > Settings > Emails** for customization and contains the final sending logic.

## Detailed Explanation

The file's structure and logic are consistent with the other email template files, divided into three main responsibilities:

1.  **Legacy Data Import**:
    -   The file begins with `wpbc_import6_...` functions, which are responsible for backward compatibility.
    -   `wpbc_import6_get_old_email_new_visitor_data()`: This function looks for old option names like `booking_email_newbookingbyperson_content`. It imports these legacy settings into the new API-driven format, ensuring a seamless upgrade for users of older plugin versions.

2.  **Email Template Definition (`WPBC_Emails_API_NewVisitor`)**:
    -   This class `extends WPBC_Emails_API`.
    -   **`init_settings_fields()`**: This core method defines the specific settings fields for this email template. The fields are structurally identical to other email templates (`enabled`, `from`, `subject`, `content`, styling options, etc.).
    -   A key difference from the admin notification is that this template does not have a "To" field in its settings; the recipient is dynamically determined from the `email` field in the submitted booking form.
    -   The default content is tailored for visitors and includes visitor-specific shortcodes like `[visitorbookingediturl]` (if the premium version is active), which allows users to view and manage their booking.

3.  **Admin Page and Sending Logic**:
    -   **`WPBC_Settings_Page_Email_NewVisitor`**: This class builds the admin UI. It hooks into the plugin's menu system to add the **"New Booking (visitor)"** sub-tab to the **Booking > Settings > Emails** page.
    -   **`wpbc_send_email_new_visitor()`**: This is the function that executes the email sending, called immediately after a booking is successfully saved.
        -   It fetches the new booking's data from the database.
        -   It instantiates `WPBC_Emails_API_NewVisitor` to load the customized template settings.
        -   It calls `wpbc__get_replace_shortcodes__email_new_visitor()` to gather all the dynamic data for the shortcodes.
        -   It dynamically determines the recipient's email address by looking for the `email` key in the returned shortcode replacement array (`$replace['email']`).
        -   It then calls the API's generic `$mail_api->send()` method to dispatch the final email to the visitor.

## Features Enabled

### Admin Menu

-   This file adds the **"New Booking (visitor)"** sub-tab to the **Booking > Settings > Emails** page.
-   It creates the entire settings interface for this specific email template, allowing administrators to customize its content, subject, and styling.

### User-Facing

-   The file's primary user-facing feature is the **sending of the initial booking confirmation email** to the visitor. This is a critical first touchpoint, confirming that their booking request has been received and is being processed.

## Extension Opportunities

The file itself is a final implementation, but it uses the plugin's highly extensible Email API, offering several points for customization.

-   **Adding Custom Shortcodes**: The `wpbc__get_replace_shortcodes__email_new_visitor` function applies the `wpbc_replace_params_for_booking` filter. This is the ideal, update-safe way to add new, custom shortcodes for use in the visitor confirmation email. For example, you could add a shortcode for a coupon code for their next booking.

    ```php
    function my_custom_visitor_email_shortcodes( $replace, $booking_id ) {
        // Generate a coupon for 10% off the next booking
        $coupon_code = 'WELCOME10';
        $replace['next_booking_coupon'] = $coupon_code;
        return $replace;
    }
    add_filter( 'wpbc_replace_params_for_booking', 'my_custom_visitor_email_shortcodes', 10, 2 );
    ```

-   **General Email Hooks**: All the filters from the base `WPBC_Emails_API` class are available, such as `wpbc_email_api_is_allow_send` to conditionally block the email or `wpbc_email_api_get_content_after` to modify the final email body before it's sent.

## Next File Recommendations

Having now analyzed all the primary email templates, it's clear they follow a consistent and robust pattern. It is now time to move on to other core, un-analyzed areas of the plugin.

1.  **`core/admin/wpbc-gutenberg.php`**: This is the top priority for understanding modern WordPress integration. It will define how the plugin provides a "Booking Form" block for the Gutenberg editor, which is the modern alternative to the classic widget.
2.  **`core/sync/wpbc-gcal.php`**: This file is responsible for the Google Calendar synchronization feature. Analyzing it will reveal how the plugin handles OAuth authentication, interacts with external APIs, and manages data syncing.
3.  **`core/admin/page-timeline.php`**: The booking "Timeline" is a core administrative UI. This file will show how booking data is queried and rendered in a visual timeline format, providing insight into the plugin's data visualization techniques.