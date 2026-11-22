---
breadcrumb:
  - HTTP Website
  - Exceptions
keywords:
  - exception
  - error
---

# HTTP exceptions

The `HttpException` class allows you to throw a specific exception from context.
You can thus define an HTTP status code and message.

By default, if an exception is thrown, the framework generates an `InternalServerErrorHttpException` exception with the
main exception as its previous exception.
If you throw an `HttpException`, the framework does not generate this exception and uses yours, with the defined status
code and message.

The error page is rendered by the [error handler](error-handler.md).

## Main exception

`\Berlioz\Http\Core\Exception\HttpException` is the main exception used by the framework to generate errors for the
user.

For the most common HTTP exceptions, some classes are already defined in the namespace
`\Berlioz\Http\Core\Exception\Http`:

 Exception                           | Code | Reason
-------------------------------------|------|----------------------------------------------
 `BadRequestHttpException`           | 400  | Bad Request
 `ConflictHttpException`             | 409  | Conflict
 `ForbiddenHttpException`            | 403  | Forbidden
 `InternalServerErrorHttpException`  | 500  | Internal Server Error
 `NotFoundHttpException`             | 404  | Not Found
 `NotImplementedHttpException`       | 501  | Not Implemented
 `ServiceUnavailableHttpException`   | 503  | Service Unavailable
 `UnauthorizedHttpException`         | 401  | Unauthorized
 `UnsupportedMediaTypeHttpException` | 415  | Unsupported Media Type *(since version 3.4)*
