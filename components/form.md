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

3 types of elements exists in **Berlioz Form**:

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

Constructor of `Form` object accept 3 parameters:

- Name of form
- Mapped object
- Array of options

Example:

```php
$form = new Form('my_form', null, ['method' => 'post']);
```

## Declare form control

`add` method accept 3 parameters:

- Name of control (must be the same that the mapped element)
- Type (class name or object)
- Array of options

Options are different between controls.

Example:

```php
$form->add('my_control', Text::class, ['label' => 'My control']);
```

## Handle

**Berlioz Form** implements **PSR-7** (HTTP message interfaces). You must give the server request to the `handle`
method.

```php
$form = new Form('my_form', null, ['method' => 'post']);
// ...

$form->handle($request);

if ($form->isSubmitted() && $form->isValid()) {
    // ...
}
```

## Group

Example for an postal address:

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
