---
breadcrumb:
  - Guides
  - Forms
  - Collection
summary-order: ;;4
keywords:
  - form
  - collection
---

# Collection

A collection is an array of elements. An element can be added, removed, etc.

Use the `Berlioz\Form\Collection` class to construct a collection of inputs.

## Options

| Name             | Type               | Default value | Description                                                               |
|------------------|--------------------|---------------|---------------------------------------------------------------------------|
| **prototype**    | `ElementInterface` | *none*        | The element template that will be cloned to create collection elements    |
| **editable**     | boolean            | true          | If the collection is editable (can add/remove elements)                   |
| **min_elements** | int                | *none*        | The minimum number of elements in the collection                          |
| **max_elements** | int                | *none*        | The maximum number of elements in the collection (enforced on submission) |
| **type**         | string             | "collection"  | The type name of the collection (to customize the template)               |

> 🆕 **Info**: *Since version 3.1*
>
> The `max_elements` limit is now enforced: submitted data exceeding this limit is silently truncated.

## Creating sub-objects

Use the special "data_type" option in the prototype options to create an object of the specified class. Or use a
closure in this option to create the object dynamically.

## JavaScript interaction

A library is available to handle interactions with JavaScript and the user: <https://github.com/BerliozFramework/Form.js>

## Example

Creation of a collection of addresses.

```php
use Berlioz\Form\Collection;
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
$form->add('addresses', Collection::class, ['prototype' => $address]);
```

The result of the form will be:

```php
[
  "addresses" => [
    [
      "address" => "Address value",
      "address_next" => "Address next value",
      "zip" => "01234",
      "city" => "City value"
    ],
    [
      "address" => "Address 2 value",
      "address_next" => "Address 2 next value",
      "zip" => "12345",
      "city" => "City 2 value"
    ]
  ]
]
```
