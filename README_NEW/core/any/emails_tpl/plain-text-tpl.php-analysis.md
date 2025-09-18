# File Analysis: `core/any/emails_tpl/plain-text-tpl.php`

## High-Level Overview

This file is a PHP template responsible for generating the plain-text version of emails sent by the plugin. It serves as a companion to the HTML email templates (like `plain-html-tpl.php`). Its primary purpose is to take the same core content (header, body, footer) and format it as a clean, readable text-only string, suitable for email clients that do not render HTML or for the text part of a multipart email.

Architecturally, this file is a simple view component within the plugin's email framework. It is called by the core email engine (`WPBC_Emails_API`) when it needs to generate the plain-text content for an email. It ensures maximum compatibility and accessibility by providing a fallback for users who cannot or prefer not to view HTML emails.

## Detailed Explanation

The file's entire logic is contained within a single function:

-   **`wpbc_email_template_plain_text( $fields_values )`**: This function constructs the plain-text email body.
    -   It accepts an array, `$fields_values`, which contains the raw content for the `header_content`, `content`, and `footer_content`.
    -   It uses an output buffer (`ob_start()` and `ob_get_clean()`) to capture its output into a string.
    -   **Content Processing**: It concatenates the header, content, and footer, separated by double newlines (`

`) to create paragraph breaks in a text format.
    -   **Sanitization**: Crucially, it passes all content through `wp_kses_post()`. In this context, `wp_kses_post` acts as a powerful HTML stripper. Since it only allows a set of tags typically found in posts (like `<p>`, `<a>`, `<strong>`), and this template has no HTML structure, its primary effect is to remove any and all HTML tags from the content, ensuring the final output is pure plain text.
    -   It also uses `wptexturize()` to improve the typography of quotes and other punctuation.

```php
// The core logic of the template
if ( ! empty($fields_values['header_content'] ) ) {
    echo wp_kses_post( wptexturize( $fields_values['header_content'] ) ) . "\n\n";
}

echo ( wp_kses_post( wptexturize( $fields_values['content'] ) ) );

if ( ! empty( $fields_values['footer_content'] ) ) {
    echo "\n\n" .  wp_kses_post( wptexturize( $fields_values['footer_content'] ) );
}
```

## Features Enabled

This file is a passive template and does not directly enable any features in the admin panel or on the user-facing side.

### Admin Menu

-   This file has no effect on the WordPress admin menu or any settings pages.

### User-Facing

-   This file provides the **plain-text alternative** for all emails sent by the plugin. This is a critical feature for:
    -   **Accessibility**: Users with screen readers may prefer plain-text emails.
    -   **Compatibility**: Some older or security-focused email clients do not render HTML.
    -   **Multipart Emails**: For `multipart/alternative` emails, this template provides the `text/plain` version that is sent alongside the `text/html` version.

## Extension Opportunities

Directly modifying this template file is not recommended, as any changes would be overwritten during a plugin update.

-   **Safe Extension Method**: The `WPBC_Emails_API` class contains a hook, `wpbc_email_api_get_plain_content_after`, which is the ideal place to modify the plain-text content. A developer could hook into this filter to perform additional text transformations, add extra information, or completely reformat the plain-text body before it is sent.

    ```php
    function my_custom_plain_text_modifier( $plain_text_content, $email_id, $fields_values ) {
        // Example: Add a disclaimer to all plain-text emails
        $plain_text_content .= "\n\n---
This is a custom disclaimer.";
        return $plain_text_content;
    }
    add_filter( 'wpbc_email_api_get_plain_content_after', 'my_custom_plain_text_modifier', 10, 3 );
    ```

-   **Potential Risks**: The template is very simple. Any custom logic hooked into the modification process should be careful to only output plain text and properly encode any special characters to avoid formatting issues in email clients.

## Next File Recommendations

Having analyzed the core email templating system, we should now focus on other major, un-analyzed features of the plugin. The following files remain the highest priority for achieving a complete architectural understanding.

1.  **`core/sync/wpbc-gcal.php`** — This is the top priority. It is responsible for the Google Calendar synchronization feature. Analyzing it will reveal how the plugin handles complex, authenticated interactions with a major third-party API and manages data syncing.
2.  **`core/timeline/flex-timeline.php`** — The booking "Timeline" is a core administrative UI. This file will show how booking data is queried and rendered in a visual timeline format, providing insight into the plugin's data visualization techniques.
3.  **`core/class/wpbc-class-notices.php`** — This file likely defines a standardized system for creating and displaying admin notices throughout the plugin. Understanding it is key to learning how the plugin communicates important information to the site administrator.

```