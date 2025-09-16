# File Analysis: `core/admin/page-email-deny.php`

This document provides a detailed analysis of the `core/admin/page-email-deny.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file defines the specific implementation for the "Pending" email template. This notification is sent to a visitor when their booking status is changed to pending (awaiting approval) by an administrator.

Architecturally, this file is another excellent example of the plugin's reusable, object-oriented Email API, following the exact same pattern as the "Approved" and "Deleted" email template files. It extends the abstract `WPBC_Emails_API` class to define the unique fields and default content for this specific email. It also creates the corresponding admin interface under **Booking > Settings > Emails** and contains the final sending logic.

## Detailed Explanation

The file's structure and logic are consistent with the plugin's established design pattern for email templates, divided into three main responsibilities.

1.  **Legacy Data Import**:
    -   The file begins with `wpbc_import6_...` functions, which are responsible for backward compatibility.
    -   `wpbc_import6_get_old_email_deny_data()`: This function specifically looks for old option names like `booking_email_deny_content` and `booking_email_deny_subject`. It imports these legacy settings into the new API-driven format, ensuring a seamless upgrade for users of older plugin versions.

2.  **Email Template Definition (`WPBC_Emails_API_Deny`)**:
    -   This class `extends WPBC_Emails_API`.
    -   **`init_settings_fields()`**: This core method defines the specific settings fields for this email template. The fields are structurally identical to other email templates (`enabled`, `copy_to_admin`, `to`, `from`, `subject`, `content`, styling options, etc.).
    -   The key difference is in the default values. The default subject is `__( 'Your booking has been declined', 'booking' )`, and the default content includes the `[denyreason]` shortcode. While the class and file names use "deny," the admin UI tab is labeled "Pending," and the hint text clarifies its use for when a booking is set to pending.

3.  **Admin Page and Sending Logic**:
    -   **`WPBC_Settings_Page_Email_Deny`**: This class builds the admin UI. It hooks into the plugin's menu system to add the "Pending" sub-tab to the **Booking > Settings > Emails** page. Its `content()` method renders the form by calling the API's `show()` method, which displays the fields defined in `WPBC_Emails_API_Deny`.
    -   **`wpbc_send_email_deny()`**: This is the function that executes the email sending. It is called from elsewhere in the plugin whenever a booking is set to pending.
        -   It fetches the booking data from the database.
        -   It instantiates `WPBC_Emails_API_Deny` to load the customized template from the database.
        -   It calls `wpbc__get_replace_shortcodes__email_deny()` to gather all the dynamic data for the shortcodes (e.g., `[id]`, `[dates]`, `[content]`). This function also specifically populates the `[denyreason]` and `[reason]` shortcodes with the reason for the status change, if provided.
        -   It then calls the API's generic `$mail_api->send()` method to dispatch the final email to the visitor.

## Features Enabled

### Admin Menu

-   This file adds the **"Pending"** sub-tab to the **Booking > Settings > Emails** page.
-   It creates the entire settings interface for this specific email template, allowing administrators to customize its content, subject, and styling.

### User-Facing

-   The file's primary user-facing feature is the **sending of the booking pending/denied email** to the visitor. This provides essential communication to the user, informing them that their booking is awaiting approval or has been declined, and can include a reason provided by the admin.

## Extension Opportunities

The file itself is a final implementation, but it uses the plugin's highly extensible Email API, offering several points for customization.

-   **Adding Custom Shortcodes**: The `wpbc__get_replace_shortcodes__email_deny` function applies the `wpbc_replace_params_for_booking` filter. This is the ideal, update-safe way to add new, custom shortcodes for use in the "Pending" email template.

    ```php
    // Add a custom shortcode, e.g., [next_steps_url], to the email
    function my_custom_pending_email_shortcodes( $replace, $booking_id ) {
        $replace['next_steps_url'] = get_site_url() . '/what-to-expect-next';
        return $replace;
    }
    add_filter( 'wpbc_replace_params_for_booking', 'my_custom_pending_email_shortcodes', 10, 2 );
    ```

-   **General Email Hooks**: All the filters from the base `WPBC_Emails_API` class are available here, such as `wpbc_email_api_is_allow_send` to conditionally block the email or `wpbc_email_api_get_headers_after` to add custom email headers.

## Next File Recommendations

Having analyzed several email templates that follow the same robust pattern, we have a solid understanding of the email API. It is time to move on to other core, un-analyzed areas of the plugin.

1.  **`core/admin/wpbc-gutenberg.php`**: This is the top priority for understanding modern WordPress integration. It will define how the plugin provides a "Booking Form" block for the Gutenberg editor, which is the modern alternative to the classic widget.
2.  **`core/sync/wpbc-gcal.php`**: This file is responsible for the Google Calendar synchronization feature. Analyzing it will reveal how the plugin handles OAuth authentication, interacts with external APIs, and manages data syncing.
3.  **`core/admin/page-timeline.php`**: The booking "Timeline" is a core administrative UI. This file will show how booking data is queried and rendered in a visual timeline format, providing insight into the plugin's data visualization techniques.
