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

**Berlioz Router** is a PHP library to manage HTTP routes, respecting PSR-7 (HTTP message interfaces) standard.

## Routes

#### Create route

You can create simple route like this:

```php
use Berlioz\Router\Route;

$route = new Route('/path-of/my-route');
$route = new Route('/path-of/my-route/{attribute}/with-attribute');
```

Constructor arguments are:

- **defaults**: an associated array to set default values of attributes when route is generated
- **requirements**: an associated array to restrict the format of attributes. Key is the name of attribute and value is
  the validation regex
- **name**: name of route
- **method**: an array of allowed HTTP methods, or just a method. When omitted, the route accepts `GET`, `HEAD`,
  `POST`, `OPTIONS`, `CONNECT`, `TRACE`, `PUT`, `PATCH`, and `DELETE`
- **host**: an array of allowed hosts, or just a host
- **priority**: you can specify the priority for a route (default: -1)

#### Route group

A route can be transformed to a group, only associate another route to them.

```php
use Berlioz\Router\Route;

$route = new Route('/path');
$route->addRoute($route2 = new Route('/path2')); // Path will be: /path/path2
```

Children routes inherit parent route attributes, requirements, ...

#### Attributes

Route accept optional attributes, you need to wrap the optional part by brackets.

```php
$route = new \Berlioz\Router\Route('/path[/optional-part/{with-attribute}]');
```

You can also define requirements directly in the path :

- Add a regular expression after the name of attribute (separate by ":").
- Add a type name after the name of attribute (separate by "::").

```php
$route = new \Berlioz\Router\Route('/path/{attributeName:\d+}');
$route = new \Berlioz\Router\Route('/path/{attributeName::int}');
```

Supported defined types:

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

The router is the main functionality in the package, it is defined by `Router` class. He is able to find the
good `Route` object according to a `ServerRequestInterface` object (see PSR-7).

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

#### Options

| Options            | Type           | Description                                                                               |
|--------------------|----------------|-------------------------------------------------------------------------------------------|
| X-Forwarded-Prefix | boolean/string | Default to **false**, **true** to use "X-Forwarded-Prefix" value or custom name of header | 

#### Generate path

You can generate a path with some parameters directly with `Router` object.

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

The return of method, is the path in string format or thrown an exception if not able to generate path (not all required
parameters for example).

#### Valid path

You can be valid a `ServerRequestInterface` to known if a path can be treated by a route.

```php
use Berlioz\Http\Message\ServerRequest;
use Berlioz\Router\Router;

$serverRequest = new ServerRequest(...);
$router = new Router();
// ...add routes

/** bool $valid Valid path ?*/
$valid = $router->isValid($serverRequest);
```

The return of method is a `boolean` value.

#### Finalize path

The `finalizePath()` method applies reverse-proxy prefix rewriting to a path:

```php
$path = $router->finalizePath('/users/1');
// If X-Forwarded-Prefix is "/app", returns "/app/users/1"
// Absolute URLs (containing "://") are returned unchanged
```

This method is the universal path-rewriting hook for the framework — routes, assets, entry points, and preload links
all pass through it.

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
