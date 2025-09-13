# File Analysis: core/wpbc_functions.php

## High-Level Overview

- **Summary:**  
  This file provides a large set of utility, helper, and integration functions for the Booking Calendar plugin. It acts as a "toolbox" for both core and extended plugin logic, supporting everything from SVG logo generation, calendar skin management, admin messaging, booking status management, meta box UI, JS enqueueing, pagination, MU (multi-user) features, currency handling, and booking workflow logic.
- **Architecture Fit:**  
  Loaded early during plugin initialization, it’s available to all other plugin modules (admin, frontend, AJAX, booking logic). Many functions are called by admin pages, booking actions, shortcodes, and UI elements, making this file a critical part of the plugin’s infrastructure.

---

## Detailed Explanation

### Key Functions, Classes, and Hooks Used

- **Logo and Branding:**
  - `wpbc_get_svg_logo`, `wpbc_get_svg_logo_for_html` – SVG logo generation for UI and branding.
- **Calendar Skin Management:**
  - `wpbc_get_calendar_skin_options`, `wpbc_is_calendar_skin_legacy`, `wpbc_dir_list` – Dynamically build options for calendar appearance, scan skin folders, and handle legacy/custom skins.
- **Shortcode and UI Helpers:**
  - `wpbc_get_preview_for_shortcode` – Provides a placeholder preview for booking calendar shortcodes in block editors.
- **Admin UI:**
  - Meta box open/close: `wpbc_open_meta_box_section`, `wpbc_close_meta_box_section`
  - Admin messaging: `wpbc_show_message`, `wpbc_show_message_in_settings`
  - Admin footer: `wpbc_show_booking_footer`
  - Pagination: `wpbc_show_pagination`
  - JS enqueueing for footer: `wpbc_enqueue_js`, `wpbc_print_js`
- **Booking Workflow:**
  - **New bookings cache:**  
    - `wpbc_booking_cache__new_bookings__reset`, `wpbc_db_get_number_new_bookings`, `wpbc_db_update_number_new_bookings` – Cache and count new bookings, reset on workflow actions.
    - Actions hooked:  
      ```php
      add_action( 'wpbc_track_new_booking', 'wpbc_booking_cache__new_bookings__reset' );
      ```
  - **Status management:**  
    - `wpbc_is_booking_approved`, `wpbc_db__booking_approve`, `wpbc_auto_approve_booking`, `wpbc_auto_pending_booking`, `wpbc_auto_cancel_booking`
    - Integrates with database and sends emails (via other functions).
    - Support for multi-user logic (super admin, resource owner).
- **Multi-user (MU) Support:**
  - Functions for switching user context based on booking resource owner (used in multiuser installs).
  - `wpbc_mu_set_environment_for_owner_of_resource`, `wpbc_mu_set_environment_for_user`, `wpbc_mu__is_simulated_login_as_user`
- **User and Role Helpers:**
  - `wpbc_is_current_user_have_this_role`, `wpbc_get_current_user_id`, `wpbc_get_current_user`
- **Currency and Cost Functions:**
  - `wpbc_get_cost_with_currency_for_user`, `wpbc_get_currency_symbol_for_user`, `wpbc_get_cost_per_period_for_user`
- **Logging and Notes:**
  - `wpbc_db__add_log_info` – Add remarks to booking notes.
- **CSV Support:**  
  - `wpbc_get_request_url_path` – Parse CSV export paths from requests.
- **Miscellaneous:**
  - Performance tracking: `php_performance_START`, `php_performance_END`
  - Utility: `wpbc_is_file_exist`, `wpbc_get_bytes_from_str`
  - Email validation: `wpbc_is_not_blank_email`
  - Inline CSS hack: `wpbc_add_css_to_hide_seasons`

### Interaction with WordPress Core APIs or Database

- **Database:**
  - Uses `$wpdb` for direct queries (bookings, bookingdates, etc.).
  - Updates custom plugin options using wrappers around WP's option API.
- **WordPress APIs:**
  - Leverages functions like `current_user_can`, `get_current_user_id`, `wp_upload_dir`, `wp_parse_url`, `wp_unslash`, `esc_attr`, `esc_html`, `wp_kses_post`.
  - Hooks: both native WP (`add_action`) and plugin-specific hooks.
- **Admin and Frontend:**
  - Many functions are context-aware (`is_admin()`) and used by both admin pages and frontend booking forms/shortcodes.

---

## Features Enabled

### Admin Menu

- **Meta Boxes for Settings Pages:**
  - Enables collapsible meta box sections in the admin UI for plugin settings.
- **Admin Messaging:**
  - Provides formatted messages, notices, and info panels for admin interactions.
- **Booking Pagination:**
  - Admin booking listings use pagination generated here.
- **Footer Branding:**
  - Displays plugin credit and review prompt in the admin footer.
- **Inline JS/CSS:**
  - Queues JS for admin pages and injects temporary CSS for specific admin views.
- **Booking Status Workflow:**
  - Admin actions for approving, pending, canceling bookings, including email notifications and logs.

### User-Facing

- **Shortcode Previews:**
  - Block editors and Elementor get a placeholder preview for booking calendar shortcodes.
- **Calendar Appearance:**
  - Enables selectable calendar skins for front-end display.
- **Currency Formatting:**
  - Dynamically formats costs and symbols per resource/user for frontend display.
- **Booking Workflow:**
  - Indirectly, user booking actions update status, trigger emails, and affect availability.

---

## Extension Opportunities

### Recommended Safe Extension Points

- **Hooks (WP and Plugin-specific):**
  - Extend booking workflow via actions (e.g., `wpbc_booking_approved`, `wpbc_track_new_booking`).
    ```php
    add_action('wpbc_booking_approved', 'my_custom_approved_handler', 10, 2);
    ```
  - Use plugin filters for SQL, cache, and context logic (see `apply_bk_filter` in related files).
- **Add New Helper Functions:**
  - Create new utilities and helper functions for booking, UI, or data management.
- **Override UI Components:**
  - Use provided functions for meta boxes and messaging, extend with custom behaviors.
- **Multi-user Logic:**
  - Extend or customize resource/user mapping for advanced multi-tenant installs.

### Suggested Improvements or Modularization Ideas

- **Separate Admin/UI and Booking Logic:**
  - Split UI functions into their own file to clarify separation of concerns.
- **Use WP Metadata API for Bookings:**
  - Replace direct SQL for booking meta with WP's metadata API for portability.
- **Strict Typing and Return Types:**
  - Add type hints and PHPDoc for better IDE support and fewer runtime bugs.
- **Refactor Inline JS/CSS:**
  - Move inline assets to dedicated files and enqueue properly.

### Potential Risks When Extending

- **Direct DB Queries:**
  - Changes may break compatibility or introduce security issues if queries aren't properly escaped or filtered.
- **Tight Coupling:**
  - Many helpers assume certain plugin architecture—major refactors could break dependent logic.
- **Global State:**
  - Some functions rely on global variables (e.g., `$wpbc_queued_js`); avoid collisions.
- **Performance:**
  - Overuse of hooks or inefficient cache logic could impact admin load times.

---

## Next File Recommendations

1. **core/wpbc-dates.php**  
   - *Reason:* Handles booking dates, capacities, and time logic; critical for availability, pricing, and calendar rendering.
2. **includes/page-bookings/bookings__actions.php**  
   - *Reason:* Implements booking approval, denial, cancelation, and other core booking actions that leverage functions from this file.
3. **core/wpbc-css.php / core/wpbc-js.php**  
   - *Reason:* Manages the plugin's CSS/JS asset logic, closely tied to UI helpers and JS enqueueing functions found here.

*By analyzing these files, you’ll get deeper insight into booking data handling, user/admin UI, and the full booking workflow of the plugin.*