```index
breadcrumb: Guides; Forms ; Inputs
summary-order: ; ; 2
keywords: form, input, text
```

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

| Name | Type | Default value | Description |
| ---- | ---- | ------------- | ----------- |
| **name** | string | *none* | Input name |
| **label** | string/false | *none* | Input label |
| **label_attributes** | array | *none* | List of label attributes |
| **required** | boolean | true | Input is required |
| **disabled** | boolean | false | Disabled input |
| **readonly** | boolean | false | Input is readonly |
| **helper** | string/false | *none* | Helper |
| **attributes** | array | *none* | List of attributes |
| **null_if_empty** | boolean | false | If empty value, it's casted to NULL |
| **transformer** | `TransformerInterface` | *Depends of input type* | Convert format into another |
| **validators** | `ValidatorInterface[]` | *Depends of input type* |  Valid data from user form |

## Attributes

Some attributes are used by forms library to do validations or interactions. Refers to the input description.