---
breadcrumb:
  - HTTP Website
  - Middlewares
keywords:
  - middleware
---

# Middlewares

**Berlioz v2** introduced the middleware concept into the framework. The implementation is based
on the [PSR-15](https://www.php-fig.org/psr/psr-15/) recommendation.

> A middleware component is an individual component participating,
> often together with other middleware components, in the processing
> of an incoming request and the creation of a resulting response,
> as defined by PSR-7.
>
> Source: [PHP-FIG](https://www.php-fig.org/psr/psr-15/)

## Default middlewares

Some middlewares are configured by default:

- `Berlioz\Http\Core\Http\Middleware\MaintenanceMiddleware`:
  stop the execution of controllers and display the maintenance page if
  [maintenance is enabled](../guides/maintenance.md).
- `Berlioz\Http\Core\Http\Middleware\DebugConsoleMiddleware` *(since version 3.2)*:
  restrict the [debug console](../getting-started/debug.md) (`/_console`) to the debug mode and to the allowed IPs;
  other requests get a 404 error.
- `Berlioz\Http\Core\Http\Middleware\AcceptQueryMiddleware` *(since version 3.4)*:
  handle the `acceptQuery` route option of the [QUERY method](routing.md#query-method): `Accept-Query` header and
  rejection of `QUERY` requests with a missing, malformed (400) or unsupported (415) content type. Does nothing for
  routes without the option.
- `Berlioz\Http\Core\Http\Middleware\RedirectionMiddleware`:
  perform the [redirections configured](../guides/redirections.md) in the configuration if no controller is found.

## Forwarded-prefix middleware

> 🆕 **Info**: *Since version 3.3*

HTTP Core automatically appends `ForwardedPrefixMiddleware` when forwarded-prefix handling and
`berlioz.router.rewriteRequestUri` are enabled. It runs after configured middlewares and route matching, immediately
before the controller. Configure it through the router options rather than adding it to the middleware list.

See [reverse-proxy prefixes](routing.md#reverse-proxy-prefixes) for configuration, request-path visibility and the
deprecation, since version 3.3, of handling trusted prefixes without URI rewriting.

## Declare middlewares

Middlewares are declared in the configuration. To manage the execution priority of middlewares, the configuration is
based on an incremental priority:

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
