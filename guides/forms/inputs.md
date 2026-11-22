---
breadcrumb:
  - Guides
  - Forms
  - Inputs
summary-order: ;;2
keywords:
  - form
  - input
  - text
---

# Inputs

## List

### Texts

- [`Email`](input-email.md)
- [`Password`](input-password.md)
- [`Search`](input-search.md)
- [`Tel`](input-tel.md)
- [`Text`](input-text.md)
- [`TextArea`](input-textarea.md)
- [`Url`](input-url.md)

### Numbers

- [`Number`](input-number.md)
- [`Range`](input-range.md)

### Dates

- [`Date`](input-date.md)
- [`DateTime`](input-datetime.md)
- [`Month`](input-month.md)
- [`Time`](input-time.md)
- [`Week`](input-week.md)

### Choices

- [`Checkbox`](input-checkbox.md)
- [`Choice`](input-choice.md)

### Buttons

- [`Button`](input-button.md)
- [`Reset`](input-reset.md)
- [`Submit`](input-submit.md)

### Others

- [`File`](input-file.md)
- [`Hidden`](input-hidden.md)
- [Composite inputs](input-composite.md)

## Common options

Options available:

| Name                 | Type                   | Default value           | Description                         |
|----------------------|------------------------|-------------------------|-------------------------------------|
| **name**             | string                 | *none*                  | Input name                          |
| **label**            | string/false           | *none*                  | Input label                         |
| **label_attributes** | array                  | *none*                  | List of label attributes            |
| **required**         | boolean                | true                    | Input is required                   |
| **disabled**         | boolean                | false                   | Disabled input                      |
| **readonly**         | boolean                | false                   | Input is readonly                   |
| **helper**           | string/false           | *none*                  | Helper                              |
| **attributes**       | array                  | *none*                  | List of attributes                  |
| **null_if_empty**    | boolean                | false                   | An empty value is cast to NULL      |
| **transformer**      | `TransformerInterface` | *Depends on input type* | Convert a format into another       |
| **validators**       | `ValidatorInterface[]` | *Depends on input type* | Validate data from the user form    |

### Mapping

The `mapped` option accepts a boolean, a property name or, since version 3.3, a `FormMapping` instance.
It defaults to `true` for ordinary inputs; button types disable mapping by default.
See [Mapping fields to an object](../forms.md#mapping-fields-to-an-object) for details and examples.

### Non-scalar values

> 🆕 **Info**: *Since version 3.4*

Inputs expect a single scalar value. When an array or an object is submitted to an input (for example
`?user[name][]=foo`, or a JSON array in an API body), the input is invalid with a
`Berlioz\Form\Validator\Constraint\ScalarConstraint` ("The field "name" has an invalid value.") and the value is not
hydrated into the mapped object.

This check is done by `Berlioz\Form\Validator\ScalarValidator`, added by default to all inputs except
[`File`](input-file.md) and [`Choice`](input-choice.md), which handle multiple values by design.
An `Email` input with the `multiple` option still receives a single comma-separated string.

The scalar check accepts `null` and scalar values (`string`, `int`, `float`, `bool`); the usual required,
format and other validators still apply. It does not enforce an exact JSON scalar type or preserve integers
through field transformations. In particular, it does not distinguish `3` from `"3"` as a type requirement.
Use custom validation for API-specific type rules; [`strictJson`](data-providers.md#strict-json) only preserves
the provider's object/array representation.

To receive several values, use a [`Collection`](collection.md) of inputs, a multiple [`Choice`](input-choice.md),
or `Berlioz\Form\Type\RawType` for a free-form value (a JSON structure, for example).

> ⚠️ **Warning**: The `scalar` option is deprecated since version 3.4 and will be removed in version 4, where
> these scalar-only inputs will always reject non-scalar values. Setting it triggers an `E_USER_DEPRECATED` notice: `scalar: false`
> disables the check (behavior before 3.4) and only eases the migration, `scalar: true` is the default behavior and
> can be removed.

## Attributes

Some attributes are used by the forms library for validations or interactions. Refer to the input description.
