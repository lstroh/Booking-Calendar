# File Analysis: `core/timeline/v2/_src/timeline_v2.js`

This document provides a detailed analysis of the JavaScript file `core/timeline/v2/_src/timeline_v2.js`, which is part of the Booking Calendar plugin.

## High-Level Overview

This file is the client-side script responsible for handling the dynamic navigation of the "Flex Timeline" view. Its primary purpose is to fetch and display different time periods (e.g., next/previous month) for the booking timeline using AJAX, providing a smooth user experience without requiring a full page reload.

Architecturally, this file is the **source code** for the timeline's navigation logic. In this specific case, the source code is identical to the distributed version found in the `_out` directory, indicating that no transpilation or complex build process is applied to this particular file. It is the direct counterpart to the `WPBC_FLEXTIMELINE_NAV` AJAX action handled by the plugin's PHP backend.

## Detailed Explanation

The file contains a single primary function, `wpbc_flextimeline_nav`, which orchestrates the timeline navigation.

-   **Key Function:** `wpbc_flextimeline_nav(timeline_obj, nav_step)`
    -   `timeline_obj`: A JavaScript object containing configuration and state for a specific timeline instance on the page, including its unique `html_client_id`.
    -   `nav_step`: An integer or string indicating the direction and magnitude of the navigation (e.g., `1`, `-1`).

-   **AJAX Interaction:**
    -   The function initiates an AJAX POST request to the WordPress AJAX handler (`admin-ajax.php`), using the global `wpbc_url_ajax` variable (which is provided by the server-side PHP).
    -   It sends the following data in the request payload:
        -   `action`: `'WPBC_FLEXTIMELINE_NAV'` — This is the WordPress AJAX action hook that the plugin's PHP backend uses to identify and handle the request.
        -   `timeline_obj`: The timeline configuration object.
        -   `nav_step`: The navigation step.
        -   `wpdev_active_locale`: The current site language, retrieved from the global `_wpbc` object.
        -   `wpbc_nonce`: A security nonce retrieved from a hidden field in the DOM to prevent CSRF attacks.
    -   Upon a successful response, the script injects the returned HTML directly into the timeline's container (`.wpbc_timeline_ajax_replace`), effectively redrawing the timeline with the new data.

```javascript
function wpbc_flextimeline_nav( timeline_obj, nav_step ){

    // Trigger a custom event for extensibility
    jQuery( ".wpbc_timeline_front_end" ).trigger( "timeline_nav" , [ timeline_obj, nav_step ] );

    // Show a loading indicator
    jQuery( '#'+timeline_obj.html_client_id + ' .flex_tl_prev,#'+timeline_obj.html_client_id + ' .flex_tl_next').remove();
    jQuery('#'+timeline_obj.html_client_id + ' .flex_tl_title').html( '<span class="wpbc_icn_rotate_right wpbc_spin"></span> &nbsp Loading...' );

    // Perform AJAX request
    jQuery.ajax({
        url: wpbc_url_ajax,
        type:'POST',
        success: function ( data, textStatus ){
            if( textStatus == 'success') {
                jQuery('#' + timeline_obj.html_client_id + ' .wpbc_timeline_ajax_replace' ).html( data );
                return true;
            }
        },
        error:  function ( XMLHttpRequest, textStatus, errorThrown){
            window.status = 'Ajax Error! Status: ' + textStatus;
            alert( 'Ajax Error! Status: ' + XMLHttpRequest.status + ' ' + XMLHttpRequest.statusText );
        },
        data:{
                action:              'WPBC_FLEXTIMELINE_NAV',
                timeline_obj:        timeline_obj,
                nav_step:            nav_step,
                wpdev_active_locale: _wpbc.get_other_param( 'locale_active' ),
                wpbc_nonce:          document.getElementById('wpbc_nonce_'+ timeline_obj.html_client_id).value
        }
    });
}
```

## Features Enabled

### Admin Menu

This file does not directly register any admin menus or pages. It provides the client-side logic for the timeline component, which may be rendered on various admin pages (e.g., a dashboard widget or the dedicated **Booking > Timeline** page) to display booking data.

### User-Facing

This script is primarily for the user-facing side of the website.

-   **AJAX Timeline:** It enables the core functionality of the booking timeline, allowing visitors to navigate through availability without reloading the page.
-   **Shortcode/Block Functionality:** This script is loaded to support a shortcode (e.g., `[bookingtimeline]`) or a Gutenberg block that renders the timeline view for users to see booking availability.

## Extension Opportunities

-   **Primary Extension Point:** The safest and most effective way to extend this functionality is by using the custom jQuery event triggered at the start of the function:
    ```javascript
    jQuery(".wpbc_timeline_front_end").trigger("timeline_nav", [timeline_obj, nav_step]);
    ```
    You can hook into this event to execute custom JavaScript before the AJAX request is sent. This could be used for analytics, modifying the request, or triggering other UI changes.

    **Example:**
    ```javascript
    jQuery(document).ready(function($) {
        $('.wpbc_timeline_front_end').on('timeline_nav', function(event, timeline_obj, nav_step) {
            console.log('Timeline is navigating.', {
                id: timeline_obj.html_client_id,
                step: nav_step
            });
            // Add your custom logic here
        });
    });
    ```

-   **Potential Risks:**
    -   While this is the source file, direct modification is still discouraged in favor of using the provided hooks, as this is more maintainable and less prone to breaking with future plugin updates.
    -   The server-side AJAX handler (`WPBC_FLEXTIMELINE_NAV`) is a critical dependency. Any changes to the data sent must be compatible with the data structure expected by the PHP backend.

## Next File Recommendations

The analysis of the timeline feature's client-side and styling aspects is now complete. The next logical steps are to investigate the remaining core data structures and low-level libraries of the plugin.

1.  **`includes/page-resource-free/page-resource-free.php`**: This is the most important remaining file. It will show how booking resources (the fundamental "bookable" items) are created and managed in the free version of the plugin, which is key to understanding the plugin's core data model.
2.  **`js/datepick/jquery.datepick.wpbc.9.0.js`**: This is the core jQuery plugin that powers the calendar itself. Analyzing it would reveal the low-level client-side logic for date selection, rendering days, and applying CSS classes, which `client.js` and `admin.js` build upon.
3.  **`css/calendar.css`**: This is the base stylesheet for the calendar. Analyzing it would show the default styling for dates, which is then customized by the various skin files and is fundamental to the front-end user experience.
