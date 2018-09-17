<meta name="docparser-index" content="Basics usage; Routing" />
<meta name="docparser-index-order" content="5" />

# Routing

The default router of Berlioz Framework is the package [**berlioz/router**](https://github.com/BerliozFramework/Router).

Declaration of routes are done with annotations for the moment, we add in future the capacity to choice between configuration file and annotations.

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
 * @route("/my-route/{attribute}", requirements={"attribute":"\d+"})
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