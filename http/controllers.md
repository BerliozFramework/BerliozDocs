```index
breadcrumb: HTTP Website; Controllers
keywords: controller
summary-order: 2; 1
```

# Controllers

It's classes whose control interactions between models, services and templates.
The controller is instanced, and the matched method is called, by router service when the application started.

## Class

`\Berlioz\Http\Core\Controller\AbstractController`

It's the main controller for website projects who offer methods for services, routing, templating, flash messages, and redirection.

## Example

```php
use Berlioz\Http\Core\Controller\AbstractController;
use Berlioz\Http\Message\Response;
use Berlioz\Http\Message\ServerRequest;

class MyController extends AbstractController
{
    /**
     * Method description.
     *
     * @param ServerRequest $request
     * @param Response $response
     *
     * @return Response $response
     */
    #Route['/my-route/{attr1}']
    public function myMethod(ServerRequest $request): Response
    {
        // Do something
        $attribute = $request->getAttribute('attr1');

        return $response;
    }
}
```

## Parameters

Parameters of the controllers methods are automatically inject by the class [`Instantiator`](../getting-started/service-container.md) of [service container](../getting-started/service-container.md).

Attributes of routes are available with `ServerRequest` parameter, with method `getAttributes()`.