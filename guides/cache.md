---
breadcrumb:
  - Guides
  - Cache
description: Configure and use the PSR-16 cache of Berlioz Framework, including the Redis, file, memory and fallback drivers
keywords:
  - cache
  - redis
  - psr-16
  - fallback
---

# Cache

**Berlioz Framework** ships with a [PSR-16 (Simple Cache)](https://www.php-fig.org/psr/psr-16/) cache used internally by
the kernel (compiled configuration, service container, routes…) and available to your application.

The cache is held by the `Core` object and managed by `Berlioz\Core\Cache\CacheManager`. It is provided to the kernel at
**bootstrap time**, through the second argument of the `Core` constructor.

```php
use Berlioz\Core\Core;
use Berlioz\Http\Core\App\HttpApp;

$app = new HttpApp(new Core(cache: $cache));
```

> The cache is an **infrastructure** service of the framework. It is configured by code at bootstrap, **not** through the
> configuration files, because the cache is needed *before* the configuration is loaded.

## Drivers

All drivers implement `Psr\SimpleCache\CacheInterface`.

| Driver                 | Class                                    | Description                                  |
|------------------------|------------------------------------------|----------------------------------------------|
| File _(default)_       | `Berlioz\Core\Cache\FileCacheDriver`     | Persists entries on the filesystem.          |
| Memory                 | `Berlioz\Core\Cache\MemoryCacheDriver`   | In-memory, non-persistent (lives one request).|
| Null                   | `Berlioz\Core\Cache\NullCacheDriver`     | No-op cache, stores nothing.                 |
| Redis                  | `Berlioz\Core\Cache\RedisCacheDriver`    | Distributed cache backed by Redis.           |
| Fallback               | `Berlioz\Core\Cache\FallbackCacheDriver` | Resilience decorator chaining other drivers. |

By default, when you pass `true` (or nothing) to the `Core` constructor, the `FileCacheDriver` is used. Passing `false`
uses the `MemoryCacheDriver`. Passing a `CacheInterface` instance uses it as-is.

### File driver

> 🆕 **Info**: *Since version 3.2*
>
> `FileCacheDriver` now also accepts a directory **path string**, in addition to a `DirectoriesInterface`. Both are
> fully supported.

```php
use Berlioz\Core\Cache\FileCacheDriver;

// From a directory path
$cache = new FileCacheDriver('/var/cache');

// From the framework directories (historical usage, still supported)
$cache = new FileCacheDriver($core->getDirectories());
```

### Redis driver

> 🆕 **Info**: *Since version 3.2*
>
> The `RedisCacheDriver` is a new PSR-16 cache driver backed by [phpredis](https://github.com/phpredis/phpredis)
> (`ext-redis`).

The `RedisCacheDriver` is well-suited for **multi-server** deployments and **containerized / ephemeral** environments,
where a shared and persistent cache is required across nodes. It takes an already-connected `Redis` instance and an
optional key **namespace** (used to prefix every key and to scope `clear()`).

```php
use Redis;
use Berlioz\Core\Cache\RedisCacheDriver;

$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
// $redis->auth('secret');
// $redis->select(1);

$cache = new RedisCacheDriver(
    redis: $redis,            // Connected Redis instance
    namespace: 'my-project',  // Key prefix (default: "berlioz")
);
```

**Key characteristics**:

- **Namespaced keys**: every key is prefixed with the namespace, so several applications can share the same Redis
  instance without collisions.
- **Native TTL**: expirations are handled by Redis itself (millisecond precision via `PSETEX`).
- **Scoped `clear()`**: only the keys of the configured namespace are removed (via `SCAN` + `DEL`); the rest of the
  Redis instance is never flushed.

> **Tip:** Use a dedicated namespace (and/or a dedicated Redis database) per application to keep `clear()` isolated.

### Fallback driver

> 🆕 **Info**: *Since version 3.2*
>
> The `FallbackCacheDriver` is a new resilience decorator.

The `FallbackCacheDriver` chains several drivers, ordered from the **most preferred** to the **fallback** one. If an
operation **fails** on a driver (for example, Redis becomes unreachable), it is retried on the next driver. A cache
**miss is not a failure** and does not trigger any fallback. Once a driver fails, it is short-circuited for the lifetime
of the instance.

```php
use Berlioz\Core\Cache\FallbackCacheDriver;
use Berlioz\Core\Cache\FileCacheDriver;
use Berlioz\Core\Cache\MemoryCacheDriver;
use Berlioz\Core\Cache\RedisCacheDriver;

$cache = new FallbackCacheDriver([
    new RedisCacheDriver($redis, 'my-project'),
    new FileCacheDriver('/var/cache'),
    new MemoryCacheDriver(),
]);
```

With this configuration, the application keeps working even if Redis goes down: it transparently falls back to the file
cache, then to the in-memory cache.

## Cache driver factory

> 🆕 **Info**: *Since version 3.2*
>
> The `CacheDriverFactory` builds a cache driver from an array of options, a JSON file or environment variables.

`Berlioz\Core\Cache\CacheDriverFactory` provides convenient ways to build a cache driver.

### From an array

```php
use Berlioz\Core\Cache\CacheDriverFactory;

$cache = CacheDriverFactory::create([
    'driver' => 'redis',       // redis | file | memory | null
    'host' => '127.0.0.1',     // redis only
    'port' => 6379,            // redis only
    'auth' => 'secret',        // redis only, optional
    'database' => 0,           // redis only, optional
    'timeout' => 0.0,          // redis only, optional
    'namespace' => 'my-project', // redis only, optional
]);

// File driver
$cache = CacheDriverFactory::create(['driver' => 'file', 'path' => '/var/cache']);
```

### From environment variables

`fromEnv()` reads the following variables (prefix `BERLIOZ_CACHE_` by default):

| Variable                      | Driver | Description                            |
|-------------------------------|--------|----------------------------------------|
| `BERLIOZ_CACHE_DRIVER`        | all    | `redis`, `file`, `memory` or `null`.   |
| `BERLIOZ_CACHE_REDIS_HOST`    | redis  | Redis host.                            |
| `BERLIOZ_CACHE_REDIS_PORT`    | redis  | Redis port.                            |
| `BERLIOZ_CACHE_REDIS_AUTH`    | redis  | Redis authentication.                  |
| `BERLIOZ_CACHE_REDIS_DB`      | redis  | Redis database index.                  |
| `BERLIOZ_CACHE_REDIS_TIMEOUT` | redis  | Connection timeout (seconds).          |
| `BERLIOZ_CACHE_NAMESPACE`     | redis  | Key namespace.                         |
| `BERLIOZ_CACHE_FILE_PATH`     | file   | Cache directory path.                  |

```php
use Berlioz\Core\Cache\CacheDriverFactory;

$cache = CacheDriverFactory::fromEnv();
```

### From a JSON file

The file must contain a JSON object matching the options of `create()`.

```json
{
  "driver": "redis",
  "host": "127.0.0.1",
  "port": 6379,
  "namespace": "my-project"
}
```

```php
use Berlioz\Core\Cache\CacheDriverFactory;

$cache = CacheDriverFactory::fromFile('/path/to/cache.json');
```

### Automatic detection

`auto()` builds a sensible driver from the environment (defaulting to the file cache, or to memory when no path is
available) and wraps any persistent driver in a `FallbackCacheDriver` with a `MemoryCacheDriver` fallback, so the cache
keeps working if the main driver becomes unavailable.

```php
use Berlioz\Core\Cache\CacheDriverFactory;
use Berlioz\Core\Core;

$app = new HttpApp(new Core(cache: CacheDriverFactory::auto()));
```

## Using the cache in your application

The cache is accessible from the `Core` object:

```php
/** @var \Psr\SimpleCache\CacheInterface $cache */
$cache = $this->getApp()->getCore()->getCache();

$cache->set('my-key', $value, 3600);
$value = $cache->get('my-key', 'default');
```

## Requirements

To use the `RedisCacheDriver`, the [phpredis](https://github.com/phpredis/phpredis) extension (`ext-redis`) must be
installed:

```bash
pecl install redis
```
