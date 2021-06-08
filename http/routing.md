```index
breadcrumb: HTTP Website; Routing
```

# Routing

Routes redirect HTTP requests to the good controller and method of him.

The default router of Berlioz Framework is the package [**
berlioz/router**](https://github.com/BerliozFramework/Router) (version ^2.0).

## Basic declaration

You can add PHP attributes to the methods of your controllers, like this:

```php
use Berlioz\Http\Core\Attribute as Berlioz;

class MyController
{
    #[Berlioz\Route('/my-route')]
    public function methodName()
    {
        // ...
    }
}
```

Accepted arguments on routes attributes:

- `path`: path of route *(type: string|null)*
- `defaults`: an array of default values of attributes *(type: array)*
- `requirements`: an array of attributes requirements *(type: array)*
- `name`: name of route *(type: string|null)*
- `method`: a method or array of HTTP methods accepted by the route *(type: string|array|null)*
- `host`: a host or an array of hosts accepted by the route *(type: string|array|null)*
- `priority`: priority of the route *(type: int)*
- `options`: options for application *(type: array)*

Example with some options:

```php
use Berlioz\Http\Core\Attribute as Berlioz;

class MyController
{
    #[Berlioz\Route('/my-route/{attr}', name: 'myRoute', defaults: ['attr' => 'foo'])]
    public function methodName()
    {
        // ...
    }
}
```

## Route with attributes

You can add some dynamic attributes in your routes, attributes are named and must be encapsulated by `{` and `}`.

Attributes are transmitted in a `ServerRequest` object of **PSR-7** in argument of controller method. And accessible
with `getAttribute()` method.

```php
use Berlioz\Http\Core\Attribute as Berlioz;
use Berlioz\Http\Message\ServerRequest;

class MyController
{
    #[Berlioz\Route('/my-route/{attr}')]
    public function methodName(ServerRequest $request)
    {
        $request->getAttribute('attr'); // Value of 'attr' attribute in the path

        // ...
    }
}
```

### Requirement mask

You can define a requirement mask for a specific attribute, it's very useful to limit internal errors if you search need
only `int` values for an attribute for example.

The `requirements` option accept only array object like value. The key represents the attribute name and value a regex
mask.

```php
use Berlioz\Http\Core\Attribute as Berlioz;

class MyController
{
    #[Berlioz\Route('/my-route/{attr}', requirements: ['attr' => '\\d+'])]
    public function methodName()
    {
        // ...
    }
}
```

### Priority

In some cases, you need to set priority between routes, because the global mask of 2 requests are concurrent, like this
routes:

```php
use Berlioz\Http\Core\Attribute as Berlioz;
use Berlioz\Http\Message\ServerRequest;

class MyController
{
    #[Berlioz\Route('/my-route/{attr}')]
    public function methodName(ServerRequest $request)
    {
        // ...
    }

    #[Berlioz\Route('/my-route/new')]
    public function methodName2()
    {
        // ...
    }
}
```

To define the priority to a route, add `priority` option to the route with `int` value, more the value is greater, more
the route will be priority (default value: **-1**). In our example, to do the second route priority:

```php
use Berlioz\Http\Core\Attribute as Berlioz;
use Berlioz\Http\Message\ServerRequest;

class MyController
{
    #[Berlioz\Route('/my-route/{attr}', priority: 0)]
    public function methodName(ServerRequest $request)
    {
        // ...
    }

    #[Berlioz\Route('/my-route/new', priority: 1)]
    public function methodName2()
    {
        // ...
    }
}
```

### Default values

You can define default values in option of the route, in case of you generate them without specify attribute.

The `defaults` option accept only array object like value. The key represents the attribute name and value the default
value.

```php
use Berlioz\Http\Core\Attribute as Berlioz;
use Berlioz\Http\Message\ServerRequest;

class MyController
{
    #[Berlioz\Route('/my-route/{attr}', defaults: ['attr' => 'new'])]
    public function methodName(ServerRequest $request)
    {
        // ...
    }
}
```

### Group of routes

For routes with same parameters, you can define a route group attribute to the controller.

Example:

```php
use Berlioz\Http\Core\Attribute as Berlioz;

#[Berlioz\RouteGroup('/root-path', requirements: ['id' => '\d+'])]
class MyController
{
    // ...
}
```

All the parameters will be merged with the parameters of the route, except the path which will be concatenated with the
path of the route.

## Declaration of routes in configuration

You can also declare routes in your configuration. To do, create a `routes.json` file in your configuration directory.

An example of `routes.json` file:

```json
{
  "routes": [
    {
      "path": "/my-route",
      "context": [
        "My\\Project\\Controller\\MyController",
        "myMethod"
      ]
    },
    {
      "path": "/my-route/{attribute}",
      "requirements": {
        "attribute": "\\d+"
      },
      "priority": 0,
      "context": [
        "My\\Project\\Controller\\MyController",
        "mySecondMethod"
      ]
    }
  ]
}
```