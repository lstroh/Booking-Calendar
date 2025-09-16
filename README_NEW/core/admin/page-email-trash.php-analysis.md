# File Analysis: `core/admin/page-email-trash.php`

This document provides a detailed analysis of the `core/admin/page-email-trash.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file defines the implementation for the "Trash / Reject" email template. This notification is sent to a visitor when an administrator cancels their booking by moving it to the trash.

Architecturally, this file is another excellent example of the plugin's reusable, object-oriented Email API, following the exact same pattern as the other email template files (`approved`, `deny`, etc.). It extends the abstract `WPBC_Emails_API` class to define the unique fields and default content for this specific email. It also creates the corresponding admin interface under **Booking > Settings > Emails** for customization and contains the final sending logic.

## Detailed Explanation

The file's structure and logic are consistent with the plugin's established design pattern for email templates, divided into three main responsibilities:

1.  **Legacy Data Import**:
    -   The file begins with `wpbc_import6_...` functions for backward compatibility.
    -   `wpbc_import6_get_old_email_trash_data()`: Interestingly, this function imports data from the same legacy options as the "deny" email (`booking_email_deny_content`). This indicates that in older versions, a single template was used for both "denied" and "trashed" bookings, and this has since been split into two separate, configurable templates for more granular control.

2.  **Email Template Definition (`WPBC_Emails_API_Trash`)**:
    -   This class `extends WPBC_Emails_API`.
    -   **`init_settings_fields()`**: This core method defines the specific settings fields for this email template. The fields are structurally identical to other email templates (`enabled`, `copy_to_admin`, `to`, `from`, `subject`, `content`, styling options, etc.).
    -   The default content is tailored for a cancellation scenario and includes the `[denyreason]` shortcode, which is populated with the reason for cancellation if an admin provides one.

3.  **Admin Page and Sending Logic**:
    -   **`WPBC_Settings_Page_Email_Trash`**: This class builds the admin UI. It hooks into the plugin's menu system to add the **"Trash / Reject"** sub-tab to the **Booking > Settings > Emails** page.
    -   **`wpbc_send_email_trash()`**: This is the function that executes the email sending. It is called from elsewhere in the plugin whenever a booking is moved to the trash.
        -   It fetches the booking data from the database.
        -   It instantiates `WPBC_Emails_API_Trash` to load the customized template settings.
        -   It calls `wpbc__get_replace_shortcodes__email_trash()` to gather all the dynamic data for the shortcodes. It also populates the `[denyreason]` and `[reason]` shortcodes with the reason for trashing the booking.
        -   It gets the recipient's email from the form data.
        -   It calls the generic `$mail_api->send()` method to dispatch the email.

## Features Enabled

### Admin Menu

-   This file adds the **"Trash / Reject"** sub-tab to the **Booking > Settings > Emails** page.
-   It creates the entire settings interface for this specific email template, allowing administrators to customize its content, subject, and styling.

### User-Facing

-   The file's primary user-facing feature is the **sending of the booking cancellation email** to the visitor. This provides essential communication to the user, informing them that their booking is no longer valid.

## Extension Opportunities

The file itself is a final implementation, but it uses the plugin's highly extensible Email API, offering several points for customization.

-   **Adding Custom Shortcodes**: The `wpbc__get_replace_shortcodes__email_trash` function applies the `wpbc_replace_params_for_booking` filter. This is the ideal, update-safe way to add new, custom shortcodes for use in the cancellation email template.

    ```php
    // Add a custom shortcode, e.g., [rebook_link], to the email
    function my_custom_trash_email_shortcodes( $replace, $booking_id ) {
        $replace['rebook_link'] = get_site_url() . '/booking-page/';
        return $replace;
    }
    add_filter( 'wpbc_replace_params_for_booking', 'my_custom_trash_email_shortcodes', 10, 2 );
    ```

-   **General Email Hooks**: All the filters from the base `WPBC_Emails_API` class are available here, such as `wpbc_email_api_is_allow_send` to conditionally block the email or `wpbc_email_api_get_headers_after` to add custom email headers.

## Next File Recommendations

Having now analyzed all the primary email templates, it's clear they follow a consistent and robust pattern. It is now time to move on to other core, un-analyzed areas of the plugin.

1.  **`core/admin/wpbc-gutenberg.php`**: This is the top priority for understanding modern WordPress integration. It will define how the plugin provides a "Booking Form" block for the Gutenberg editor, which is the modern alternative to the classic widget.
2.  **`core/sync/wpbc-gcal.php`**: This file is responsible for the Google Calendar synchronization feature. Analyzing it will reveal how the plugin handles OAuth authentication, interacts with external APIs, and manages data syncing.
3.  **`core/admin/page-timeline.php`**: The booking "Timeline" is a core administrative UI. This file will show how booking data is queried and rendered in a visual timeline format, providing insight into the plugin's data visualization techniques.
