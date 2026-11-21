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

## Upgrading from version 3.3 to 3.4

- **Hector ORM:** `berlioz/hector-package` now requires `hectororm/hectororm` **^1.5** so that PDO options are
  actually forwarded to the connection. Remove any project-level pin below 1.5 and update the Composer lock file
  as part of the upgrade. See [PDO options](../guides/orm/hector.md#pdo-options).

- **API forms:** Constructing `ApiDataProvider` without `strictJson: true` is deprecated. Pass
  `strictJson: true` and check that your forms accept nested JSON objects as `stdClass`. See
  [Strict JSON](../guides/forms/data-providers.md#strict-json).
- **API forms:** `ApiDataProvider` now also handles `PATCH` and `DELETE` requests.
- **Twig preloads:** Every file rendered by `entrypoints()` with preload enabled now receives a preload hint.
  The obsolete `h2pushes` cookie cache is removed and the `Link` header is sent on every response. Remove the deprecated
  `nopush` parameter; see [Preload](../guides/templating/twig-extensions.md#preload).
- **Custom Twig extensions:** If you override `DefaultExtension::filterDateFormat()`, its first parameter now accepts
  `DateTimeInterface|string|int` instead of `DateTime|string|int`. Overrides of
  `RouterRuntimeExtension::functionPathExists()` must accept `array|RouteAttributes` for the parameters argument.
  Update narrower overrides to match or widen these types. Subclasses overriding
  `Berlioz\Package\Twig\Twig::__debugInfo()` must now declare the `array` return type.

- **SMTP transport:** The `options` argument of `Smtp` is deprecated, use the `timeout` argument instead. Consider
  enabling encryption with the new `encryption` argument. See
  [SMTP connection modes](../components/mailer.md#smtp-connection-modes).

- **Forms:** Inputs now reject arrays and objects submitted as their value and no longer hydrate them into the mapped
  object. If an input intentionally receives several values, use a `Collection`, a multiple `Choice` or a `RawType`
  instead; the deprecated `scalar: false` option restores the previous behavior until version 4. See
  [Non-scalar values](../guides/forms/inputs.md#non-scalar-values).

- **Service declarations:** Declaring services in the `container.services` configuration key is deprecated. Move
  them to a service provider listed in `container.providers`. See
  [Migrating from `container.services`](service-container.md#migrating-from-containerservices).

- **Front controller:** Call `$app->terminate()` after `$app->print($response)` in your `public/index.php`, to run
  terminate listeners and [deferred queues](../guides/queues.md#deferred-queues). See
  [After the response](../http/terminate.md#front-controller).
- **Connection closed after the response:** With PHP-FPM or LiteSpeed, `HttpApp::print()` now closes the connection
  with the client once the response is printed: anything printed afterwards is no longer sent. Set
  `berlioz.http.finish_request` to `false` to keep the previous behavior.
- **Queue workers:** `Worker` installs its signal handlers when `run()` starts, no longer in its constructor. See
  [Graceful Shutdown](../components/queue-manager.md#graceful-shutdown).
- **Debug console:** An unknown snapshot now returns a `404` error, and the toolbar shows the response time instead of
  the total time. See [Debug](debug.md#response-time).

- **Custom queues:** `QueueInterface` has two new methods, `getMaxAttempts()` and `getRetryTime()`. Custom
  implementations must implement them. See
  [Attempts and retry time](../components/queue-manager.md#attempts-and-retry-time).
- **Custom queue jobs:** `JobInterface` has two new methods, `getMaxAttempts()` and `isLastAttempt()`. Custom
  implementations not extending `Berlioz\QueueManager\Job\Job` must implement them. See
  [Last attempt](../components/queue-manager.md#last-attempt).
- **Redis queues:** failed jobs are no longer retried indefinitely: after `maxAttempts` (5 by default), they are moved
  to the `<name>:dead` list. Set `maxAttempts` on `RedisQueue` if you need more attempts. Consumed jobs are now made
  available again if neither deleted nor released within `retryTime` (30 seconds by default): set a `retryTime` greater
  than your longest job.
- **AMQP queues:** messages redelivered by the broker after a consumer crash now count as attempts and end in the
  dead-letter queue after `maxAttempts`.
- **SQS attempt metadata:** Reading the maximum attempts from the redrive policy requires `sqs:GetQueueAttributes`.
  Grant that permission, or configure `max_attempts` explicitly to match the actual SQS redrive policy.
  See [Attempts and retry time](../components/queue-manager.md#attempts-and-retry-time).
- **Retry safety:** Make handlers idempotent and keep job processing time below the retry/visibility timeout;
  duplicate or concurrent processing is possible. See
  [Delivery guarantees and idempotency](../components/queue-manager.md#delivery-guarantees-and-idempotency).

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

> **Note:** The `psr/http-message` upgrade from v1 to v2 introduces return types on interface methods. If you have
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

- Packages methods signature changed

#### Breaking changes for HttpCore

- Namespace `Berlioz\HttpCore` moved to `Berlioz\Http\Core`
- Namespace `Berlioz\Router` moved to `Berlioz\Http\Router`
- Magic methods `_b_pre()` and `_b_post()` of controllers are removed in favor of [middlewares](../http/middleware.md)
- Remove usage of PhpDoc annotations in favor of [PHP 8 attributes](https://www.php.net/manual/language.attributes.php),
  like [routes annotations](../http/routing.md)
- Method `AbstractController::getService()` renamed to `AbstractController::get()`
- `ErrorHandler`

#### Breaking changes for CliCore

- Namespace `Berlioz\CliCore` moved to `Berlioz\Cli\Core`
- Signature of `CommandInterface` changed

### Steps to upgrade

1. Update versions into your composer.json
2. Run command `composer update`
3. Create a directory "resources" at root project directory
4. Move directories "assets" and "templates" into new "resources" directory
5. Update your webpack config if necessary
6. Fixes breaking changes
7. Update files "public/index.php"
   like [WebsiteSkeleton](https://github.com/BerliozFramework/WebsiteSkeleton/tree/2.x/public) (do not forget .htaccess
   file)
