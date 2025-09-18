# File Analysis: `core/class/welcome_current.php`

## High-Level Overview

This file acts as the content provider for the plugin's "What's New" page. It is not a standalone file but is included and utilized by the `WPBC_Welcome` class (defined in `wpbc-class-welcome.php`). Its sole purpose is to house the HTML and text for the release notes that are displayed to administrators after a plugin update.

Architecturally, this is a "view" file. It contains a series of version-specific functions (e.g., `wpbc_welcome_section_10_14`, `wpbc_welcome_section_10_13`) that are called by the main `WPBC_Welcome` class to render the content for each update. This modular structure makes it easy for the developers to add new sections for future releases.

## Detailed Explanation

The file consists of a series of procedural functions, each following a consistent pattern:

-   **`wpbc_welcome_section_*()` functions**: Each function is named after a specific plugin version (e.g., `wpbc_welcome_section_10_14`) and is responsible for rendering the release notes for that version.
    -   It takes a single argument, `$obj`, which is the instance of the `WPBC_Welcome` class that called it.
    -   It uses methods from the `$obj` instance, such as `$obj->expand_section_start()` and `$obj->section_img_url()`, to create a consistent collapsible layout and to generate correct image paths.
    -   The content is primarily static HTML, using `div`s with flexbox classes (`wpbc_wn_container`, `wpbc_wn_section`, `wpbc_wn_col`) to create multi-column layouts for text and images.
    -   All outputted strings are sanitized with `wp_kses_post()` to ensure security, and formatted with helper functions like `wpbc_replace_to_strong_symbols()`.
    -   Images and GIFs that showcase new features are loaded from an external asset path on `wpbookingcalendar.com`.

```php
// Example of the modular structure
function wpbc_welcome_section_10_14( $obj ){

	$section_param_arr = array( 'version_num' => '10.14', 'show_expand' => false );

	$obj->expand_section_start( $section_param_arr );

    // ... HTML content for version 10.14 ...

	$obj->expand_section_end( $section_param_arr );
}

function wpbc_welcome_section_10_13( $obj ){
    // ... content for version 10.13 ...
}
```

## Features Enabled

This file does not directly enable any features but provides the content for an important administrative user experience.

### Admin Menu

-   This file has no effect on the admin menu. It provides the informational content for the hidden "What's New" page (`index.php?page=wpbc-about`), which is rendered by the `WPBC_Welcome` class.

### User-Facing

-   This file has no user-facing features.

## Extension Opportunities

This file is not designed to be extended. The content for each version is hardcoded within its respective function, and there are no actions or filters provided to modify the output.

-   **Safe Extension Method**: It is not possible to safely extend or modify the content of the "What's New" page without directly editing this file, which is not recommended as changes would be lost on update.
-   **Suggested Improvements**: The only way to make this extensible would be for the developers to add `do_action()` hooks within each section function, for example `do_action( 'wpbc_welcome_section_10_14_after_content', $obj );`. This would allow other add-ons or custom plugins to append their own release notes to the page.

## Next File Recommendations

This analysis completes our understanding of the "What's New" feature. It is now time to focus on the remaining major, un-analyzed core functionalities of the plugin.

1.  **`core/sync/wpbc-gcal.php`** — **Top Priority.** This file is responsible for the Google Calendar synchronization feature. Analyzing it will reveal how the plugin handles complex, authenticated interactions with a major third-party API and manages data syncing.
2.  **`core/timeline/flex-timeline.php`** — The booking "Timeline" is a core administrative UI. This file will show how booking data is queried and rendered in a visual timeline format, providing insight into the plugin's data visualization techniques.
3.  **`core/class/wpbc-class-notices.php`** — This file likely defines a standardized system for creating and displaying admin notices throughout the plugin. Understanding it is key to learning how the plugin communicates important information to the site administrator.
