# File Analysis: includes/_capacity/create_booking.php

## High-Level Overview
This file is the server-side endpoint for creating a new booking. It contains the function `ajax_WPBC_AJX_BOOKING__CREATE`, which is hooked into WordPress's AJAX system for both logged-in users (`wp_ajax_*`) and visitors (`wp_ajax_nopriv_*`).

When a user submits a booking form, the client-side scripts send a request to this endpoint. This file's code is responsible for the entire booking creation lifecycle: it validates the incoming data, checks for date availability and capacity, calculates the cost, saves the new booking to the database, and triggers email notifications. It then sends a structured JSON response back to the client with the booking confirmation details.

This file orchestrates the entire server-side booking creation by calling a series of other specialized functions.

## Detailed Explanation

The process is primarily handled by two key functions:

### 1. `ajax_WPBC_AJX_BOOKING__CREATE()`
This function is the main entry point for the AJAX request.
- **Security**: It first performs a nonce check using `check_ajax_referer` to ensure the request is valid and secure.
- **Data Sanitization**: It uses a custom `WPBC_AJX__REQUEST` class to sanitize all incoming `$_REQUEST` parameters, such as the resource ID, selected dates, and form data.
- **CAPTCHA Validation**: It calls `wpbc_captcha__in_ajx__check()` to validate the CAPTCHA response, if enabled.
- **Orchestration**: Its primary role is to gather and prepare all the request data and then pass it to the master `wpbc_booking_save()` function.
- **Response**: After `wpbc_booking_save()` completes, this function takes the returned data and sends it back to the client-side script as a JSON object using `wp_send_json()`.

### 2. `wpbc_booking_save( $request_params )`
This is the master function that contains the core business logic for creating or updating a booking.

- **Data Parsing**: It receives the request parameters, parses the custom-formatted `form_data` string into a usable PHP array, and converts the date strings into a standard format.
- **Edit/Duplicate Handling**: It checks for a `booking_hash` in the request to determine if this is an edit of an existing booking. This allows the same function to handle both new bookings and updates.
- **Availability & Capacity Check**: It calls a critical function, `wpbc__where_to_save_booking()`. This function performs the final, authoritative check to see if the requested dates and times are available, considering the resource's capacity, existing bookings, and rules for the number of items being booked. If no available slots are found, it returns an error, and the process is halted.
- **Database Insertion (`wpbc_db__booking_save`)**: If availability is confirmed, this function is called to save the data.
  - It determines if it's a new booking (`INSERT`) or an update (`UPDATE`).
  - It constructs the SQL query using `$wpdb->prepare`.
  - For a new booking, it first performs an `INSERT` into the `{$wpdb->prefix}booking` table.
  - It retrieves the new `booking_id` using `$wpdb->insert_id`.
  - It then constructs a multi-row `INSERT` query to save all the individual dates into the `{$wpdb->prefix}bookingdates` table, linking them to the new `booking_id`.
- **Payment & Cost Calculation**: After the booking is saved, it calls `wpbc_maybe_get_payment_form()` (if a payment gateway is active). This function calculates the final cost and generates the HTML for any payment forms.
- **Email Notifications**: It then calls `wpbc_send_email_new_admin()` and `wpbc_send_email_new_visitor()` to send out the relevant email confirmations.
- **Confirmation Message**: Finally, it calls `wpbc_booking_confirmation()` to generate the HTML for the "Thank You" message that will be displayed to the user.

## Features Enabled
This file is the engine behind the single most important feature of the plugin: **creating a booking**. It handles the entire server-side process, ensuring data is validated, availability is double-checked, and the booking is correctly saved to the database before the user receives a confirmation.

## Extension Opportunities
This file provides two excellent action hooks for developers to integrate custom functionality into the booking process.

- **`do_action( 'wpbc_track_new_booking', $payment_params )`**: This hook fires immediately after a new booking has been successfully saved and all costs have been calculated. The docblocks specifically mention its usefulness for conversion tracking (e.g., Google Ads). It passes a large array of booking data, which can be used to send information to third-party analytics or marketing automation systems.

- **`do_action( 'wpbc_track_edit_booking', $payment_params )`**: This hook is similar to the one above but fires when an existing booking is edited and saved.

These hooks are the ideal, update-safe way to trigger custom actions upon booking creation or modification.

## Final Summary
This file completes our end-to-end analysis of the booking creation process. We have now seen:
1.  How the booking form and calendar are rendered (**Shortcodes**).
2.  How the client-side scripts (`client.js`, `wpbc_all.js`) handle user interaction, validation, and data serialization.
3.  How data (settings, translations, nonces) is passed from PHP to JS (`wpbc-js-vars.php`).
4.  How the AJAX request is sent and received (`wpbc_all.js` -> `create_booking.php`).
5.  How this file (`create_booking.php`) handles the server-side validation, database insertion, and final response.

This concludes the deep-dive analysis of the plugin's core functionality.