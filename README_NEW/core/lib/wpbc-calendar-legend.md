# File Analysis: core/lib/wpbc-calendar-legend.php

## High-Level Overview
This file is responsible for generating the HTML for the booking calendar's legend. The legend displays the meaning of different date colors and styles (e.g., available, pending, approved).

Its functionality is delivered in two ways:
1.  **Automatically**: Based on settings configured in the WordPress admin panel, a legend can be shown alongside the calendar.
2.  **Manually**: A user can insert a `[legend_items]` shortcode into a booking form to display a customized version of the legend.

The file reads plugin options to determine which legend items to show, what text to display, and how to style them, ensuring it integrates with the overall plugin configuration.

## Detailed Explanation
The file contains three primary functions that work together to produce the legend.

- **`wpbc_get_calendar_legend()`**
  This function acts as a controller. It checks if the legend is enabled via the plugin option `booking_is_show_legend`. If enabled, it gathers which specific items (available, pending, etc.) are also enabled via their own options and then calls the main HTML rendering function.

- **`wpbc_get_calendar_legend__content_html( $params )`**
  This is the core rendering engine. It builds the raw HTML for each possible legend item by constructing strings of `<table>`, `<td>`, and `<div>` elements with specific CSS classes (`date_available`, `date_approved`, `full_day_booking`, etc.) to mimic the appearance of actual calendar cells.
  - It takes a `$params` array to customize the output, including which items to show (`items`), orientation (`is_vertical`), and the text for the day number (`text_for_day_cell`).
  - The descriptive titles for each item are retrieved using `wpbc_lang( get_bk_option( '...' ) )`, making them both configurable and translatable.

- **`wpbc_replace_shortcodes_in_booking_form__legend_items( $return_form, ... )`**
  This function is hooked into the `wpbc_replace_shortcodes_in_booking_form` filter. It parses the content of a booking form to find and replace `[legend_items ...]` shortcodes.
  - It uses a helper function, `wpbc_get_params_of_shortcode_in_string()`, to extract attributes from the shortcode (e.g., `items="available,pending"`).
  - This allows an admin to override the global legend settings and place a highly customized legend anywhere within the form.

```php
// The filter that enables the [legend_items] shortcode functionality
add_filter(  'wpbc_replace_shortcodes_in_booking_form', 'wpbc_replace_shortcodes_in_booking_form__legend_items', 10, 3 );
```

## Features Enabled
### Admin Menu
This file does not directly create any admin menus, pages, or settings. However, it is a consumer of settings configured elsewhere in the plugin. The admin can control the legend's appearance and content via options on a settings page (likely **Booking > Settings > Form** or a similar general settings page). Examples of options it reads include:
- `booking_is_show_legend` (to show/hide the legend)
- `booking_legend_is_vertical` (to control orientation)
- `booking_legend_is_show_numbers` (to show/hide day numbers in the legend items)
- `booking_legend_text_for_item_available` (to set the descriptive text)

### User-Facing
- **Calendar Legend**: The primary feature is the visual legend that appears next to the front-end booking calendar, helping users understand the booking status of each day.
- **`[legend_items]` Shortcode**: It registers a powerful shortcode that can be used within the booking form editor. This gives administrators fine-grained control over the legend's content and placement on a per-form basis.

Example shortcode usage:
```shortcode
[legend_items items="available,unavailable,pending" is_vertical="1" text_for_day_cell=""]
```

## Extension Opportunities
The current implementation has limited extensibility as it does not contain any WordPress `actions` or `filters` for developers to hook into.

- **Potential Risks**:
  - **Rigid HTML Generation**: The HTML is constructed via string concatenation, which is inflexible and can be difficult to modify without breaking the layout.
  - **No Easy Extension**: Adding a new custom legend item (e.g., "Maintenance Day") would require modifying the `wpbc_get_calendar_legend__content_html` function directly. Any such changes would be lost during a plugin update.

- **Recommended Improvements**:
  To make the legend extensible, a filter could be introduced before the final HTML is assembled. For example, inside `wpbc_get_calendar_legend__content_html`, the array defining the legend items could be passed through a filter:

  ```php
  // Inside wpbc_get_calendar_legend__content_html()...

  $items_arr = array(
      'available' => array( /* ... */ ),
      'approved' => array( /* ... */ ),
      // ... other items
  );

  /**
   * Filter the calendar legend item definitions.
   *
   * @param array $items_arr The array of legend item definitions.
   * @param array $params    The parameters passed to the function.
   */
  $items_arr = apply_filters( 'wpbc_calendar_legend_items_definition', $items_arr, $params );

  // ... then proceed to loop through $items_arr to build the HTML
  ```
  With this filter, a developer could safely add, remove, or modify legend items from their theme's `functions.php` or a separate custom plugin.

## Next File Recommendations
1.  **`core/admin/page-settings.php`**: Since the legend's behavior is heavily dependent on options set in the admin panel, analyzing this file is critical. It will reveal how the settings UI is constructed, where the `get_bk_option` values originate, and how they are saved to the database.
2.  **`core/wpbc-translation.php`**: The legend titles are processed by the `wpbc_lang()` function. Understanding this file is essential for grasping the plugin's internationalization (i18n) and localization strategy, which appears to be a custom implementation.
3.  **`core/form_parser.php`**: The `[legend_items]` shortcode is parsed by a custom function, `wpbc_get_params_of_shortcode_in_string`. This function is likely located in `core/form_parser.php`. Analyzing the form parser will explain the core mechanism for how the plugin handles shortcodes and other dynamic template tags within forms.
