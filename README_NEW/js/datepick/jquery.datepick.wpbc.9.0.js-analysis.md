# File Analysis: `js/datepick/jquery.datepick.wpbc.9.0.js`

## High-Level Overview

This file is a heavily customized version of a third-party library, `jQuery.datepick` v3.7.1. It serves as the low-level rendering engine for the entire front-end calendar grid. Its core responsibility is to generate the HTML for the calendar, handle user interactions like clicking on dates and navigating between months, and provide a system of callbacks that the Booking Calendar plugin uses to inject its dynamic availability logic.

Architecturally, this file is the foundation of the calendar's UI. Higher-level scripts like `client.js` and `admin.js` interact with this library by calling the `.datepick()` function on a target element and passing a large configuration object. The most important parts of this configuration are the callback functions, especially `beforeShowDay`, which allows the Booking Calendar plugin to control the appearance and selectability of every single day in the calendar.

## Detailed Explanation

While the file is large, its core functionality and customizations can be understood by focusing on several key areas.

-   **Customization and Namespacing**: The file's header comments indicate a significant customization effort to prevent conflicts with other plugins that might use the same library. The developers have noted their intention to rename core components (e.g., `$.datepick` to `$.wpbc_calendar`, `.datepick` class to `.wpbc_calendar`), effectively namespacing the library to make it unique to Booking Calendar.

-   **The Callback System**: The library's primary integration point is its extensive callback system. The Booking Calendar plugin uses these callbacks to inject its own logic:
    -   **`beforeShowDay`**: This is the most critical callback. For every day cell that the library is about to render, it executes the function provided by Booking Calendar (e.g., `wpbc__calendar__apply_css_to_days`). This function returns an array containing `[is_selectable, 'css_classes', 'tooltip_text']`. The library then uses this information to apply the correct classes (`date_available`, `date_approved`, `timespartly`, etc.) to the cell, making it the core mechanism for displaying availability.
    -   **`onSelect`**: Booking Calendar provides a function to this callback to handle what happens after a user clicks on a date, such as updating the booking form with the selected dates.
    -   **`onHover`**: Used to implement the visual feedback when a user is selecting a date range.
    -   **`onChangeMonthYear`**: Used to trigger an AJAX call to load booking data for the new month and year when the user navigates the calendar.

-   **HTML Generation (`_generateHTML`)**: This is the main rendering loop of the library. It programmatically builds the entire HTML string for the calendar grid.
    -   It iterates through the weeks and days of the month.
    -   For each day, it calls the `beforeShowDay` callback to get the day's status.
    -   **Booking Calendar Customization**: The HTML generated for each cell is highly customized. It includes the specific `div` structure required for the modern calendar, including `<div class="wpbc-cell-box">`, `<div class="wpbc-diagonal-el">` (which contains the SVG polygons for check-in/out triangles), and the containers for in-cell info: `<div class="date-content-top">` and `<div class="date-content-bottom">`.
    -   It also calls the custom functions `wpbc_show_date_info_top()` and `wpbc_show_date_info_bottom()` to inject content like cost and availability hints into the cells.

-   **Custom Event Emitters**: The library has been modified to trigger custom jQuery events at key moments, which allows other scripts to be decoupled.
    -   `jQuery( 'body' ).trigger('wpbc_datepick_inline_calendar_loaded', ...)`: Fired once the calendar has been fully rendered for the first time.
    -   `jQuery( 'body' ).trigger('wpbc_datepick_inline_calendar_refresh', ...)`: Fired every time the calendar is redrawn (e.g., after month navigation).

## Features Enabled

This file is the engine that renders the entire interactive calendar grid on the front-end.

### User-Facing

-   **Calendar Grid Rendering**: It draws the entire calendar, including month headers, weekday names, and date cells.
-   **Date Selection**: It handles the logic for single date, multiple date, and date range selection.
-   **Dynamic Styling**: Through the `beforeShowDay` callback, it enables the dynamic application of all availability-related styling (e.g., green for available, red for booked).
-   **Check-in/Out Display**: It renders the complex HTML required for the split-day (changeover) view.
-   **In-Cell Information**: It provides the structure for displaying hints like cost and availability directly within the date cells.

## Extension Opportunities

-   **Overriding Callbacks**: The primary way to extend the calendar's functionality is to interact with its initialization. A developer could use JavaScript to wrap the `.datepick()` call, allowing them to intercept or extend the configuration object. For example, one could add a custom `onSelect` callback that runs after the plugin's default callback.

-   **Listening for Custom Events**: The safest and most decoupled way to interact with the calendar is to listen for the custom events it fires. A developer can safely run their own code after the calendar is loaded or refreshed by hooking into these events:
    ```javascript
    jQuery( 'body' ).on('wpbc_datepick_inline_calendar_loaded', function( event, resource_id, jCalContainer, inst ) { 
        // Your custom code here, runs once after calendar is first drawn
    } );
    jQuery( 'body' ).on('wpbc_datepick_inline_calendar_refresh', function( event, resource_id, inst ) { 
        // Your custom code here, runs every time the calendar is redrawn
    } );
    ```

## Next File Recommendations

This file represents the lowest level of the calendar UI. Having analyzed it, the understanding of the client-side architecture is very comprehensive. The next steps should be to investigate the plugin's third-party dependencies and its integration with other major platforms.

1.  **`vendors/` (Directory)**: We have not yet explored the contents of this directory. It likely contains other third-party libraries that the plugin depends on. Listing its contents is a good next step to understand all external dependencies.
2.  **`includes/elementor-booking-form/elementor-booking-form.php`**: This file will show how the plugin integrates with the Elementor page builder, which is a key feature for modern WordPress sites and provides a good contrast to the Gutenberg integration.
3.  **`css/skins/premium-black.css`**: We have analyzed one skin (`multidays.css`). Analyzing a different one, like a premium dark skin, would provide a more complete picture of the skinning system's capabilities and CSS overrides.
