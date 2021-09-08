```index
breadcrumb: Guides; Forms ; Inputs ; Choice
summary-visible: false
keywords: form, input, choice
```

# Choice

## Description

- Class for this input: `Berlioz\Form\Type\Choice`
- HTML equivalents:
  - `<input type="checkbox">`
  - `<input type="radio">`
  - `<select>...</select>`

## Additional options

All [commons options](inputs.md#common-options), and:

| Name | Type | Default value | Description |
| ---- | ---- | ------------- | ----------- |
| **multiple** | boolean | false | If multiple choice accepted |
| **expanded** | boolean | false | If expanded choices (radio if not multiple, else checkbox) |
| **allow_clear** | boolean/string | false | If a blank element is added in first of list (if string value, it will be the label) |
| **choice_transformer** | `ChoiceTransformerInterface` | *none* | Transformer for individual value |
| **choice_label** | `\Closure`/array/string` | *none* | Function called to determine the label of a choice ; be able to an array ; be able to a method name to call for object elements |
| **choice_value** | `\Closure`/array/string` | *none* | Function called to determine the value of a choice ; be able to an array ; be able to a method name to call for object elements |
| **choice_attributes** | `\Closure`/array/string` | *none* | Function called to determine the attributes of a choice ; be able to an array ; be able to a method name to call for object elements |
| **preferred_choices** | `\Closure`/array` | *none* | Function called to determine the preferred choices (first in the list) ; be able to an array of elements |

## Attributes

No attribute.