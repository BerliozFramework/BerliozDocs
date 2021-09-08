```index
breadcrumb: Guides; Forms ; Group
summary-order: ; ; 3
keywords: form, group
```

# Group

You can organize your inputs into groups. If you map an object with a form with named groups, the system map with sub
objects, excepted if group has no name.

Use class `Berlioz\Form\Group` to construct a group of inputs.

## Example

Creation of a group "address".

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
$form->add('address', $address);
```

Result of form will be:

```php
[
  "address" => [
    "address" => "Address value",
    "address_next" => "Address next value",
    "zip" => "01234",
    "city" => "City value"
  ]
]
```