```index
breadcrumb: HTTP Website; Redirections
keywords: redirection
```

# HTTP Redirections

Sometimes, you need to redirect an old route to a new route. You have two methods to redirect route to another route.

## With config

You can create a `redirections.json` file into your [configuration directory](../getting-started/directories.md) and add
a similar content:

```json
{
  "berlioz": {
    "http": {
      "redirections": {
        "^/old$": "/new",
        "^/old-route/(.*)": "/new-route/$1",
        "^/another-old-route/": "/new/"
      }
    }
  }
}
```

The key `berlioz.http.redirections` is an JSON object. The key of property it's a regular expression to find, and the
value is the replacement value.

In internal, Berlioz use `preg_replace()` PHP function. So you can use mask and replacement value with backreferences.

Examples for redirection with the config given in example above:

Original path              | Redirect path
---------------------------|---------------
/old                       | /new
/old/foo                   | ***No redirection***
/old-route/foo             | /new-route/foo
/old-route/foo/bar         | /new-route/foo/bar
/old-route/                | /new-route/
/another-old-route/        | /new/
/another-old-route/foo/bar | /new/foo/bar

The default redirection HTTP status code is `301 Moved Permanently`, so if you want specify another HTTP status code,
it's possible with this config example:

```json
{
  "berlioz": {
    "http": {
      "redirections": {
        "^/old$": {
          "url": "/new",
          "type": 302
        }
      }
    }
  }
}
```

In this example, the HTTP status code will be `302 Found`.

> **Warning**:
>
> With this method, controllers routes are priority from list of redirection.
> So if you have a conflict between a controller route and a defined route, the controller wins!

## With controller

You can create redirection with methods of controllers, to use `redirection()` helper, who redirect with `302 Found`HTTP
status.

```php
class MyController extends \Berlioz\Http\Core\Controller\AbstractController
{
    /**
     * @route( "/old-route" )
     */
    public function oldMethod(): Response
    {
        return $this->redirect('/new-route');
    }
}
```

If you can also generate the route from the router, like this:

```php
use Berlioz\Http\Core\Attribute as Berlioz;
use Berlioz\Http\Core\Controller\AbstractController;
use Berlioz\Http\Message\Response;

class MyController extends AbstractController
{
    #[Berlioz\Route('/old-route')]
    public function oldMethod(): Response
    {
        return $this->redirect($this->getRouter()->generate('foo-route'));
    }

    #[Berlioz\Route('/new-route', name: 'foo-route')]
    public function newMethod(): Response
    {
        // ...
    }
}
```

If you want to specify the HTTP response code, pass it in the second parameter of `redirect()` method:

```php
return $this->redirect('/new-route', 301);
```