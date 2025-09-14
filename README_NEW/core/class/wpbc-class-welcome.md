# File Analysis: `core/class/wpbc-class-welcome.php`

This document provides a detailed analysis of the `core/class/wpbc-class-welcome.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file defines the `WPBC_Welcome` class, which is responsible for displaying the "What's New" page to users after a plugin update. This is a common feature in WordPress plugins designed to highlight new features and guide users.

The class handles two primary functions:
1.  **Post-Update Redirect:** It contains the logic that automatically redirects an administrator to the "What's New" page immediately after the plugin has been activated or updated.
2.  **Content Rendering:** It builds the entire "What's New" page, complete with a tabbed interface, version-specific feature sections, images, and helpful links.

Architecturally, it's a self-contained UI component that enhances the post-update user experience but is not involved in the plugin's core booking functionality.

## Detailed Explanation

The file's logic is encapsulated within the `WPBC_Welcome` class.

### 1. Post-Update Redirect Logic

-   **`welcome()` method**: This is the core of the redirect functionality and is hooked into `admin_init`.
    -   It first checks for a WordPress transient named `_booking_activation_redirect`. This transient is set during the plugin's activation/update process. If it doesn't exist, the function does nothing.
    -   Once the transient is found, it's immediately deleted to ensure the redirect only happens once.
    -   It then performs a version check. It compares the current plugin version (`WP_BK_VERSION_NUM`) against a value stored in the database (`booking_activation_redirect_for_version`). If the user has already seen the welcome screen for this version, it stops.
    -   If it's a new version for the user, it updates the database option with the current version number and performs a `wp_safe_redirect()` to the hidden "What's New" admin page.

### 2. Admin Page Registration and Content Rendering

-   **`admin_menus()` method**: This method uses a common WordPress technique for creating a "hidden" admin page.
    -   It uses `add_dashboard_page()` to register the page, making it accessible via the URL `wp-admin/index.php?page=wpbc-about`.
    -   It then immediately calls `remove_submenu_page()` to hide the link from the "Dashboard" menu. This ensures users can only access the page via the direct redirect.

-   **`content_whats_new()` method**: This is the callback function that renders all the HTML for the page.
    -   It builds the page in a modular way by calling a series of external functions like `wpbc_welcome_section_10_14()`, `wpbc_welcome_section_10_13()`, etc. These functions are defined in the included `core/class/welcome_current.php` file, with each function containing the specific content for a major version update.
    -   It uses its own helper methods like `show_col_section()` to render content in responsive two- or three-column layouts.
    -   Images for the feature sections are loaded from an external asset path on `wpbookingcalendar.com`.

-   **`title_section()` method**: This function renders the main page title and the tabbed navigation (`What's New`, `Get Started`, `Get even more functionality`).

## Features Enabled

### Admin Menu

-   **"What's New" Page:** It creates the hidden admin page at `index.php?page=wpbc-about` that showcases new features after an update.
-   **Automatic Redirect:** It implements the logic that automatically sends users to this page once after a plugin update, ensuring they are aware of recent changes.

### User-Facing

-   This file has **no user-facing features**. It is purely for administrator information within the WordPress dashboard.

## Extension Opportunities

This class is a self-contained feature and is not designed with explicit hooks for extension. The content is hardcoded within the modular functions it calls.

-   **Disabling the Welcome Page:** A developer could prevent the redirect from happening by removing the action hook in their own plugin or theme's `functions.php`:
    ```php
    // Note: You would need to get the global $wpbc_welcome object to do this.
    global $wpbc_welcome;
    if ( class_exists( 'WPBC_Welcome' ) && isset( $wpbc_welcome ) ) {
        remove_action( 'admin_init', array( $wpbc_welcome, 'welcome' ) );
    }
    ```
-   **Potential Risks:** There are no significant risks in extending this file, primarily because there are no easy ways to do so. Modifying the core file directly would be overwritten on the next plugin update.

## Next File Recommendations

This file gives a clear picture of the post-update experience. To continue, we should focus on the core architectural components of the admin panel.

1.  **`core/any/class-admin-menu.php`**: **Top Priority.** We've now seen several hidden or specialized pages. This file is essential for understanding how the main, visible admin menu structure ("Bookings", "Settings", etc.) is defined and rendered.
2.  **`core/any/api-emails.php`**: This file was referenced in the `wpbc-emails.php` analysis. It likely contains the core API for defining, storing, and parsing email templates, making it a critical part of the notification system.
3.  **`core/class/welcome_current.php`**: Since the `WPBC_Welcome` class includes this file to render its content, analyzing it would reveal the actual text, images, and layout for all the "What's New" sections. This would provide a complete picture of the welcome page feature.