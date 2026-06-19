---
breadcrumb:
  - Components
  - HTTP Client
keywords:
  - http
  - client
  - psr-18
  - curl
  - cookies
  - session
---

# HTTP Client

> ℹ️ **Note**: The HTTP client is part of the **Berlioz** ecosystem but is available as a standalone package:
> [`berlioz/http-client`](https://github.com/BerliozFramework/HttpClient).
> You can find it on [Packagist](https://packagist.org/packages/berlioz/http-client).
> You can use it independently of the framework, in any PHP application.

**Berlioz HTTP Client** is a PHP library to request HTTP server with continuous navigation, including cookies,
sessions... Implements PSR-18 (HTTP Client), PSR-7 (HTTP message interfaces) and PSR-17 (HTTP Factories) standards.

## Requests

#### With RequestInterface

You can construct your own request object whose implements `RequestInterface` interface (PSR-7).

```php
use Berlioz\Http\Client\Client;
use Berlioz\Http\Message\Request;

/** @var \Psr\Http\Message\RequestInterface $request */
$request = new Request(...);

$client = new Client();
$response = $client->sendRequest($request);

print $response->getBody();
```

#### Get/Post/Patch/Put/Delete/Options/Head/Connect/Trace

Methods are available to do request with defined HTTP method:

- `Client::get(...)`
- `Client::post(...)`
- `Client::patch(...)`
- `Client::put(...)`
- `Client::delete(...)`
- `Client::options(...)`
- `Client::head(...)`
- `Client::connect(...)`
- `Client::trace(...)`

Example with `Client::get()`:

```php
use Berlioz\Http\Client\Client;

$client = new Client();
$response = $client->get('https://getberlioz.com');

print $response->getBody();
```

You also can pass HTTP method in argument to `Client::request(...)` method:

```php
use Berlioz\Http\Client\Client;

$client = new Client();
$response = $client->request('get', 'https://getberlioz.com');

print $response->getBody();
```

Each method accept an array of options with `$options` argument.
List of options:

- **baseUri** (string|null): Base of URI if not given in requests (default: null)
- **followLocation** (int|false): Maximum number of redirections to follow, or `false` to disable (default: 5)
- **sleepTime** (int): Minimum sleep time between requests in milliseconds (default: 0)
- **logFile** (string|null): Log file path for request/response logging (default: null)
- **exceptions** (bool): Throw exceptions on HTTP error status (4xx/5xx) (default: true)
- **retry** (int|false): Maximum number of retry attempts on network errors, or `false` to disable (default: 3)
- **retryTime** (int): Wait time between retries in milliseconds (default: 1000)
- **cookies** (null|false|CookiesManager): `null` to use default cookie manager; `false` to disable cookies; a `CookiesManager` instance to use
- **history** (int|float): Maximum number of history entries to retain, `INF` for unlimited (default: INF)
- **callback** (Closure|null): Callback after each successful request `fn(RequestInterface, ResponseInterface): void`
- **callbackException** (Closure|null): Callback on HTTP error instead of throwing `fn(HttpException, Options): ResponseInterface`
- **headers** (array): Default headers merged into every request
- **context** (HttpContext|null): SSL/TLS and proxy configuration

Options passed in argument replace default options of client.

## Session

The session is accessible with method `Client::getSession()`.

#### History

The browsing history is saved in the session. If you serialize the object `Session`, the history is preserve.

The method `Session::getHistory()` returns an `History` object:

```php
use Berlioz\Http\Client\Client;

$client = new Client();
$history = $client->getSession()->getHistory();
```

#### Cookies

A cookie manager is available to manage cookies of session and between requests. The manager is available
with `Session::getCookies()` method.

If you serialize the object `Session`, the cookies are preserves.

#### HAR file

HAR file of session is accessible with method `Session::getHar()`.

If you serialize the object `Session`, the HAR is preserved.

Refers to the documentation of **elgigi/har-parser** library: https://github.com/ElGigi/HarParser

## Adapters

#### Usage

Default adapter used by library is the `AutoAdapter`, which picks the best available transport automatically: it
prefers `CurlAdapter` when the CURL extension is installed, and falls back to the `StreamAdapter` otherwise.

> 🆕 **Info**: *Since version 3.2*
>
> The default adapter is now the `AutoAdapter`. In previous versions, the client used `CurlAdapter` directly when the
> CURL extension was installed, and `StreamAdapter` otherwise.

You can specify adapters to the client constructor, with argument `adapter`:

```php
use Berlioz\Http\Client\Client;
use Berlioz\Http\Client\Adapter;

$client = new Client(adapter: new Adapter\CurlAdapter(), adapter: new Adapter\StreamAdapter());
```

The first specified adapter is the default adapter.

If you want force an adapter for a request, you can pass is name in the request options:

```php
use Berlioz\Http\Client\Client;
use Berlioz\Http\Client\Adapter;

$client = new Client(adapter: new Adapter\CurlAdapter(), adapter: new Adapter\StreamAdapter());
$client->get('https://getberlioz.com', options: ['adapter' => 'stream']);
```

#### List

List of adapters:

- **auto**: `Berlioz\Http\Client\Adapter\AutoAdapter` *(since version 3.2)*
- **curl**: `Berlioz\Http\Client\Adapter\CurlAdapter`
- **stream**: `Berlioz\Http\Client\Adapter\StreamAdapter`
- **har**: `Berlioz\Http\Client\Adapter\HarAdapter`

#### AutoAdapter

> 🆕 **Info**: *Since version 3.2*

The `AutoAdapter` selects the best available transport automatically: `CurlAdapter` is preferred when the CURL
extension is loaded, otherwise the `StreamAdapter` is used as a fallback. It's the default adapter of the client.

```php
use Berlioz\Http\Client\Client;
use Berlioz\Http\Client\Adapter\AutoAdapter;

$client = new Client(adapter: new AutoAdapter());
$client->get('https://getberlioz.com'); // Uses curl if available, else stream
```

Because the resolved adapter reports its own name (`curl` or `stream`), forcing an adapter by name in the request
options keeps working as expected.

You can also pass pre-configured adapters that the `AutoAdapter` should use once resolved:

```php
use Berlioz\Http\Client\Adapter\AutoAdapter;
use Berlioz\Http\Client\Adapter\CurlAdapter;
use Berlioz\Http\Client\Adapter\StreamAdapter;

$adapter = new AutoAdapter(
    curl: new CurlAdapter([CURLOPT_TIMEOUT => 20]),
    stream: new StreamAdapter(timeout: 5),
);
```

#### HarAdapter

The `HarAdapter` is specially made to simulate a navigation, coming from a desktop browser for example.

It's very useful for test units. You only need to store your cleaned HAR file into your repository to launch tests with
simulated HTTP dialogs.

```php
use Berlioz\Http\Client\Client;
use Berlioz\Http\Client\Adapter\HarAdapter;
use ElGigi\HarParser\Parser;

// Create HAR object from library `elgigi/har-parser`
$har = (new Parser())->parse('/path/of/my/file.har', contentIsFile: true);

$client = new Client(adapter: new HarAdapter(har: $har));
$client->get('https://getberlioz.com'); // Get response from HAR object, without making an HTTP request
```

Har adapter accept an option `strict` (default: `false`) to force the way of navigation.

#### Create an adapter

You can create an adapter for your project.
You must implement the interface `Berlioz\Http\Client\Adapter\AdapterInterface`.
