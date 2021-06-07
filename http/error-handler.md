```index
breadcrumb: HTTP Website; Error handler
keywords: exception, error
```

# Error handler

You can personalize errors pages with an error handler. Also, you can define an error handler by status code.

## Error handler

Error handler is a class whose implements `\Berlioz\Http\Core\Http\Handler\Error\ErrorHandlerInterface` interface.

```php
use Berlioz\Http\Core\Http\Handler\Error\ErrorHandlerInterface;
use Berlioz\Http\Message\Response;

class MyHttpErrorHandler implements ErrorHandlerInterface
{
    /**
     * @inheritDoc
     */
    public function handle(ServerRequestInterface $request, ?Throwable $throwable = null): ResponseInterface
    {
        // ...

        return new Response(statusCode: 500);
    }
}
```

If you want use the template rendering engine or access to core functionalities, you need to
extend `\Berlioz\Http\Core\Controller\AbstractController` class.

```php
use Berlioz\Http\Core\Controller\AbstractController;
use Berlioz\Http\Core\Http\Handler\Error\ErrorHandlerInterface;

class MyHttpErrorHandler extends AbstractController implements ErrorHandlerInterface
{
    /**
     * @inheritDoc
     */
    public function handle(ServerRequestInterface $request, ?Throwable $throwable = null): ResponseInterface
    {
        // ...

        return $this->response($this->render('error.html.twig'));
    }
}
```

## Configuration

Your error handler must be declared in your configuration file like this:

```json
{
  "berlioz": {
    "http": {
      "errors": {
        "default": "App\\Http\\MyHttpErrorHandler"
      }
    }
  }
}
```

The key `default`, it's for all errors attempted if no other handler is declared. So you can declare others handlers for
specific http errors, like `500` errors:

```json
{
  "berlioz": {
    "http": {
      "errors": {
        "default": "App\\Http\\MyHttpErrorHandler",
        "500": "App\\Http\\InternalHttpErrorHandler"
      }
    }
  }
}
```