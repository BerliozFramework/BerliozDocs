---
breadcrumb:
  - HTTP Website
  - Middlewares
keywords:
  - middleware
---

# Middlewares

**Berlioz v2** introduce the middlewares concept into the framework. The implementation is based
on [PSR-15](https://www.php-fig.org/psr/psr-15/) recommendation.

> A middleware component is an individual component participating,
> often together with other middleware components, in the processing
> of an incoming request and the creation of a resulting response,
> as defined by PSR-7.
>
> Source: [PHP-FIG](https://www.php-fig.org/psr/psr-15/)

## Default middlewares

Some middlewares are configured by default:

- `Berlioz\Http\Core\Http\Middleware\MaintenanceMiddleware`:
  stop execution of controllers and display maintenance page if [maintenance is enabled](../guides/maintenance.md).
- `Berlioz\Http\Core\Http\Middleware\RedirectionMiddleware`:
  do [redirections configured](../guides/redirections.md) in configuration if no controller found.

## Forwarded-prefix middleware

> 🆕 **Info**: *Since version 3.3*

HTTP Core automatically appends `ForwardedPrefixMiddleware` when forwarded-prefix handling and
`berlioz.router.rewriteRequestUri` are enabled. It runs after configured middlewares and route matching, immediately
before the controller. Configure it through the router options rather than adding it to the middleware list.

See [reverse-proxy prefixes](routing.md#reverse-proxy-prefixes) for configuration, request-path visibility and the
deprecation, since version 3.3, of handling trusted prefixes without URI rewriting.

## Declare middlewares

Middlewares are declared into the configuration. To manage priority of middlewares execution, the configuration is based
on incremental priority:

```json
{
  "berlioz": {
    "http": {
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
