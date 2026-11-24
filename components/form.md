---
breadcrumb:
  - Components
  - Form
keywords:
  - form
  - input
  - validation
  - collection
  - group
---

# Form

> ℹ️ **Note**: The form component is part of the **Berlioz** ecosystem but is available as a standalone package:
> [`berlioz/form`](https://github.com/BerliozFramework/Form).
> You can find it on [Packagist](https://packagist.org/packages/berlioz/form).
> You can use it independently of the framework, in any PHP application.

**Berlioz Form** is a PHP library to manage your forms.

## Description

Three types of elements exist in **Berlioz Form**:

- `AbstractType`: it's a form control
- `Group`: represents an object in OOP
- `Collection`: represents a collection of AbstractType or Group

Input types available:

- `Button`
- `Checkbox`
- `Choice`
- `Date`
- `DateTime`
- `Email`
- `File`
- `Hidden`
- `Month`
- `Number`
- `Password`
- `Range`
- `Reset`
- `Search`
- `Submit`
- `Tel`
- `Text`
- `TextArea`
- `Time`
- `Url`
- `Week`

## Form creation

The constructor of the `Form` object accepts 3 parameters:

- Name of the form
- Mapped object
- Array of options

Example:

```php
$form = new Form('my_form', null, ['method' => 'post']);
```

## Declare a form control

The `add` method accepts 3 parameters:

- Name of the control (used as the mapped property name by default)
- Type (class name or object)
- Array of options

Options differ between controls.

Use the `mapped` option to target a different property, disable mapping or provide custom read/write logic.
See [Mapping fields to an object](../guides/forms.md#mapping-fields-to-an-object).

Example:

```php
$form->add('my_control', Text::class, ['label' => 'My control']);
```

## Handle

**Berlioz Form** uses **PSR-7** (HTTP message interfaces). You must give the server request to the `handle`
method.

```php
$form = new Form('my_form', null, ['method' => 'post']);
// ...

$form->handle($request);

if ($form->isSubmitted() && $form->isValid()) {
    // ...
}
```

### Manual invalidation

> 🆕 **Info**: *Since version 3.4*

Some checks can only be done after the validators, for example a uniqueness check in a database. Use the `invalid()`
method to add constraints to the form or to any of its elements:

```php
use Berlioz\Form\Validator\Constraint\BasicConstraint;

$form->handle($request);

if ($form->isSubmitted() && $form->isValid()) {
    if ($userRepository->emailExists($form['email']->getValue())) {
        $form['email']->invalid(new BasicConstraint(message: 'This email is already used.'));
    }
}

if ($form->isSubmitted() && $form->isValid()) {
    // Save...
}
```

The element and all its parents are then invalid, and the constraint is returned by `getConstraints()` to be displayed
with the other errors. Manual constraints are kept until:

- the `resetConstraints()` method is called (it also resets the children of a group, a collection or a form),
- the form is handled again with `handle()`,
- the element is cloned (a clone, like a new collection entry, never copies the constraints of the original).

## Group

Example for a postal address:

```php
$addressGroup = new Group(['type' => 'address']);
$addressGroup
    ->add('address', Text::class, ['label' => 'Address'])
    ->add(
        'address_next',
        Text::class,
        ['label' => 'Address (next)',
         'required' => false]
    )
    ->add('postal_code', Text::class, ['label' => 'Postal code'])
    ->add('city', Text::class, ['label' => 'City']);

$form->add('address', $addressGroup);
```

## Collection

Example for a list of addresses:

```php
// Create group
$addressGroup = new Group(['type' => 'address']);
$addressGroup
    ->add('address', Text::class, ['label' => 'Address'])
    ->add(
        'address_next',
        Text::class,
        ['label' => 'Address (next)',
         'required' => false]
    )
    ->add('postal_code', Text::class, ['label' => 'Postal code'])
    ->add('city', Text::class, ['label' => 'City']);

// Create collection
$collection = new Collection([
    'prototype' => $addressGroup,
    'data_type' => ArrayObject::class
]);

// Add collection to form
$form->add('addresses', $collection);
```
