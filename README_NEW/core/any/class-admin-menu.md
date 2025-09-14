# File Analysis: `core/any/class-admin-menu.php`

This document provides a detailed analysis of the `core/any/class-admin-menu.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file defines the `WPBC_Admin_Menus` class, a powerful object-oriented wrapper for the standard WordPress menu creation functions (`add_menu_page`, `add_submenu_page`). It standardizes the process of creating admin pages, handling capabilities, setting menu icons, and, most importantly, ensuring that CSS and JavaScript assets are loaded only on the specific pages where they are needed. It acts as a factory for creating the plugin's backend pages.

## Detailed Explanation

The logic is entirely encapsulated within the `WPBC_Admin_Menus` class.

-   **`__construct( $slug, $param )`**: The constructor is the entry point for creating a new menu page. It accepts a unique `$slug` for the page and a `$param` array with all configuration options:
    -   `in_menu`: A critical parameter that determines the page's location. If set to `'root'`, it creates a top-level menu. Otherwise, it's the slug of the parent page (e.g., `'wpbc'` for the main plugin menu).
    -   `menu_title`, `page_header`, `browser_header`: Define the various titles for the menu item and page.
    -   `user_role`: A simplified way to set permissions (e.g., `'editor'`). The class translates this into the proper WordPress capability (e.g., `'publish_pages'`) using a static `$capability` array.
    -   `mune_icon_url`: Handles menu icons. It can accept a relative URL, the string `'none'`, or even a raw SVG string, which it will correctly base64-encode for use as a data URI.
    -   The constructor hooks the class's `new_admin_page` method into the `admin_menu` action.

-   **`new_admin_page()`**: This is the core method that actually registers the page with WordPress.
    -   It checks the `in_menu` property to decide whether to call its internal `create_plugin_menu()` (for `add_menu_page`) or `create_plugin_submenu()` (for `add_submenu_page`).
    -   After creating the page, it fires two important custom actions: `do_action('wpbc_menu_created', ...)` and `do_action('wpbc_define_nav_tabs', ...)`. These hooks allow other parts of the plugin to attach their functionality (like page content or navigation tabs) to this specific page.

-   **`content()`**: This is the function that WordPress calls to render the page's content.
    -   It follows a delegation pattern by immediately firing another action: `do_action('wpbc_page_structure_show', $this->menu_tag)`. This means that this class is only responsible for *creating* the page, and another class or file is responsible for listening for this action and *rendering* the actual content.

-   **`create_plugin_menu()` & `create_plugin_submenu()`**: These protected methods are wrappers for the WordPress core functions. Their most important feature is how they handle asset loading. After `add_menu_page()` returns a page hook (e.g., `toplevel_page_wpbc`), they use it to create page-specific hooks for loading assets:
    -   `add_action( 'admin_print_styles-' . $page, ... )`
    -   `add_action( 'admin_print_scripts-' . $page, ... )`
    This is a WordPress best practice that prevents the plugin from loading its scripts and styles on every single admin page.

## Features Enabled

### Admin Menu

-   This class is the foundational engine that **enables the creation of the entire admin menu system** for the Booking Calendar plugin.
-   While the menu definitions themselves are located elsewhere (likely in `core/wpbc.php`), this class provides the standardized, reusable mechanism for registering every top-level menu (like "Bookings") and submenu ("Settings", "Resources", etc.).
-   It ensures that user roles are correctly mapped to capabilities and that assets are loaded efficiently.

### User-Facing

-   This file has no user-facing features. It is exclusively for building the backend interface.

## Extension Opportunities

The class is designed for internal use but provides clear patterns for extension through its action hooks.

-   **Creating a New Page**: The primary way to extend the admin UI is to follow the plugin's pattern and instantiate this class to create your own page. This will ensure your page integrates correctly with the plugin's menu and style.
    ```php
    // In your custom plugin file, after Booking Calendar has loaded
    $my_page = new WPBC_Admin_Menus(
        'my-custom-page-slug',
        array(
            'in_menu'      => 'wpbc', // Add under the main "Booking" menu
            'menu_title'   => 'My Custom Page',
            'browser_header' => 'My Custom Page Title',
            'user_role'    => 'editor'
        )
    );
    ```
-   **Adding Content to Existing Pages**: The delegation hooks are powerful extension points.
    -   `add_action( 'wpbc_page_structure_show', 'my_content_render_function' )`: You can hook into this to add content to a specific page by checking the `$menu_tag` passed to your function.
    -   `add_action( 'wpbc_define_nav_tabs', 'my_nav_tab_function' )`: This allows you to add a new tab to the navigation bar on a specific plugin page.

## Next File Recommendations

Now that we understand the "factory" for creating admin pages, the next logical step is to analyze the "blueprints" for the content of those pages.

1.  **`core/admin/page-new.php`**: This file is the most likely place to find the content and logic for the **Bookings > Add New** page. It will almost certainly hook into `wpbc_page_structure_show` for the `wpbc-new` page slug and will reveal how manual bookings are created from the admin side.
2.  **`includes/page-availability/availability__page.php`**: This appears to be the main file for the **Availability** page. Analyzing it will explain how the UI for managing date availability is constructed and handled.
3.  **`core/any/api-emails.php`**: This remains a high-priority file. It likely contains the core API for defining and managing the email templates that are configured on the **Settings > Emails** page.
