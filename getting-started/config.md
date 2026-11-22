---
breadcrumb:
  - Getting started
  - Configuration
---

# Configuration

The default configuration manager of Berlioz Framework is the package
[**berlioz/config**](../components/config.md).

## Write the configuration

All your configuration files must be in the [configuration directory](directories.md).

Allowed formats:

| Type                                          | Extension | Mime              |
|-----------------------------------------------|-----------|-------------------|
| [JSON](https://www.json.org/)                 | .json     | application/json  |
| [JSON5](https://json5.org/)                   | .json5    | application/json5 |
| [YAML](https://yaml.org/)                     | .yml      | text/yaml         |
| [INI](https://en.wikipedia.org/wiki/INI_file) | .ini      | text/plain        |

Berlioz automatically merges all configuration files into a single configuration, accessible in your application.

In some cases, it is useful to split your configuration into different files, for example to separate the
configuration of packages.

## Get the configuration object

The configuration is accessible from the core object with the method `Core::getConfig()`. The core is accessible from
the application object.

From controllers that inherit from the `AbstractController` class, the config is accessible with
the method `AbstractController::getApp()->getConfig()`.

```php
// ...in controller method

/** @var \Berlioz\Config\ConfigInterface $config */
$config = $this->getApp()->getConfig();
```

## Usage

You can access the configuration variables with the method `Config::get(string $name)`. The given parameter is the path
of your variable in your JSON.

```php
/** @var \Berlioz\Config\ConfigInterface $config */
$config = $this->getApp()->getConfig();
$value = $config->get('app.varname.subvar');
```

## Dist files

All configuration files with `.dist` in their filename have a lower priority than the others.

It's recommended to write a dist configuration to push to the repository, and to write your sensitive data in a
`config.json` file added to the `.gitignore` file.

## Default configuration of Berlioz

Your configuration automatically extends a default configuration.

### Core configuration

This is the base configuration provided by **berlioz/core**:

```json
{
  "berlioz": {
    "environment": "prod",
    "locale": null,
    "debug": {
      "enable": false,
      "ip": [],
      "pending_ttl": null,
      "gc": {
        "probability": 1,
        "divisor": 100,
        "max_age": 7,
        "max_files": 100
      }
    },
    "proxies": {
      "trusted": []
    },
    "directories": {
      "app": "{var: berlioz.directories.app}",
      "cache": "{var: berlioz.directories.cache}",
      "config": "{var: berlioz.directories.config}",
      "debug": "{var: berlioz.directories.debug}",
      "log": "{var: berlioz.directories.log}",
      "tmp": "{config: berlioz.directories.var}/tmp",
      "var": "{var: berlioz.directories.var}",
      "vendor": "{var: berlioz.directories.vendor}",
      "working": "{var: berlioz.directories.working}"
    },
    "assets": {
      "manifest": null,
      "entrypoints": null,
      "entrypoints_key": null
    }
  },
  "events": {
    "listeners": {},
    "subscribers": []
  },
  "container": {
    "services": {},
    "providers": []
  }
}
```

Where each key is documented:

| Key                    | Documentation                                                                          |
|------------------------|----------------------------------------------------------------------------------------|
| `berlioz.debug`        | [Debug](debug.md)                                                                      |
| `berlioz.proxies`      | [Client IP resolution behind a proxy](debug.md#client-ip-resolution-behind-a-proxy)    |
| `berlioz.directories`  | [Directories](directories.md)                                                          |
| `events`               | [Events](../guides/events.md)                                                          |
| `container`            | [Service container](service-container.md)                                              |

> ⚠️ **Warning**: The `container.services` key is deprecated since version 3.4. Declare your services in a provider
> listed in `container.providers`. See [Service container](service-container.md#migrating-from-containerservices).

### HTTP configuration

When using **berlioz/http-core**, the following configuration is merged on top:

```json
{
  "berlioz": {
    "directories": {
      "templates": "{config:berlioz.directories.app}/resources/templates"
    },
    "assets": {
      "manifest": "{config:berlioz.directories.app}/public/assets/manifest.json",
      "entrypoints": "{config:berlioz.directories.app}/public/assets/entrypoints.json"
    },
    "http": {
      "finish_request": true,
      "errors": {
        "default": "Berlioz\\Http\\Core\\Http\\Handler\\Error\\DefaultErrorHandler"
      },
      "redirections": {},
      "middlewares": {
        "00": {
          "maintenance": "Berlioz\\Http\\Core\\Http\\Middleware\\MaintenanceMiddleware"
        },
        "01": {
          "debug_console": "Berlioz\\Http\\Core\\Http\\Middleware\\DebugConsoleMiddleware"
        },
        "02": {
          "accept_query": "Berlioz\\Http\\Core\\Http\\Middleware\\AcceptQueryMiddleware"
        },
        "99": {
          "redirection": "Berlioz\\Http\\Core\\Http\\Middleware\\RedirectionMiddleware"
        }
      }
    },
    "router": {
      "rewriteRequestUri": false
    },
    "maintenance": false
  },
  "controllers": [
    "Berlioz\\Http\\Core\\Controller\\DebugController"
  ],
  "twig": {
    "paths": {
      "Berlioz-HttpCore": "{config:berlioz.directories.vendor}/berlioz/http-core/resources"
    },
    "globals": {
      "app": "@AppProfile"
    }
  }
}
```

Where each key is documented:

| Key                            | Documentation                                                          |
|--------------------------------|------------------------------------------------------------------------|
| `berlioz.http.finish_request`  | [After the response](../http/terminate.md#configuration)               |
| `berlioz.http.errors`          | [Error handler](../http/error-handler.md#configuration)                |
| `berlioz.http.redirections`    | [HTTP redirections](../guides/redirections.md)                         |
| `berlioz.http.middlewares`     | [Middlewares](../http/middleware.md)                                   |
| `berlioz.router`               | [Reverse-proxy prefixes](../http/routing.md#reverse-proxy-prefixes)    |
| `berlioz.maintenance`          | [Maintenance mode](../guides/maintenance.md)                           |
| `controllers`                  | [Routing](../http/routing.md#basic-declaration)                        |
| `twig`                         | [Templating](../guides/templating.md#configuration)                    |
