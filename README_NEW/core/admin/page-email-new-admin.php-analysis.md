# File Analysis: `core/admin/page-email-new-admin.php`

This document provides a detailed analysis of the `core/admin/page-email-new-admin.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file defines the implementation for the "New Booking" email template, which is the crucial notification sent to the site administrator(s) whenever a new booking is submitted.

Architecturally, this file is another perfect demonstration of the plugin's consistent, object-oriented Email API pattern. It extends the abstract `WPBC_Emails_API` class to define the unique fields and default content for the admin notification email. It also creates the corresponding admin interface under **Booking > Settings > Emails** for customization and contains the final sending logic.

## Detailed Explanation

The file's structure and logic are consistent with the other email template files, divided into three main responsibilities:

1.  **Legacy Data Import**:
    -   The file begins with `wpbc_import6_...` functions, which are responsible for backward compatibility.
    -   `wpbc_import6_get_old_email_new_admin_data()`: This function looks for old option names like `booking_email_reservation_content` and `booking_email_reservation_subject`. It imports these legacy settings into the new API-driven format, ensuring a seamless upgrade for users of older plugin versions.

2.  **Email Template Definition (`WPBC_Emails_API_NewAdmin`)**:
    -   This class `extends WPBC_Emails_API`.
    -   **`init_settings_fields()`**: This core method defines the specific settings fields for this email template. The fields are structurally identical to other email templates (`enabled`, `to`, `from`, `subject`, `content`, styling options, etc.).
    -   A key feature specific to this template is the **`enable_replyto`** checkbox, which allows the admin to set the `Reply-To` header of the notification email to the visitor's email address, making it easy to reply directly to the customer.
    -   The default content is tailored for administrators, including important shortcodes like `[moderatelink]`, `[click2approve]`, and `[click2decline]` for quick booking management directly from the email.

3.  **Admin Page and Sending Logic**:
    -   **`WPBC_Settings_Page_Email_NewAdmin`**: This class builds the admin UI. It hooks into the plugin's menu system to add the **"New Booking (admin)"** sub-tab to the **Booking > Settings > Emails** page, which is set as the default tab for the Emails section.
    -   **`wpbc_send_email_new_admin()`**: This is the function that executes the email sending. It is called from the booking creation workflow.
        -   It fetches the new booking's data from the database.
        -   It instantiates `WPBC_Emails_API_NewAdmin` to load the customized template settings.
        -   It calls `wpbc__get_replace_shortcodes__email_new_admin()` to gather all the dynamic data for the shortcodes, including booking details, cost, dates, and admin-specific action links.
        -   It retrieves the admin email address(es) from the template's "To" setting.
        -   It then calls the API's generic `$mail_api->send()` method to dispatch the final email.

## Features Enabled

### Admin Menu

-   This file adds the **"New Booking (admin)"** sub-tab to the **Booking > Settings > Emails** page, making it the default view for email settings.
-   It creates the entire settings interface for this specific email template, allowing administrators to customize its recipients, content, subject, and styling.

### User-Facing

-   This file has no direct user-facing features. However, it is a critical component of the booking workflow, as it ensures administrators are promptly notified of new bookings, enabling them to manage reservations efficiently.

## Extension Opportunities

The file itself is a final implementation, but it uses the plugin's highly extensible Email API, offering several points for customization.

-   **Adding Custom Shortcodes**: The `wpbc__get_replace_shortcodes__email_new_admin` function applies the `wpbc_replace_params_for_booking` filter. This is the ideal, update-safe way to add new, custom shortcodes for use in the admin notification email. For example, you could add a shortcode to include a link to a customer's profile in your CRM.

    ```php
    function my_custom_admin_email_shortcodes( $replace, $booking_id ) {
        $user_email = $replace['email']; // Get email from existing shortcodes
        $replace['crm_link'] = 'https://my-crm.com/customers?email=' . urlencode( $user_email );
        return $replace;
    }
    add_filter( 'wpbc_replace_params_for_booking', 'my_custom_admin_email_shortcodes', 10, 2 );
    ```

-   **General Email Hooks**: All the filters from the base `WPBC_Emails_API` class are available, such as `wpbc_email_api_is_allow_send` to conditionally block the email (e.g., for low-value bookings) or `wpbc_email_api_get_headers_after` to add custom email headers for routing or tracking.

## Next File Recommendations

Having analyzed the primary email templates, it's clear they follow a consistent and robust pattern. It is now time to move on to other core, un-analyzed areas of the plugin.

1.  **`core/admin/wpbc-gutenberg.php`**: This is the top priority for understanding modern WordPress integration. It will define how the plugin provides a "Booking Form" block for the Gutenberg editor, which is the modern alternative to the classic widget.
2.  **`core/sync/wpbc-gcal.php`**: This file is responsible for the Google Calendar synchronization feature. Analyzing it will reveal how the plugin handles OAuth authentication, interacts with external APIs, and manages data syncing.
3.  **`core/admin/page-timeline.php`**: The booking "Timeline" is a core administrative UI. This file will show how booking data is queried and rendered in a visual timeline format, providing insight into the plugin's data visualization techniques.
