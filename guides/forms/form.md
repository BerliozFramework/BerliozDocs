---
breadcrumb:
  - Guides
  - Forms 
  - Form
summary-order: ;;1
keywords:
  - form
  - group
---

# Form

The `Berlioz\Form\Form` object is the main object to manipulate forms.
The class inherits from [`Berlioz\Form\Group`](group.md), so you can add a form to another form!

## Options

| Name             | Type                    | Default value      | Description                                                      |
|------------------|-------------------------|--------------------|------------------------------------------------------------------|
| **method**       | string                  | "post"             | HTTP method for form                                             |
| **required**     | boolean                 | true               | Default requirement for all elements                             |
| **dataProvider** | `DataProviderInterface` | `FormDataProvider` | Extracts submitted data, see [Data providers](data-providers.md) |

## Methods

- `Form::handle(ServerRequestInterface $request): void`: handle a server request for the form
- `Form::isSubmitted(): bool`: whether the form is submitted
- `Form::isValid(): bool`: whether all elements of the form pass validation
- `Form::getConstraints(): ConstraintInterface[]`: the constraints not respected by the form itself, after
  `isValid()`, including the ones added with `invalid()`; the constraints of an element are given by the element
  (`$form['name']->getConstraints()`) and displayed with it as its `errors`
- `Form::invalid(ConstraintInterface ...$constraint): void`: manually invalidate the form (also available on every
  element), see [Manual invalidation](../../components/form.md#manual-invalidation)
- `Form::resetConstraints(): void`: clear the constraints of the form and its elements, including manual ones
- `Form::getValue(): array`: form value (without processing by transformers); if the form is submitted, the value will
  be the user value, otherwise the default value
- `Form::getFinalValue(): array`: same as `Form::getValue()` with processing by transformers

## Request lifetime

Create a fresh form instance for each request, including in long-running application processes. Calling
`handle()` again on the same instance does not reset every child's submitted state: an omitted field can retain
its previous submitted value. Do not register a mutable form as a shared service across requests; use a factory
to build a new form instead.

## Example

Creation of a form.

```php
use Berlioz\Form\Form;
use Berlioz\Form\Group;
use Berlioz\Form\Transformer\JsonTransformer;
use Berlioz\Form\Type\Text;

$address = new Group();
$address
    ->add(
        'address',
        Text::class,
        [
            'label' => 'Address',
            'attributes' => ['maxlength' => 128],
        ]
    )
    ->add(
        'address_next',
        Text::class,
        [
            'label' => 'Address (next)',
            'required' => false,
            'attributes' => ['maxlength' => 128],
        ]
    )
    ->add(
        'zip',
        Text::class,
        [
            'label' => 'Zip code',
            'attributes' => ['maxlength' => 5],
        ]
    )
    ->add(
        'city',
        Text::class,
        [
            'label' => 'City',
            'attributes' => ['maxlength' => 128],
        ]
    );
   

$form = new Form('form');
$form
    ->add('input1', Text::class, ['label' => 'My first input'])
    ->add('input2', Text::class, ['label' => 'My second input']);
```

The result of the form will be:

```php
[
  "input1" => "Input 1 value",
  "input2" => "Input 2 value"
]
```
