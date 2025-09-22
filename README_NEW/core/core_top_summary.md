Plugin Analysis Summary
Files Included
The following files were analyzed based on the provided source documents. All paths are relative to the plugin root directory.
1. core/form_parser.php
2. core/index.php
3. core/wpbc-core.php
4. core/wpbc-css.php
5. core/wpbc-dates.php
6. core/wpbc-debug.php
7. core/wpbc-dev-api.php
8. core/wpbc-emails.php
9. core/wpbc-js-vars.php
10. core/wpbc-js.php
11. core/wpbc-translation.php
12. core/wpbc.php
13. core/wpbc_functions.php
14. core/wpbc_functions_dates.php
Table of Contents
• File Analysis: core/form_parser.php
• File Analysis: core/index.php
• File Analysis: core/wpbc-core.php
• File Analysis: core/wpbc-css.php
• File Analysis: core/wpbc-dates.php
• File Analysis: core/wpbc-debug.php
• File Analysis: core/wpbc-dev-api.php
• File Analysis: core/wpbc-emails.php
• File Analysis: core/wpbc-js-vars.php
• File Analysis: core/wpbc-js.php
• File Analysis: core/wpbc-translation.php
• File Analysis: core/wpbc.php
• File Analysis: core/wpbc_functions.php
• File Analysis: core/wpbc_functions_dates.php
File-by-File Summaries
core/form_parser.php
• Source MD file name: form_parser.md
• Role (short sentence): Dedicated utility for parsing the plugin's custom text-based form configuration (shortcode-like syntax) into a structured PHP array.
• Key Technical Details: The main parsing engine is wpbc_parse_form($booking_form_configuration). It uses complex regular expressions (preg_match_all) to identify field types (e.g., text*, select) and captures their name, options, and values. Helper function wpbc_parse_form_shortcode_values() handles value parsing, including @@ syntax for display/submission separation.
• Features (Admin vs User): Admin: It is the engine that interprets the syntax used in the "Form" editor found in plugin settings. User: It is a prerequisite for rendering any booking form HTML on the front-end.
• Top Extension Opportunities: The list of supported shortcode types is hardcoded into the regex, presenting a notable limitation. Suggested improvement: Make the shortcode types filterable (e.g., using a wpbc_form_parser_shortcode_types filter) to allow custom fields.
• Suggested Next Files (from that MD): includes/page-form-simple/form_simple.php, core/lib/wpbc-booking-new.php, core/wpbc-translation.php.
core/index.php
• Source MD file name: index.md
• Role (short sentence): A standard WordPress security measure to prevent unauthorized directory listing on web servers.
• Key Technical Details: Contains no executable code, only a single PHP comment ("Silence is golden."). It interacts with no functions, hooks, APIs, or database.
• Features (Admin vs User): Admin: None. User: None.
• Top Extension Opportunities: None; modifying this file would violate WordPress best practices, as its purpose is purely to be an empty placeholder.
• Suggested Next Files (from that MD): core/any/class-admin-menu.php, core/form_parser.php, core/lib/wpbc-booking-new.php.
core/wpbc-core.php
• Source MD file name: wpbc-core.md, wpbc-core_v2.md
• Role (short sentence): A foundational architectural file that provides an internal hooks system and data abstraction wrappers for options and booking metadata.
• Key Technical Details: Implements a Custom Event System using global variables ($wpdev_bk_action, $wpdev_bk_filter) and functions like add_bk_action, apply_bk_filter, and make_bk_action. It uses option management wrappers (get_bk_option, update_bk_option) that apply filters before calling standard WordPress option functions. Booking meta functions (wpbc_save_booking_meta_option, wpbc_get_booking_meta_option) interact directly with the custom {$wpdb->prefix}booking table, storing meta data as a serialized array in the booking_options column using raw SQL via $wpdb.
• Features (Admin vs User): Admin: Indirectly underpins admin features by powering option and booking meta management. User: Indirectly enables user-facing features (e.g., custom fields, dynamic options) by providing data access infrastructure.
• Top Extension Opportunities: Utilize the custom hooks (add_bk_filter, add_bk_action). Specifically, filter wpdev_bk_get_option or wpdev_bk_update_option to intercept or modify settings. Use wpbc_save_booking_meta_option() to safely add custom data to a booking.
• Suggested Next Files (from that MD): core/wpbc-functions.php, includes/page-bookings/bookings__actions.php, core/wpbc-dates.php (from wpbc-core.md); core/any/class-admin-menu.php, core/form_parser.php, core/lib/wpbc-booking-new.php (from wpbc-core_v2.md).
core/wpbc-css.php
• Source MD file name: wpbc-css.md
• Role (short sentence): The central controller responsible for enqueueing all CSS stylesheets (core, libraries, and custom skins) for both the admin and front-end using WordPress asset hooks.
• Key Technical Details: Defines the WPBC_CSS class. Enqueues global styles like wpdevelop-bts (custom Bootstrap), wpbc-ui-both.css, and icon fonts. It loads extensive admin-specific stylesheets (e.g., wpbc-all-admin.min.css). wpbc_enqueue_styles__calendar() loads calendar.css and dynamically retrieves the skin URL via wpbc_get_calendar_skin_url(). Includes a remove_conflicts() method to explicitly wp_dequeue_style known conflicting assets.
• Features (Admin vs User): Admin: Loads all CSS for consistent UI styling of custom elements, buttons, and settings pages. User: Implements the logic for loading different calendar skins selected by the user, controlling the front-end appearance.
• Top Extension Opportunities: Use the custom action hook do_action( 'wpbc_enqueue_css_files', $where_to_load ) to add custom stylesheets. New calendar skins can be added by placing CSS files in the /wp-content/uploads/wpbc_skins/ directory.
• Suggested Next Files (from that MD): css/wpbc_ui_both.css, css/admin.css, core/wpbc-js.php.
core/wpbc-dates.php
• Source MD file name: wpbc-dates.md
• Role (short sentence): The central "dates engine" providing all date/time parsing, formatting, conversion, validation, and availability calculation logic for the plugin.
• Key Technical Details: Contains functions for date parsing and conversion (wpbc_get_dates_in_diff_formats), time handling (wpbc_get_time_in_24_hours_format), and date comparison. It uses $wpdb for direct queries to custom tables (booking, bookingdates) to retrieve booked dates (wpbc__sql__get_booked_dates) and calculate season availability (wpbc__sql__get_season_availability). All queries use prepared statements.
• Features (Admin vs User): Admin: Provides all date/time data for admin booking listings, filters, and calendar displays, supporting backend availability calculation. User: Drives all front-end calendar rendering, date selection, availability checking, and validation for booking forms.
• Top Extension Opportunities: Use or extend existing date/time functions for custom calendar needs. Hook into availability calculation by filtering/enhancing wpbc__sql__get_season_availability.
• Suggested Next Files (from that MD): includes/page-bookings/bookings__actions.php, core/wpbc-functions.php, core/wpbc-core.php.
core/wpbc-debug.php
• Source MD file name: wpbc-debug.md
• Role (short sentence): A collection of utility functions providing tools for debugging, performance monitoring, and server configuration diagnostics for developers.
• Key Technical Details: Functions are globally scoped and wrapped in ! function_exists() checks. Includes debuge() (variable dumping, optionally killing execution), debuge_error() (formatted error messages including file/line info and database errors via $EZSQL_ERROR), and debuge_speed() (query count/execution time). Critically, wpbc_check_post_key_max_number() diagnoses PHP/Suhosin configuration issues that prevent form submissions from saving. It uses a helper (wpbc_admin_show_top_notice) to display dynamic notices via client-side JavaScript.
• Features (Admin vs User): Admin: Provides mechanisms for displaying error messages, system notices, and diagnostic warnings (e.g., Suhosin check results) on admin pages. User: Has no user-facing features.
• Top Extension Opportunities: Developers can call debuge($variable, true) for inspection. The action do_action( 'wpbc_admin_show_top_notice', ...) can be used to trigger custom admin notices from other parts of the plugin.
• Suggested Next Files (from that MD): core/any/class-admin-menu.php, core/lib/wpbc-booking-new.php, includes/page-form-simple/form_simple.php.
core/wpbc-dev-api.php
• Source MD file name: wpbc-dev-api.md
• Role (short sentence): The official Developer API, providing a stable, programmatic abstraction layer for creating, modifying, and checking the availability of bookings.
• Key Technical Details: The core function is wpbc_api_booking_add_new(), which handles booking creation/editing by translating developer-friendly input into the plugin's complex internal serialized form string before calling the core wpbc_booking_save() function. wpbc_api_is_dates_booked() checks availability using the internal wpbc__where_to_save_booking() engine. It returns WP_Error objects on failure, following WordPress best practices. The file includes extensive documentation for action and filter hooks.
• Features (Admin vs User): Admin/User: None directly; its purpose is to enable third-party integrations (e.g., syncing with CRMs or creating custom front-ends).
• Top Extension Opportunities: Call wpbc_api_booking_add_new() programmatically. Utilize the documented action hooks (e.g., wpbc_track_new_booking, wpbc_booking_approved) to trigger custom logic based on plugin events.
• Suggested Next Files (from that MD): core/any/class-admin-menu.php, core/lib/wpbc-booking-new.php, core/wpbc-emails.php.
core/wpbc-emails.php
• Source MD file name: wpbc-emails.md
• Role (short sentence): The central hub for all email-related functionality, providing helper functions for validation, formatting, shortcode generation, and interacting with the underlying Email API.
• Key Technical Details: wpbc_validate_emails() cleans and formats email addresses, supporting "Name email@example.com" syntax. wpbc_wp_mail() is a wrapper for wp_mail() that temporarily hooks into phpmailer_init to fix the Sender header for better deliverability. It uses the wpbc_email_api_is_allow_send filter as a central kill-switch for sending emails. wpbc_get_email_help_shortcodes() dynamically generates the list of shortcodes available for use in email templates.
• Features (Admin vs User): Admin: Provides the content and list of available shortcodes for the email template editing pages (Settings > Emails). Supports Reply-To functionality on admin notifications. User: Responsible for sending all transactional emails (confirmation, approval, modification) to customers and administrators.
• Top Extension Opportunities: Use the wpbc_email_api_get_headers_after filter to modify or add custom email headers. Use the wpbc_email_api_is_allow_send filter to conditionally block specific emails from being sent.
• Suggested Next Files (from that MD): core/any/api-emails.php, core/any/class-admin-menu.php, core/lib/wpbc-booking-new.php.
core/wpbc-js-vars.php
• Source MD file name: wpbc-js-vars.md
• Role (short sentence): Acts as the primary data bridge, gathering configuration settings, translated strings, and timezone data from PHP and injecting it into the client-side JavaScript environment.
• Key Technical Details: Generates inline JavaScript that populates a global _wpbc object using methods like _wpbc.set_other_param() (for settings, URLs, time/locale) and _wpbc.set_message() (for translated strings). Crucially, it wraps this payload in a resilient bootstrap loader that polls and listens to events (wpbc-ready, DOMContentLoaded) to ensure data injection occurs successfully even when scripts are deferred or cached.
• Features (Admin vs User): This file is foundational, enabling dynamic configuration, internationalization (i18n), timezone awareness, and AJAX capability for the client-side scripts.
• Top Extension Opportunities: Use the wpbc_js_vars filter to add, modify, or remove variables from the main _wpbc object before injection. Alternatively, use the wpbc_define_js_vars action hook to call wp_localize_script and create a custom data object.
• Suggested Next Files (from that MD): js/client.js, core/lib/wpbc-booking-new.php, wpdev-booking.php.
core/wpbc-js.php
• Source MD file name: wpbc-js.md
• Role (short sentence): The master controller that manages the conditional loading, enqueueing, localization, and conflict resolution of all JavaScript assets used by the plugin.
• Key Technical Details: Defines the WPBC_JS class. The main orchestrator method, WPBC_JS::enqueue(), calls wpbc_js_load_vars() (for localization), wpbc_js_load_libs() (jQuery, UI), and wpbc_js_load_files() (core scripts like wpbc_all.js, client.js, admin.js). A critical function is filter_script_loader_tag(), which hooks script_loader_tag to programmatically remove async and defer attributes from the plugin's and jQuery's script tags to prevent race conditions. It also explicitly removes known conflicting scripts via wpbc_remove_js_conflicts().
• Features (Admin vs User): Admin: Powers all interactive elements of the admin panel, including AJAX requests for managing bookings and dynamic UI. User: Enables the entire front-end booking process, date selection, form validation, and AJAX submission.
• Top Extension Opportunities: Use the designated hook do_action( 'wpbc_enqueue_js_files', $where_to_load ) to safely enqueue custom scripts. Use do_action( 'wpbc_define_js_vars', $where_to_load ) to add custom data to the JavaScript environment via localization.
• Suggested Next Files (from that MD): js/admin.js, js/client.js, core/wpbc-js-vars.php.
core/wpbc-translation.php
• Source MD file name: wpbc-translation.md
• Role (short sentence): The comprehensive internationalization and localization engine, managing translation file loading, multilingual plugin compatibility (WPML/Polylang), and a custom inline translation system.
• Key Technical Details: wpbc_get_maybe_reloaded_booking_locale() intelligently detects the correct locale, checking Polylang and supporting AJAX requests by checking $_REQUEST parameters. wpbc_lang() parses the custom shortcode [lang=xx_XX]...[/lang] allowing administrators to embed translations directly within a single settings field. It includes explicit compatibility functions for WPML (wpbc_check_wpml_tags) and qTranslate. It contains logic (wpbc_update_translations__from_wp()) to update language packs programmatically.
• Features (Admin vs User): Admin: Powers the translation management tools (e.g., "Update Translations," "Show translations status"). Enables inline translation in settings fields. User: Entirely responsible for ensuring the calendar, forms, validation messages, and emails are displayed in the correct language and format based on the active locale.
• Top Extension Opportunities: Developers can wrap their strings in wpbc_lang() to utilize the plugin's built-in inline translation shortcode system. The plugin_locale filter can be used to implement custom logic for locale selection.
• Suggested Next Files (from that MD): core/any/class-admin-menu.php, core/lib/wpbc-booking-new.php, core/any/api-emails.php.
core/wpbc.php
• Source MD file name: wpbc.md
• Role (short sentence): The main initializer and central controller for the plugin, establishing the Booking_Calendar singleton class and managing the overall lifecycle, dependency loading, and hook setup.
• Key Technical Details: Establishes the Booking_Calendar singleton. It performs WordPress version compatibility checks. It dynamically builds the admin menu structure (Bookings, Add Booking, Settings, etc.) using a custom WPBC_Admin_Menus class. It registers core hooks (like init and asset loading hooks) and initializes major components (cron, notices, JS, CSS). It loads dependencies via includes/wpbc-include.php.
• Features (Admin vs User): Admin: Registers multiple admin menu entries, provides error notices for compatibility failures, and offers optional debug info output in the footer. User: Enqueues frontend JS/CSS assets and instantiates the main booking object (wpdev_booking).
• Top Extension Opportunities: Use the internal action define_admin_menu to register new admin menu items. Leverage plugin filters/actions such as make_bk_action('wpbc_booking_calendar_started').
• Suggested Next Files (from that MD): includes/wpbc-include.php, core/lib/wpbc-ajax.php, core/classes/wpdev_booking.php.
core/wpbc_functions.php
• Source MD file name: wpbc_functions.md
• Role (short sentence): A critical "toolbox" providing a large, diverse set of helper functions for UI components, branding, booking status workflow, multi-user logic, and currency formatting.
• Key Technical Details: Contains utilities for SVG logo generation (wpbc_get_svg_logo), calendar skin management, shortcode previews, and admin UI (wpbc_open_meta_box_section). It includes functions for managing booking status (wpbc_db__booking_approve), new booking caching, and support for multi-user/multi-tenant installs (e.g., wpbc_mu_set_environment_for_owner_of_resource). Uses $wpdb for direct queries related to booking status updates.
• Features (Admin vs User): Admin: Enables collapsible meta box sections, provides formatted admin messages, powers pagination on booking listings, and controls the core status workflow (approve/deny/cancel). User: Provides selectable calendar skins, shortcode previews for block editors, and dynamically formats costs and currency symbols.
• Top Extension Opportunities: Extend booking workflow via custom actions (e.g., wpbc_booking_approved). Override UI components using the provided meta box and messaging functions.
• Suggested Next Files (from that MD): core/wpbc-dates.php, includes/page-bookings/bookings__actions.php, core/wpbc-css.php / core/wpbc-js.php.
core/wpbc_functions_dates.php
• Source MD file name: wpbc_functions_dates.md
• Role (short sentence): A specialized date and time utility library focusing on localization, ensuring dates and times are correctly formatted and displayed according to the user's local timezone and site settings.
• Key Technical Details: The core function is wpbc_datetime_localized(), which temporarily sets the server timezone to 'UTC', applies the site's configured offset, and uses date_i18n() to format the output. It provides helpers like wpbc_date_localized() and wpbc_time_localized(). It also includes logic (wpbc_get_redable_dates) for converting date arrays into human-readable ranges (e.g., "Start Date - End Date"). It contains the developer debugging tool [wpbc_test_dates_functions] shortcode.
• Features (Admin vs User): Admin: Provides dynamic, descriptive text hints on the Settings > Availability page based on calculated date ranges. Includes a developer debugging shortcode. User: Ensures all front-end dates and times (calendar, forms, emails) are correctly displayed in the localized format, language, and timezone.
• Top Extension Opportunities: Custom extensions should reuse these functions (e.g., wpbc_datetime_localized__use_wp_timezone()) to ensure consistent timezone-aware output. Developers can use the existing logic as a template for new, unique date display formats.
• Suggested Next Files (from that MD): core/any/class-admin-menu.php, core/any/api-emails.php, includes/page-form-simple/form_simple.php.
Common Features and Patterns
Across the analyzed files, several architectural patterns and common practices emerge:
1. Custom Internal Extensibility: The plugin utilizes a parallel custom hook system (add_bk_action, apply_bk_filter, make_bk_action) defined in core/wpbc-core.php. This system, which uses global variables ($wpdev_bk_action, $wpdev_bk_filter), allows internal plugin modules to communicate and modifies data flow independently of the standard WordPress hook API.
2. Data Abstraction and Filtering: Key components wrap standard WordPress functions to allow for internal interception. Functions for managing global plugin options (get_bk_option, update_bk_option) trigger custom filters (like wpdev_bk_get_option) before interacting with WordPress storage.
3. Direct Database Interaction: Database operations, particularly for custom booking metadata (booking_options) and date queries, frequently rely on raw SQL queries using the global $wpdb object. This approach bypasses the standardized WordPress metadata API.
4. Robust Asset Control: Dedicated controller files (core/wpbc-css.php, core/wpbc-js.php) manage asset loading. These files implement advanced features: conditional loading (admin vs. client, specific pages), explicit conflict prevention via dequeuing/deregistering known scripts/styles, and defensive measures like preventing async or defer attributes on critical scripts (jQuery, plugin files) to ensure load order predictability.
5. Sophisticated Localization (i18n/l10n): The translation system (core/wpbc-translation.php) handles complex scenarios beyond standard WordPress translation loading. This includes specific compatibility for multilingual plugins (Polylang, WPML) and a unique shortcode syntax ([lang=xx_XX]) that allows administrators to embed multilingual content within a single setting field.
6. PHP-to-JS Data Bridge: Critical backend configuration, security nonces, and translated strings are passed to the frontend via a specialized localization script (core/wpbc-js-vars.php). This system uses a highly resilient bootstrap loader to guarantee the data is available to the client-side _wpbc object even when facing caching and deferred loading mechanisms.
Extension Opportunities
The plugin offers several intentional and unintentional extension opportunities:
Extension Point
Context
Source MD(s)
Custom Actions & Filters
Use add_bk_action and add_bk_filter to hook into the plugin's extensive internal event system.
wpbc-core.md, wpbc-core_v2.md
Option Interception
Use filters like add_bk_filter('wpdev_bk_get_option', ...) to intercept and modify plugin settings before they are retrieved.
wpbc-core_v2.md
Booking Meta Storage
Safely add custom data to individual bookings using the existing wpbc_save_booking_meta_option() function without altering the DB schema.
wpbc-core_v2.md
Custom CSS/JS Loading
Use the designated custom actions do_action( 'wpbc_enqueue_css_files', ...) and do_action( 'wpbc_enqueue_js_files', ...) to load custom assets.
wpbc-css.md, wpbc-js.md
Client-Side Data Injection
Use the wpbc_js_vars filter or wpbc_define_js_vars action to add custom data, settings, or URLs to the global _wpbc object.
wpbc-js-vars.md, wpbc-js.md
Programmatic Booking
Use the high-level Developer API function wpbc_api_booking_add_new() to create or edit bookings from external systems or custom code.
wpbc-dev-api.md
Event-Driven Logic
Hook into critical documented events like wpbc_track_new_booking or wpbc_booking_approved (found in the API documentation) to trigger custom code.
wpbc-dev-api.md
Custom Date Display
Leverage core localization utilities like wpbc_datetime_localized() and its wrappers (wpbc_date_localized) to ensure custom dates/times are displayed correctly.
wpbc_functions_dates.md
Inline Translation
Wrap extension strings in wpbc_lang() to utilize the plugin’s native inline multilingual configuration system.
wpbc-translation.md
Next Files to Analyze
This list aggregates all suggested files from the MD documents, deduplicates them, and excludes all files already listed in completed_files.txt.
Exact relative path
Priority
One-line reason
Which MD(s) recommended it
core/lib/wpbc-booking-new.php
High
Likely contains the critical server-side logic for receiving form data and finalizing new booking insertions.
form_parser.md, index.md, wpbc-debug.md, wpbc-dev-api.md, wpbc-emails.md, wpbc-js-vars.md, wpbc-translation.md
core/any/api-emails.php
High
Strongly suggested by related files to contain the core Email API class, templates, and primary sending logic.
wpbc-emails.md, wpbc-translation.md, wpbc_functions_dates.md
includes/page-form-simple/form_simple.php
Medium
Expected to be responsible for rendering the final HTML booking form structure using the data provided by the form parser.
form_parser.md, wpbc-debug.md, wpbc_functions_dates.md
css/wpbc_ui_both.css
Medium
A high-priority stylesheet containing core UI styles used universally on both the admin and front-end.
wpbc-css.md
core/classes/wpdev_booking.php
Low
Main booking object instantiated for frontend and backend interactions.
wpbc.md
css/admin.css
Low
Main stylesheet defining the bulk of layout and styling rules for the dedicated admin pages.
wpbc-css.md
Excluded Recommendations
The following files were suggested in the analysis documents but are excluded from the Next Files to Analyze list because they were already marked as completed/reviewed in completed_files.txt:
• core/any/class-admin-menu.php
• core/form_parser.php
• core/wpbc-translation.php
• core/wpbc-js.php
• core/wpbc-functions.php
• core/wpbc-dates.php
• core/wpbc-core.php
• includes/page-bookings/bookings__actions.php
• core/lib/wpbc-ajax.php
• includes/wpbc-include.php
• core/wpbc-js-vars.php
• js/admin.js
• js/client.js
• wpdev-booking.php
Sources
• completed_files.txt
• form_parser.md
• index.md
• wpbc-core.md
• wpbc-core_v2.md
• wpbc-css.md
• wpbc-dates.md
• wpbc-debug.md
• wpbc-dev-api.md
• wpbc-emails.md
• wpbc-js-vars.md
• wpbc-js.md
• wpbc-translation.md
• wpbc.md
• wpbc_functions.md
• wpbc_functions_dates.md
