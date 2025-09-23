FAQ on the Booking Calendar Plugin's CSS Architecture
What is the overall architectural approach of the Booking Calendar plugin's styling, and how is the UI split between the front-end and back-end?
The plugin employs a robust, modern, and highly modular CSS architecture, primarily utilizing CSS Flexbox for complex layouts and responsiveness across all components. Styling is strictly segregated:

Admin/Back-End: Styles are handled by files like admin.css, admin-skin.css, and modal.css. These files focus on creating a consistent, branded, and modern look for the control panel, including two-tier tabbed navigation, modern settings page layouts (often using a two-column, sticky vertical menu), and styling complex UI elements like toolbars and modal dialogs.
Client/Front-End: Styles are handled by files like calendar.css, client.css, and wpbc_time-selector.css. These focus on the public-facing booking forms, calendar grid, and user feedback messages. The front-end calendar uses modern techniques like aspect-ratio to ensure square date cells and @container queries for advanced responsiveness.
How does the plugin ensure a cohesive and modern user experience (UX) for administrators?
The plugin uses several specialized stylesheets to standardize and modernize the admin UX:

Branding and Consistency (admin-skin.css): This file acts as the primary "skin," overriding default WordPress and Bootstrap styles to enforce a consistent look. It implements a "New design 2.0, 2.1" system for flat, tabbed navigation and uses techniques like Flexbox for layout management and prominent left borders for row highlighting in tables. It also actively hides standard WordPress notices to prevent UI clutter.
Layout and Structure (admin.css): This file introduced a modern, flexbox-based, two-column layout for main settings pages, featuring a sticky vertical navigation menu that collapses on smaller screens.
Complex Interactions (modal.css): This file ensures all administrative modal dialogs (for tasks like shortcode generation or payment requests) are responsive and consistent, often using high z-index values to guarantee they overlay all other admin content.
What modern CSS techniques are used to make the front-end calendar and booking forms responsive?
The front-end styling leverages advanced CSS features for superior responsiveness and layout:

Flexbox: Both the calendar grid (calendar.css) and the booking forms (client.css) are built using Flexbox (display: flex) for multi-column layouts that easily wrap and stack vertically on mobile devices (e.g., the two-column calendar/form layout).
Square Cells and Container Queries (calendar.css): The calendar uses aspect-ratio: 1 / 1 to ensure date cells are perfectly square. Critically, it uses @container queries to allow the font size inside a single month block to shrink if the block is placed in a very narrow container (like a sidebar), providing a level of responsiveness independent of the main viewport size.
Time Slots (wpbc_time-selector.css): Time slot buttons are arranged in a flexible, wrapping row, using flex: 1 1 auto to ensure they fill the available space while preventing excessive width with max-width: Min(250px, 100%).
How does the plugin handle visual feedback and availability status for users?
The plugin uses color-coding, modern iconography, and state-based styling to provide clear visual feedback:

Calendar Availability: Skins like multidays.css define clear color palettes (e.g., bright green for available, reddish-orange for approved) that are applied based on date status.
Changeover Days: The base calendar (calendar.css) uses modern techniques with inline SVG polygons within date cells, which are then styled by the skins, to create flexible diagonal or vertical lines indicating check-in/check-out days.
Time Slot Status: The time selector (wpbc_time-selector.css) uses the .wpbc_time_picker_disabled class to apply a distinct red background and cursor: not-allowed to unavailable slots, providing immediate feedback.
General Messages (client.css): Front-end messages (success, warning, error) use a modern UI pattern with a colored border-left to visually categorize the type of user feedback.
Where and how are loading indicators and visual feedback used during AJAX operations?
Reusable loading indicators (spinners) and blur effects are centralized in the css_wpbc_ui_both.css utility file. These styles are designed for use in both the client-facing and administrative interfaces.

Spinners: The file defines various pure CSS spinners (.wpbc_spins_loader, .wpbc_one_spin_loader_micro) animated with simple CSS @keyframes rules.
Feedback Mechanism: During an asynchronous operation (like navigating a month on the front-end or saving settings in the admin panel), JavaScript adds a blur class (e.g., .wpbc_calendar_blur) to the content container, making it unreadable and non-interactive (pointer-events: none), and positions a spinner on top. This consistently communicates that the system is processing a request.
What is the role of client.css, and how does it enhance the booking form experience?
client.css is the core stylesheet for the front-end booking form and post-booking elements. It enhances the user experience through:

Layout Flexibility: It defines multiple Flexbox-based form layouts (e.g., centered, two-column form-right) that can be selected by the administrator.
Professional Elements: It styles all form fields, ensuring clear focus states (for accessibility), and provides distinct, polished styles for buttons and payment gateway icons (using embedded SVG data URIs).
Post-Booking UX: It includes a large section dedicated to styling the post-booking confirmation page (.wpbc_after_booking_thank_you_section), using a clean, card-based Flexbox layout for booking details and payment options.
How does the plugin provide a WYSIWYG experience for Gutenberg block users?
The css_wpbc-gutenberg.css file is dedicated to styling the Booking Calendar's custom blocks within the WordPress Gutenberg editor.

Visual Previews: It provides styled previews for the booking form, calendar, and timeline blocks, giving content editors a good idea of the front-end appearance without leaving the editor.
Contextual Controls: It styles and controls the visibility of UI elements like the "Configure" button, which is leveraged by the core Gutenberg class .is-selected to appear only when the administrator actively clicks on the block, thus reducing visual clutter.
Shortcode Display: It includes a styled footer area within the block preview to display the underlying shortcode, aiding administrators in debugging and configuration.
What is the recommended method for developers to safely customize the plugin's appearance without losing changes during updates?
The recommended safe extension method for both admin and client-facing styles is to never modify the plugin's core CSS files directly.

Instead, developers should:

Create a Custom CSS File: Create a separate stylesheet within their theme or via a custom CSS plugin.
Enqueue with Later Priority: Ensure this custom file is loaded after the plugin's default stylesheets.
Use Specific Selectors: Write CSS rules using more specific selectors (e.g., targeting a specific ID or adding more classes) to override the properties defined in the plugin's files.
Skinning System (Front-end Calendar): For changing the calendar's colors, the most robust method is to copy an existing skin file (like multidays.css), rename it, modify the color values, and upload it to the /wp-content/uploads/wpbc_skins/ directory, allowing the plugin to detect and use the new theme.