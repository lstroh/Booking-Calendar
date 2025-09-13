# File Analysis: core/wpbc-css.php

## High-Level Overview
This file is the central controller for loading all CSS stylesheets used by the Booking Calendar plugin. It defines the `WPBC_CSS` class, which contains the logic for enqueueing styles for both the front-end (what site visitors see) and the back-end (the admin panel). 

Its primary role is to hook into WordPress's standard asset loading system (`admin_enqueue_scripts` and `wp_enqueue_scripts`) and register all necessary `.css` files. It handles loading core styles, Bootstrap, icon fonts, and dynamically loads different "skin" files for the calendar based on user settings. It also includes a mechanism for preventing conflicts by dequeueing specific stylesheets from other plugins.

## Detailed Explanation
The file's functionality is primarily within the `WPBC_CSS` class and a few helper functions.

- **`WPBC_CSS::enqueue( $where_to_load )`**: This is the main method, which is called on the appropriate WordPress hook (`admin_enqueue_scripts` or `wp_enqueue_scripts`).
  - **Global Styles**: It starts by enqueueing several stylesheets that are used in both the admin and front-end contexts:
    - `wpdevelop-bts`: A custom version of Bootstrap.
    - `wpbc-tippy-popover` & `wpbc-tippy-times`: Styles for the Tippy.js tooltip and popover library.
    - `wpbc-material-design-icons`: An icon font.
    - **`wpbc-ui-both.css`**: A key stylesheet whose name implies it contains UI styles used in *both* admin and front-end.

  - **Admin Styles**: Inside an `if ( 'admin' === $where_to_load )` block, it enqueues a comprehensive list of stylesheets specifically for the admin panel:
    - `wpbc-all-admin.min.css`: A minified file containing several concatenated stylesheets, likely for performance.
    - A modular set of individual files for different admin components: `admin-menu.css`, `admin-toolbar.css`, `settings-page.css`, `admin-listing-table.css`, etc.
    - **`admin.css`** and **`admin-skin.css`**: The main stylesheets that define the overall look and feel of the admin pages.

  - **Inline & Conditional Styles**: It uses `wp_add_inline_style()` to add dynamic CSS rules, for example, to modify the admin menu when a user is being impersonated. It also contains logic to load front-end styles on certain admin pages (like the Add New Booking page) where a calendar is displayed.

  - **Conflict Removal**: The `remove_conflicts()` method is called to explicitly `wp_dequeue_style` stylesheets from other popular plugins and themes (e.g., WPML, The7 theme, Chosen) that are known to cause CSS conflicts.

- **`wpbc_enqueue_styles__front_end()`**: This function is responsible for loading stylesheets on the public-facing side of the site, including `client.css` and styles for the time picker.

- **`wpbc_enqueue_styles__calendar()`**: This function loads the base `calendar.css` and then calls `wpbc_get_calendar_skin_url()` to load the specific skin file chosen by the admin.

- **`wpbc_get_calendar_skin_url()`**: A helper function that retrieves the path to the selected calendar skin. It contains logic to check for the skin file first in a custom user directory (`/wp-content/uploads/wpbc_skins/`) and then falls back to the plugin's default `/css/skins/` directory. This makes the calendar skins easily extensible.

## Features Enabled
This file is responsible for the entire visual presentation of the plugin. It doesn't add HTML elements, but it ensures that all elements are styled correctly.

- **Consistent Admin UI**: It loads all the CSS required to style the custom buttons, dropdowns, toggles, and meta boxes we have analyzed.
- **Skinnable Calendar**: It implements the logic for loading different calendar skins, allowing users to customize the front-end appearance.
- **Conflict Prevention**: It proactively prevents UI issues by removing known conflicting stylesheets from other plugins.

## Extension Opportunities
- **`do_action( 'wpbc_enqueue_css_files', $where_to_load )`**: This custom action hook at the end of the `enqueue` method is the primary, intended way for developers to add their own stylesheets. You can hook a function here to run `wp_enqueue_style` for your own CSS, ensuring it loads after the plugin's main styles.

  ```php
  function my_custom_wpbc_styles() {
      wp_enqueue_style( 'my-wpbc-customizations', get_stylesheet_directory_uri() . '/my-wpbc-styles.css' );
  }
  add_action( 'wpbc_enqueue_css_files', 'my_custom_wpbc_styles' );
  ```

- **Custom Calendar Skins**: The `wpbc_get_calendar_skin_url()` function provides a built-in mechanism for overriding or adding new calendar skins. By creating a `.css` file and placing it in the `/wp-content/uploads/wpbc_skins/` directory, it will become available in the skin selection dropdown in the settings.

## Next File Recommendations
We have now identified the map of all CSS files. The next logical step is to read the most important ones to see the actual styling rules.

1.  **`css/wpbc_ui_both.css`**: **Top Priority.** This file is loaded on both the front-end and back-end. Its name suggests it contains the core styles for the UI elements used everywhere, including the `wpbc_flex_*` components. It's the key to understanding the base styling of the modern UI.
2.  **`css/admin.css`**: This is the main stylesheet for the admin panel. It will contain the bulk of the layout and styling rules for settings pages, tables, and other admin-specific containers.
3.  **`core/wpbc-js.php`**: We have now covered the HTML structure and the CSS styling. The final piece of the puzzle is the client-side interactivity. This file is the JavaScript counterpart to `wpbc-css.php` and will show us how the JS assets are loaded.
