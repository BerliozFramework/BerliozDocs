```index
breadcrumb: Templating; Introduction
summary-order: 4; 1
keywords: templating
```

# Templates

We provide a **Twig** package. Twig is a PHP template engine with is own syntax.

- Official website: [https://twig.symfony.com/](https://twig.symfony.com/)
- Documentation: [https://twig.symfony.com/doc/](https://twig.symfony.com/doc/)

## Integration in Berlioz

If you use the main repository of Berlioz : **berlioz/berlioz**, the package it's already available.

If you use customizable repositories, you need to [install the package](packages.md) by yourself:

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
/** @var \Berlioz\Package\Twig\Twig $twig */
$twig = $this->getCore()->getServiceContainer()->get(\Berlioz\Package\Twig\Twig::class);
$rendering = $twig->render(
    'my-template/path/file.html.twig',
    ['myVar' => 'value']
);
```

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
            "templates": "%berlioz.directories.app%/templates"
        }
    },
    "twig": {
        "paths": {
            "__main__": "%berlioz.directories.templates%"
        },
        "options": {
            "cache": "%berlioz.directories.cache%/twig",
            "debug": "%berlioz.debug.enable%"
        },
        "extensions": [
            "Berlioz\\Package\\Twig\\TwigExtension"
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