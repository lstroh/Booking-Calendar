# File Analysis: `core/wpbc-translation.php`

This document provides a detailed analysis of the `core/wpbc-translation.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file is the complete internationalization (i18n) and localization (l10n) engine for the Booking Calendar plugin. It goes far beyond the standard WordPress translation system, providing a multi-layered approach to ensure the plugin is displayed in the correct language across different contexts (front-end, admin, AJAX).

Its primary responsibilities are:
1.  **Loading Translation Files:** It robustly detects the correct locale and loads the appropriate `.mo` language file, with fallbacks.
2.  **Compatibility Layers:** It includes specific compatibility logic for multilingual plugins like **Polylang** and **WPML**, ensuring it works seamlessly within those ecosystems.
3.  **Custom Inline Translations:** It implements a powerful custom shortcode, `[lang=xx_XX]...[/lang]`, that allows administrators to provide translations for a single text field directly within the admin panel.
4.  **Translation Management Tools:** It contains the backend logic for tools on the Settings page that allow admins to update translations from wordpress.org and check the status of existing language files.

## Detailed Explanation

The file's functionality can be broken down into several key areas:

### 1. Locale Detection and Loading

-   **`wpbc_load_translation()`**: Hooked into `plugins_loaded`, this is the main function that orchestrates the loading process.
-   **`wpbc_get_maybe_reloaded_booking_locale()`**: This is the brain of the locale detection. It determines the correct language to use by checking several sources in order:
    1.  It has a specific check for the **Polylang** plugin, using `pll_current_language()` to adopt its active locale.
    2.  It checks for `$_REQUEST` parameters (`wpdev_active_locale` or `wpbc_ajx_locale`) to allow forcing a specific locale during **AJAX requests**. This is critical for ensuring the correct language is used in dynamic front-end and back-end interactions.
    3.  If neither of the above is present, it defaults to the standard WordPress locale via `get_locale()`.
-   **`wpbc_load_plugin_translation_file__mo()`**: This function handles the actual loading of the `.mo` file. It has a complex fallback system, first trying the standard WordPress function `load_plugin_textdomain` (which checks `wp-content/languages/plugins/`), and if that fails, it tries to load the file directly from the plugin's own `/languages` directory.

### 2. Custom Inline Translation System

-   **`wpbc_lang( $content_orig )`**: This is a wrapper for `wpdev_check_for_active_language()`. This function is the parser for the `[lang=xx_XX]` shortcode.
-   **Functionality**: An admin can enter text like `Thank you[lang=fr_FR]Merci` into a settings field. The `wpbc_lang` function parses this string, identifies the sections for each language, and returns only the content that matches the currently active locale. This provides a simple yet powerful way to manage multilingual content without needing separate settings for each language.

### 3. Third-Party Plugin Compatibility

-   **`wpbc_check_wpml_tags( $text, $locale )`**: Provides explicit support for **WPML**. It finds text wrapped in `[wpml]...[/wpml]` tags and uses WPML's `wpml_register_single_string` and `wpml_translate_single_string` hooks to register the string with WPML and retrieve the correct translation.
-   **`wpbc_bk_check_qtranslate( $text, $locale )`**: Provides legacy support for the **qTranslate** plugin by parsing its `<!--:xx-->` language tags.

### 4. Translation Management Tools

-   **`wpbc_update_translations__from_wp()`**: Contains the logic for the "Update Translations" button. It uses the `WP_Upgrader` class with a custom skin to programmatically download and install the latest language packs for the plugin from both wordpress.org and the wpbookingcalendar.com servers.
-   **`wpbc_show_translation_status...()` functions**: These functions power the "Show translations status" tool. They scan the `.po` files in both the WordPress languages directory and the plugin's local directory, parse them to count the number of translated, untranslated, and "fuzzy" strings, and then generate a report for the administrator.
-   **`wpbc_pot_to_php()`**: A developer-only tool that reads the main `.pot` file and generates PHP files containing all translatable strings. This is a clever technique to ensure that dynamic strings are discoverable by translation tools.

## Features Enabled

### Admin Menu

-   Provides the backend logic for the **"Update Translations"** and **"Show translations status"** buttons on the Booking > Settings > System Info page.
-   Enables the use of `[lang=xx_XX]` and `[wpml]` shortcodes in various text fields throughout the admin panel, allowing for inline translation of settings.

### User-Facing

-   This file is entirely responsible for ensuring that the **correct language is displayed to the front-end user**. It ensures the calendar, form, validation messages, and confirmation emails are all properly translated based on the site's active locale.

## Extension Opportunities

-   **Using `wpbc_lang()`**: Developers creating custom extensions can wrap their strings in `wpbc_lang()` to make them compatible with the plugin's built-in inline translation system.
-   **Using `plugin_locale` filter**: The file uses the `plugin_locale` filter to force its own detected locale. A developer could also use this filter with a high priority to implement a custom logic for choosing a language.
-   **Translation Consistency Check**: The `wpbc_check_translations` function, hooked to `gettext_booking`, prevents broken translations by ensuring the number of placeholders (like `%s`) matches between the original string and the translation. This is a good pattern to follow for custom code.

## Next File Recommendations

Understanding the translation system is a major step. The most logical next step is to analyze the files that structure the admin panel where these settings and tools are presented.

1.  **`core/any/class-admin-menu.php`**: **Top Priority.** This file is responsible for creating the plugin's entire menu structure in the WordPress admin panel. Analyzing it will provide a complete map of the backend UI and show how pages like "Settings" and "Bookings" are registered.
2.  **`core/lib/wpbc-booking-new.php`**: This file likely handles the manual creation of bookings from the admin panel. This is a core workflow that will use many of the translated strings and settings we've seen.
3.  **`core/any/api-emails.php`**: The analysis of `wpbc-emails.php` showed that it's a companion to a core Email API. This file almost certainly contains that API, and analyzing it would complete our understanding of the email system.
