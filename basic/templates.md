<meta name="docparser-index" content="Basic uses; Templates" />
<meta name="docparser-index-order" content="6" />

# Templates

In **Berlioz**, we recommend to use **Twig** template engine:

- Official website: [https://twig.symfony.com/](https://twig.symfony.com/)
- Documentation: [https://twig.symfony.com/doc/2.x/](https://twig.symfony.com/doc/2.x/)

## Integration in Berlioz

If you use the main repository of Berlioz : **berlioz/berlioz**, the package it's already available.

If you use customizable repositories, you need to install by yourself the **TwigPackage**.

## Installation of Twig package

Use composer to install package:

```bash
composer require berlioz/twig-package
```

For more detail package installation, referred to the [package description](./packages.md) page.

## Template service

Template engine in Berlioz implements the `\Berlioz\Core\Package\TemplateEngine` interface, who have some methods:

- `TemplateEngine::addGlobal(string $name, $value): TemplateEngine`

  > Add global variable.

- `TemplateEngine::registerPath(string $path, string $namespace = null)`

  > Register a new path for template engine.

- `TemplateEngine::render(string $name, array $variables = []): string`

  > Add global variable.

- `TemplateEngine::addGlobal(string $name, $value): TemplateEngine`

  > Render a template.

- `TemplateEngine::hasBlock(string $tplName, string $blockName): bool`

  > Has block in template?

- `TemplateEngine::renderBlock(string $tplName, string $blockName, array $variables = []): string`

  > Render a block in template.

## Usage

In [controllers](./controllers.md), the method `render(string $name, array $variables = []): string` is available to process of the rendering of a template file.

Example inside controllers methods:

```php
// In controller method
$rendering = $this->render('my-template/path/file.html.twig',
                           ['myVar' => 'value']);
```

Example outside controllers:

```php
/** @var \Berlioz\Core\Package\TemplateEngine $templateEngine */
$templateEngine = $this->getApp()->getServiceContainer()->get('templating');
$rendering = $templateEngine->render('my-template/path/file.html.twig',
                                     ['myVar' => 'value']);
```