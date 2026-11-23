---
breadcrumb:
  - HTTP Website
  - After the response
keywords:
  - terminate
  - deferred
  - fastcgi_finish_request
  - background
---

# After the response

> 🆕 **Info**: *Since version 3.4*

Some work does not need to delay the response: sending an email, calling a webhook, warming a cache... Berlioz lets you
run it **after the response has been sent to the client**, in the same PHP process.

## Front controller

The application must be terminated explicitly in your front controller (`public/index.php`), after the response has been
printed:

```php
use Berlioz\Http\Core\App\HttpApp;

require_once __DIR__ . '/../vendor/autoload.php';

$app = new HttpApp();
$response = $app->handle();
$app->print($response);
$app->terminate();
```

> ℹ️ **Note**: Projects created before version 3.4 must add the `$app->terminate()` call. Without it, nothing breaks, but
> terminate listeners and [deferred queues](../guides/queues.md#deferred-queues) are never executed.

## How it works

1. **`HttpApp::print()`** sends the headers and the body, then closes the connection with the client using
   `fastcgi_finish_request()` (PHP-FPM or a compatible SAPI) or `litespeed_finish_request()` (LiteSpeed) when available.
   The client gets the full response and does not wait for what follows.
   Before sending anything, it validates the complete response: an invalid status line or header throws an
   `InvalidResponseException` and nothing is sent
   (see [HTTP/1 emission](../components/http-message.md#http1-emission)).
2. **`HttpApp::terminate()`** releases the session lock (`session_write_close()`), then dispatches an
   `HttpTerminateEvent`. Its listeners run while the client is already done.

`terminate()` dispatches the event only once per application instance. Exceptions thrown by listeners are caught, added
to the [debug report](../getting-started/debug.md) and reported as PHP warnings (`E_USER_WARNING`). A custom PHP error
handler that converts warnings into exceptions can still interrupt termination. A failing listener can also prevent
later listeners from running, so handle expected failures within your own listener.

## Listen to the terminate event

`HttpTerminateEvent` extends `Berlioz\Core\Event\TerminateEvent`, also dispatched at the end of
[CLI commands](../cli.md) (`CliTerminateEvent`). Listen to the base class to run your work in both contexts, or to the
HTTP one to access the request:

```json
{
  "events": {
    "listeners": {
      "Berlioz\\Http\\Core\\Event\\HttpTerminateEvent": "App\\Listener\\AuditListener::onTerminate"
    }
  }
}
```

```php
namespace App\Listener;

use Berlioz\Http\Core\Event\HttpTerminateEvent;

class AuditListener
{
    public function __construct(private AuditService $audit)
    {
    }

    public function onTerminate(HttpTerminateEvent $event): void
    {
        $request = $event->getRequest();             // ServerRequestInterface|null
        $responseInfo = $event->getResponseInfo();   // status code, reason phrase, protocol version, headers

        $this->audit->log(
            $request?->getUri()->getPath(),
            $responseInfo['statusCode'] ?? null,
        );
    }
}
```

| Event | Dispatched by | Data |
|---|---|---|
| `Berlioz\Core\Event\TerminateEvent` | Any application | `getApp()` |
| `Berlioz\Http\Core\Event\HttpTerminateEvent` | `HttpApp::terminate()` | `getRequest()`, `getResponseInfo()` (the response body is not kept) |
| `Berlioz\Cli\Core\Event\CliTerminateEvent` | `CliApp::terminate()` | `getExitCode()` |

> 💡 **Tip**: To run jobs after the response, prefer the [deferred queues](../guides/queues.md#deferred-queues): your
> jobs keep working unchanged the day you move them to a persistent queue consumed by a worker.

## Configuration

The connection with the client is closed by default. You can disable it, for example if you print something after the
response:

```json
{
  "berlioz": {
    "http": {
      "finish_request": false
    }
  }
}
```

The terminate event is still dispatched, but the client waits until the end of the script.

## Caveats

- **Output after `print()` is lost**: once the connection is closed, anything printed is no longer sent to the client.
- **The PHP worker stays busy**: a PHP-FPM worker running terminate work cannot serve other requests. Keep this work
  short, and use a [worker](../cli/queues.md) for heavy or critical jobs.
- **PHP execution limits still apply**: `max_execution_time` can stop terminate work, but it is not a reliable
  wall-clock limit for blocking jobs. On Unix-like systems, time spent sleeping or waiting on streams and database
  operations is generally excluded. Configure timeouts on network clients and other blocking operations as well.
- **PHP-FPM timeout after the response**: `request_terminate_timeout` is **not** enforced after
  `fastcgi_finish_request()` by default. Set `request_terminate_timeout_track_finished = yes` in the FPM pool to
  enforce it during after-response work too, together with a suitable `request_terminate_timeout`.
  See the [PHP-FPM configuration reference](https://www.php.net/manual/en/install.fpm.configuration.php).
- **Deferred drain limit**: `berlioz.queues.deferred.time_limit` is checked between jobs. It prevents starting another
  job after the limit, but never interrupts an already-started handler or a blocked queue operation. It does not
  replace per-operation timeouts or a process-level limit.
- **Errors are invisible to the client**: log them, the response has already been sent.
- **No guarantee of execution**: if the process is killed, the work is lost (no retry, unlike a persistent queue).
- **Session written before terminate**: changes made to `$_SESSION` in terminate listeners are not saved.
- **Unsupported SAPIs**: with Apache `mod_php` or the PHP built-in server, the connection cannot be closed early. The
  terminate work still runs, but the client waits for it.
