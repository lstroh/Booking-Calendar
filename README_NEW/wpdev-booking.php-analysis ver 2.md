# File Analysis: Main Plugin Bootstrap and Activation

This report covers the plugin's main entry point (`wpdev-booking.php`), its core engine (`core/wpbc.php`), and the activation process (`core/wpbc-activation.php`) where the database schema is defined.

## 1. `wpdev-booking.php` - The Bootstrapper

This file is the main plugin file recognized by WordPress. Its role is minimal and serves only as a bootstrapper.

- **Plugin Header**: It contains the standard WordPress plugin header with metadata like the plugin name, version, and author.
- **Constants**: It defines several critical global constants used throughout the plugin, including `WPBC_FILE`, `WPBC_PLUGIN_DIR`, and `WPBC_PLUGIN_URL`.
- **Core Engine Include**: Its final and most important action is to load the main plugin engine: `require_once WPBC_PLUGIN_DIR . '/core/wpbc.php';`.

## 2. `core/wpbc.php` - The Core Engine

This file contains the main `Booking_Calendar` class, which acts as the central orchestrator for the entire plugin.

- **Singleton Pattern**: The class uses a static `init()` method to ensure only a single instance of the plugin's main class is ever created.
- **Initialization**: The `init()` method is the primary entry point for the plugin's logic. It:
  - **Includes Files**: Calls an `includes()` method which loads `includes/wpbc-include.php`, a manifest file that in turn `require_once`s all other necessary PHP files.
  - **Instantiates Core Classes**: It creates new instances of key classes, including `WPBC_JS`, `WPBC_CSS`, and the main `wpdev_booking` class which handles shortcodes.
  - **Hooks Activation**: Critically, it instantiates `new WPBC_BookingInstall()`, the class responsible for handling plugin activation, deactivation, and uninstallation.
  - **Registers Hooks**: It adds all the necessary actions and filters for the plugin to function, such as hooking methods into `admin_menu` to build the admin pages and `wp_enqueue_scripts` to load assets.

## 3. `core/wpbc-activation.php` - Activation & Database Schema

This file handles the one-time setup process when the plugin is activated.

#### High-Level Overview
This file defines the `WPBC_BookingInstall` class and the `wpbc_booking_activate()` function. The `wpbc_booking_activate()` function is hooked into the WordPress activation process and is responsible for creating the custom database tables and populating the `wp_options` table with default settings.

#### Detailed Explanation & Database Schema
The core of the activation process is the `wpbc_booking_activate()` function. It checks if the plugin's tables exist and, if not, creates them using direct `$wpdb->query()` calls.

The two primary tables created are:

**1. `{$wpdb->prefix}booking`**

This table stores the main record for each booking.

| Column | Type | Notes |
| :--- | :--- | :--- |
| `booking_id` | `bigint(20) unsigned` | **Primary Key**, Auto Increment. |
| `form` | `text` | Stores the serialized booking form data submitted by the user. |
| `booking_type` | `bigint(10)` | The ID of the booking resource. |
| `modification_date` | `datetime` | The date and time the booking was last modified. |
| `creation_date` | `timestamp` | The date and time the booking was created. |
| `sort_date` | `datetime` | The first date of the booking, used for efficient sorting. |
| `status` | `varchar(200)` | A custom status string for the booking. |
| `is_new` | `bigint(10)` | A flag (1 or 0) to indicate if the booking is "new". |
| `sync_gid` | `varchar(200)` | Stores a Google Calendar event ID for synchronization. |
| `trash` | `bigint(10)` | A flag (1 or 0) to indicate if the booking is in the trash. |
| `hash` | `text` | A unique hash for the booking, used for editing/payment links. |
| `booking_options` | `text` | Stores additional serialized options for the booking. |

**2. `{$wpdb->prefix}bookingdates`**

This table stores the specific dates associated with each booking.

| Column | Type | Notes |
| :--- | :--- | :--- |
| `booking_dates_id` | `bigint(20) unsigned` | **Primary Key**, Auto Increment. |
| `booking_id` | `bigint(20) unsigned` | Foreign key linking to the `booking` table. |
| `booking_date` | `datetime` | The specific date. For time slots, this includes the time. |
| `approved` | `bigint(20) unsigned` | A flag (1 or 0) indicating if the date is approved or pending. |
| `type_id` | `bigint(20) unsigned`| *(In Business+ versions)* The ID of a child resource for this specific date. |

A **unique index** is created on `(booking_id, booking_date)` in the `bookingdates` table to ensure fast lookups and prevent duplicate date entries for the same booking.