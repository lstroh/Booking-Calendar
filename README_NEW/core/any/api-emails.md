# File Analysis: `core/any/api-emails.php`

This document provides a detailed analysis of the `core/any/api-emails.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file defines the `WPBC_Emails_API`, an abstract class that serves as the core engine for the plugin's entire email notification system. It provides a standardized and extensible framework for creating, storing, and sending different types of transactional emails (e.g., "New Booking," "Approved," "Declined").

Architecturally, this file is a cornerstone of the plugin's notification system. It cleverly extends the plugin's own settings framework (`WPBC_Settings_API`), treating each email template as a distinct set of configurable options (subject, body, headers, etc.) that are stored in the WordPress options table. Any specific email type in the plugin (like the "admin new booking" email) is an implementation of this abstract class.

## Detailed Explanation

The file's logic is encapsulated within the `abstract class WPBC_Emails_API`.

-   **Inheritance from `WPBC_Settings_API`**: The class extends `WPBC_Settings_API`, which means every email template is, under the hood, a settings page. The constructor configures these settings to be saved as a single serialized array in the `wp_options` table with a key like `booking_email_{template_id}` (e.g., `booking_email_new_admin`).

-   **`init_settings_fields()` (Abstract Method)**: This is an `abstract` function, meaning any child class that extends `WPBC_Emails_API` **must** implement this method. This is where the specific fields for an email template (like 'subject', 'to', 'from', 'content') are defined.

-   **Shortcode Replacement Engine**:
    -   `set_replace( $replace )`: A method to define an associative array of placeholders and their values (e.g., `['booking_id' => 123, 'name' => 'John']`).
    -   `replace_shortcodes( $subject )`: This method iterates through the defined replacements and substitutes placeholders like `[booking_id]` or `{booking_id}` in any given string (typically the email subject or body).

-   **Dynamic Content Templating**:
    -   `get_content()`: The main function for retrieving the final email body. It checks the template's configured content type (`plain`, `html`, or `multipart`) and calls the appropriate rendering function.
    -   `get_content_html()` and `get_content_plain()`: These functions implement a flexible templating system. They dynamically `require_once` a PHP template file from the `/core/any/emails_tpl/` directory based on a setting. This allows for different email layouts (e.g., a branded HTML template vs. a simple plain text one) to be used and selected in the settings.

-   **Email Sending Workflow**:
    -   `send( $to, $replace )`: This is the primary public method for sending an email. It orchestrates the entire process:
        1.  Validates the recipient's email address.
        2.  Sets the shortcode replacements.
        3.  Calls internal methods to get the processed subject, content, and headers.
        4.  Checks a filter, `wpbc_email_api_is_allow_send`, which acts as a global "kill switch" to programmatically prevent sending.
        5.  Calls the standard `wp_mail()` function to send the email.
        6.  Handles logic for sending a copy to the admin if the option is enabled.

-   **Header and Error Handling**:
    -   `get_headers()`: Constructs the `From:` and `Content-Type:` headers. It includes the `wpbc_email_api_get_headers_after` filter for customization.
    -   `process_multipart( $mailer )`: Hooks into the WordPress `phpmailer_init` action. If the email is multipart, it generates a plain text version for email clients that don't support HTML. It also sets the `Sender` property to match the `From` address, a common technique for improving email deliverability.
    -   `email_error_parse( $wp_error_object )`: Hooks into `wp_mail_failed` to catch any errors during sending and fires a custom `wpbc_email_sending_error` action.

## Features Enabled

### Admin Menu

-   This file itself does not add any menu items. However, it provides the **foundational framework** used by other files to create the settings pages for each email template, which are found under **Booking > Settings > Emails**. Each template (New booking, Approved, etc.) is a concrete implementation of this API.

### User-Facing

-   This file is the engine that **sends all transactional emails** to both site visitors and administrators.
-   It ensures that booking confirmations, approval/denial notifications, and other communications are correctly formatted, populated with dynamic data (like booking dates and customer names), and reliably delivered.

## Extension Opportunities

This API is rich with filters, making it highly extensible.

-   **Create a Custom Email Notification**: The primary extension pattern is to create a new class that `extends WPBC_Emails_API` and implement the `init_settings_fields()` method to define your new email type.

-   **Conditionally Block Emails**: The `wpbc_email_api_is_allow_send` filter is a powerful "kill switch." You can use it to prevent certain emails from being sent based on custom logic.
    ```php
    function my_custom_email_sending_logic( $is_send_email, $email_id, $fields_values ) {
        // Example: Don't send 'pending' emails for bookings made on weekends.
        if ( 'pending' === $email_id && in_array( date('N'), array(6, 7) ) ) {
            return false; // Do not send email
        }
        return $is_send_email; // Otherwise, allow sending
    }
    add_filter( 'wpbc_email_api_is_allow_send', 'my_custom_email_sending_logic', 10, 3 );
    ```

-   **Modify Headers or Content**: You can use filters like `wpbc_email_api_get_headers_after` to add custom headers (e.g., for tracking or CRM integration) or `wpbc_email_api_get_content_after` to modify the final email body before it's sent.

## Next File Recommendations

Now that we have a deep understanding of the email framework, we should explore other key un-analyzed areas of the plugin.

1.  **`core/admin/wpbc-gutenberg.php`**: This file is critical for understanding how the plugin integrates with the modern WordPress Block Editor. It will define how the "Booking Form" block is registered, what its settings are, and how it's rendered on the front-end.
2.  **`core/sync/wpbc-gcal.php`**: This file likely contains the core logic for the Google Calendar synchronization feature. Analyzing it will provide valuable insight into how the plugin handles OAuth, interacts with a major third-party API, and manages data syncing.
3.  **`core/admin/wpbc-toolbars.php`**: The admin interface relies on toolbars for filtering and actions on pages like the Booking Listing. This file likely contains the definitions for these toolbars, and analyzing it will complete our understanding of the admin UI construction.
