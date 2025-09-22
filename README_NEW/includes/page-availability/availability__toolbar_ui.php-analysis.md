# File Analysis: `includes/page-availability/availability__toolbar_ui.php`

This file is a dedicated UI component library responsible for rendering the toolbar on the **Booking > Availability** admin page. It contains a set of functions that build the various buttons, selectors, and informational text that allow an administrator to interact with the availability calendar.

## High-Level Overview

Architecturally, this file acts as a "view" layer for the Availability page's toolbar. It uses lower-level UI helper functions (from `flex_ui_elements.php`) to construct complex, page-specific components. Its primary role is to generate the HTML for the toolbar and to embed the necessary JavaScript `onclick` events that trigger the AJAX functionality defined in `availability__class.php`. It does not contain any data-fetching or business logic itself; it is purely for presentation and user interaction.

## Detailed Explanation

The file consists of several procedural functions, each responsible for a specific part of the toolbar.

-   **`wpbc_ajx_availability__toolbar( $escaped_search_request_params )`**:
    -   This is the main function that assembles the entire toolbar.
    -   It creates a container with two main sections (tabs): "Info" and "Calendar Settings".
    -   The visibility of these sections is controlled by the `$selected_tab` parameter, which comes from the request parameters. This means the toolbar has different states.
    -   The "Info" section contains placeholders for the resource selection dropdown (`wpbc_hidden_template__select_booking_resource`) and an info/hint area (`wpbc_toolbar_dates_hint`).
    -   The "Calendar Settings" section contains buttons for "Reset availability" and a general "Reset" button. It also has a placeholder for the "months in a row" selector.

-   **`wpbc_ajx_avy__ui__info(...)`**:
    -   Renders a help text message for the user, explaining how to use the interface.
    -   It uses the `wpbc_flex_addon` function to display the text.
    -   It includes a jQuery script that makes this help text "blink" when the page content is first loaded (`wpbc_page_content_loaded` event), drawing the user's attention to the instructions.

-   **UI Element Functions (`wpbc_ajx_avy__ui__*`)**:
    -   **`wpbc_ajx_avy__ui__available_radio()`** and **`wpbc_ajx_avy__ui__unavailable_radio()`**: These functions render the "Available" and "Unavailable" radio buttons. They use helper functions `wpbc_flex_vertical_color` to add a colored border (green for available, red for unavailable) and `wpbc_flex_radio` to create the input element itself.
    -   **`wpbc_ajx_avy__ui__availability_apply_btn()`**: This renders the main "Apply" button. The key logic is in the `action` parameter, which contains an `onclick` JavaScript call. This script gathers the selected dates and the chosen availability status and passes them to the `wpbc_ajx_availability__send_request_with_params()` JavaScript function with `do_action: 'set_availability'`. This is the direct trigger for the AJAX update.
    -   **`wpbc_avy__ui__toolbar_erase_availability_button()`**: Renders the "Reset availability" button. It includes a JavaScript confirmation dialog (`wpbc_are_you_sure`) to prevent accidental data deletion before sending the AJAX request with `do_action: 'erase_availability'`.

## Features Enabled

This file is exclusively for the admin panel and provides no user-facing features.

### Admin Menu

-   This file does not add any menu items but is responsible for building the entire interactive toolbar on the **Booking > Availability** page.
-   It renders the core UI for setting date availability: the "Available" / "Unavailable" radio buttons and the "Apply" button.
-   It renders secondary actions, such as the "Reset availability" button, which allows an admin to clear all custom availability rules for a resource.
-   It provides contextual help text to guide the user.

### User-Facing

-   This file has no impact on the user-facing side of the website.

## Extension Opportunities

-   **Limitations**: The file has no WordPress action or filter hooks, making it difficult to extend in an update-safe way. The toolbar structure is hardcoded in the `wpbc_ajx_availability__toolbar` function, and the JavaScript calls are embedded directly as strings within the PHP function calls.
-   **Suggested Improvements**: To improve extensibility, the main toolbar function could be refactored to include `do_action` hooks. For example:
    ```php
    // Suggestion for improvement
    do_action( 'wpbc_availability_toolbar_before_actions' );
    // ... render existing buttons ...
    do_action( 'wpbc_availability_toolbar_after_actions' );
    ```
    This would allow developers to easily add their own buttons or UI elements to the toolbar.
-   **Potential Risks**: The hardcoded JavaScript `onclick` actions are brittle. A syntax error in the JS within the PHP string would only be discoverable at runtime in the browser and could break the page's functionality.

## Next File Recommendations

The server-side components of the Availability page have been thoroughly analyzed. The next logical step is to investigate the client-side JavaScript that controls this page's behavior and then to explore the fundamental data structures of the plugin.

1.  **`includes/page-availability/_out/availability_page.js`**: **Top Priority.** This is the JavaScript file enqueued by `availability__class.php`. It is the client-side counterpart to the AJAX functionality and will contain the `wpbc_ajx_availability__send_request_with_params` function, which is critical for understanding how the UI is rendered and updated.
2.  **`includes/page-resource-free/page-resource-free.php`**: This remains a key un-analyzed file. Understanding how booking resources (the "bookable" items) are created and managed is fundamental to the entire plugin's data model.
3.  **`js/user-data-saver.js`**: This file is still relevant as it likely handles the client-side logic for saving user preferences, such as the last-selected resource or view mode on the Availability page.
