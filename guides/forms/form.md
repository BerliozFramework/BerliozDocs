```index
breadcrumb: Guides; Forms ; Form
summary-order: ; ; 1
keywords: form, group
```

# Form

Object `Berlioz\Form\Form` is the main object to manipulate forms.
The class inherits [`Berlioz\Form\Group`](group.md), so you can add a form to another form!

## Options

| Name | Type | Default value | Description |
| ---- | ---- | ------------- | ----------- |
| **method** | string | "post" | HTTP method for form |
| **required** | boolean | true | Default requirement for all elements |

## Methods

- `Form::handle(ServerRequestInterface $request): void`: handle a server request to form
- `Form::isSubmitted(): bool`: form is submitted
- `Form::isValid(): bool`: all elements of form pass validation
- `Form::getConstraints(): ConstraintInterface[]`: all the thrown constraints by the validation
- `Form::getValue(): array`: form value (without treatment by transformers) ; if form is submitted, the value will be the user value else the default value
- `Form::getFinalValue(): array`: same as `Form::getValue()` with treatment by transformers

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

Result of form will be:

```php
[
  "input1" => "Input 1 value",
  "input2" => "Input 2 value"
]
```