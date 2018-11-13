<meta name="docparser-index" content="Basic uses; Controllers" />
<meta name="docparser-index-order" content="4" />

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
     * @route( "/my-route/{attr1}" )
     */
    public function myMethod(\Berlioz\Http\Message\ServerRequest $request, \Berlioz\Core\Http\Response $response)
    {
        // Do something
        $attribute = $request->getAttribute('attr1');

        return $response;
    }
}
```

## Parameters

Parameters given to the methods are injected automatically by the [`Instantiator`](./service-container.md) class of [service container](./service-container.md).

Attributes of routes are available with `ServerRequest` parameter, with method `getAttributes()`.