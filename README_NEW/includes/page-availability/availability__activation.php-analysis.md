# File Analysis: `includes/page-availability/availability__activation.php`

This file handles the database schema setup and teardown for a core feature of the Booking Calendar plugin: "Availability." Its primary responsibility is to manage a custom database table that stores properties for specific dates, which is fundamental for defining when a booking resource is available, unavailable, or has other special conditions.

## High-Level Overview

The script ensures that a custom database table, `{$wpdb->prefix}booking_dates_props`, exists when the plugin is activated and is removed when the plugin is deactivated. This table is designed to store fine-grained details about specific dates, such as their availability status, pricing adjustments (`rate`), or rules about booking start days.

This file is a foundational piece of the plugin's "Availability" module. It does not implement any user-facing logic itself but provides the necessary database structure that other parts of the plugin rely on to manage and display date availability to both admins and users.

## Detailed Explanation

The file contains two main functions hooked into the plugin's custom activation and deactivation lifecycle.

-   **`wpbc_activation__dates_availability()`**:
    -   **Hook**: `add_bk_action( 'wpbc_free_version_activation', 'wpbc_activation__dates_availability' )`
    -   **Purpose**: This function runs when the plugin is activated. It checks if the `booking_dates_props` table already exists using a helper function, `wpbc_is_table_exists()`.
    -   **Database Interaction**: If the table does not exist, it executes a `CREATE TABLE` SQL query using the global `$wpdb` object.
    -   **Schema**: The created table `wp_booking_dates_props` has the following columns:
        -   `booking_dates_prop_id`: (BIGINT, PK, auto_increment) - Unique ID for the row.
        -   `resource_id`: (BIGINT) - The ID of the booking resource this property applies to.
        -   `calendar_date`: (DATETIME) - The specific date for which the property is defined.
        -   `prop_name`: (VARCHAR) - The name of the property (e.g., 'unavailable', 'rate').
        -   `prop_value`: (TEXT) - The value of the property.

    ```php
    function wpbc_activation__dates_availability() {
        // ...
        if ( ! wpbc_is_table_exists( 'booking_dates_props' ) ) {
            $simple_sql = "CREATE TABLE {$wpdb->prefix}booking_dates_props (
                         booking_dates_prop_id bigint(20) unsigned NOT NULL auto_increment,
                         resource_id bigint(10) NOT NULL default 1,
                         calendar_date datetime NOT NULL default '0000-00-00 00:00:00',
                         prop_name varchar(200) NOT NULL default '',
                         prop_value text,
                         PRIMARY KEY  (booking_dates_prop_id)
                        ) {$charset_collate}";
            // ...
            $wpdb->query( $simple_sql );
        }
    }
    ```

-   **`wpbc_deactivation__dates_availability()`**:
    -   **Hook**: `add_bk_action( 'wpbc_free_version_deactivation', 'wpbc_deactivation__dates_availability' )`
    -   **Purpose**: This function runs upon plugin deactivation.
    -   **Database Interaction**: It executes a `DROP TABLE IF EXISTS` query to completely remove the `booking_dates_props` table and all its data.

## Features Enabled

This file does not directly enable visible features but is a critical prerequisite for the Availability functionality.

### Admin Menu

-   No UI elements, settings pages, or menu items are added by this file.
-   Its logic is executed in the background during the plugin's activation/deactivation process, which is typically initiated from the WordPress "Plugins" page.

### User-Facing

-   There are no direct user-facing features like shortcodes, blocks, or scripts.
-   However, the database table it creates is essential for the front-end calendar to function correctly, as it determines which dates are shown as available, booked, or having special pricing. Without this table, the logic for displaying date availability would fail.

## Extension Opportunities

-   **Safe Extension**: The safest way to extend this is to use the same custom hooks with a later priority. You could create a separate helper plugin that hooks into `wpbc_free_version_activation` to add new columns to the table using `dbDelta` or an `ALTER TABLE` query.

    ```php
    // Example: Adding a new column in your own custom plugin
    function my_custom_add_column_to_availability_table() {
        global $wpdb;
        $table_name = $wpdb->prefix . 'booking_dates_props';
        $column_exists = $wpdb->get_results( "SHOW COLUMNS FROM {$table_name} LIKE 'my_custom_flag'" );

        if ( empty( $column_exists ) ) {
            $wpdb->query( "ALTER TABLE {$table_name} ADD my_custom_flag VARCHAR(10) NOT NULL DEFAULT 'off'" );
        }
    }
    // Hook with a priority > 10 to run after the original
    add_action( 'wpbc_free_version_activation', 'my_custom_add_column_to_availability_table', 20 );
    ```

-   **Potential Risks**:
    -   Directly modifying this file is not recommended, as your changes will be lost during plugin updates.
    -   The deactivation function (`wpbc_deactivation__dates_availability`) permanently deletes the table and all availability data. This is a destructive action, and any logic hooked into `wpbc_free_version_deactivation` should be aware of this. If you store critical data in this table, you might consider removing the `DROP TABLE` call or backing up the data first.

## Next File Recommendations

To understand how the database table created by this file is used, the next logical step is to investigate the files responsible for managing and displaying availability and resources.

1.  **`includes/page-resource-free/page-resource-free.php`** — The `booking_dates_props` table has a `resource_id` column. This file is explicitly named for "resources" and is likely where resources and their availability are managed. It's not on the completed list.
2.  **`js/user-data-saver.js`** — Availability rules directly impact the user experience. This file sounds like it handles client-side data, possibly related to saving user selections or form data which would be constrained by the availability rules defined in the database. It's not on the completed list.
3.  **`includes/page-settings-form-options/page-settings-form-options.php`** — Availability settings might be intertwined with general booking form options. This file could define how availability rules (e.g., "allow start day selection") are configured and applied to the booking form. It's not on the completed list.
