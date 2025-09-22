# File Analysis: `css/skins/multidays.css`

## High-Level Overview

This file is a "skin" for the front-end booking calendar. Its sole purpose is to override the base structural styles from `calendar.css` to provide a specific and distinct color theme. This particular skin, "Multidays," is characterized by a bright green for available dates, a reddish-orange for approved (booked) dates, and a yellow for pending dates.

Architecturally, this file is a clear example of the plugin's theming system. It demonstrates how a new visual appearance can be created simply by targeting the specific CSS classes that the plugin's JavaScript applies to date cells based on their status. It contains no layout or structural rules, focusing only on presentational properties like `background-color`, `color`, and `border`.

## Detailed Explanation

This is a purely presentational CSS file and contains no PHP or JavaScript logic. It works by providing more specific CSS rules that override the default styles.

-   **Color Palette**: The skin defines a clear color-coded system for date statuses:
    -   **Available (`.date_available`)**: A solid, bright green background (`#00D025`) with white text.
    -   **Approved/Booked (`.date_approved`)**: A reddish-orange background (`#f50`) with dark, brownish-red text (`#a23c08`).
    -   **Pending (`.date2approve`)**: A yellow/orange background (`#FFD200`) with dark, brownish-yellow text (`#8e7c28`).
    -   **Selected (`.datepick-current-day`)**: A dark grey background (`#555`) with light grey text.
    -   **Hover (`.datepick-days-cell-over`)**: A medium grey background (`#aaa`).

-   **Integration with Modern Features**: The skin is up-to-date with the plugin's modern rendering engine.
    -   **SVG Polygon Styling**: It correctly styles the check-in/check-out day triangles by targeting the SVG polygons directly. It first sets a default `fill` color (the "available" green) and then provides specific overrides for pending and approved states. This demonstrates how the skinning system is tied to the SVG-based changeover day feature.
        ```css
        /* Approved */
        .datepick-inline td.datepick-days-cell.check_in_time.check_in_time_date_approved .wpbc-cell-box .wpbc-co-in svg polygon {
            fill: #f50;
        }
        ```
    -   **Partially Booked Day Theming**: It supports the "Show partially booked days with available background" option by using a CSS Custom Property (`--wpbc_cal-available-day-color`) to ensure consistency.

-   **Admin-Specific Styles**: The skin includes a dedicated section (`.wpbc_ajx_availability_container`) that defines styles specifically for how unavailable dates should appear on the admin "Availability" page, using a striped background. This shows that skins can be context-aware and provide different styling for front-end vs. back-end views.

## Features Enabled

This file has no effect on the WordPress admin menu.

### User-Facing

-   **Calendar Theme**: It provides a distinct "Multidays" color scheme for the front-end calendar, allowing site administrators to choose a visual theme that fits their website's design.
-   **Clear Visual States**: The strong color contrast between available, pending, and booked dates makes it very easy for users to understand the calendar's availability at a glance.

## Extension Opportunities

-   **Creating Custom Skins (Primary Method)**: This file serves as a perfect template for creating a new custom calendar skin. A developer can:
    1.  Copy `multidays.css`.
    2.  Rename it to something unique, like `my-custom-skin.css`.
    3.  Change the color values for the various states (`.date_available`, `.date_approved`, etc.) to match their desired theme.
    4.  Upload the new file to the `/wp-content/uploads/wpbc_skins/` directory on their server.
    The plugin will automatically detect the new file and make it a selectable option on the **Appearance / Color Theme** settings page.

## Next File Recommendations

The analysis of the plugin's skinning system is now complete. The next logical steps are to do a deep dive into the core calendar library that the skins are applied to, or to investigate other major un-analyzed features.

1.  **`js/datepick/jquery.datepick.wpbc.9.0.js`**: **Top Priority.** This is the core, third-party jQuery Datepick library that has been customized for the plugin. It is the engine that renders the calendar grid, handles date selection logic, and applies the CSS classes (like `.date_available`) that this skin file targets. Analyzing it is essential for a deep understanding of the calendar's functionality.
2.  **`includes/elementor-booking-form/elementor-booking-form.php`**: This file will show how the plugin integrates with the Elementor page builder, which is a key feature for modern WordPress sites.
3.  **`vendors/` (Directory)**: We have not yet explored the contents of this directory. It likely contains other third-party libraries that the plugin depends on. Listing its contents would be a good step to understand the plugin's external dependencies.
