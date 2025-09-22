Plugin Analysis Summary
Files Included
The following 19 plugin file paths were analyzed across the uploaded Markdown documents:
File Path
Source MD File
core/admin/api-settings.php
api-settings.md
core/admin/page-email-approved.php
page-email-approved.md
core/admin/page-email-deleted.php
page-email-deleted.md
core/admin/page-email-deny.php
page-email-deny.php-analysis.md
core/admin/page-email-new-admin.php
page-email-new-admin.php-analysis.md
core/admin/page-email-new-visitor.php
page-email-new-visitor.php-analysis.md
core/admin/page-email-trash.php
page-email-trash.php-analysis.md
core/admin/page-ics-export.php
page-ics-export.php-analysis.md
core/admin/page-ics-general.php
page-ics-general.php-analysis.md
core/admin/page-ics-import.php
page-ics-import.php-analysis.md
core/admin/page-import-gcal.php
page-import-gcal-analysis.md
core/admin/page-new.php
page-new.md
core/admin/page-settings.php
page-settings.md
core/admin/page-timeline.php
page-timeline-analysis.md
core/admin/wpbc-dashboard.php
wpbc-dashboard-analysis.md
core/admin/wpbc-gutenberg.php
wpbc-gutenberg-analysis.md
core/admin/wpbc-settings-functions.php
wpbc-settings-functions-analysis.md
core/admin/wpbc-sql.php
wpbc-sql.md
core/admin/wpbc-toolbars.php
wpbc-toolbars-analysis.md
Table of Contents
• core/admin/api-settings.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/page-email-approved.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/page-email-deleted.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/page-email-deny.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/page-email-new-admin.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/page-email-new-visitor.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/page-email-trash.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/page-ics-export.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/page-ics-general.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/page-ics-import.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/page-import-gcal.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/page-new.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/page-settings.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/page-timeline.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/wpbc-dashboard.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/wpbc-gutenberg.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/wpbc-settings-functions.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/wpbc-sql.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
• core/admin/wpbc-toolbars.php
    ◦ Overview
    ◦ Details
    ◦ Features
    ◦ Extensions
    ◦ Next Files
File-by-File Summaries
core/admin/api-settings.php
<a id="coreadminapi-settingsphp"></a>
Field
Summary
Source MD file name
api-settings.md
Role (short sentence)
This file acts as the configuration hub and logic source for all fields displayed on the Booking > Settings > General page.
Key Technical Details
Defines the concrete class WPBC_Settings_API_General; implements init_settings_fields() to build a large configuration array of settings options; uses wpbc_settings_validate_fields_before_saving__all filter for data cleanup before DB save; contains complex jQuery logic in enqueue_js() for dynamic UI interactivity (showing/hiding fields based on other values).
Features (Admin vs User)
Admin: Defines the content, fields, and interactive UI for the General Settings page, including Calendar, Legend, Availability, Confirmation, Permissions, and Advanced settings. User: Has no direct user-facing features; it is purely backend configuration.
Top Extension Opportunities
Use apply_filters calls interspersed within init_settings_fields() to inject new settings fields into specific sections of the General Settings page.
Suggested Next Files (from that MD)
core/any/admin-bs-ui.php, core/wpbc-js.php, core/wpbc-translation.php.
core/admin/page-email-approved.php
<a id="coreadminpage-email-approvedphp"></a>
Field
Summary
Source MD file name
page-email-approved.md
Role (short sentence)
Implements the specific email template and sending logic for the "Approved" booking notification sent to the visitor.
Key Technical Details
Defines WPBC_Emails_API_Approved (extends WPBC_Emails_API) to set template fields and default content; uses wpbc_import6_... functions for backward compatibility of legacy settings; contains wpbc_send_email_approved() which retrieves booking data, processes shortcodes via wpbc__get_replace_shortcodes__email_approved(), and dispatches the email.
Features (Admin vs User)
Admin: Adds the "Approved" sub-tab to Booking > Settings > Emails for customizing content and styling. User: Triggers the critical booking approval email sent to the visitor.
Top Extension Opportunities
Hook into the wpbc_replace_params_for_booking filter inside wpbc__get_replace_shortcodes__email_approved to add custom shortcodes to the template.
Suggested Next Files (from that MD)
core/admin/wpbc-gutenberg.php, core/sync/wpbc-gcal.php, core/admin/wpbc-toolbars.php.
core/admin/page-email-deleted.php
<a id="coreadminpage-email-deletedphp"></a>
Field
Summary
Source MD file name
page-email-deleted.md
Role (short sentence)
Implements the email template and sending logic for the "Deleted" notification, sent when a booking is canceled or declined by an administrator.
Key Technical Details
Defines WPBC_Emails_API_Deleted (extends WPBC_Emails_API); imports legacy settings from options like booking_email_deny_content; the sending function, wpbc_send_email_deleted(), populates the specific shortcodes like [denyreason] and [reason] before dispatching the email via the API.
Features (Admin vs User)
Admin: Adds the "Deleted" sub-tab to Booking > Settings > Emails for template customization. User: Sends the booking cancellation/deletion email, including the reason for decline.
Top Extension Opportunities
Use the wpbc_replace_params_for_booking filter within wpbc__get_replace_shortcodes__email_deleted to safely inject new, custom shortcodes into the template.
Suggested Next Files (from that MD)
core/admin/wpbc-gutenberg.php, core/sync/wpbc-gcal.php, core/admin/wpbc-toolbars.php.
core/admin/page-email-deny.php
<a id="coreadminpage-email-denyphp"></a>
Field
Summary
Source MD file name
page-email-deny.php-analysis.md
Role (short sentence)
Implements the email template and sending logic for the "Pending" notification, sent when a booking is awaiting administrator approval.
Key Technical Details
Defines WPBC_Emails_API_Deny (extends WPBC_Emails_API); uses wpbc_import6_get_old_email_deny_data() for backward compatibility; wpbc_send_email_deny() handles fetching data, populating shortcodes like [denyreason] and [reason], and dispatching the email. Note: The class/file uses "Deny," but the UI tab is labeled "Pending".
Features (Admin vs User)
Admin: Adds the "Pending" sub-tab to Booking > Settings > Emails for template customization. User: Sends the booking pending/denied email to the visitor.
Top Extension Opportunities
Hook into the wpbc_replace_params_for_booking filter within wpbc__get_replace_shortcodes__email_deny to add custom shortcodes.
Suggested Next Files (from that MD)
core/admin/wpbc-gutenberg.php, core/sync/wpbc-gcal.php, core/admin/page-timeline.php.
core/admin/page-email-new-admin.php
<a id="coreadminpage-email-new-adminphp"></a>
Field
Summary
Source MD file name
page-email-new-admin.php-analysis.md
Role (short sentence)
Implements the email template and sending logic for the critical "New Booking" notification sent to the site administrator(s).
Key Technical Details
Defines WPBC_Emails_API_NewAdmin (extends WPBC_Emails_API); features an enable_replyto checkbox to set the Reply-To header to the visitor's email; default content includes admin-specific shortcodes like [moderatelink], [click2approve], and [click2decline] for quick actions; wpbc_send_email_new_admin() retrieves admin recipient emails from the template settings before sending.
Features (Admin vs User)
Admin: Adds the "New Booking (admin)" sub-tab (the default email settings tab) to Booking > Settings > Emails; enables efficient booking management directly from the notification email. User: No direct user-facing features, but critical for administrator workflow.
Top Extension Opportunities
Use the wpbc_replace_params_for_booking filter inside wpbc__get_replace_shortcodes__email_new_admin to add custom shortcodes, such as CRM links.
Suggested Next Files (from that MD)
core/admin/wpbc-gutenberg.php, core/sync/wpbc-gcal.php, core/admin/page-timeline.php.
core/admin/page-email-new-visitor.php
<a id="coreadminpage-email-new-visitorphp"></a>
Field
Summary
Source MD file name
page-email-new-visitor.php-analysis.md
Role (short sentence)
Implements the initial "New Booking" confirmation email template sent directly to the visitor after they submit a booking.
Key Technical Details
Defines WPBC_Emails_API_NewVisitor (extends WPBC_Emails_API); the template definition does not include a configurable "To" field, as the recipient is determined dynamically from the submitted booking form data; default content includes visitor-specific shortcodes like [visitorbookingediturl] (if premium is active); wpbc_send_email_new_visitor() dynamically determines the recipient's email using the shortcode replacement array.
Features (Admin vs User)
Admin: Adds the "New Booking (visitor)" sub-tab to Booking > Settings > Emails for content customization. User: Sends the initial booking confirmation email, acknowledging receipt of the request.
Top Extension Opportunities
Use the wpbc_replace_params_for_booking filter inside wpbc__get_replace_shortcodes__email_new_visitor to add custom shortcodes, such as coupon codes.
Suggested Next Files (from that MD)
core/admin/wpbc-gutenberg.php, core/sync/wpbc-gcal.php, core/admin/page-timeline.php.
core/admin/page-email-trash.php
<a id="coreadminpage-email-trashphp"></a>
Field
Summary
Source MD file name
page-email-trash.php-analysis.md
Role (short sentence)
Implements the specific email template and sending logic for the "Trash / Reject" notification, sent when a booking is moved to the trash by an administrator.
Key Technical Details
Defines WPBC_Emails_API_Trash (extends WPBC_Emails_API); legacy import functions (wpbc_import6_get_old_email_trash_data()) retrieve data from the same options used by the 'deny' email, indicating a split in template granularity over time; wpbc_send_email_trash() populates the [denyreason] and [reason] shortcodes with the cancellation reason.
Features (Admin vs User)
Admin: Adds the "Trash / Reject" sub-tab to Booking > Settings > Emails for template customization. User: Sends the booking cancellation email when a booking is trashed.
Top Extension Opportunities
Hook into the wpbc_replace_params_for_booking filter inside wpbc__get_replace_shortcodes__email_trash to add custom shortcodes.
Suggested Next Files (from that MD)
core/admin/wpbc-gutenberg.php, core/sync/wpbc-gcal.php, core/admin/page-timeline.php.
core/admin/page-ics-export.php
<a id="coreadminpage-ics-exportphp"></a>