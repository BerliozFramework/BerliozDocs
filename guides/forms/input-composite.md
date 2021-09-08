```index
breadcrumb: Guides; Forms ; Inputs ; Composite
summary-visible: false
keywords: form, input, composite
```

# Composite

You can create an input composed of other inputs, and transform values into another.

Use class `Berlioz\Form\Type\CompositeType` to construct a composite type.

## Example

Creation of a SEO composite type. The objective is to get the value of type in JSON format.

```php
use Berlioz\Form\Form;
use Berlioz\Form\Transformer\JsonTransformer;
use Berlioz\Form\Type\CompositeType;
use Berlioz\Form\Type\Text;

$seoType = new CompositeType(
    [
        'required' => false,
        'transformer' => new JsonTransformer(),
    ],
    new Text([
        'name' => 'title',
        'label' => 'Title',
        'required' => false,
        'attributes' => ['maxlength' => 60]
    ]),
    new Text([
        'name' => 'description',
        'label' => 'Description',
        'required' => false,
        'attributes' => ['maxlength' => 160]
    ])
);

$form = new Form('form');
$form->add('seo', $seoType);
```

Result of form will be:

```json
{
    "title": "Value of title",
    "description": "Value of description"
}
```