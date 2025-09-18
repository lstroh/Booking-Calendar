# File Analysis: `core/class/wpbc-class-notices.php`

## High-Level Overview

This file defines the `WPBC_Notices` class, a system dedicated to managing and displaying persistent, dismissible admin notices. Its primary purpose is to show important, non-transient warnings to the site administrator on the main pages of the Booking Calendar plugin.

Unlike standard WordPress admin notices that appear on every admin page, this system is designed to hook into specific plugin pages. It also includes logic to make notices permanently dismissible on a per-user basis, preventing them from reappearing after a user has acknowledged them. The file currently implements a single, critical notice: a warning that appears if the plugin detects it may have been downgraded from a paid version to the free version.

## Detailed Explanation

The file's logic is encapsulated within the `WPBC_Notices` class.

-   **Class Structure and Hooks**:
    -   The `__construct()` method hooks the class into the WordPress lifecycle.
    -   `add_action( 'init', ... )`: The `set_messages` method is hooked into `init` to define the notice content early on.
    -   `add_action( 'wpbc_hook_*_page_header', ... )`: The `show_system_messages` method is hooked into several custom plugin actions (`wpbc_hook_booking_page_header`, `wpbc_hook_settings_page_header`, etc.). This ensures the notices are displayed consistently at the top of the plugin's own pages.

-   **Message Definition and Display**:
    -   `set_messages()`: This method populates a private `$this->messages` array. Currently, it defines only one message, `updated_paid_to_free`, which contains the warning text.
    -   `show_system_messages()`: This is the main rendering function. It checks for a specific condition by calling an external helper function, `wpbc_is_updated_paid_to_free()`. If this function returns true, it proceeds to render the notice.

-   **Formatting and Dismissal Logic**:
    -   `get_formated_message()`: This private method is responsible for building the final HTML for the notice.
    -   **Dismissible Behavior**: This is the key feature. The function uses another helper, `wpbc_is_dismissed( 'wpbc_message_update_free_to_paid', ... )`, to check if the current user has already dismissed this specific notice. This helper function (defined elsewhere, likely in `wpbc-functions.php`) is responsible for:
        1.  Checking a user meta option to see if the notice was previously dismissed.
        2.  If not dismissed, rendering the "Dismiss Forever" button, which will trigger an AJAX call to set the user meta option.
    -   The notice HTML also includes a simple "Hide" link that uses jQuery to temporarily hide the notice for the current page view without permanently dismissing it.
    -   If `wpbc_is_dismissed()` returns `false` (meaning the user has permanently dismissed it), `get_formated_message()` returns an empty string, and no notice is shown.

## Features Enabled

This file is exclusively for the admin panel and provides no user-facing features.

### Admin Menu

-   This file does not add any admin menu pages.
-   It creates a **persistent, dismissible warning notice** that appears at the top of the main Booking Calendar pages (Bookings, Settings, etc.) if the plugin detects that a paid version may have been overwritten by the free version. This is a crucial user support feature to prevent data loss and confusion.

### User-Facing

-   This file has no effect on the user-facing side of the website.

## Extension Opportunities

The `WPBC_Notices` class itself is not designed to be easily extended, as there are no filters to add new messages to its internal array. However, the pattern it uses is reusable.

-   **Safe Extension Method**: A developer could create their own notice system by replicating the pattern used here. This would involve:
    1.  Creating a new class or function.
    2.  Hooking it into the same `wpbc_hook_*_page_header` actions.
    3.  Calling the global `wpbc_is_dismissed( 'my_custom_notice_id', ... )` helper function to manage the display and dismissal of their own custom notice.

-   **Potential Risks**: The sanitization method used (`html_entity_decode( esc_js( $message ) )`) is unusual. While safe, it might be overly aggressive in stripping content if a developer wanted to include complex HTML in a custom notice. Using `wp_kses_post` would be a more standard approach.

## Next File Recommendations

This file provides a clear example of a specific admin UI feature. To continue building a comprehensive understanding of the plugin, we should now focus on the remaining major, un-analyzed core functionalities.

1.  **`core/sync/wpbc-gcal.php`** — **Top Priority.** This file is responsible for the Google Calendar synchronization feature. Analyzing it will reveal how the plugin handles complex, authenticated interactions with a major third-party API and manages data syncing.
2.  **`core/timeline/flex-timeline.php`** — The booking "Timeline" is a core administrative UI. This file will show how booking data is queried and rendered in a visual timeline format, providing insight into the plugin's data visualization techniques.
3.  **`includes/page-resource-free/page-resource-free.php`** — This file (or one in its directory) is responsible for managing booking resources, which is a fundamental concept in the plugin. Analyzing it is key to understanding the plugin's core data model.
