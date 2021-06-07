```index
breadcrumb: HTTP Website; Middlewares
keywords: middleware
```

# Middlewares

**Berlioz 2** introduce the middlewares concept into the framework. The implementation is based
on [PSR-15](https://www.php-fig.org/psr/psr-15/) recommendation.

> A middleware component is an individual component participating,
> often together with other middleware components, in the processing
> of an incoming request and the creation of a resulting response,
> as defined by PSR-7.
>
> Source: [PHP-FIG](https://www.php-fig.org/psr/psr-15/)

## Default middlewares

Some middlewares are configured by default:

- `Berlioz\Http\Core\Http\Middleware\MaintenanceMiddleware`: stop execution of controllers and display maintenance page
  if maintenance is enabled.
- `Berlioz\Http\Core\Http\Middleware\RedirectionMiddleware`: do redirections configured in configuration if no controller
  found.

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