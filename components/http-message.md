---
breadcrumb:
  - Components
  - HTTP Message
keywords:
  - http
  - message
  - psr-7
  - psr-17
  - request
  - response
  - stream
---

# HTTP Message

> ℹ️ **Note**: The HTTP message library is part of the **Berlioz** ecosystem but is available as a standalone package:
> [`berlioz/http-message`](https://github.com/BerliozFramework/HttpMessage).
> You can find it on [Packagist](https://packagist.org/packages/berlioz/http-message).
> You can use it independently of the framework, in any PHP application.

**Berlioz HTTP Message** is a PHP library that implements the PSR-7 (HTTP message interfaces) and PSR-17
(HTTP Factories) standards.

## PSR references

See the **PSR** documentation:

- **PSR-7** (HTTP message interfaces): <https://www.php-fig.org/psr/psr-7/>
- **PSR-17** (HTTP Factories): <https://www.php-fig.org/psr/psr-17/>

## Factory

Only one factory class implements **PSR-17**:
`\Berlioz\Http\Message\HttpFactory`

To help you, the factory is split into several traits:

- `\Berlioz\Http\Message\Factory\RequestFactoryTrait`
- `\Berlioz\Http\Message\Factory\ResponseFactoryTrait`
- `\Berlioz\Http\Message\Factory\ServerRequestFactoryTrait`
- `\Berlioz\Http\Message\Factory\StreamFactoryTrait`
- `\Berlioz\Http\Message\Factory\UploadedFileFactoryTrait`
- `\Berlioz\Http\Message\Factory\UriFactoryTrait`

## Specialized Streams

Beyond the base `Stream` class, several specialized stream implementations are available:

- `Stream\MemoryStream`: In-memory stream backed by `php://memory`
- `Stream\FileStream`: Opens a file on disk as a stream
- `Stream\PhpInputStream`: Caches `php://input` (readable only once) for safe multiple reads
- `Stream\GzStream`: Gzip-compressed stream using the `zlib` extension
- `Stream\GzFileStream`: Opens a gzip-compressed file on disk
- `Stream\Base64Stream`: Transparent base64 encoding via PHP stream filters
- `Stream\AppendStream`: Concatenates multiple streams into a single read-only virtual stream
- `Stream\MultipartStream`: Builds MIME multipart bodies (e.g. for `multipart/form-data` requests)

Example with `MultipartStream`:

```php
use Berlioz\Http\Message\Stream\MultipartStream;

$multipart = new MultipartStream();
$multipart->addElements(['field1' => 'value1', 'field2' => 'value2']);
$multipart->addFile('file', '/path/to/file.pdf');

$contentType = 'multipart/form-data; boundary=' . $multipart->getBoundary();
```

## Body parsers

The `Message` class includes a body parser registry. Built-in parsers handle common content types:

- `application/json` via `JsonParser`
- `application/x-www-form-urlencoded` via `FormUrlEncodedParser`
- `multipart/form-data` via `FormDataParser`

You can register custom parsers for any MIME type:

```php
use Berlioz\Http\Message\Message;

Message::addBodyParser('application/xml', MyXmlParser::class);
```

The parser class must implement `Berlioz\Http\Message\Parser\ParserInterface`.

Content-type suffixes are handled automatically (e.g. `application/vnd.api+json` resolves to the `json` parser).

### Malformed bodies

> 🆕 **Info**: *Since version 3.4*

`getParsedBody()` throws a `Berlioz\Http\Message\Exception\MalformedBodyException` when the body cannot be used:
invalid JSON, or a parsed value that is neither an array, an object nor `null` (e.g. a JSON string `"text"`).
The exception extends `RuntimeException`, and `getHttpMessage()` returns the message whose body is malformed.

A custom parser should throw this exception when it cannot parse the contents; `getParsedBody()` attaches the message
to it. In an HTTP application, the framework answers with a 400 Bad Request when the body of the incoming **request**
is malformed (see [HTTP exceptions](../http/exceptions.md)). A malformed body of another message, such as a response
received from an upstream API, remains a server error (500).

## HTTP/1 emission

> 🆕 **Info**: *Since version 3.4*

`Berlioz\Http\Message\Validator\Http1ResponseValidator` checks that any PSR-7 response, from this library or another
one, can be sent as a valid HTTP/1 message. Validate the response before sending anything, so that an invalid response
is never sent halfway:

```php
use Berlioz\Http\Message\Exception\InvalidResponseException;
use Berlioz\Http\Message\Validator\Http1ResponseValidator;

try {
    $head = (new Http1ResponseValidator())->validate($response);
} catch (InvalidResponseException $exception) {
    // Nothing has been sent yet: log the error and send another response
    throw $exception;
}

header($head->statusLine);
foreach ($head->headers as $name => $values) {
    foreach ($values as $i => $value) {
        // Pass the status code, so PHP does not change it (e.g. with a "Location" header)
        header(sprintf('%s: %s', $name, $value), 0 === $i, $head->statusCode);
    }
}
```

The validator checks:

- **Protocol version:** a digit, optionally followed by a dot and a digit (`1.0`, `1.1`, `2`...)
- **Status code:** from 100 to 599
- **Reason phrase:** no control character, except the horizontal tab
- **Header names:** an RFC 9110 token; pseudo-headers such as `:status` are rejected
- **Header values:** no control character except the horizontal tab, and no leading or trailing space or tab;
  empty values are allowed

An invalid field throws a `Berlioz\Http\Message\Exception\InvalidResponseException`, which extends
`InvalidArgumentException`. The validator never corrects a value and never reads the body.

On success, `validate()` returns an `Http1ResponseHead` with three read-only properties:

- `statusLine`: the status line, e.g. `HTTP/1.1 200 OK`
- `statusCode`: the status code
- `headers`: the header values by name, in the order of the response; each value stays separate (e.g. one
  `Set-Cookie` per cookie)

Messages of this library still accept pseudo-headers in their header names, for example when an HTTP client session
is replayed from a HAR file. Only an HTTP/1 emission rejects them.

`HttpApp::print()` uses this validator before sending a response (see [After the response](../http/terminate.md)).

## Additional methods

Some methods are provided beyond the PSR-7/PSR-17 standards:

- `Request::HTTP_METHOD_*` constants — HTTP method names (`Request::HTTP_METHOD_GET`, `Request::HTTP_METHOD_POST`...),
  including `Request::HTTP_METHOD_QUERY` since version 3.4
- `ServerRequest::getServerParam(string $name, mixed $default = null): mixed` — single server parameter access
- `ServerRequest::getQueryParam(string $name, mixed $default = null): mixed` — single query parameter access
- `ServerRequest::isAjaxRequest(): bool` — detects AJAX requests
- `Uri::getQueryValue(string $name, mixed $default = null): mixed` — extract a single query parameter from the URI
- `Uri::withAddedQuery(string $query): static` — merge additional query parameters
- `Uri::withoutQuery(string $name): static` — remove a query parameter
- `UploadedFile::getHash(string $algo = 'sha1'): string` — compute a file content hash
- `UploadedFile::getMediaType(): ?string` — server-side MIME detection via `finfo`
