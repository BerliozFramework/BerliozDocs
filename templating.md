```index
breadcrumb: Templating
summary-order: 4
keywords: templating, twig
```

# Templates

We provide a **Twig** package. Twig is a PHP template engine with its own syntax.

- Official website: [https://twig.symfony.com/](https://twig.symfony.com/)
- Documentation: [https://twig.symfony.com/doc/](https://twig.symfony.com/doc/)

## Integration in Berlioz

If you use the main repository of Berlioz : **berlioz/berlioz**, the package it's already available.

If you use customizable repositories, you need to [install the package](guides/packages.md) by yourself:

```bash
composer require berlioz/twig-package
```

## Usage

In [controllers](http/controllers.md), the method `render(string $name, array $variables = []): string` is available to process of the rendering of a template file.

Example inside controllers methods:

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

An inflector is provided with interface: `\Berlioz\Package\Twig\TwigAwareInterface`.

For more information on inflectors, refers you to [container inflectors](./getting-started/service-container.md).

## Methods

Twig package have some methods:

- `Twig::render(string $name, array $variables = []): string`

  > Add global variable.

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
      "templates": "{config: berlioz.directories.app}/templates"
    }
  },
  "twig": {
    "paths": {
      "__main__": "{config: berlioz.directories.templates}",
      "Berlioz-TwigPackage": "{config: berlioz.directories.vendor}/berlioz/twig-package/resources"
    },
    "options": {
      "cache": "{config: berlioz.directories.cache}/twig",
      "optimizers": null
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

### Cache

The internal cache of Twig is use, and can be disabled:

```json
{
    "twig": {
        "options": {
            "cache": false
        }
    }
}
```