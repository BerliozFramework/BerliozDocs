```index
breadcrumb: Basic uses; Configuration
summary-order: 2
```

# Configuration

The default configuration manager of Berlioz Framework is the package [**berlioz/config**](https://github.com/BerliozFramework/Config).

## Write configuration

All your configuration file must be in [configuration directory](../directories.md).
Berlioz merge automatically all **JSON** files (`*.json`) in only one configuration accessible in your application.

It's interesting in some case to separate your configuration in different files. Like separate packages configuration...

## Get configuration object

Configuration is accessible from the core object with method `Core:getConfig()`. The core is accessible from the application object.

From controllers whose inherit `AbstractController` class, the core is accessible with method `AbstractController:getCore()`.

```php
// ...in controller method

/** @var \Berlioz\Core\Config $config */
$config = $this->getCore()->getConfig();
```

## Usage

You can access to the variables of configuration with method `Config:get(string $name)`. `$name` given parameter is the path of your variable in your JSON.

```php
/** @var \Berlioz\Core\Config $config */
$config = $this->getCore()->getConfig();
$value = $config->get('app.varname.subvar');
```

## Default configuration

Your configuration is automatically extended from a default configuration:

```json
{
  "berlioz": {
    "debug": false,
    "directories": {
      "templates": "%berlioz.directories.app%/templates",
      "var": "%berlioz.directories.app%/var",
      "debug": "%berlioz.directories.app%/var/debug",
      "cache": "%berlioz.directories.app%/var/cache",
      "log": "%berlioz.directories.app%/var/log",
      "tmp": "%berlioz.directories.app%/var/tmp",
      "vendor": "%berlioz.directories.app%/vendor"
    },
    "http": {
      "errors": {
        "default": "Berlioz\\HttpCore\\Http\\DefaultHttpErrorHandler"
      }
    }
  }
}
```

The variables encapsulated by `%` are automatically replaced by associate values.

## Extend configuration

It's possible to extend a configuration file with special root key `@extends` with extended file in value.

### config.json.dist

We recommend to create a file `config.json.dist` with the default parameters. File that you can commit on your GIT repository.

This file **do not must** include any password or address server, or other critical data.

So you can create a `config.json` file who inherit the `config.json.dist` file, and you just add inherit parameters like password... and **do not commit this file** and add this in your `.gitignore` file.

### Example

File **config.json**:

```json
{
  "@extends": "config.json.dist",
  "app": {
    "var1": "value1",
    "var2": {
      "var3": "value3"
    }
  }
}
```

File **config.json.dist**:

```json
{
  "app": {
    "var.another": "value",
    "var1": "valueX"
  }
}
```

The final configuration file is:

```json
{
  "app": {
    "var.another": "value",
    "var1": "value1",
    "var2": {
      "var3": "value3"
    }
  }
}
```