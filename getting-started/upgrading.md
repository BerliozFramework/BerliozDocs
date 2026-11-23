---
breadcrumb:
  - Getting started
  - Upgrading
keywords:
  - installation
  - upgrade
  - migration
summary-order: 2
---

# Upgrading

This page lists what you must do to upgrade. To discover the new features of each version, see
[What's new?](../whats-new.md).

## Preparing for version 4

Version 3 keeps deprecated features working, so you can migrate at your own pace. Fix the deprecations below before
you upgrade to version 4: the features of the first table are announced for removal in that version.

Each feature of the first table triggers an `E_USER_DEPRECATED` notice when used. Look for these notices in your
logs or in the [debug console](debug.md) to find what remains to migrate.

| Deprecated                                               | Since | What to do                                                                                                                           |
|----------------------------------------------------------|-------|--------------------------------------------------------------------------------------------------------------------------------------|
| Services declared in the `container.services` key        | 3.4   | Declare them in a service provider: [Migrating from `container.services`](service-container.md#migrating-from-containerservices)     |
| `ApiDataProvider` constructed without `strictJson: true` | 3.4   | Pass `strictJson: true`: [Strict JSON](../guides/forms/data-providers.md#strict-json)                                                |
| `scalar` option of form inputs                           | 3.4   | Remove the option: [Non-scalar values](../guides/forms/inputs.md#non-scalar-values)                                                  |
| `options` argument of the `Smtp` transport               | 3.4   | Use the `timeout` argument: [Transports](../components/mailer.md#transports)                                                         |
| `nopush` parameter of the Twig `preload()` function      | 3.4   | Remove the parameter: [Preload](../guides/templating/twig-extensions.md#preload)                                                     |
| Trusted forwarded prefix without request URI rewriting   | 3.3   | Set `berlioz.router.rewriteRequestUri` to `true`: [Reverse-proxy prefixes](../http/routing.md#deprecated-behavior-and-v4-transition) |

The features below are deprecated too, without an announced removal version. Stop using them as well:

| Deprecated                                                                            | Since      | What to do                                                                                                       |
|---------------------------------------------------------------------------------------|------------|------------------------------------------------------------------------------------------------------------------|
| `AssetRuntimeExtension::H2PUSH_CACHE_COOKIE` and `AssetRuntimeExtension::setCookie()` | 3.4        | Remove their usages: the framework no longer uses them                                                           |
| `QueueManager::stats()`                                                               | 3.1        | Iterate over `getQueues()`, or use `size()`: [Queue Monitoring](../components/queue-manager.md#queue-monitoring) |
| `reload()` controller helper                                                          | Before 3.0 | Use `redirect()`: [Controllers](../http/controllers.md)                                                          |
| `uuid` type of route attributes                                                       | Before 3.0 | Use the `uuid4` type: [Router](../components/router.md)                                                          |
| `Router::isValid()` called with a string                                              | Before 3.0 | Pass a `ServerRequestInterface` object                                                                           |
| `UploadedFile::parseUploadedFiles()` and `UploadedFile::setStream()`                  | Before 3.0 | Use `UploadedFileFactoryTrait::createUploadedFiles()` and the constructor                                        |
| `Session::createFromHar()`, `createFromHarFile()`, `getHar()` and `writeHar()`        | Before 3.0 | Stop using them: the HTTP client no longer uses them                                                             |
| `Berlioz\Mailer\Transport\Mail` transport                                             | Before 3.0 | Use `Berlioz\Mailer\Transport\PhpMail`                                                                           |

---

## Upgrading from version 3.3 to 3.4

Start with the required actions. Then read the behavior changes for the features you use, and plan the deprecations
before version 4. The last part only concerns you if you extend framework classes or interfaces.

### Required actions

- **Front controller:** Call `$app->terminate()` after `$app->print($response)` in your `public/index.php`. Without
  this call, terminate listeners and [deferred queues](../guides/queues.md#deferred-queues) never run. See
  [After the response](../http/terminate.md#front-controller).
- **Custom CLI entry script:** The `berlioz` binary terminates the application itself. If you start `CliApp` from
  your own script, call `$app->terminate()` after `$app->handle()`. See [CLI](../cli.md).
- **Hector ORM:** `berlioz/hector-package` now requires `hectororm/hectororm` **^1.5**, so that PDO options are
  actually forwarded to the connection. Remove any project-level pin below 1.5 and update the Composer lock file
  as part of the upgrade. See [PDO options](../guides/orm/hector.md#pdo-options).

### Behavior changes

#### HTTP and debug

- **Connection closed after the response:** With PHP-FPM or LiteSpeed, `HttpApp::print()` now closes the connection
  with the client once the response is printed: anything printed afterwards is no longer sent. Set
  `berlioz.http.finish_request` to `false` to keep the previous behavior. See
  [After the response](../http/terminate.md#configuration).
- **HTTP QUERY method:** Routes declared without `method` now also accept `QUERY` requests
  ([RFC 10008](https://www.rfc-editor.org/info/rfc10008/)). If such a controller reads the request body, it now
  receives `QUERY` requests too: restrict `method` on routes that must not accept them, and reject a `QUERY` request
  without `Content-Type`, or use the `acceptQuery` route option. See [QUERY method](../http/routing.md#query-method).
- **HTTP client redirections:** `QUERY` requests now keep their method and body on 301 and 302 redirections.
- **Malformed request bodies:** A malformed body of the incoming request, such as invalid JSON, now produces a `400`
  error instead of a `500`: the error handler configured for `400` renders it, and it is no longer written to the
  error log. `getParsedBody()` throws a `MalformedBodyException`, which extends `RuntimeException`, so existing
  `catch` blocks keep working. See [Malformed bodies](../components/http-message.md#malformed-bodies).
- **Header values:** Messages of `berlioz/http-message` now remove leading and trailing spaces and tabs from header
  values, and reject header names ending with a line feed.
- **Response validation:** `HttpApp::print()` validates the complete response before sending anything. A response
  that cannot be sent as a valid HTTP/1 message, for example with a control character in a header value, a status
  code outside 100-599 or a pseudo-header such as `:status`, throws an `InvalidResponseException` and nothing is sent.
  The exception is not handled by the error handler: it reaches your front controller, where PHP logs it and, with
  `display_errors` disabled, answers with a 500 error. See
  [HTTP/1 emission](../components/http-message.md#http1-emission).
- **Debug console:** An unknown snapshot now returns a `404` error. A snapshot still being written returns a `202`
  page that reloads itself, and a snapshot that was never completed returns a `410` page. The toolbar shows the
  response time instead of the total time. See [Debug](debug.md#response-time).

#### Forms

- **Non-scalar values:** Inputs now reject arrays and objects submitted as their value and no longer hydrate them into
  the mapped object. If an input intentionally receives several values, use a `Collection`, a multiple `Choice` or a
  `RawType` instead. See [Non-scalar values](../guides/forms/inputs.md#non-scalar-values).
- **API forms:** `ApiDataProvider` now also handles `PATCH` and `DELETE` requests.

#### Templates and mailer

- **Twig preloads:** Every file rendered by `entrypoints()` with preload enabled now receives a preload hint.
  The obsolete `h2pushes` cookie cache is removed and the `Link` header is sent on every response. See
  [Preload](../guides/templating/twig-extensions.md#preload).
- **SMTP timeout:** The `timeout` of the SMTP transport (10 seconds by default) now also applies to reads, not only to
  the connection. Increase it if your SMTP server needs more time to answer. See
  [Transports](../components/mailer.md#transports).

#### Queues

- **Retry safety:** Make handlers idempotent and keep job processing time below the retry/visibility timeout;
  duplicate or concurrent processing is possible. See
  [Delivery guarantees and idempotency](../components/queue-manager.md#delivery-guarantees-and-idempotency).
- **Redis queues:** Failed jobs are no longer retried indefinitely: after `maxAttempts` (5 by default), they are moved
  to the `<name>:dead` list. Set `maxAttempts` on `RedisQueue` if you need more attempts. Consumed jobs are now made
  available again if neither deleted nor released within `retryTime` (30 seconds by default): set a `retryTime` greater
  than your longest job. See [RedisQueue](../components/queue-manager.md#redisqueue).
- **AMQP queues:** Messages redelivered by the broker after a consumer crash now count as attempts and end in the
  dead-letter queue after `maxAttempts`.
- **SQS attempt metadata:** Reading the maximum attempts from the redrive policy requires `sqs:GetQueueAttributes`.
  Grant that permission, or configure `max_attempts` explicitly to match the actual SQS redrive policy.
  See [Attempts and retry time](../components/queue-manager.md#attempts-and-retry-time).
- **Queue workers:** `Worker` installs its signal handlers when `run()` starts, no longer in its constructor. See
  [Graceful Shutdown](../components/queue-manager.md#graceful-shutdown).
- **Queue factories:** `berlioz/queue-manager-package` now provides factories for Redis, AMQP, RabbitMQ and Null
  queues, registered by default. A factory you wrote for one of these queue classes keeps precedence over the built-in
  one: you can keep it, or remove it and use the built-in configuration. See
  [Redis queue](../guides/queues.md#redis-queue), [AMQP queue](../guides/queues.md#amqp-queue) and
  [Custom queue factories](../guides/queues.md#custom-queue-factories).

### Deprecations

These features still work in version 3.4. Unless stated otherwise, using them triggers an `E_USER_DEPRECATED` notice,
and they will be removed in version 4. All deprecations, including older ones, are listed in
[Preparing for version 4](#preparing-for-version-4).

- **Service declarations:** Declaring services in the `container.services` configuration key is deprecated. Move
  them to a service provider listed in `container.providers`. See
  [Migrating from `container.services`](service-container.md#migrating-from-containerservices).
- **API forms:** Constructing `ApiDataProvider` without `strictJson: true` is deprecated. Pass
  `strictJson: true` and check that your forms accept nested JSON objects as `stdClass`. See
  [Strict JSON](../guides/forms/data-providers.md#strict-json).
- **Form `scalar` option:** The `scalar` option of inputs is deprecated. `scalar: false` restores the behavior before
  3.4 until version 4; `scalar: true` is the default and can be removed. See
  [Non-scalar values](../guides/forms/inputs.md#non-scalar-values).
- **SMTP transport:** The `options` argument of `Smtp` is deprecated, use the `timeout` argument instead. Consider
  enabling encryption with the new `encryption` argument. See
  [SMTP connection modes](../components/mailer.md#smtp-connection-modes).
- **Twig preloads:** The `nopush` parameter of `preload()` is deprecated: remove it. The
  `AssetRuntimeExtension::H2PUSH_CACHE_COOKIE` constant and the `AssetRuntimeExtension::setCookie()` method are
  deprecated too; they are no longer used and trigger no notice. See
  [Preload](../guides/templating/twig-extensions.md#preload).

### If you extend the framework

- **Custom queues:** `QueueInterface` has two new methods, `getMaxAttempts()` and `getRetryTime()`. Custom
  implementations must implement them. See
  [Attempts and retry time](../components/queue-manager.md#attempts-and-retry-time).
- **Custom queue jobs:** `JobInterface` has two new methods, `getMaxAttempts()` and `isLastAttempt()`. Custom
  implementations not extending `Berlioz\QueueManager\Job\Job` must implement them. See
  [Last attempt](../components/queue-manager.md#last-attempt).
- **Custom Twig extensions:** If you override `DefaultExtension::filterDateFormat()`, its first parameter now accepts
  `DateTimeInterface|string|int` instead of `DateTime|string|int`. Overrides of
  `RouterRuntimeExtension::functionPathExists()` must accept `array|RouteAttributes` for the parameters argument.
  Update narrower overrides to match or widen these types. Subclasses overriding
  `Berlioz\Package\Twig\Twig::__debugInfo()` must now declare the `array` return type.

---

## Upgrading from version 3.2 to 3.3

- **Reverse proxies:** If your application uses `X-Forwarded-Prefix`, review your trusted proxy configuration
  and enable request URI rewriting. See [Reverse-proxy prefixes](../http/routing.md#reverse-proxy-prefixes).
- **Custom routers:** Review the [compatibility notes for custom routers](../components/router.md#notes-for-custom-routers).
- **Custom form elements:** Review the
  [compatibility notes for custom form elements](../guides/forms.md#notes-for-custom-form-elements).

---

## Upgrading from version 2

Version 3.0 marks the transition to a **monorepo** structure. All Berlioz components are now developed, tested and
released together from a single repository.

### What changed

#### Monorepo structure

All Berlioz packages are now maintained in a single repository:
[github.com/BerliozFramework/Berlioz](https://github.com/BerliozFramework/Berlioz).

- The individual component repositories (Config, Router, ServiceContainer, etc.) are now **read-only mirrors**
  automatically synchronized via subsplits.
- **Issues and pull requests** should be submitted to the main monorepo.
- Composer packages remain the same (`berlioz/config`, `berlioz/router`, etc.) and can still be required individually.

#### PHP 8.2 minimum

All components now require **PHP ^8.2** (previously ^8.0 or ^7.1 for some).

#### Updated PSR dependencies

| Package              | Version 2.x  | Version 3.x      |
|----------------------|---------------|-------------------|
| `psr/http-message`   | ^1.0          | ^2.0              |
| `psr/container`      | ^1.0 \|\| ^2.0 | ^2.0            |
| `psr/log`            | ^1.0 \|\| ^2.0 | ^2.0 \|\| ^3.0 |
| `psr/http-factory`   | ^1.0          | ^1.0 \|\| ^2.0   |
| `psr/simple-cache`   | ^1.0 \|\| ^2.0 | ^2.0 \|\| ^3.0 |

> ℹ️ **Note**: The `psr/http-message` upgrade from v1 to v2 introduces return types on interface methods. If you have
> custom implementations of PSR-7 interfaces, you will need to add the appropriate return types.

#### Removed package: Atlas ORM

The `berlioz/atlas-package` has been removed in version 3.0. If you were using Atlas ORM, you will need to migrate
to [Hector ORM](../guides/orm/hector.md) or manage your ORM integration manually.

#### No other API breaking changes

Beyond the dependency updates and Atlas removal above, version 3.0 does **not** introduce any breaking changes in the
Berlioz API itself. All namespaces, class names, method signatures and behaviors remain the same as in version 2.x.

### Steps to upgrade

1. **Update your `composer.json`** to require version `^3.0` for Berlioz packages, or require the monorepo directly:

   ```json
   {
       "require": {
           "berlioz/berlioz": "^3.0"
       }
   }
   ```

2. **Update PHP** to version 8.2 or higher if not already done.

3. **Run `composer update`** and fix any dependency conflicts.

4. **Check your PSR implementations**: if you have custom classes implementing PSR-7 (`MessageInterface`,
   `RequestInterface`, etc.), add the required return types introduced in `psr/http-message` v2.

5. **Test your application** thoroughly. No other code changes should be required.

---

## Upgrading from version 1

### Breaking changes

#### Breaking changes for Core

- The signature of package methods changed

#### Breaking changes for HttpCore

- Namespace `Berlioz\HttpCore` moved to `Berlioz\Http\Core`
- Magic methods `_b_pre()` and `_b_post()` of controllers are removed in favor of [middlewares](../http/middleware.md)
- PhpDoc annotations are removed in favor of [PHP 8 attributes](https://www.php.net/manual/language.attributes.php),
  like [route annotations](../http/routing.md)
- Method `AbstractController::getService()` renamed to `AbstractController::get()`
- Error handler classes moved to the `Berlioz\Http\Core\Http\Handler\Error` namespace: `HttpErrorHandler` became
  `ErrorHandler` and `DefaultHttpErrorHandler` became `DefaultErrorHandler`. See
  [Error handler](../http/error-handler.md)

#### Breaking changes for CliCore

- Namespace `Berlioz\CliCore` moved to `Berlioz\Cli\Core`
- Signature of `CommandInterface` changed

### Steps to upgrade

1. Update the versions in your composer.json
2. Run the command `composer update`
3. Create a "resources" directory at the root of the project directory
4. Move the "assets" and "templates" directories into the new "resources" directory
5. Update your webpack config if necessary
6. Fix the breaking changes
7. Update the file "public/index.php"
   like in [WebsiteSkeleton](https://github.com/BerliozFramework/WebsiteSkeleton/tree/2.x/public) (do not forget the
   .htaccess file)
