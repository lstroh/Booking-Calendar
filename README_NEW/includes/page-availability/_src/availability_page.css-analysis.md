# File Analysis: `includes/page-availability/_src/availability_page.css`

## High-Level Overview

This file is the human-readable **source stylesheet** for the **Booking > Availability** admin page. Its purpose is to define the layout and structural styling for this specific page, creating a modern, two-column interface using CSS Flexbox. It arranges the main availability calendar on the left and a series of action widgets on the right.

Architecturally, this file is the "blueprint" for the page's visual structure. It is intended to be processed by a build tool (minified) and saved to the `_out` directory as `availability_page.min.css`, which is the file loaded in production for performance. It works in direct concert with the PHP files that generate the page's HTML (`availability__page.php`, `availability__toolbar_ui.php`).

## Detailed Explanation

This is a CSS file and does not contain any PHP code, functions, or WordPress hooks. Its functionality is delivered entirely through CSS selectors that target HTML elements.

-   **Key CSS Selectors & Structure**:
    -   `.wpbc_ajx_avy__container`: The main flexbox container for the page, which arranges the primary sections in a row (`flex-flow: row nowrap`).
    -   `.wpbc_ajx_avy__section_left`: The primary content area holding the calendar. It is configured with `flex: 1 1 auto;` to allow it to grow and fill available space.
    -   `.wpbc_ajx_avy__section_right`: The sidebar area for widgets. It is configured with `flex: 0 1 auto;`, preventing it from growing and creating a fixed-width sidebar effect.
    -   `.wpbc_widgets` & `.wpbc_widget`: These classes define the structure and styling for the individual widgets within the right-hand sidebar.

-   **Styling Approach**:
    -   **Layout**: The file exclusively uses CSS Flexbox for layout, which is a modern and robust approach for creating responsive grid systems.
    -   **Responsive Design**: It includes a `@media (max-width: 470px)` query to handle mobile layouts. On small screens, it changes the flex-flow to `wrap`, causing the right sidebar to stack vertically below the calendar.
    -   **Sticky Mobile Widget**: A key UX enhancement for mobile is the `.wpbc_widget_available_unavailable` rule within the media query. It applies `position: fixed` and `bottom: 0` to the main action widget, ensuring the "Apply" button is always visible and accessible at the bottom of the screen on mobile devices.
    -   **Style Overrides**: The file contains a specific override (`border-radius: 0 !important;`) for `.datepick-days-cell`. This ensures that on the Availability page, the calendar cells are always square, providing a clean, grid-like appearance suitable for an admin tool, regardless of the front-end skin settings.

## Features Enabled

### Admin Menu

-   This file does not add any pages or options to the admin menu.
-   It is responsible for the entire layout and responsive structure of the **Booking > Availability** page, making it a usable and well-organized interface.
-   It enables a key mobile usability feature: the "sticky footer" action widget that keeps primary controls accessible on small screens.

### User-Facing

-   This file has **no effect** on the user-facing side of the website. It is exclusively for styling the admin panel.

## Extension Opportunities

-   **Potential Risks**: While this is the source file, **direct modification is not recommended for typical users or developers extending the plugin**. Any changes will be overwritten by future plugin updates. For developers contributing directly to the plugin, any changes to the flexbox properties must be tested carefully to avoid breaking the responsive layout.

-   **Recommended Safe Extension Points**:
    1.  **Override with a Custom CSS File**: The safest and most update-proof method is to create your own custom CSS file and load it on the Availability page via the `admin_enqueue_scripts` WordPress action. Ensure your stylesheet is loaded *after* the plugin's to allow your rules to take precedence.
    2.  **Build Process (for Core Contributors)**: If you are contributing to the plugin itself, the intended workflow is to edit this source file and then use the project's build process (e.g., a Grunt, Gulp, or Webpack script) to re-compile and minify it into the `_out` directory.

## Next File Recommendations

The analysis of the Availability page's CSS is now complete. The following unreviewed files are recommended to broaden the understanding of other core plugin areas.

1.  **`js/wpbc_times.js`** — This file is the likely JavaScript counterpart to the time-slot selection feature. Analyzing it is crucial for understanding how time-based bookings are handled on the front-end, including how available slots are displayed and how user selections are processed.
2.  **`includes/page-settings-form-options/page-settings-form-options.php`** — This file appears to be responsible for the admin settings page where users configure the booking form fields. Understanding this file is key to learning how the plugin allows for form customization and how those settings are saved and used.
3.  **`css/skins/multidays.css`** — We have analyzed the base calendar CSS. Looking at a specific skin file like this one will reveal how the plugin's theming system works, applying custom colors and styles for different calendar appearances, such as for multi-day selections.
