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

**Berlioz HTTP Message** is a PHP library that implements PSR-7 (HTTP message interfaces) and PSR-17 (HTTP Factories) standards.

## PSR references

Looks at **PSR** documentations:
- **PSR-7** (HTTP message interfaces): https://www.php-fig.org/psr/psr-7/
- **PSR-17** (HTTP Factories): https://www.php-fig.org/psr/psr-17/

## Factory

Only one factory class implements the **PSR-17**:
`\Berlioz\Http\Message\HttpFactory`

To help you, the factory is cut into some traits:

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

## Additional methods

Some methods are provided beyond the PSR-7/PSR-17 standards:

- `ServerRequest::getServerParam(string $name, mixed $default = null): mixed` — single server parameter access
- `ServerRequest::getQueryParam(string $name, mixed $default = null): mixed` — single query parameter access
- `ServerRequest::isAjaxRequest(): bool` — detects AJAX requests
- `Uri::getQueryValue(string $name, mixed $default = null): mixed` — extract a single query parameter from the URI
- `Uri::withAddedQuery(string $query): static` — merge additional query parameters
- `Uri::withoutQuery(string $name): static` — remove a query parameter
- `UploadedFile::getHash(string $algo = 'sha1'): string` — compute a file content hash
- `UploadedFile::getMediaType(): ?string` — server-side MIME detection via `finfo`
