```index
breadcrumb: Guides; Forms
summary-order: ; 3
```

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