---
breadcrumb:
  - HTTP Website
  - Error handler
keywords:
  - exception
  - error
---

# Error handler

You can customize error pages with an error handler. You can also define an error handler per status code.

To raise an HTTP error from your code, see [HTTP exceptions](exceptions.md).

## Error handler

An error handler is a class that implements the
`\Berlioz\Http\Core\Http\Handler\Error\ErrorHandlerInterface` interface.

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

If you want to use the template rendering engine or access core functionalities, you need to
extend the `\Berlioz\Http\Core\Controller\AbstractController` class.

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

## Error logging

> 🆕 **Info**: *Since version 3.3*

The framework error handler logs caught server exceptions to the configured PHP error log, even when debug
mode is disabled. This includes non-HTTP exceptions, HTTP exceptions with a status code of 500 or higher,
and failures in custom or default error handlers. HTTP exceptions below 500, such as a missing page (404),
are not logged as server errors.

Logging uses PHP's `error_log()` function and its configured destination; enabling the debug toolbar is not required.

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

The `default` key is used for all errors if no other handler is declared. So you can declare other handlers for
specific HTTP errors, like `500` errors:

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
