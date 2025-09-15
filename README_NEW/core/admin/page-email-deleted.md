# File Analysis: `core/admin/page-email-deleted.php`

This document provides a detailed analysis of the `core/admin/page-email-deleted.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file defines the specific implementation for the "Deleted" email template, which is sent to a visitor when their booking is canceled or declined by an administrator.

Architecturally, this file is a direct parallel to `page-email-approved.php` and serves as a perfect example of the plugin's reusable and object-oriented Email API. It extends the abstract `WPBC_Emails_API` class to define the unique fields and default content for the "Deleted" email. It also creates the corresponding admin interface under **Booking > Settings > Emails** for customizing this template and contains the final sending logic.

## Detailed Explanation

The file's structure and logic mirror the other email template pages, demonstrating a consistent and robust design pattern.

1.  **Legacy Data Import**:
    -   The file begins with `wpbc_import6_...` functions. These are responsible for backward compatibility.
    -   `wpbc_import6_get_old_email_deleted_data()`: This function specifically looks for old option names like `booking_email_deny_content` and `booking_email_deny_subject`. This indicates that in previous versions, a single "deny" email existed, and its settings are now being imported to serve as the basis for the new "Deleted" template, ensuring a seamless upgrade for long-time users.

2.  **Email Template Definition (`WPBC_Emails_API_Deleted`)**:
    -   This class `extends WPBC_Emails_API`.
    -   **`init_settings_fields()`**: This core method defines the specific settings fields for this email template. The fields are structurally identical to other email templates (`enabled`, `to`, `from`, `subject`, `content`, styling options, etc.).
    -   The key difference is in the default values. For example, the default subject is `__( 'Your booking has been declined', 'booking' )`, and the default content includes the `[denyreason]` shortcode, tailoring the template for its specific purpose.

3.  **Admin Page and Sending Logic**:
    -   **`WPBC_Settings_Page_Email_Deleted`**: This class builds the admin UI. It hooks into the plugin's menu system to add the "Deleted" sub-tab to the "Emails" settings page. Its `content()` method renders the form by calling the API's `show()` method, displaying the fields defined in `WPBC_Emails_API_Deleted`.
    -   **`wpbc_send_email_deleted()`**: This is the function that executes the email sending. It is called from elsewhere in the plugin whenever a booking is deleted or declined.
        - It fetches the booking data from the database.
        - It instantiates `WPBC_Emails_API_Deleted` to load the customized template from the database.
        - It calls `wpbc__get_replace_shortcodes__email_deleted()` to gather all the dynamic data for the shortcodes (e.g., `[id]`, `[dates]`, `[content]`). This function also specifically populates the `[denyreason]` and `[reason]` shortcodes with the reason for cancellation, if provided.
        - It then calls the API's generic `$mail_api->send()` method to dispatch the final email.

## Features Enabled

### Admin Menu

-   This file adds the **"Deleted"** sub-tab to the **Booking > Settings > Emails** page.
-   It creates the entire settings interface for this specific email template, allowing administrators to customize its content, subject, and styling.

### User-Facing

-   The file's primary user-facing feature is the **sending of the booking cancellation/deletion email** to the visitor. This provides essential communication to the user, informing them that their booking is no longer valid and providing a reason if the admin entered one.

## Extension Opportunities

The file itself is a final implementation, but it uses the plugin's extensible Email API, offering several points for customization.

-   **Adding Custom Shortcodes**: The `wpbc__get_replace_shortcodes__email_deleted` function applies the `wpbc_replace_params_for_booking` filter. This is the ideal, update-safe way to add new, custom shortcodes for use in the "Deleted" email template.

    ```php
    // Add a custom shortcode, e.g., [cancellation_policy_url], to the email
    function my_custom_deleted_email_shortcodes( $replace, $booking_id ) {
        $replace['cancellation_policy_url'] = get_site_url() . '/cancellation-policy';
        return $replace;
    }
    add_filter( 'wpbc_replace_params_for_booking', 'my_custom_deleted_email_shortcodes', 10, 2 );
    ```

-   **General Email Hooks**: All the filters from the base `WPBC_Emails_API` class are available here, such as `wpbc_email_api_is_allow_send` to conditionally block the email or `wpbc_email_api_get_headers_after` to add custom email headers.

## Next File Recommendations

Having analyzed several email templates that follow the same pattern, we have a solid understanding of the email API. It is time to move on to other core, un-analyzed areas of the plugin.

1.  **`core/admin/wpbc-gutenberg.php`**: This is the top priority for understanding modern WordPress integration. It will define how the plugin provides a "Booking Form" block for the Gutenberg editor, which is the modern alternative to the classic widget.
2.  **`core/sync/wpbc-gcal.php`**: This file is responsible for the Google Calendar synchronization feature. Analyzing it will reveal how the plugin handles OAuth authentication, interacts with external APIs, and manages data syncing.
3.  **`core/admin/wpbc-toolbars.php`**: The admin UI, particularly the Booking Listing, relies on toolbars for filtering and actions. This file likely defines those toolbars and is key to understanding the construction of the main admin pages.
