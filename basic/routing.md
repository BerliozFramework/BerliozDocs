```index
breadcrumb: Basic uses; Routing
summary-order: 6
description: Manage the routes (urls) of your project with annotations or directly into configuration file
```

# Routing

The default router of Berlioz Framework is the package [**berlioz/router**](https://github.com/BerliozFramework/Router).

## Basic declaration

You can add annotation to the methods of your controllers, like this:

```php
/**
 * @route("/my-route")
 */
public function methodName()
{
    // ...
}
```

The first argument unnamed of `@route` annotation must be the path. If named, call him **path** :

```php
/**
 * @route(path="/my-route")
 */
```

## Route with attribute

You can add some dynamic attributes in your routes, attributes are named and must be encapsulated by `{` and `}`.

Attributes are transmitted in a `ServerRequest` object of **PSR-7** in argument of controller method. And accessible with `getAttribute()` method.

```php
/**
 * @route("/my-route/{attribute}")
 */
public function methodName(ServerRequest $request)
{
    $request->getAttribute('attribute');
    // ...
}
```

## Options

You can add options to the `@route` annotation like the requirement mask for attributes, the priority of routes, default values...
Options must be separated by a comma.

### Attribute requirement mask

You can define a requirement mask for a specific attribute, it's very useful to limit internal errors if you search need only `int` values for an attribute for example.

The `requirements` option accept only JSON object like value. The key represents the attribute name and value a regex mask.

```php
/**
 * @route("/my-route/{attribute}", requirements={"attribute":"\\d+"})
 */
```

## Priority

In some cases, you need to set priority between routes, because the global mask of 2 requests are concurrent, like this routes:

```php
/**
 * @route("/my-route/{attribute}")
 */
public function methodName(ServerRequest $request)
{
    // ...
}

/**
 * @route("/my-route/new")
 */
public function methodName2()
{
    // ...
}
```

To define the priority to a route, add `priority` option to the route with `int` value, more the value is greater, more the route will be priority (default value: **-1**).
In our example, to do the second route priority:

```php
/**
 * @route("/my-route/{attribute}", priority=0)
 */
public function methodName(ServerRequest $request)
{
    // ...
}

/**
 * @route("/my-route/new", priority=1)
 */
public function methodName2()
{
    // ...
}
```

## Default values

You can define default values in option of the route, in case of you generate them without specify attribute.

The `defaults` option accept only JSON object like value. The key represents the attribute name and value the default value.

```php
/**
 * @route("/my-route/{attribute}", defaults={"attribute":"new"})
 */
```

## Declaration of routes in configuration

You can also declare routes in your configuration. To do, create a `routes.json` file in your configuration directory.

An example of `routes.json` file:

```json
{
  "routes": [
    {
      "path": "/my-route",
      "context": {
        "_class": "My\\Project\\Controller\\MyController",
        "_method": "myMethod"
      }
    },
    {
      "path": "/my-route/{attribute}",
      "options": {
        "requirements": {
          "attribute": "\\d+"
        },
        "priority": 0
      },
      "context": {
        "_class": "My\\Project\\Controller\\MyController",
        "_method": "mySecondMethod"
      }
    }
  ]
}
```