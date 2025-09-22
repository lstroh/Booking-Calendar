Booking Calendar Plugin Architectural FAQ
1. What is the fundamental architecture used for managing settings and email templates within the plugin?
The plugin uses a consistent, object-oriented design pattern based on an Abstract API framework for both general settings and email templates. For settings, files like core/admin/api-settings.php define the concrete class WPBC_Settings_API_General, which specifies the complete catalog of settings fields (type, title, default value, group) for the Booking > Settings page. For email templates, files like core/admin/page-email-approved.php define concrete classes (e.g., WPBC_Emails_API_Approved) that extend an abstract WPBC_Emails_API. This structure defines the specific fields, default content, and sending logic for each notification type (Approved, Deleted, New Admin, New Visitor, Pending, Trash/Reject), ensuring a uniform and extensible system.

2. How are administrators notified of new bookings, and what unique features are provided in the admin notification email?
When a new booking is submitted, the core/admin/page-email-new-admin.php file is responsible for defining and sending the notification to the administrator(s). This email template is managed by the WPBC_Emails_API_NewAdmin class. Crucially, the default content of this template is tailored for quick booking management, including dynamic shortcodes such as [moderatelink], [click2approve], and [click2decline]. This allows administrators to quickly change the status of a booking directly from their email inbox. Additionally, this template features an enable_replyto option that sets the email's Reply-To header to the visitor's address, facilitating direct communication with the customer.

3. What role does the "Timeline View" play, and how is its functionality architecturally separated in the admin panel?
The Timeline View is a core administrative feature for visualizing bookings laid out chronologically. The file core/admin/page-timeline.php acts purely as the page controller. Its responsibilities are limited to setting up the admin page, rendering the necessary filter toolbars (like the date and resource selection controls), and ensuring timezones are handled correctly. The actual complex tasks—querying the database for booking data, processing the filters, and generating the HTML for the visual timeline—are entirely delegated to a separate, specialized class, WPBC_TimelineFlex, which is contained in another file (likely core/timeline/flex-timeline.php). This separation ensures modularity between the UI structure and the data visualization engine.

4. How does the plugin handle data synchronization with external calendars using .ics feeds and Google Calendar?
The plugin supports external synchronization through two primary mechanisms, both configured under Booking > Settings > Sync:

.ics Feeds (Import/Export): The admin pages for .ics Import and Export (page-ics-import.php and page-ics-export.php) allow users to manually import bookings from external calendar URLs (like Airbnb or VRBO) or generate public URLs for exporting their bookings. The core logic for parsing and importing/exporting .ics files is not in these UI files; it is delegated to a required companion plugin, "Booking Manager."
Google Calendar (Import): The settings page for Google Calendar Import (page-import-gcal.php) manages the connection settings, including the Google API Key. Its critical function is managing automation: it checks the administrator's settings (frequency, activation) and uses wp_cron (WordPress's scheduling system) to set up a recurring cron job that triggers the actual import logic (wpbc_silent_import_all_events).
5. What is the purpose of the core/admin/wpbc-sql.php file, and how is the filtering system made extensible?
The core/admin/wpbc-sql.php file is the data engine for the administrative booking listings and timeline views. Its purpose is to dynamically construct and execute complex SQL queries based on user inputs from the admin filter toolbars (date ranges, keywords, status, etc.).

The filtering system is highly extensible through numerous apply_filters calls within the wpbc_get_sql_for_booking_listing() function (e.g., get_bklist_sql_keyword). Developers can hook into these filters to inject their own custom SQL conditions into the WHERE clause, allowing them to add custom data filters to the booking list without modifying the core query logic.

6. How does the plugin integrate with the modern WordPress Gutenberg Block Editor?
The integration with Gutenberg is handled by core/admin/wpbc-gutenberg.php. This file acts as the necessary PHP bridge:

It checks for Gutenberg compatibility.
It uses standard WordPress functions (wp_register_script, wp_register_style) to register the required JavaScript (js/wpbc-gutenberg.js) and CSS assets.
It calls register_block_type() to officially register the booking/booking block.
This architecture delegates the entire visual appearance, settings, and functionality of the block within the editor to the enqueued JavaScript file (likely using React/wp-element), providing a modern, intuitive way for content creators to insert and configure booking forms.

7. How are administrators able to customize the appearance and filtering options on key admin pages?
Admin pages often feature toolbars, defined in core/admin/wpbc-toolbars.php, that allow for a highly personalized experience. For instance, on the Bookings > Add New page, the toolbar includes a "Calendar View" tab. This tab allows the administrator to customize the calendar's display (e.g., number of months visible, layout width). When the administrator clicks "Save," this file triggers an AJAX call (wpbc_save_custom_user_data) to save these specific view preferences as user meta, ensuring that the next time they visit the page, the calendar retains their personalized settings.

8. What hidden tools are available for advanced debugging and system maintenance, and how are they accessed?
A powerful "System Info / Debug panel" is available but hidden on the main Booking > Settings page, controlled by core/admin/wpbc-settings-functions.php. This panel is revealed only when a specific URL parameter (&system_info=show) and a valid nonce are present.

The panel provides several on-demand utilities, including:

Displaying a detailed report of the server environment (PHP/MySQL versions, memory limits, active plugins).
Tools to force-update remote translation files.
Destructive actions like resetting all custom booking forms to their default state.
Functions to restore all dismissed admin notices and panels. These tools are intended for advanced users or support staff to diagnose complex, server-related issues.
