---
breadcrumb:
  - Guides
  - Forms 
  - Group
summary-order: ;;3
keywords:
  - form
  - group
---

# Group

You can organize your inputs into groups. If you map an object to a form with named groups, the system maps the groups
to sub-objects, except if a group has no name.

Use the `Berlioz\Form\Group` class to construct a group of inputs.

## Example

Creation of an "address" group.

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

The result of the form will be:

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
