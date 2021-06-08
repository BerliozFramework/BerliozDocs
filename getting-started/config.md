```index
breadcrumb: Getting started; Configuration
```

# Configuration

The default configuration manager of Berlioz Framework is the package
[**berlioz/config**](https://github.com/BerliozFramework/Config).

## Write configuration

All your configuration file must be in [configuration directory](directories.md).

Allowed formats:

Type | Extension | Mime
-----|-----------|-----
[JSON](https://www.json.org/) | .json | application/json
[JSON5](https://json5.org/) | .json5 | application/json5
[YAML](https://yaml.org/) | .yml | text/yaml
[INI](https://en.wikipedia.org/wiki/INI_file) | .ini | text/plain

Berlioz merge automatically all configuration files in only one configuration accessible in your application.

It's interested in some case to separate your configuration in different files. Like separate packages configuration...

## Get configuration object

Configuration is accessible from the core object with method `Core:getConfig()`. The core is accessible from the
application object.

From controllers whose inherit `AbstractController` class, the config is accessible with
method `AbstractController::getApp()->getConfig()`.

```php
// ...in controller method

/** @var \Berlioz\Config\ConfigInterface $config */
$config = $this->getApp()->getConfig();
```

## Usage

You can access to the variables of configuration with method `Config:get(string $name)`. Given parameter is the path of
your variable in your JSON.

```php
/** @var \Berlioz\Config\ConfigInterface $config */
$config = $this->getApp()->getConfig();
$value = $config->get('app.varname.subvar');
```

## Dist files

All configuration files with `.dist` in filename have a lower priority than others.

It's recommended to write a dist configuration to push on the repository ; and add a `config.json` into `.gitignore`
file where you can write your sensitive data...

## Default configuration of Berlioz

Your configuration is automatically extends from a default configuration:

```json
{
  "berlioz": {
    "environment": "prod",
    "locale": null,
    "debug": {
      "enable": false,
      "ip": []
    },
    "maintenance": false,
    "directories": {
      "app": "{var: berlioz.directories.app}",
      "cache": "{var: berlioz.directories.cache}",
      "config": "{var: berlioz.directories.config}",
      "debug": "{var: berlioz.directories.debug}",
      "log": "{var: berlioz.directories.log}",
      "templates": "{config:berlioz.directories.app}/resources/templates",
      "tmp": "{config: berlioz.directories.var}/tmp",
      "var": "{var: berlioz.directories.var}",
      "vendor": "{var: berlioz.directories.vendor}",
      "working": "{var: berlioz.directories.working}"
    },
    "assets": {
      "manifest": "{config:berlioz.directories.app}/public/assets/manifest.json",
      "entrypoints": "{config:berlioz.directories.app}/public/assets/entrypoints.json",
      "entrypoints_key": null
    },
    "http": {
      "errors": {
        "default": "Berlioz\\Http\\Core\\Http\\Error\\DefaultErrorHandler"
      },
      "redirections": {},
      "middlewares": {
        "00": {
          "maintenance": "Berlioz\\Http\\Core\\Http\\Middleware\\MaintenanceMiddleware"
        },
        "99": {
          "redirection": "Berlioz\\Http\\Core\\Http\\Middleware\\RedirectionMiddleware"
        }
      }
    }
  }
}
```