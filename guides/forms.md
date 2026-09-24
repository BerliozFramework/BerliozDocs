---
breadcrumb:
  - Guides
  - Forms
summary-order: ;3
---

# Forms

## Installation

Use composer to install library:

```bash
composer require berlioz/form
```

## Description

Library `berlioz/form` permit you to create HTML forms from your PHP application.
Use Twig extension to transpose into HTML.

Forms are composed of:

- [Form](forms/form.md)
    - [Inputs](forms/inputs.md)
    - [Collection](forms/collection.md)
    - [Group](forms/group.md)

## Mapping fields to an object

The `mapped` option controls how a field reads its initial value from an object and writes its submitted value
back to that object. The field name remains the public name used in the form.

| `mapped` value | Behavior |
|----------------|----------|
| `true` (default for ordinary inputs) | Use the field name as the object property name. |
| A property name (`string`) | Read and write a different property through the object's accessors. |
| `false` | Do not read from or write to the mapped object for this field. |
| A `FormMapping` instance | Use custom read and write callbacks (since version 3.3). |

For example, a field named `display_name` can read and write the object's `name` property:

```php
use Berlioz\Form\Type\Text;

$form->add('display_name', Text::class, ['mapped' => 'name']);
```

### Custom mapping

> 🆕 **Info**: *Since version 3.3*

Use `FormMapping` when a field needs to access a nested property or apply custom read/write logic.
Both callbacks receive the object mapped to the containing form or group.

```php
use Berlioz\Form\Form;
use Berlioz\Form\FormMapping;
use Berlioz\Form\Type\Text;

$user = (object) ['profile' => (object) ['name' => 'Alice']];
$form = new Form('user', $user);
$form->add('display_name', Text::class, [
    'mapped' => new FormMapping(
        get: fn(object $user): string => $user->profile->name,
        set: function (object $user, mixed $value): bool {
            $user->profile->name = $value;

            return true;
        },
    ),
]);
```

The field displays `Alice`; when the form hydrates the object, its submitted value is written to
`$user->profile->name`. Its public name remains `display_name`.

- The `get` callback returns the value. It can optionally accept `?bool &$exists = null` as a second parameter
  and set it to `false` when the target does not exist. Returning `null` alone represents an existing null value.
- The `set` callback receives the value after form transformation and returns `true` on success or `false`
  if it cannot write the value. A failed write causes a hydration error.

For a simple property, `FormMapping::forProperty('name')` is equivalent to using `'mapped' => 'name'`.
The existing boolean and string options continue to work as before.

## Notes for custom form elements

> 🆕 **Info**: *Since version 3.3*

Custom classes implementing `ElementInterface` directly must now provide `getMapping(): ?FormMapping`
(`Berlioz\Form\FormMapping`). Elements extending `AbstractElement` inherit this implementation and require
no change for this addition.

## Validators

Validators can be added to an input, a group or form to validate data from user.

A validator must implements interface `Berlioz\Form\Validator\ValidatorInterface`:

```php
interface ValidatorInterface
{
    /**
     * Validate.
     *
     * @param ElementInterface $element
     *
     * @return ConstraintInterface[]
     * @throws ValidatorException
     */
    public function validate(ElementInterface $element): array;
}
```

Method `ValidatorInterface::validate(): array` must return the thrown constraints.

## Transformers

Transformers permit the data conversion from user to application, and invert.

A transformer must implements interface `Berlioz\Form\Transformer\TransformerInterface`:

```php
interface TransformerInterface
{
    /**
     * Transform data to form.
     *
     * @param mixed $data
     * @param ElementInterface $element
     *
     * @return mixed
     */
    public function toForm(mixed $data, ElementInterface $element): mixed;

    /**
     * Transform data from form.
     *
     * @param mixed $data
     * @param ElementInterface $element
     *
     * @return mixed
     */
    public function fromForm(mixed $data, ElementInterface $element): mixed;
}
```

Existent transformers:

- `Berlioz\Form\Transformer\CheckboxTransformer`
- `Berlioz\Form\Transformer\DateTimeTransformer`
- `Berlioz\Form\Transformer\DefaultTransformer`
- `Berlioz\Form\Transformer\JsonTransformer`
- `Berlioz\Form\Transformer\NumberTransformer`
