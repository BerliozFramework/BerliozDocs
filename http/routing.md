---
breadcrumb:
  - HTTP Website
  - Routing
---

# Routing

Routes redirect HTTP requests to the good controller and method of him.

The default router of Berlioz Framework is the package [**berlioz/router**](../components/router.md).

## Reverse-proxy prefixes

> 🆕 **Info**: *Since version 3.3*

When a reverse proxy exposes an application under `/app` and removes that mount before forwarding the request,
configure the prefix header and the trusted direct proxy address:

```json
{
  "berlioz": {
    "proxies": {
      "trusted": ["10.0.0.1"]
    },
    "router": {
      "X-Forwarded-Prefix": true,
      "rewriteRequestUri": true
    }
  }
}
```

`X-Forwarded-Prefix` accepts `true` for the standard header, a custom header name, or `false` to disable handling.
The header is accepted only when `REMOTE_ADDR` matches the configured trust list. Use proxy IPs or CIDRs;
`*` trusts every direct peer. The proxy must overwrite client-supplied forwarding headers.

The router inherits `berlioz.proxies.trusted` unless `berlioz.router.trustedProxies` is explicitly set.
The router-specific value takes precedence, including an empty list, which trusts nobody.

### Request URI rewriting

With `rewriteRequestUri: true`, the controller's request URI and `HttpApp::getRequest()` expose the public path:
an internal `/articles?page=2` becomes `/app/articles?page=2`. URLs derived from that URI, including pagination
and self-redirects, consequently include the mount. Remove manual prefix additions when enabling this behavior.

HTTP Core appends `ForwardedPrefixMiddleware` after configured middlewares, closest to the controller:

- Route matching still uses the internal path.
- Preceding middlewares retain their original immutable request, including after downstream execution.
- A middleware returning early does not reach rewriting; the outer error handler also retains its original request.
- A rewritten request exposes the normalized mount through its `berlioz.forwarded_prefix` attribute.
- Passing that rewritten request through the middleware again does not add another prefix.

Internal paths may themselves start with the mount name. For example, internal route `/app/articles` behind mount
`/app` correctly produces public path `/app/app/articles`.

HTTP Core supplies each request's server parameters to the router before matching, even when rewriting is disabled.
Generated routes and Twig asset URLs therefore use the supplied PSR-7 context rather than unrelated `$_SERVER` values.
Custom request adapters must supply the direct peer address and forwarded header server parameters.

### Deprecated behavior and v4 transition

Starting with version 3.3, `rewriteRequestUri` defaults to **false** throughout v3 to preserve the historical path.
However, handling a valid trusted prefix without rewriting the request URI is **deprecated**.

An `E_USER_DEPRECATED` notice is emitted when such a prefix is received and rewriting is disabled, including when
the option is omitted. The request still uses its internal path in v3. Set `rewriteRequestUri: true` to adopt the
target behavior. No notice is emitted for disabled prefix handling, an untrusted peer, or an absent/invalid prefix.

In v4, rewriting will be systematic when prefix handling is enabled and a valid prefix comes from a trusted peer;
the mode that leaves the controller's request path unprefixed will no longer be supported.

### Notes for existing applications

When upgrading to version 3.3, applications already using `X-Forwarded-Prefix` must configure trusted proxies:
the header is no longer accepted
without a trust list. Rebuild configuration and container/router caches after updating dependencies, including
`berlioz/helpers` >= 1.15. Old serialized routers lack the new trust options and ignore the prefix until rebuilt.

Custom router implementations and overrides must also account for the
[public contract changes](../components/router.md#notes-for-custom-routers).

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
path of the route. This includes `requirements`, `defaults`, `method`, and `host` restrictions.

> 🆕 **Info**: *Since version 3.1*
>
> Host restrictions declared on a `RouteGroup` are inherited by all child routes.

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
