# File Analysis: `vendors` Directory

## High-Level Overview

The `vendors` directory contains a collection of third-party JavaScript and CSS libraries that the Booking Calendar plugin depends on. These libraries provide a wide range of functionality, from UI components and styling to advanced features like input masking and drag-and-drop. The directory also contains some of the plugin's own custom-built assets.

Architecturally, this directory represents the plugin's external dependencies. By keeping these libraries in a dedicated `vendors` folder, the developers separate the core plugin code from the third-party code, which is a standard and good practice for maintainability.

## Library Breakdown

The libraries can be grouped by their functionality:

### UI and Styling

-   **Bootstrap (`_custom/bootstrap-css`)**: A custom build of the Bootstrap CSS framework. This provides the foundational grid system, buttons, and other UI components that are then customized by the plugin's own stylesheets.
-   **Icon Fonts (`_custom/bootstrap-icons`, `_custom/material-design-icons`)**: The plugin uses both Bootstrap Icons and Google's Material Design Icons to provide a rich set of icons throughout the admin panel.
-   **Chosen (`chosen`)**: A popular jQuery plugin that makes long, unwieldy `<select>` boxes more user-friendly. This is likely used for the resource selection dropdowns and other complex select fields in the admin panel.
-   **Tippy.js (`_custom/tippy.js`)**: A modern tooltip and popover library. This is used to power the informational pop-ups that appear when hovering over dates in the calendar.
-   **Tether & Shepherd (`tether`, `tether-shepherd`)**: Tether is a library for positioning elements next to each other (a dependency for Shepherd). Shepherd is a library for creating guided user tours, which is likely used for onboarding new users.
-   **Simplebar (`simplebar`)**: A library for creating custom, cross-browser scrollbars, used to improve the appearance of scrollable areas.

### Form Functionality

-   **IMask.js (`imask`)**: A vanilla JavaScript input masking library. As seen in the analysis of `wpbc_phone_validator.js`, this is the engine that powers the smart phone number formatting.

### Admin Panel Interactivity

-   **SortableJS (`sortablejs`)**: A JavaScript library for creating sortable, drag-and-drop lists. This is likely used in the admin panel for reordering items, such as booking resources or custom form fields.

### Custom Plugin Utilities

-   **Print Utility (`_custom/wpbc_js_print`)**: Contains a custom script, `wpbc_js_print.js`, which is likely a utility for handling the printing of booking details or reports from the admin panel.
-   **Dropdown Modal (`_custom/dropdown_modal`)**: Contains a custom script for dropdown modals, likely used for some of the more complex UI interactions in the admin panel.

## Extension Opportunities

There are no direct extension opportunities within this directory, as it contains third-party code. Developers should not modify these files directly, as any changes would be overwritten by a plugin update.

However, understanding these dependencies is crucial for extension. For example, a developer wanting to add a new sortable list to a custom admin page could enqueue the `Sortable.js` script from this directory to ensure they are using the same version as the core plugin, preventing conflicts.

## Next File Recommendations

This review provides a comprehensive overview of the plugin's third-party dependencies. The next logical steps are to investigate the remaining major un-analyzed features.

1.  **`includes/elementor-booking-form/elementor-booking-form.php`**: This is a top priority. It will show how the plugin integrates with the Elementor page builder, a key feature for modern WordPress sites, and provides a good comparison to the Gutenberg integration we've already analyzed.
2.  **`css/skins/premium-black.css`**: We have analyzed one skin (`multidays.css`). Analyzing a different one, like a premium dark skin, would provide a more complete picture of the skinning system's capabilities and CSS overrides.
3.  **`includes/_functions/wpbc-booking-functions.php`**: This file is in a directory we have not explored. Its name suggests it contains core functions related to the booking process and would be valuable to analyze.
