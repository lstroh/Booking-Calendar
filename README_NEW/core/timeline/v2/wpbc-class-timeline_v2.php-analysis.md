# File Analysis: `core/timeline/v2/wpbc-class-timeline_v2.php`

## High-Level Overview

This file defines the `WPBC_TimelineFlex` class, the powerful engine responsible for rendering the booking **Timeline**—a Gantt-chart-like visual representation of bookings over time. This is a core data visualization feature for both the admin panel and the front-end.

Architecturally, this class is a versatile component that can be initialized in three different contexts: for the admin panel, for the front-end via a shortcode, and for subsequent navigation via AJAX. It fetches all relevant booking and resource data, processes it into a complex grid structure, and then renders the complete HTML for the timeline, including the booking bars and popover details.

## Detailed Explanation

The class is a complete, self-contained system for generating the timeline.

-   **Initialization Methods**:
    -   `admin_init()`: Initializes the timeline for the **Booking > Timeline** admin page. It gets its view parameters (e.g., which resources to show, the date range) from the URL `$_REQUEST` variables.
    -   `client_init( $attr )`: Initializes the timeline for the front-end, typically called by the `[bookingtimeline]` shortcode. It parses shortcode attributes (`$attr`) to configure the view.
    -   `ajax_init( $attr )`: Handles AJAX requests for navigation (e.g., clicking "Next" or "Previous" month). It recalculates the date range based on the navigation step and re-renders the timeline content.

-   **Data Fetching and Processing**:
    -   The class does not query the database directly. Instead, it calls the global function `wpbc_get_bookings_objects()`, which is the plugin's main data-access layer for retrieving bookings based on a wide range of filters.
    -   **`wpbc_get_dates_and_times_for_timeline( $bookings )`**: This is the most complex and critical method in the class. It takes the raw booking data and transforms it into two structured arrays optimized for rendering:
        1.  `$dates_array`: A simple associative array mapping a date string (`Y-m-d`) to an array of booking IDs that fall on that date.
        2.  `$time_array_new`: A multi-dimensional array that maps bookings to specific hourly slots within each day. It has sophisticated logic to handle full-day bookings (which fill all slots), as well as bookings with specific start and end times. It correctly processes check-in/out times by looking at the last digit of the time's "seconds" component (`:01` for check-in, `:02` for check-out).

-   **Rendering Logic**:
    -   `show_timeline()`: The main public method that orchestrates the rendering. It iterates through the booking resources and calls `wpbc_show_timeline_booking_row()` for each one.
    -   `wpbc_show_timeline_header_row()`: Renders the top two rows of the timeline grid: the month/year row and the day/time row. The structure of this header changes dynamically based on the current view mode (day, week, month, year).
    -   `wpbc_show_timeline_booking_row()`: Renders a single resource's row. It iterates through the days and time slots, checks the processed data arrays, and renders the colored "booking bars" (pipelines) for any bookings that exist in that slot.
    -   `get_booking_title_for_timeline()` and `wpbc_get_booking_info_4_popover()`: Helper methods that generate the text content displayed inside the booking bars and in the pop-up details, respectively.

## Features Enabled

### Admin Menu

-   **Timeline Page**: This class is the engine that renders the entire timeline on the **Booking > Timeline** page. This provides administrators with a powerful visual tool to see availability and manage bookings across multiple resources at a glance.

### User-Facing

-   **`[bookingtimeline]` Shortcode**: This class provides the backend logic for the `[bookingtimeline]` shortcode, allowing a public-facing timeline to be embedded on any page or post.
-   **Customer-Specific View**: The timeline supports a `booking_hash` parameter, which allows it to be filtered to show only the bookings belonging to a specific customer, which is useful for "My Bookings" pages.

## Extension Opportunities

This class is a large, monolithic component with very few internal hooks, making it difficult to extend safely.

-   **Filtering the Data Source**: The most viable extension point is not in this file, but in the `wpbc_get_bookings_objects()` function (defined in `core/admin/wpbc-sql.php`), which this class calls. That function contains numerous filters that allow a developer to modify the SQL query that fetches the bookings. By filtering the data before it even reaches the timeline class, one could add custom bookings or alter existing ones.

-   **JavaScript/CSS Overrides**: For purely visual changes, a developer could use custom JavaScript to modify the DOM after the timeline has been rendered, or use custom CSS to override the default styles. This is a brittle approach, as changes to the timeline's HTML structure in a plugin update could break the custom code.

-   **Potential Risks**: The data processing logic, especially in `wpbc_get_dates_and_times_for_timeline`, is highly complex and interdependent. A small change could have unintended consequences on how bookings are displayed, potentially showing incorrect availability. Direct modification is strongly discouraged.

## Next File Recommendations

This file represents the last major un-analyzed UI component. The next steps should focus on the remaining core data structures and the low-level libraries that power the UI.

1.  **`includes/page-resource-free/page-resource-free.php`** — This is the most important remaining file. It will show how booking resources (the fundamental "bookable" items) are created and managed in the free version of the plugin, which is key to understanding the plugin's core data model.
2.  **`js/datepick/jquery.datepick.wpbc.9.0.js`** — This is the core jQuery Datepick library that has been customized for the plugin. Analyzing it would provide a deep understanding of how the calendar UI is rendered and how date selection, styling, and user interactions are handled at a low level.
3.  **`css/calendar.css`** — This is the base stylesheet for the calendar. Analyzing it would show the default styling for dates, which is then customized by the various skin files and is fundamental to the front-end user experience.
