---
breadcrumb:
  - Guides
  - Templating
summary-order: ;2
keywords:
  - templating
  - template
  - twig
---

# Templates

We provide a **Twig** package. Twig is a PHP template engine with its own syntax.

- Official website: [https://twig.symfony.com/](https://twig.symfony.com/)
- Documentation: [https://twig.symfony.com/doc/](https://twig.symfony.com/doc/)

## Integration in Berlioz

If you use the main repository of Berlioz, **berlioz/berlioz**, the package is already available.

If you use customizable repositories, you need to [install the package](packages.md) yourself:

```bash
composer require berlioz/twig-package
```

## Usage

In [controllers](../http/controllers.md), the method `render(string $name, array $variables = []): string` is available
to render a template file.

Example inside controller methods:

```php
// In controller method
$rendering = $this->render(
    'my-template/path/file.html.twig',
    ['myVar' => 'value']
);
```

Example outside controllers:

```php
use Berlioz\Package\Twig\Twig;

/** @var Twig $twig */
$twig = $this->getCore()->getContainer()->get(Twig::class);
$rendering = $twig->render(
    'my-template/path/file.html.twig',
    ['myVar' => 'value']
);
```

## Service inflector

An inflector is provided with the interface `\Berlioz\Package\Twig\TwigAwareInterface`.

For more information on inflectors, see [container inflectors](../getting-started/service-container.md).

## Methods

The Twig package has some methods:

- `Twig::render(string $name, array $variables = []): string`

  > Render a template.

- `Twig::hasBlock(string $name, string $blockName): bool`

  > Has block in template?

- `Twig::renderBlock(string $name, string $blockName, array $variables = []): string`

  > Render a block in template.

## Configuration

### Default configuration

```json
{
  "berlioz": {
    "directories": {
      "templates": "{config: berlioz.directories.app}/resources/templates"
    }
  },
  "twig": {
    "paths": {
      "__main__": "{config: berlioz.directories.templates}",
      "Berlioz-TwigPackage": "{config: berlioz.directories.vendor}/berlioz/twig-package/resources"
    },
    "options": {
      "debug": "{config: berlioz.debug.enable}",
      "cache": "{config: berlioz.directories.cache}/twig",
      "optimizations": -1
    },
    "extensions": [
      "Berlioz\\Package\\Twig\\Extension\\AssetExtension",
      "Berlioz\\Package\\Twig\\Extension\\DefaultExtension",
      "Berlioz\\Package\\Twig\\Extension\\RouterExtension"
    ],
    "globals": {}
  }
}
```

### Global variable `app`

The HTTP core package registers an `AppProfile` object as a Twig global variable `app`. It's available in all
templates:

```twig
{{ app.env }}                {# Current environment (e.g. "prod", "dev") #}
{{ app.locale }}             {# Current locale #}
{{ app.debugEnabled }}       {# Boolean: is debug enabled? #}
{{ app.request }}            {# Current ServerRequestInterface #}
{{ app.route }}              {# Current matched RouteInterface #}
{{ app.config('key') }}      {# Get a configuration value #}
{{ app.flashBag }}           {# FlashBag instance #}
{{ app.assets }}             {# Assets instance #}
```

### Cache

The internal cache of Twig is used, and can be disabled:

```json
{
    "twig": {
        "options": {
            "cache": false
        }
    }
}
```
