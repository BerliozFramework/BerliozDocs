---
breadcrumb:
  - HTTP Website
  - Controllers
summary-order: 2; 1
keywords:
  - controller
---

# Controllers

It's classes whose control interactions between models, services and templates. The controller is instanced, and the
matched method is called, by router service when the application started.

## Class `\Berlioz\Http\Core\Controller\AbstractController`

It's the main controller for website projects who offer methods for services, routing, templating, flash messages, and
redirection.

## Example

```php
use Berlioz\Http\Core\Controller\AbstractController;
use Berlioz\Http\Message\Response;
use Berlioz\Router\Attribute\Route;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Message\ResponseInterface;

class MyController extends AbstractController
{
    #[Route('/my-route/{attr1}')]
    public function myMethod(ServerRequestInterface $request): ResponseInterface
    {
        $attribute = $request->getAttribute('attr1');

        return $this->response('Hello ' . $attribute);
    }
}
```

## Return values

Controller methods do not have to return a `ResponseInterface`. The framework automatically converts the return value:

| Return type | Behavior |
|---|---|
| `ResponseInterface` | Passed through directly |
| `null` or empty string | Returns a **204 No Content** response |
| Scalar (`string`, `int`, ...) | Returns a **200** response with the value as body |
| Other (array, object, ...) | JSON-encoded with a `Content-Type: application/json` header. If encoding fails, a **500** error is thrown |

> 🆕 **Info**: *Since version 3.1*
>
> If `json_encode()` fails on a non-scalar return value, an `InternalServerErrorHttpException` is thrown instead of
> returning a silent empty response.

## Parameters

Parameters of the controllers methods are automatically inject by the
class [`Instantiator`](../getting-started/service-container.md)
of [service container](../getting-started/service-container.md).

Attributes of routes are available with `ServerRequest` parameter, with method `getAttributes()`.

## Helper methods

The `AbstractController` class provides several helper methods through traits.

### Response helpers

From `ResponseHelperTrait`:

- `response(mixed $body = null, int $statusCode = 200, array $headers = []): ResponseInterface`

  > Create a response. If the body is empty and status is 200, it will be automatically changed to 204 (No Content).

- `jsonResponse(mixed $body = null, int $flags = 0, int $statusCode = 200, array $headers = []): ResponseInterface`

  > Create a JSON response. The body is encoded with `json_encode()`. A `Content-Type: application/json` header is
  > automatically added.

- `redirect(UriInterface|string $uri, int $httpResponseCode = 302, ?ResponseInterface $response = null): ResponseInterface`

  > Redirect to a given URI. If a response object is provided, it will be completed with the Location header.

### Router helpers

From `RouterHelperTrait`:

- `getRouter(): RouterInterface`

  > Get the router service.

- `getRoute(): ?RouteInterface`

  > Get the current matched route.

- `path(string|RouteInterface $name, array|RouteAttributes $parameters = []): UriInterface`

  > Generate a path for a named route.

- `finalize_path(string $path): string`

  > Finalize a path (apply router prefix, etc.).

### Reload helper

From `ReloadHelperTrait`:

- `reload(array $queryParams = [], bool $mergeQueryParams = false, ?ResponseInterface $response = null): ResponseInterface`

  > Reload the current page. Deprecated: use `redirect()` instead.

### Flash messages

- `addFlash(string $type, string $message): static`

  > Add a flash message to the [flash bag](../components/flash-bag.md).

### Service access

- `get(string $id): mixed`

  > Get a service from the [service container](../getting-started/service-container.md).

### Templating

From `TwigAwareTrait` (when **berlioz/twig-package** is installed):

- `render(string $name, array $variables = []): string`

  > Render a [Twig template](../guides/templating.md).
