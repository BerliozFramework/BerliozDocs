---
breadcrumb:
  - Guides
  - Forms
  - Data providers
summary-order: ;;5
keywords:
  - form
  - data provider
  - api
  - json
---

# Data providers

A data provider extracts the submitted data from the server request when `Form::handle()` is called.
Set it with the `dataProvider` option of the [form](form.md):

```php
use Berlioz\Form\DataProvider\ApiDataProvider;
use Berlioz\Form\Form;

$form = new Form('user', $user, ['dataProvider' => new ApiDataProvider(strictJson: true)]);
```

The form only asks its data provider for data when the request method matches the form `method` option.

## Available providers

| Class                                         | Submitted data                                                                                 |
|-----------------------------------------------|------------------------------------------------------------------------------------------------|
| `Berlioz\Form\DataProvider\FormDataProvider`  | **Default.** Data under the form name, from the query string (`GET`...) or the parsed body.   |
| `Berlioz\Form\DataProvider\RootFormDataProvider` | The whole query string or parsed body, without the form name level.                         |
| `Berlioz\Form\DataProvider\ApiDataProvider`   | The whole parsed body (typically JSON) of `POST`, `PUT`, `PATCH` and `DELETE` requests.      |

Uploaded files are merged into the submitted data.

## API data provider

`ApiDataProvider` is designed for APIs receiving a JSON body. Its constructor accepts:

| Parameter          | Type           | Default | Description                                                         |
|--------------------|----------------|---------|---------------------------------------------------------------------|
| **mapToElement**   | string\|null   | null    | Name of the element receiving the whole parsed body.                |
| **strictJson**     | bool\|null     | null    | Preserve JSON types (since version 3.4). See below.                 |

> 🆕 **Info**: *Since version 3.4*, `PATCH` and `DELETE` requests are handled too.

### Strict JSON

> 🆕 **Info**: *Since version 3.4*

By default, every JSON object of the body is converted to an associative array: a nested `{}` cannot be
distinguished from `[]`, and an object with numeric keys becomes a list.

With `strictJson: true`, only the root object is converted to an array; nested objects stay `stdClass` and arrays
stay arrays. A field can then require a JSON object:

```php
use Berlioz\Form\DataProvider\ApiDataProvider;

// {"name": "foo", "attributes": {}}
$provider = new ApiDataProvider(strictJson: true);
// Submitted data: ['name' => 'foo', 'attributes' => stdClass]
```

[Groups](group.md) accept a submitted `stdClass` and [collections](collection.md) iterate over its properties, so
existing form structures keep working.

> ⚠️ **Warning**: Constructing `ApiDataProvider` without `strictJson: true` is deprecated since version 3.4 and
> triggers an `E_USER_DEPRECATED` notice. Strict JSON will be the only behavior in version 4, where the parameter
> will be removed.
