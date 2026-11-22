---
breadcrumb:
  - Components
  - Router
keywords:
  - router
  - route
  - routing
  - psr-7
  - http
---

# Router

> ℹ️ **Note**: The router is part of the **Berlioz** ecosystem but is available as a standalone package:
> [`berlioz/router`](https://github.com/BerliozFramework/Router).
> You can find it on [Packagist](https://packagist.org/packages/berlioz/router).
> You can use it independently of the framework, in any PHP application.

**Berlioz Router** is a PHP library to manage HTTP routes, respecting the PSR-7 (HTTP message interfaces) standard.

## Routes

### Create a route

You can create a simple route like this:

```php
use Berlioz\Router\Route;

$route = new Route('/path-of/my-route');
$route = new Route('/path-of/my-route/{attribute}/with-attribute');
```

The constructor arguments are:

- **defaults**: an associative array to set the default values of attributes when the route is generated
- **requirements**: an associative array to restrict the format of attributes. The key is the name of the attribute
  and the value is the validation regex
- **name**: name of the route
- **method**: an array of allowed HTTP methods, or just a method. When omitted, the route accepts `GET`, `HEAD`,
  `POST`, `OPTIONS`, `CONNECT`, `TRACE`, `PUT`, `PATCH`, `DELETE`, and `QUERY` (since version 3.4,
  [RFC 10008](https://www.rfc-editor.org/info/rfc10008/))
- **host**: an array of allowed hosts, or just a host
- **priority**: you can specify the priority for a route (default: -1)

### Route group

A route can be turned into a group: you only need to add another route to it.

```php
use Berlioz\Router\Route;

$route = new Route('/path');
$route->addRoute($route2 = new Route('/path2')); // Path will be: /path/path2
```

Child routes inherit parent route attributes, requirements, methods, host restrictions, options, and context.

> 🆕 **Info**: *Since version 3.1*
>
> Host restrictions declared on a parent route group are now inherited by child routes.

### Attributes

Routes accept optional attributes; you need to wrap the optional part in brackets.

```php
$route = new \Berlioz\Router\Route('/path[/optional-part/{with-attribute}]');
```

You can also define requirements directly in the path:

- Add a regular expression after the name of the attribute (separated by ":").
- Add a type name after the name of the attribute (separated by "::").

```php
$route = new \Berlioz\Router\Route('/path/{attributeName:\d+}');
$route = new \Berlioz\Router\Route('/path/{attributeName::int}');
```

Supported predefined types:

- `int` (equivalent of `\d+`)
- `float` (equivalent of `\d+(?:\.\d+)?`, so integer values are accepted too)
- `uuid4` (equivalent of `[0-9A-Fa-f]{8}\-[0-9A-Fa-f]{4}\-[0-9A-Fa-f]{4}\-[0-9A-Fa-f]{4}\-[0-9A-Fa-f]{12}`)
- `slug` (equivalent of `[a-z0-9]+(?:-[a-z0-9]+)*`)
- `md5` (equivalent of `[0-9a-fA-F]{32}`)
- `sha1` (equivalent of `[0-9a-fA-F]{40}`)
- `domain` (equivalent of `([a-zA-Z0-9]([a-zA-Z0-9\-]{0,61}[a-zA-Z0-9])?\.)+[a-zA-Z]{2,6}`)

> 🆕 **Info**: *Since version 3.1*
>
> Duplicate attribute names in the same route path now throw a `RoutingException` during route construction.

## Router

The router is the main feature of the package; it is defined by the `Router` class. It is able to find the
right `Route` object according to a `ServerRequestInterface` object (see PSR-7).

```php
use Berlioz\Http\Message\ServerRequest;
use Berlioz\Router\Route;
use Berlioz\Router\Router;

// Create server request or get them from another place in your app
$serverRequest = new ServerRequest(...);

// Create router
$router = new Router();
$router->addRoute(
    new Route('/path-of/my-route'),
    new Route('/path-of/my-route/{attribute}/with-attribute')
);

$route = $router->handle($serverRequest);
```

### Options

| Options            | Type           | Description                                                                                              |
|--------------------|----------------|----------------------------------------------------------------------------------------------------------|
| X-Forwarded-Prefix | boolean/string | Defaults to **false**; **true** to use the "X-Forwarded-Prefix" value, or a custom header name           |
| trustedProxies     | array          | IPs, CIDRs or helper aliases allowed to supply the prefix; defaults to an empty list (since version 3.3) |

### Generate path

You can generate a path with some parameters directly with the `Router` object.

```php
use Berlioz\Router\Exception\NotFoundException;
use Berlioz\Router\Router;

$router = new Router();
// ...add routes

try {
    $path = $router->generate('name-of-route', ['attribute1' => 'value']);
} catch (NotFoundException $exception) {
    // ... not found route
}
```

The method returns the path as a string, or throws an exception if it is not able to generate the path (for example,
when not all required parameters are given).

### Validate a path

You can validate a `ServerRequestInterface` to know if a path can be handled by a route.

```php
use Berlioz\Http\Message\ServerRequest;
use Berlioz\Router\Router;

$serverRequest = new ServerRequest(...);
$router = new Router();
// ...add routes

/** bool $valid Valid path ?*/
$valid = $router->isValid($serverRequest);
```

The method returns a `boolean` value.

### Finalize path

The `finalizePath()` method applies a reverse-proxy mount to an internal path.

> 🆕 **Info**: *Since version 3.3*

```php
use Berlioz\Router\Router;

$router = new Router([
    'X-Forwarded-Prefix' => true,
    'trustedProxies' => ['10.0.0.1'],
]);
$router->setServerParams([
    'REMOTE_ADDR' => '10.0.0.1',
    'HTTP_X_FORWARDED_PREFIX' => '/app',
]);
$path = $router->finalizePath('/users/1');
// Returns "/app/users/1". Absolute URLs with a scheme are returned unchanged.
```

This method is the universal path-rewriting hook for the framework — routes, assets, entry points, and preload links
all pass through it.

Only a trusted direct peer's header is used. An absent or empty trust list ignores the prefix. `true` selects the
standard header; a string selects a custom header. Prefer explicit IPs/CIDRs; `*` trusts every direct peer.

Internal paths always receive the mount, even when they start with the same segment:
`finalizePath('/app/articles')` with mount `/app` returns `/app/app/articles`.
This method does not infer whether its input is already a public path.

#### Request context

Resolution priority is:

1. Explicit server parameters passed as the second argument of `finalizePath()`.
2. Parameters set with `setServerParams()`.
3. `$_SERVER` when no request context is set.

An explicit empty array prevents fallback to globals. `setServerParams(null)` clears the context and restores
fallback. The context is excluded from router serialization and must be renewed by persistent request handlers.
HTTP Core does this on each `handle()` call; standalone callers should set it when processing each request.

`ForwardedPrefixResolver` centralizes trust and validation without retaining request data. The concrete router's
`getForwardedPrefixResolver()` exposes the resolver built from its effective options. Prefixes such as `/my%20app`
are accepted; malformed escapes, query/fragment delimiters, controls, backslashes, dot segments, header lists,
empty internal segments, encoded separators and encoded percent signs are ignored. Surrounding slashes are normalized.

See [HTTP request URI rewriting](../http/routing.md#request-uri-rewriting) for controller-visible paths and the
deprecation, since version 3.3, of handling trusted prefixes without rewriting the request URI.

### Notes for custom routers

> 🆕 **Info**: *Since version 3.3*

Version 3.3 adds these methods to the `RouterInterface` contract:

```php
public function finalizePath(string $path, ?array $serverParams = null): string;
public function setServerParams(?array $serverParams): void;
```

Custom implementations must provide both methods. Subclasses overriding `Router::finalizePath()` must add the
optional parameter and forward it to the parent. Existing one-argument calls remain valid; these contract changes
are intentional compatibility changes for custom extensions.

Neither prefix resolution nor `getForwardedPrefixResolver()` is required by `RouterInterface`.
HTTP Core's default resolver service comes from the concrete `Router`; applications replacing that service with
another implementation must wire a matching resolver for prefix handling and URI rewriting.

After upgrading, rebuild persisted router/configuration caches so the new trusted-proxy configuration takes effect.
The existing cached route options otherwise continue to apply.

## RouteAttributes interface

The `RouteAttributes` interface allows entities or value objects to provide route parameters for URL generation:

```php
use Berlioz\Router\RouteAttributes;

class Article implements RouteAttributes
{
    public function routeAttributes(): array
    {
        return ['id' => $this->id, 'slug' => $this->slug];
    }
}

// Usage in path generation:
$path = $router->generate('article_show', $article);
```

`RouteAttributes` objects can also be mixed inside parameter arrays:

```php
$path = $router->generate('article_page', ['page' => 1, $article]);
// The article's routeAttributes() are merged with explicit parameters
```

## Route context

Each route has a freeform `context` property (`mixed`) to store associated metadata (e.g. a controller reference):

```php
$route = new Route('/path', context: MyController::class . '::action');
$route->getContext(); // "MyController::action"
$route->setContext($newContext);
```

The context is preserved during serialization and inherited through route groups.
