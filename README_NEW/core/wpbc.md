# File Analysis: core/wpbc.php

## High-Level Overview

- **Role:**  
  `core/wpbc.php` acts as the main initializer and controller for the Booking Calendar plugin. It establishes the singleton `Booking_Calendar` class, manages the plugin's lifecycle (activation, core loading, error handling), and sets up both backend and frontend hooks.  
- **Architecture Fit:**  
  This file is the central orchestrator, loaded by `wpdev-booking.php`. It wires up dependencies, loads required files, and defines admin menus and plugin-wide behaviors. All major plugin features (admin, frontend, AJAX, cron) are initialized from here.

---

## Detailed Explanation

### Key Functions, Classes, and Hooks

- **Singleton Class (`Booking_Calendar`):**  
  - Controls the plugin's entire lifecycle.
  - Holds references to core objects (cron, notices, booking_obj, admin_menu, js, css).
  - Implements `init()` as the global entry point, enforcing one instance.
- **Initialization Sequence:**  
  - Checks for WordPress version compatibility.
  - Loads dependencies:  
    ```php
    require_once WPBC_PLUGIN_DIR . '/includes/wpbc-include.php';
    ```
  - Sets up plugin version from file headers.
  - Handles plugin start logic for AJAX and normal requests.
- **Hooks Registered:**
  - `add_action( 'init', array( self::$instance, 'wp_inited' ) )` – sets a flag when WP is initialized.
  - Admin menu setup via:
    ```php
    add_action( '_admin_menu', array( self::$instance, 'define_admin_menu' ) );
    add_action( 'admin_footer', 'wpbc_print_js', 50 );
    ```
  - Frontend asset loading:
    ```php
    add_action( 'wp_enqueue_scripts', array(self::$instance->css, 'load'), 1000000001 );
    add_action( 'wp_enqueue_scripts', array(self::$instance->js, 'load'), 1000000001 );
    add_action( 'wp_footer', 'wpbc_print_js', 50 );
    ```
- **Admin Menu Construction:**  
  - Dynamically builds top-level and sublevel menus using `WPBC_Admin_Menus`.
  - Menu items for Bookings, Add Booking, Availability, Prices, Resources, Settings, Setup Wizard, Log Off Simulated User.
  - Menu badges and custom SVG icons are supported.
- **Error Handling:**  
  - If plugin requirements aren't met, displays admin notices for minimum WP version or legacy plugin activation.
  - Example:
    ```php
    add_action('admin_notices', 'wpbc_show_min_wp_version_error');
    ```
- **Debug Tools:**  
  - Optional debug info output in admin footer for performance and queries.

### Interaction with WordPress Core APIs and Database

- **Version Checking:**  
  Uses `version_compare` and `$wp_version` to enforce minimum requirements.
- **Admin Menus:**  
  Uses custom `WPBC_Admin_Menus` class for menu registration (likely wraps `add_menu_page`/`add_submenu_page`).
- **Options and User Data:**  
  Calls to `get_bk_option`, `get_userdata`, and `get_option` for settings and user context.
- **Front-End Asset Enqueue:**  
  Uses standard WordPress asset hooks for CSS/JS loading.
- **AJAX Support:**  
  Loads AJAX handler if `DOING_AJAX` is set.
- **Database:**  
  Interacts with `$wpdb` for debug purposes; booking-related queries likely handled elsewhere via `wpdev_booking` or other classes.

#### Example Snippet: Admin Menu Construction
```php
self::$instance->admin_menu['settings'] = new WPBC_Admin_Menus(
    'wpbc-settings', array(
        'in_menu' => 'wpbc',
        'menu_title' => __('Settings', 'booking'),
        // ... more config
    )
);
```

---

## Features Enabled

### Admin Menu

- **Settings Pages:**  
  - Multiple admin menu entries: Bookings, Add Booking, Availability, Prices, Resources, Settings, Setup Wizard, Log Off Simulated User.
  - Menu badges for new bookings.
  - Custom SVG icons for branding.
- **Admin Notices:**  
  - Error messages for version issues or conflicting plugin instances.
- **Admin Scripts/CSS:**  
  - Queues JS/CSS for admin pages.
- **Debug Info:**  
  - Optional output of performance and query statistics.

### User-Facing

- **Frontend Asset Loading:**  
  - Enqueues booking-related JS and CSS for site visitors.
- **Booking Functionality:**  
  - Instantiates main booking object (`wpdev_booking`) for frontend interactions.
- **AJAX Support:**  
  - Handles AJAX requests for booking actions via dedicated handler.
- **No Direct Shortcodes/Blocks Here:**  
  - These are likely registered in included files (e.g., `wpbc-include.php` or elsewhere).

---

## Extension Opportunities

### Recommended Safe Extension Points

- **Add New Admin Menus:**  
  - Use the `define_admin_menu` method to register new menu items.
- **Register Additional Hooks:**  
  - Use `add_action` or `add_filter` after plugin initialization (`WPBC()`).
- **Leverage Filters/Actions:**  
  - `make_bk_action('wpbc_booking_calendar_started')` and `apply_bk_filter('multiuser_is_current_user_active', true)` provide extension points.
- **Include Custom Files:**  
  - Add your own includes in the `includes()` method or load new modules after initialization.

### Suggested Improvements or Modularization Ideas

- **Separate Admin/Frontend Init:**  
  - Move admin and frontend bootstrapping into dedicated classes/files for clarity.
- **Abstract Asset Handling:**  
  - Use WordPress’s asset registration API for better dependency management.
- **More Granular Error Handling:**  
  - Use WordPress’s error handling classes for more flexible notices.

### Potential Risks or Limitations

- **Singleton Pattern:**  
  - Direct modification of the singleton can have wide-ranging effects.
- **Global State:**  
  - Heavy reliance on globals and static properties may increase risk of conflicts.
- **Direct Output in Notices/Error Handlers:**  
  - Some error and debug output is rendered directly; better to use WP’s standard notice APIs.
- **Entrenched Legacy Code:**  
  - Some legacy patterns may make refactoring or extending harder.

---

## Next File Recommendations

1. **includes/wpbc-include.php**  
   - *Reason:* This file is loaded during initialization and likely contains essential class definitions, core functions, and further hooks for booking logic, shortcodes, and UI.
2. **core/lib/wpbc-ajax.php**  
   - *Reason:* Handles AJAX requests for bookings. Critical for understanding how frontend and backend communicate asynchronously, including booking submission and status updates.
3. **core/classes/wpdev_booking.php**  
   - *Reason:* Main booking object instantiated for both admin and frontend. Central for understanding booking creation, management, and data handling.

*Reviewing these files will give you a complete view of initialization, booking logic, and async functionality, helping you extend or customize the plugin safely and effectively.*