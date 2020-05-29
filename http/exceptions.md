```index
breadcrumb: HTTP Website; Exceptions
keywords: exception
```

# HTTP exceptions

`HttpException` class allow you to throw specific exception from context.
You can so define an HTTP status code and message.

By default, if an exception is throw, the framework generate an `InternalServerErrorHttpException` exception with the main exception in previous.
If you throw an `HttpException`, the framework do not generate this exception and use yours, and the defined status code and message. 

## Main exception

`\Berlioz\HttpCore\Exception\HttpException` is the main exception used by the framework to generate errors to the user.

## Children HTTP exceptions

For the most common HTTP exceptions, some class are already define in the namespace `\Berlioz\HttpCore\Exception\Http`:

- `BadRequestHttpException`: 400 Bad Request
- `ConflictHttpException`: 409 Conflict
- `ForbiddenHttpException`: 403 Forbidden
- `InternalServerErrorHttpException`: 500 Internal Server Error
- `NotFoundHttpException`: 404 Not Found
- `NotImplementedHttpException`: 501 Not Implemented
- `ServiceUnavailableHttpException`: 503 Service Unavailable
- `UnauthorizedHttpException`: 401 Unauthorized