# File Analysis: `core/form_parser.php`

This document provides a detailed analysis of the `core/form_parser.php` file from the Booking Calendar plugin repository.

## High-Level Overview

This file is a dedicated utility for parsing the plugin's custom form-building syntax. The Booking Calendar plugin allows administrators to define booking forms using a text-based, shortcode-like syntax (similar to the popular Contact Form 7 plugin).

The primary role of `core/form_parser.php` is to take this raw string of form-definition shortcodes and transform it into a structured, predictable PHP array. This parsed array is then used by other parts of the plugin to render the actual HTML form on the front-end and to understand the form's structure for validation and data processing. It is a crucial data-transformation component that bridges the gap between the admin-configured form layout and the functional front-end form.

## Detailed Explanation

The file's logic is contained in a set of procedural functions, with `wpbc_parse_form()` being the main one.

-   **`wpbc_parse_form( $booking_form_configuration )`**: This is the main parsing engine.
    -   It takes the entire form configuration string as input (e.g., `[text* your-name] [email* your-email] [select choices "Option 1" "Option 2"]`).
    -   It uses a complex regular expression with `preg_match_all` to find and deconstruct all supported form-field shortcodes.
    -   The regex is built to recognize specific field types (`text`, `email`, `select`, `checkbox`, `radio`, `textarea`, `captcha`, etc.) and capture their distinct parts:
        1.  **Type**: The kind of field, including a `*` for required fields (e.g., `select*`).
        2.  **Name**: The unique name of the field (e.g., `your-name`).
        3.  **Options**: Optional parameters like `id:html-id` or `default:on`.
        4.  **Values**: The list of choices for fields like `select` or `radio`, enclosed in quotes.
    -   It then iterates through the results and organizes them into a clean array, calling `wpbc_parse_form_shortcode_values()` to handle the values for each field.

-   **`wpbc_parse_form_shortcode_values( $shortcode_values )`**: This helper function is responsible for parsing the `values` part of a shortcode.
    -   It takes a string like `"Option A" "Option B@@value-b"`.
    -   It uses another regular expression to extract each individually quoted value.
    -   It then processes each value, splitting it by `@@`. This syntax allows for defining a separate display `title` and submission `value` (e.g., `Display Title@@actual_value`). If `@@` is not present, the title and value are the same.

-   **Helper Functions**:
    -   `wpbc_parse_form__get_shortcodes_with_name()`: A wrapper around `wpbc_parse_form()` that filters the results to return all shortcodes that share a specific name. This is important for handling conditional forms where multiple fields might have the same name but appear under different conditions.
    -   `wpbc_parse_form__get_first_shortcode_values()`: A simpler wrapper that finds the very first shortcode with a given name and returns its possible values. The docblock correctly warns that this should be used with care, as it ignores other fields with the same name.

Here is an example of the transformation from the file's documentation:

**Input String:**
`[checkbox* term_and_condition use_label_element default:on "I Accept term and conditions@@accept"]`

**Output Array Structure:**
```php
[
    'full_shortcode' => '[checkbox* term_and_condition use_label_element default:on "I Accept term and conditions@@accept"]',
    'type' => 'checkbox*',
    'name' => 'term_and_condition',
    'options' => 'use_label_element default:on',
    'values_str' => '"I Accept term and conditions@@accept"',
    'values_arr' => [
        [
            'title' => 'I Accept term and conditions',
            'value' => 'accept'
        ]
    ]
]
```

## Features Enabled

This file is a backend utility and has no direct UI features. However, it is fundamental to the form-building functionality.

### Admin Menu

-   This file does not add any admin pages.
-   It is the engine that powers the "Form" editor found in the plugin's settings. When an administrator saves their form layout, this parser is what validates and interprets that syntax.

### User-Facing

-   This file is a prerequisite for rendering any booking form on the front-end.
-   It enables the plugin to translate the abstract shortcode syntax defined by the admin into the actual HTML `<input>`, `<select>`, and `<textarea>` elements that the user interacts with.

## Extension Opportunities

The parser itself is not designed for easy extension, which is a notable limitation.

-   **Potential Risks / Limitations**: The list of supported shortcode types (`text`, `email`, `select`, etc.) is hardcoded into the regular expression inside `wpbc_parse_form()`. This means a developer cannot add a new, custom field type (e.g., a special IBAN or phone number field) without modifying the core plugin file, which is not update-safe.

-   **Suggested Improvements**: To make the parser extensible, the list of shortcode types could be made filterable. For example, if the function was modified like this:
    ```php
    // Suggestion for improvement inside wpbc_parse_form()
    $rx_shortcode_types = 'text[*]?|email[*]?|...|quiz';
    $rx_shortcode_types = apply_filters( 'wpbc_form_parser_shortcode_types', $rx_shortcode_types );
    $regex = '%\[\s*(' . $rx_shortcode_types . ') ... %';
    ```
    A developer could then hook into the `wpbc_form_parser_shortcode_types` filter to add their own custom field type to the regex, allowing the parser to recognize it.

## Next File Recommendations

Now that we understand how the form definition is parsed into a data structure, the next logical steps are to see how that data is used.

1.  **`includes/page-form-simple/form_simple.php`**: This file is a strong candidate for containing the logic that takes the array generated by `wpbc_parse_form()` and loops through it to render the final HTML for the booking form.
2.  **`core/lib/wpbc-booking-new.php`**: After a form is rendered and submitted, the data needs to be saved. This file likely contains the server-side logic for creating a new booking record from the submitted form data.
3.  **`core/wpbc-translation.php`**: The form field labels and values can be translated. This file should explain how the plugin's custom `wpbc_lang()` function and overall internationalization system works.
