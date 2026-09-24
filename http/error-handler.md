---
breadcrumb:
  - HTTP Website
  - Error handler
keywords:
  - exception
  - error
---

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
