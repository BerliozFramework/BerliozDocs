```index
breadcrumb: Basic uses; Controllers
summary-order: 5
```

# Controllers

It's classes whose control interactions between models, services and templates.
The controller is instanced and the matched method is called, by router service when the application start.

## Class

`\Berlioz\HttpCore\Controller\AbstractController`

It's the main controller for website projects who offer methods for services, routing, templating, flash messages, and redirection.

## Magic methods

Two methods are reserved for system:

- `_b_pre()`: method called before execution of controller method
- `_b_post()`: method called after execution of controller method

## Example

```php
class MyController extends \Berlioz\HttpCore\Controller\AbstractController
{
    /**
     * Method description.
     *
     * @param \Berlioz\Http\Message\ServerRequest $request
     * @param \Berlioz\Http\Message\Response $response
     *
     * @return \Berlioz\Http\Message\Response $response
     * @route( "/my-route/{attr1}" )
     */
    public function myMethod(ServerRequest $request, Response $response): Response
    {
        // Do something
        $attribute = $request->getAttribute('attr1');

        return $response;
    }
}
```

## Parameters

Parameters of the controllers methods are automatically injected by the class [`Instantiator`](./service-container.md) of [service container](./service-container.md).

Attributes of routes are available with `ServerRequest` parameter, with method `getAttributes()`.