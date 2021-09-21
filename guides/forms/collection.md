```index
breadcrumb: Guides; Forms ; Collection
summary-order: ; ; 4
keywords: form, collection
```

# Collection

Collection is an array of element. An element can be added, removed, etc.

Use class `Berlioz\Form\Collection` to construct a collection of inputs.

## Options

| Name | Type | Default value | Description |
| ---- | ---- | ------------- | ----------- |
| **prototype** | `ElementInterface` | *none* | The element template that will be cloned to create collection elements |
| **editable** | boolean | true | If collection is editable (can add/remove element) |
| **min_elements** | int | *none* | The minimum number of elements in collection |
| **max_elements** | int | *none* | The maximum number of elements in collection |
| **type** | string | "collection" | The type name of collection (to personalize template) |

## Creation of sub objects

Use special option "data_type" in the prototype options to create the specified class object. Or use a closure in this
option to create a dynamic object.

## JavaScript interaction

A library is available to do interaction with JavaScript and user: https://github.com/BerliozFramework/Form.js

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

Result of form will be:

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