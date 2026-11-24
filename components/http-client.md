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

**Berlioz HTTP Client** is a PHP library to request HTTP servers with continuous navigation, including cookies,
sessions... It implements the PSR-18 (HTTP Client), PSR-7 (HTTP message interfaces) and PSR-17 (HTTP Factories)
standards.

## Requests

### With RequestInterface

You can construct your own request object that implements the `RequestInterface` interface (PSR-7).

```php
use Berlioz\Http\Client\Client;
use Berlioz\Http\Message\Request;

/** @var \Psr\Http\Message\RequestInterface $request */
$request = new Request(...);

$client = new Client();
$response = $client->sendRequest($request);

print $response->getBody();
```

### Get/Post/Query/Patch/Put/Delete/Options/Head/Connect/Trace

Methods are available to send a request with a given HTTP method:

- `Client::get(...)`
- `Client::post(...)`
- `Client::query(...)` *(since version 3.4)*: HTTP `QUERY` method ([RFC 10008](https://www.rfc-editor.org/info/rfc10008/)),
  the query is sent in the body
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

You can also pass the HTTP method as an argument to the `Client::request(...)` method:

```php
use Berlioz\Http\Client\Client;

$client = new Client();
$response = $client->request('get', 'https://getberlioz.com');

print $response->getBody();
```

Each method accepts an array of options with the `$options` argument.
List of options:

- **baseUri** (string|null): Base URI, if not given in requests (default: null)
- **followLocation** (int|false): Maximum number of redirections to follow, or `false` to disable (default: 5)
- **redirectSensitiveHeaders** (string[]): Additional credential headers to remove on cross-origin redirects
  (default: `[]`; since version 3.3; see [Redirect security](#redirect-security))
- **sleepTime** (int): Minimum sleep time between requests in milliseconds (default: 0)
- **logFile** (string|null): Log file path for request/response logging (default: null)
- **exceptions** (bool): Throw exceptions on HTTP error status (4xx/5xx) (default: true)
- **retry** (int|false): Maximum number of retry attempts on network errors, or `false` to disable (default: 3)
- **retryTime** (int): Wait time between retries in milliseconds (default: 1000)
- **cookies** (null|false|CookiesManager): `null` to use the default cookie manager; `false` to disable automatic cookie
  sending and collection; a `CookiesManager` instance to use
- **history** (int|float): Maximum number of history entries to retain, `INF` for unlimited (default: INF)
- **callback** (Closure|null): Callback after each successful request `fn(RequestInterface, ResponseInterface): void`
- **callbackException** (Closure|null): Callback on HTTP error instead of throwing `fn(HttpException, Options): ResponseInterface`
- **headers** (array): Default headers added to requests, subject to the redirect credential policy
- **context** (HttpContext|array|null): SSL/TLS and proxy configuration; its `credentials_origins` entry lists the
  origins allowed to receive transport credentials on redirects (since version 3.4; see
  [Transport credentials](#transport-credentials))

Array options override client defaults; headers are merged by name and `redirectSensitiveHeaders` is additive.
Passing an `Options` object supplies a complete options object rather than merging it with client defaults.

## Proxy

> 🆕 **Info**: *Since version 3.4*

The `proxy` option of the HTTP context selects the proxy of the requests:

| Value             | Behavior                                                         |
|-------------------|------------------------------------------------------------------|
| `null` (default)  | Proxy of the environment variables, see below                    |
| `false`           | No proxy, even if environment variables define one               |
| string            | Given proxy, for example `http://user:password@proxy.local:3128` |

```php
use Berlioz\Http\Client\Client;

// Never use a proxy
$client = new Client(['context' => ['proxy' => false]]);

// Always use this proxy
$client = new Client(['context' => ['proxy' => 'http://proxy.local:3128']]);
```

Only `http` and `https` proxies are supported. A proxy without scheme is an `http` proxy.

### Environment variables

With the default value, the cURL and stream adapters read the same environment variables and make the same decision:

- `https_proxy` or `HTTPS_PROXY` for HTTPS requests;
- `http_proxy` for HTTP requests;
- `all_proxy` or `ALL_PROXY` when the previous ones are not defined;
- `no_proxy` or `NO_PROXY`: comma-separated list of hosts that do not use the proxy.

The uppercase `HTTP_PROXY` variable is never read: in a web context, a client can define it with a `Proxy` request
header ([httpoxy](https://httpoxy.org/)).

Entries of `no_proxy` can be:

- `*`, for every host;
- a host name, which also matches its sub-domains: `example.com`, `.example.com` or `*.example.com` match
  `example.com` and `www.example.com`;
- an IP address or a CIDR range: `127.0.0.1`, `::1`, `10.0.0.0/8`;
- any of them followed by a port: `example.com:8080`, `[::1]:8080`.

`no_proxy` only applies to the proxy of the environment, never to a proxy given in the `proxy` option.

> ⚠️ **Warning**: Before version 3.4, the default value was `false`: the cURL adapter used the environment variables,
> the stream adapter did not. `false` now disables the proxy in every adapter. A `CURLOPT_PROXY` option given to the
> cURL adapter still takes precedence over the environment variables.

## Redirect security

> 🆕 **Info**: *Since version 3.3*

The client follows responses with a `Location` header for status codes 201, 301, 302, 303, 307 and 308 by default.
Set `followLocation` to `false` to return the response without following it.

### Origins and credentials

Redirect destinations are resolved against the current request URI before comparing origins. Two HTTP URIs have
the same origin when their scheme, case-insensitive host and effective port match. An omitted port is equivalent
to 80 for HTTP or 443 for HTTPS. A different subdomain, port or scheme is a different origin.

On the first cross-origin redirect, the client removes these headers from the options used by the redirect chain:

- `Authorization`
- `Proxy-Authorization`
- Manually supplied `Cookie`
- Application-specific headers listed in `redirectSensitiveHeaders`

Header names are compared case-insensitively. Removed credentials stay removed for the rest of that call, including
network retries and redirects back to the initial origin. Client defaults and caller-supplied options are preserved;
a subsequent independent call uses its normal credentials again.

Credentials supplied in a redirect's `Location` URI are removed. Credentials inherited from the current URI may be
retained for a same-origin relative redirect, but are removed after any origin change. Redirect URI fragments are
also removed.

### Application-specific secrets

Declare custom authentication headers explicitly; their names cannot be inferred from their values:

```php
use Berlioz\Http\Client\Client;

$client = new Client([
    'headers' => [
        'Authorization' => 'Bearer example-token',
        'X-Api-Key' => 'example-api-key',
    ],
    'redirectSensitiveHeaders' => ['X-Api-Key'],
]);

$response = $client->get('https://api.example.test/resource', options: [
    'headers' => ['X-Access-Token' => 'example-access-token'],
    'redirectSensitiveHeaders' => ['X-Access-Token'],
]);
```

In this example, all three authentication headers are removed if the request is redirected to another origin.
The additional names in array options are merged with the client's list, normalized and deduplicated; an empty
list does not clear the inherited list. The three mandatory headers are always filtered, even with no custom list.
When passing a complete `Options` object, include all applicable custom sensitive headers in that object.

Headers configured through options or default-header setters are reapplied on same-origin redirects. Headers set
only on the original PSR-7 request are not automatically copied when a redirect request is rebuilt.

### Transport credentials

> 🆕 **Info**: *Since version 3.4*

Some credentials are not headers of the request: they are configured on the transport and sent by the adapter.

- The TLS client certificate of the `context` option: `ssl_local_cert`, `ssl_local_pk`, `ssl_local_cert_passphrase`
  and `ssl_local_passphrase`
- The credential options given to `CurlAdapter`: `CURLOPT_USERPWD`, `CURLOPT_XOAUTH2_BEARER`, `CURLOPT_SSLCERT`,
  `CURLOPT_COOKIE`...

These credentials are only used for the origin of the initial request. When a redirect leaves this origin, the next
requests of the call are sent without them, including a redirect back to the initial origin. Proxy settings and
proxy credentials are kept on every request.

To also send them to other origins, list these origins in the `credentials_origins` option of the context:

```php
use Berlioz\Http\Client\Client;

$client = new Client([
    'context' => [
        'ssl_local_cert' => '/path/to/client.pem',
        'ssl_local_pk' => '/path/to/client.key',
        'credentials_origins' => [
            'https://auth.example.test',
            'https://*.partner.example',
        ],
    ],
]);
```

An entry is an origin, written `scheme://host` or `scheme://host:port`, without path. The scheme and the port must
match those of the redirect target. A leading `*.` matches exactly one subdomain level: `https://*.partner.example`
matches `https://api.partner.example`, but neither `https://partner.example` nor `https://a.b.partner.example`.
An invalid entry, such as `*` or `https://*.com`, throws an `InvalidOriginException` when the request is sent.

This list only concerns transport credentials. The headers of the previous sections are always removed on a
cross-origin redirect.

### Referer and cookies

For each followed redirect, the client replaces any configured `Referer` with a value based on the previous URI:

| Redirect | Generated `Referer` |
| --- | --- |
| Same origin | Previous URI without user information or fragment; path and query are retained |
| Different origin | Previous origin only, without user information, path or query |
| HTTPS to HTTP | No header |

This follows `strict-origin-when-cross-origin` semantics. A suppressed default `Referer` is not reapplied on the
next iteration.

Managed cookies are selected again for each destination using the cookie manager's domain, path, expiration and
Secure rules. They are distinct from manually supplied `Cookie` headers. Setting `cookies` to `false` disables
automatic cookie sending and collection; a manually supplied header still follows the credential policy above.

### Request bodies

For 307 and 308, the client preserves the method, body and content type, including across origins. Other followed
statuses rebuild the request as GET without the original body. Content length is recalculated for each redirect.

> 🆕 **Info**: *Since version 3.4*, `QUERY` requests also keep their method, body and content type on 301 and 302,
> as required by RFC 10008; only 303 switches them to GET.

The credential policy filters headers and URI credentials; it does not inspect or redact request bodies. To handle
body replay decisions yourself, disable automatic redirects with `followLocation: false` in an `Options` object,
or `'followLocation' => false` in array options.

## Session

The session is accessible with the `Client::getSession()` method.

### History

The browsing history is saved in the session. If you serialize the `Session` object, the history is preserved.

The `Session::getHistory()` method returns a `History` object:

```php
use Berlioz\Http\Client\Client;

$client = new Client();
$history = $client->getSession()->getHistory();
```

### Cookies

A cookie manager is available to manage the cookies of the session and between requests. The manager is available
with the `Session::getCookies()` method.

Serializing a `Session` preserves its cookies.

#### Cookie scope

Cookies received without `Domain` are restricted to the exact response host (`Cookie::isHostOnly()`).
An explicit `Domain` must match that host or a parent domain, with a DNS label boundary. Cookies for
unrelated domains are rejected before storage, replacement or deletion; other valid cookies from the
same response are retained. IP addresses only match exactly.

Cookie paths respect segment boundaries: `Path=/admin` matches `/admin` and `/admin/users`, but not
`/administrator`. When `Path` is absent or invalid, it is derived from the request path: a response to
`/account/login` defaults to `/account`.

The optional PHP `intl` extension provides non-transitional IDNA normalization for internationalized
cookie domains. Without IDNA support, ASCII domains (including ASCII Punycode representations) and IP
addresses remain supported; Unicode domains are rejected. Malformed domains and hosts with trailing
dots are rejected.

#### Public suffix limitation

Public suffixes are **not checked**: a response from `tenant.github.io` could set `Domain=github.io`,
or a response from `example.co.uk` could set `Domain=co.uk`. This version does not include a PSL
validator, dependency or bundled database.

#### Session serialization

`Session` serializes an array of `Cookie` objects and rebuilds `CookiesManager` using its constructor,
preserving the host-only flag and the other cookie attributes.

The `CookiesManager` constructor accepts a snapshot already deduplicated by cookie name, domain and path.
It validates that every element is a `Cookie` and restores the array in linear time, preserving order and
object identity without updating cookies. Use `addCookie()` for insertions that require replacement checks.

Sessions saved in the old format containing a cookie manager discard their cookies on restoration,
because the original host-only scope and response provenance cannot be recovered reliably.
Applications using those saved sessions may need to authenticate again.

### HAR file

HAR exports omit the domain for host-only cookies and prefix explicit domain-cookie scopes with a dot.
On import, cookies are validated against their entry URI. Missing or undotted domain metadata is treated
conservatively as host-only for that entry host; this may narrow the scope of cookies in older or external HAR files.
Invalid cookie domains are ignored individually during import and replay, preserving other valid cookies.

The HAR file of the session is accessible with the `Session::getHar()` method.

If you serialize the `Session` object, the HAR is preserved.

Refer to the documentation of the **elgigi/har-parser** library: <https://github.com/ElGigi/HarParser>

## Adapters

### Usage

The default adapter used by the library is the `AutoAdapter`, which picks the best available transport automatically:
it prefers `CurlAdapter` when the CURL extension is installed, and falls back to the `StreamAdapter` otherwise.

> 🆕 **Info**: *Since version 3.2*
>
> The default adapter is now the `AutoAdapter`. In previous versions, the client used `CurlAdapter` directly when the
> CURL extension was installed, and `StreamAdapter` otherwise.

You can pass adapters to the client constructor, after the options:

```php
use Berlioz\Http\Client\Client;
use Berlioz\Http\Client\Adapter;

$client = new Client(null, new Adapter\CurlAdapter(), new Adapter\StreamAdapter());
```

The first specified adapter is the default adapter.

If you want to force an adapter for a request, you can pass its name in the request options:

```php
use Berlioz\Http\Client\Client;
use Berlioz\Http\Client\Adapter;

$client = new Client(null, new Adapter\CurlAdapter(), new Adapter\StreamAdapter());
$client->get('https://getberlioz.com', options: ['adapter' => 'stream']);
```

### List

List of adapters:

- **auto**: `Berlioz\Http\Client\Adapter\AutoAdapter` *(since version 3.2)*
- **curl**: `Berlioz\Http\Client\Adapter\CurlAdapter`
- **stream**: `Berlioz\Http\Client\Adapter\StreamAdapter`
- **har**: `Berlioz\Http\Client\Adapter\HarAdapter`

### AutoAdapter

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

### HarAdapter

The `HarAdapter` is specially made to simulate a navigation, coming from a desktop browser for example.

It's very useful for unit tests. You only need to store your cleaned HAR file in your repository to run tests with
simulated HTTP exchanges.

```php
use Berlioz\Http\Client\Client;
use Berlioz\Http\Client\Adapter\HarAdapter;
use ElGigi\HarParser\Parser;

// Create HAR object from library `elgigi/har-parser`
$har = (new Parser())->parse('/path/of/my/file.har', contentIsFile: true);

$client = new Client(adapter: new HarAdapter(har: $har));
$client->get('https://getberlioz.com'); // Get response from HAR object, without making an HTTP request
```

The HAR adapter accepts a `strict` option (default: `false`) to force the navigation order.

### Create an adapter

You can create an adapter for your project.
You must implement the interface `Berlioz\Http\Client\Adapter\AdapterInterface`.

> 🆕 **Info**: *Since version 3.4*
>
> If your adapter holds its own credentials for the origin (authentication, client certificate, cookies...),
> implement `Berlioz\Http\Client\Adapter\OriginCredentialsAwareInterface` instead. Its
> `withoutOriginCredentials()` method returns a copy of the adapter without these credentials, and must keep the proxy
> and transport settings. The client uses this copy when a redirect leaves the origins allowed to receive them
> (see [Transport credentials](#transport-credentials)). Other adapters only receive a context without the TLS client
> certificate.
