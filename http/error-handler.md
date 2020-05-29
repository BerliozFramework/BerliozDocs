```index
breadcrumb: HTTP Website; Error handler
keywords: exception
```

# Error handler

You can personalize errors pages with an error handler. Also, you can define an error handler by status code.

## Error handler

Error handler is a class whose implements `\Berlioz\HttpCore\Http\HttpErrorHandler` interface.

```php
class MyHttpErrorHandler implements HttpErrorHandler
{
    public function handle(?ServerRequestInterface $request, HttpException $e): ResponseInterface
    {
        // ...
    }
}
```

If you want use the template rendering engine or access to core functionalities, you need to extend `\Berlioz\HttpCore\Controller\AbstractController` class.

```php
class MyHttpErrorHandler extends AbstractController implements HttpErrorHandler
{
    public function handle(?ServerRequestInterface $request, HttpException $e): ResponseInterface
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

The key `default`, it's for all errors attempted if no other handler is declared.
So you can declare others handlers for specific http errors, like `500` errors:

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