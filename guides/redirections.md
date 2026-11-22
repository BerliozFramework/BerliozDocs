---
breadcrumb:
  - Guides
  - HTTP Redirections
keywords:
  - redirection
summary-order: ;6
---

# HTTP Redirections

Sometimes, you need to redirect an old route to a new route. You have two methods to redirect a route to another route.

## With config

You can create a `redirections.json` file in your [configuration directory](../getting-started/directories.md) and add
content similar to this:

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

The key `berlioz.http.redirections` is a JSON object. The key of each property is a regular expression to find, and the
value is the replacement value.

Internally, Berlioz uses the `preg_replace()` PHP function. So you can use a pattern and a replacement value with
backreferences.

Examples of redirections with the config given in the example above:

 Original path              | Redirect path        
----------------------------|----------------------
 /old                       | /new                 
 /old/foo                   | ***No redirection*** 
 /old-route/foo             | /new-route/foo       
 /old-route/foo/bar         | /new-route/foo/bar   
 /old-route/                | /new-route/          
 /another-old-route/        | /new/                
 /another-old-route/foo/bar | /new/foo/bar         

The default redirection HTTP status code is `301 Moved Permanently`. If you want to specify another HTTP status code,
you can do so as in this config example:

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

> ⚠️ **Warning**: With this method, controller routes take priority over the list of redirections.
> So if you have a conflict between a controller route and a defined route, the controller wins!

## With a controller

You can create a redirection in controller methods with the `redirection()` helper, which redirects with the
`302 Found` HTTP status.

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

You can also generate the route from the router, like this:

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

If you want to specify the HTTP response code, pass it as the second parameter of the `redirect()` method:

```php
return $this->redirect('/new-route', 301);
```
