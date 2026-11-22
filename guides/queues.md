---
breadcrumb:
  - Guides
  - Queues
summary-order: ;8
keywords:
  - queue
  - worker
  - job
  - async
---

# Queues

**Berlioz Framework** integrates with the [Queue Manager](../components/queue-manager.md) component via the
**berlioz/queue-manager-package**. This package provides:

- Configuration-based queue setup (Memory, Database, AWS SQS, Redis, AMQP, RabbitMQ, Null)
- Automatic service container registration (`QueueManager`, `JobHandlerManager`, `Worker`)
- Built-in job handlers for running Berlioz commands (and, opt-in, system commands) asynchronously
- [CLI commands](../cli/queues.md) for running workers, purging queues, and monitoring queue metrics

## Installation

```bash
composer require berlioz/queue-manager-package
```

The package is auto-discovered by the framework. No additional registration is needed.

For more details on package installation, refer to the [Packages](packages.md) page.

## Configuration

Queue configuration goes under the `berlioz.queues` key. You can define it in your main `berlioz.json` or in a
dedicated `queues.json` file in your [configuration directory](../getting-started/directories.md).

Default configuration provided by the package:

```json
{
    "berlioz": {
        "queues": {
            "queues": [],
            "handlers": {
                "berlioz:command": "Berlioz\\Package\\QueueManager\\Handler\\BerliozCommandJobHandler"
            },
            "factories": [
                "Berlioz\\Package\\QueueManager\\Factory\\MemoryQueueFactory",
                "Berlioz\\Package\\QueueManager\\Factory\\AwsSqsQueueFactory",
                "Berlioz\\Package\\QueueManager\\Factory\\DbQueueFactory",
                "Berlioz\\Package\\QueueManager\\Factory\\RedisQueueFactory",
                "Berlioz\\Package\\QueueManager\\Factory\\AmqpQueueFactory",
                "Berlioz\\Package\\QueueManager\\Factory\\RabbitMqQueueFactory",
                "Berlioz\\Package\\QueueManager\\Factory\\NullQueueFactory"
            ],
            "deferred": {
                "queues": [],
                "time_limit": null
            }
        }
    }
}
```

- **`queues`**: Array of queue definitions, each with a `type` key matching a queue class
- **`handlers`**: Map of job name to handler class
- **`factories`**: Array of factory classes that know how to create queues from configuration
- **`deferred`**: Queues consumed after the response, see [Deferred queues](#deferred-queues)

### Database queue

```json
{
    "berlioz": {
        "queues": {
            "queues": [
                {
                    "type": "Berlioz\\QueueManager\\Queue\\DbQueue",
                    "db": {
                        "dsn": "mysql:dbname=myapp;host=127.0.0.1;port=3306;charset=UTF8",
                        "username": "myuser",
                        "password": "mypassword",
                        "table_name": "queue_jobs"
                    },
                    "retry_time": 30,
                    "max_attempts": 5,
                    "name": ["emails", "notifications"]
                }
            ]
        }
    }
}
```

The `name` key accepts a list of queue names. All queues in the same definition share the same database connection and
settings. See the [Queue Manager component documentation](../components/queue-manager.md#dbqueue) for the required MySQL
schema.

> ⚠️ **Warning**: This configuration file contains database credentials. Add it to your `.gitignore` if stored in a
> separate file.

### AWS SQS queue

> 🆕 **Info**: *Since version 3.1*
>
> You can optionally configure a `cloudwatch_client` to expose `waitTime()` monitoring metrics for SQS queues.

```json
{
    "berlioz": {
        "queues": {
            "queues": [
                {
                    "type": "Berlioz\\QueueManager\\Queue\\AwsSqsQueue",
                    "client": {
                        "region": "eu-west-1",
                        "version": "latest"
                    },
                    "cloudwatch_client": {
                        "region": "eu-west-1",
                        "version": "latest"
                    },
                    "retry_time": 30,
                    "max_attempts": 5,
                    "name": {
                        "emails": "https://sqs.eu-west-1.amazonaws.com/123456789/emails",
                        "notifications": "https://sqs.eu-west-1.amazonaws.com/123456789/notifications"
                    }
                }
            ]
        }
    }
}
```

The `client` key is passed directly to the `SqsClient` constructor. The optional `cloudwatch_client` key is passed to a
`CloudWatchClient` instance used for monitoring metrics. The `name` key is a map of queue name to SQS URL.

Each queue entry must resolve to a valid SQS URL. If a queue is misconfigured without a URL, the factory now throws a
clear configuration exception.

> 🆕 **Info**: *Since version 3.4*
>
> The optional `max_attempts` key (global or per queue) gives the maximum attempts of a job, returned by
> `JobInterface::getMaxAttempts()`. It should match the `maxReceiveCount` of the SQS redrive policy; if not given, it
> is read from the redrive policy when needed. Retries themselves are still managed by SQS.

Automatic discovery requires `sqs:GetQueueAttributes` permission on the queue. An explicit `max_attempts` avoids
that API lookup, but does not modify the SQS redrive policy. Keep it aligned with the actual `maxReceiveCount`.

### Memory queue

Useful for testing or short-lived tasks:

```json
{
    "berlioz": {
        "queues": {
            "queues": [
                {
                    "type": "Berlioz\\QueueManager\\Queue\\MemoryQueue",
                    "retry_time": 30,
                    "max_attempts": 5,
                    "name": ["default"]
                }
            ]
        }
    }
}
```

### Redis queue

> 🆕 **Info**: *Since version 3.4*

Requires the `ext-redis` PHP extension.

```json
{
    "berlioz": {
        "queues": {
            "queues": [
                {
                    "type": "Berlioz\\QueueManager\\Queue\\RedisQueue",
                    "redis": {
                        "host": "127.0.0.1",
                        "port": 6379,
                        "timeout": 0,
                        "password": null,
                        "database": 0
                    },
                    "deleted_jobs_ttl": 86400,
                    "retry_time": 30,
                    "max_attempts": 5,
                    "name": ["emails", "notifications"]
                }
            ]
        }
    }
}
```

All queues in the same definition share the same Redis connection. `password` accepts a string or a
`["username", "password"]` array (Redis ACL). `deleted_jobs_ttl`, `retry_time` and `max_attempts` can be given globally
or per queue; `deleted_jobs_ttl` is the time, in seconds, during which deleted jobs are kept.

### AMQP queue

> 🆕 **Info**: *Since version 3.4*

Requires the `ext-amqp` PHP extension. The connection is configured with a DSN, with explicit keys, or both:

```json
{
    "berlioz": {
        "queues": {
            "queues": [
                {
                    "type": "Berlioz\\QueueManager\\Queue\\AmqpQueue",
                    "amqp": {
                        "dsn": "amqp://guest:guest@127.0.0.1:5672/%2F"
                    },
                    "max_attempts": 5,
                    "name": ["emails", "notifications"]
                }
            ]
        }
    }
}
```

```json
{
    "amqp": {
        "host": "127.0.0.1",
        "port": 5672,
        "vhost": "/",
        "login": "guest",
        "password": "guest",
        "heartbeat": 30
    }
}
```

- The DSN accepts the `amqp://` and `amqps://` schemes (default port `5672` and `5671`). The vhost, login and password
  must be URL-encoded (`%2F` for the `/` vhost); an empty path means the `/` vhost. Query parameters are not supported.
- Explicit keys (`host`, `port`, `vhost`, `login`, `password`) override the values of the DSN.
- Any other key (`heartbeat`, `read_timeout`, `connect_timeout`, `cacert`, `cert`, `key`, `verify`…) is passed as is
  to `AMQPConnection`.
- Omitted keys take the value of the corresponding `amqp.*` INI setting of the extension (`amqp.login`,
  `amqp.vhost`, `amqp.cacert`…).

All queues in the same definition share the same connection. `max_attempts` can be given globally or per queue.

#### TLS connection

The `amqps://` scheme **requires** a verified TLS connection. The port alone does not enable encryption: the `ext-amqp`
extension only uses TLS when a CA certificate is given. Before connecting, the factory checks the effective settings
(options or `amqp.*` INI settings) and throws a `ConfigException`, without opening any connection, when:

- no CA certificate is given (`cacert` option or `amqp.cacert` INI setting);
- the verification is disabled (`verify` option or `amqp.verify` INI setting set to `false`);
- only one of the `cert` and `key` options is given.

Explicit `host` or `port` keys do not remove this requirement.

```json
{
    "amqp": {
        "dsn": "amqps://myuser:mypassword@rabbitmq.example.com:5671/my-vhost",
        "cacert": "/etc/ssl/rabbitmq/ca.pem",
        "verify": true
    }
}
```

- **`cacert`**: path to the CA certificate (PEM) that signed the broker certificate. No system CA path is assumed:
  give the CA explicitly, even for a broker certificate signed by a public authority (for example the CA bundle of
  your system).
- **`verify`**: verification of the broker certificate **and** hostname, enabled by default. Connect with the hostname
  present in the broker certificate. Never disable it in production: without verification, anyone able to intercept
  the connection can impersonate the broker and receive the credentials and the jobs.
- **`cert`** / **`key`**: optional client certificate and private key, for brokers requiring mutual TLS. They
  authenticate the client to the broker, and do not replace the CA certificate.

> ⚠️ **Warning**: With the `amqp://` scheme, no TLS is required: a missing CA certificate means a plain connection.
> Always use `amqps://` when the connection must be encrypted.

### RabbitMQ queue

> 🆕 **Info**: *Since version 3.4*

The RabbitMQ queue accepts the same configuration as the [AMQP queue](#amqp-queue), and adds the management API used
for monitoring metrics (`waitTime()`, `delayed()`…):

```json
{
    "berlioz": {
        "queues": {
            "queues": [
                {
                    "type": "Berlioz\\QueueManager\\Queue\\RabbitMqQueue",
                    "amqp": {
                        "dsn": "amqp://guest:guest@127.0.0.1:5672/%2F"
                    },
                    "management": {
                        "base_url": "http://127.0.0.1:15672",
                        "username": null,
                        "password": null
                    },
                    "name": ["emails", "notifications"]
                }
            ]
        }
    }
}
```

When `username` or `password` are not given, the login and password of the effective AMQP connection are used,
whether they come from the DSN, the explicit keys or the `amqp.login` / `amqp.password` INI settings. Explicit
`management.username` and `management.password` override them. Management API calls use the vhost of the effective
connection (including the `amqp.vhost` INI setting). The `amqps://` [TLS requirements](#tls-connection) also apply.

### Null queue

> 🆕 **Info**: *Since version 3.4*

A placeholder queue, always empty: pushing or consuming a job throws a `QueueException`. Useful to declare a queue name
that must not be used in an environment, or for tests:

```json
{
    "berlioz": {
        "queues": {
            "queues": [
                {
                    "type": "Berlioz\\QueueManager\\Queue\\NullQueue",
                    "name": ["emails"]
                }
            ]
        }
    }
}
```

### Per-queue overrides

Each queue name can be an object with its own `retry_time`, `max_attempts`, and `rate_limit` (and `deleted_jobs_ttl` for
Redis queues), when supported by the queue type:

```json
{
    "name": [
        {
            "name": "emails",
            "retry_time": 60,
            "max_attempts": 10,
            "rate_limit": "100/1m"
        },
        {
            "name": "notifications",
            "rate_limit": ["50/1m", "1000/1d"]
        }
    ]
}
```

Rate limits accept a string (`"100/1m"`) or an array of strings for multiple constraints. Supported units: `s`
(seconds), `m` (minutes), `h` (hours), `d` (days). You can use a multiplier: `"10/2hours"` means 10 per 2 hours.

## Dispatching jobs

The `QueueManager` service is automatically registered in the service container. You can inject it in your controllers
or services:

```php
use Berlioz\Http\Core\Controller\AbstractController;
use Berlioz\QueueManager\Job\JobDescriptor;
use Berlioz\QueueManager\QueueManager;
use Psr\Http\Message\ResponseInterface;

class OrderController extends AbstractController
{
    public function __construct(
        private QueueManager $queueManager,
    ) {
    }

    public function confirm(int $orderId): ResponseInterface
    {
        // Create and push a job
        $job = new JobDescriptor(
            name: 'order.confirmation',
            payload: ['order_id' => $orderId, 'email' => 'customer@example.com'],
        );
        $this->queueManager->push($job);

        // Push with a delay (process in 5 minutes)
        $this->queueManager->push($job, delay: 300);

        // Push to a specific queue by name
        $this->queueManager->push($job, queue: 'emails');

        return $this->response('Order confirmed');
    }
}
```

### Routing jobs to a specific queue

If a job should always go to a particular queue, implement the `JobForQueue` interface:

```php
use Berlioz\QueueManager\Job\JobDescriptor;
use Berlioz\QueueManager\Job\JobForQueue;
use Berlioz\QueueManager\Job\PayloadInterface;

class EmailJobDescriptor extends JobDescriptor implements JobForQueue
{
    public function __construct(PayloadInterface|array $payload)
    {
        parent::__construct('send_email', $payload);
    }

    public function forQueue(): string
    {
        return 'emails';
    }
}
```

When pushed through the `QueueManager`, this job will automatically be routed to the `emails` queue regardless of the
default.

## Deferred queues

> 🆕 **Info**: *Since version 3.4*

A deferred queue is consumed **in the same process, after the response has been sent to the client** (or at the end of
a CLI command), instead of by a [worker](../cli/queues.md). It suits short tasks that should not delay the response:
sending a confirmation email, calling a webhook, refreshing a cache...

Declare a memory queue and list it under `berlioz.queues.deferred.queues`:

```json
{
    "berlioz": {
        "queues": {
            "queues": [
                {
                    "type": "Berlioz\\QueueManager\\Queue\\MemoryQueue",
                    "name": "deferred",
                    "max_attempts": 1
                }
            ],
            "deferred": {
                "queues": ["deferred"],
                "time_limit": 20
            }
        }
    }
}
```

| Key | Description | Default |
|---|---|---|
| `queues` | Names of the queues consumed on terminate; empty disables the feature | `[]` |
| `time_limit` | Maximum time in seconds to consume the queues; a job already started is not interrupted | `null` (no limit) |

Then push jobs to this queue, as usual:

```php
$this->queueManager->push(
    new JobDescriptor('order.confirmation', ['order_id' => $orderId]),
    queue: 'deferred',
);
```

The jobs are processed by the same [job handlers](#creating-job-handlers) as any other queue. They require the
application to be terminated: see [After the response](../http/terminate.md) for the front controller. The `berlioz` CLI
binary terminates the application automatically after each command.

How the deferred queues are consumed:

- Jobs are processed until the queues are empty, including jobs pushed meanwhile, or until `time_limit` is reached.
- A failed job is released according to the queue `max_attempts` (retried immediately, without backoff). Prefer
  `"max_attempts": 1` for deferred memory queues.
- Memory queues are **purged** afterwards: remaining jobs (delayed, failed, or not consumed before the time limit) are
  lost. Persistent queues are left untouched.
- Errors never reach the client: failed jobs are logged if a `Psr\Log\LoggerInterface` service is available, and
  everything is visible in the [debug console](../getting-started/debug.md) (`Terminate` group of activities).
- In a CLI command, the deferred queues are consumed once, when the command ends. In a long-running command such as
  [`queue:worker`](../cli/queues.md), a job pushed to a deferred memory queue by a job handler stays in memory until
  the worker stops. From a job handler, push to a queue consumed by a worker instead.

> 💡 **Tip**: Jobs do not depend on how they are consumed. When they become too heavy or critical, move them to a
> persistent queue (database, Redis, SQS...) consumed by a worker. In your configuration, replace the queue definition
> with the `type` and the settings of the new backend (see [Configuration](#configuration)), and remove the queue from
> `berlioz.queues.deferred.queues`. No code change is needed.

> ⚠️ **Warning**: Jobs of a memory queue are lost if the process ends unexpectedly (timeout, memory limit, killed
> worker). Use a persistent queue for jobs that must not be lost. See also the
> [caveats of the work after the response](../http/terminate.md#caveats).

## Creating job handlers

A job handler implements `JobHandlerInterface` and processes a specific type of job:

```php
use Berlioz\QueueManager\Handler\JobHandlerInterface;
use Berlioz\QueueManager\Job\JobInterface;

class OrderConfirmationHandler implements JobHandlerInterface
{
    public function __construct(
        private MailerService $mailer,
        private OrderRepository $orders,
    ) {
    }

    public function handle(JobInterface $job): void
    {
        $payload = $job->getPayload();
        $order = $this->orders->find($payload->getOrFail('order_id'));
        $email = $payload->getOrFail('email');

        $this->mailer->sendOrderConfirmation($order, $email);
    }
}
```

Register the handler in your configuration:

```json
{
    "berlioz": {
        "queues": {
            "handlers": {
                "order.confirmation": "App\\Handler\\OrderConfirmationHandler"
            }
        }
    }
}
```

Handler classes are resolved from the service container, so dependency injection works automatically.

The `JobHandlerManager` supports wildcard matching: a handler registered as `order.*` will match job names like
`order.confirmation`, `order.shipped`, etc.

### Handling the last attempt

> 🆕 **Info**: *Since version 3.4*

When a handler throws, the worker releases the job for a new attempt, until the `max_attempts` of the queue is reached.
Use `JobInterface::isLastAttempt()` to behave differently when the job will not be retried:

```php
public function handle(JobInterface $job): void
{
    try {
        $this->mailer->sendOrderConfirmation($order, $email);
    } catch (MailerException $exception) {
        if ($job->isLastAttempt()) {
            $this->orders->flagConfirmationFailure($order);
            return;
        }

        throw $exception;
    }
}
```

`isLastAttempt()` returns `false` when the maximum number of attempts is unknown (`getMaxAttempts()` returns `null`).
See [Attempts and retry time](../components/queue-manager.md#attempts-and-retry-time) for details per queue type.

Returning normally after the fallback marks the job as handled; the worker deletes it. Also make handlers
[idempotent](../components/queue-manager.md#delivery-guarantees-and-idempotency): broker redeliveries or an expired
retry/visibility timeout can cause duplicate execution, including while an earlier handler is still running.

## Built-in job handlers

The package provides two job handlers out of the box.

### `berlioz:command`

Executes a Berlioz CLI command asynchronously. The payload must contain a `command` key with an array of arguments
(without the `berlioz` prefix):

```php
use Berlioz\QueueManager\Job\JobDescriptor;

// Run "vendor/bin/berlioz berlioz:cache-clear --all" asynchronously
$job = new JobDescriptor('berlioz:command', [
    'command' => ['berlioz:cache-clear', '--all'],
]);
$queueManager->push($job);
```

### `berlioz:system`

> 🆕 **Info**: *Since version 3.2*
>
> The `berlioz:system` handler is **no longer registered by default** and must be enabled explicitly. The command is
> now executed with `proc_open()` using an **array of arguments** (no shell): shell metacharacters such as `|`, `;`,
> `&&`, `$()` or `>` are treated as literal arguments and are never interpreted.

Executes a system command. The payload `command` key is an array of arguments passed directly to the binary (the first
element is the executable, the rest are its arguments):

```php
use Berlioz\QueueManager\Job\JobDescriptor;

// Run a system command asynchronously
$job = new JobDescriptor('berlioz:system', [
    'command' => ['php', '/path/to/script.php', '--option=value'],
]);
$queueManager->push($job);
```

Because it can run arbitrary executables on the worker host, this handler is **opt-in**. Register it in your
configuration only if you need it and you fully control the job payloads:

```json
{
    "berlioz": {
        "queues": {
            "handlers": {
                "berlioz:system": "Berlioz\\Package\\QueueManager\\Handler\\BerliozSystemJobHandler"
            }
        }
    }
}
```

> ⚠️ **Warning**: The `berlioz:system` handler executes arbitrary system commands. Ensure job payloads are never built
> from untrusted user input. Running it without a shell prevents metacharacter injection, but the executable and its
> arguments are still fully controlled by the payload.

## CLI commands

The package registers three CLI commands for managing queues. See the dedicated
[Queue CLI commands](../cli/queues.md) page for full usage details and production deployment tips.

- **`queue:worker`** — Start a worker to process queue jobs
- **`queue:purge`** — Purge all jobs from queues
- **`queue:size`** — Display queue monitoring metrics

## Monitoring and metrics

Queue metrics can be consumed in two ways: on demand through the CLI, or scraped over HTTP by a Prometheus server.
Depending on the backend, the following metrics are available:

- `size`: number of jobs ready to be consumed
- `waitTime`: age in seconds of the oldest consumable job
- `delayed`: number of delayed jobs waiting to become available

`waitTime` and `delayed` are only reported for backends implementing `MonitorableQueueInterface`. For such a queue they
are always exposed, defaulting to `0` when the queue is empty, so known metric series never disappear. Non-monitorable
queues omit these metrics entirely.

### CLI

> 🆕 **Info**: *Since version 3.1*

The `queue:size` command reports the metrics above for the selected queues:

```bash
$ vendor/bin/berlioz queue:size --format prometheus --total
```

See the [Queue CLI commands](../cli/queues.md#queuesize) page for output formats and Prometheus examples.

### HTTP endpoint

> 🆕 **Info**: *Since version 3.2*

The same metrics can be served over HTTP, so a Prometheus server can scrape them directly. The endpoint is **disabled by
default** and served by a middleware — no route is declared, and any existing application route on the same path always
takes precedence.

> ℹ️ **Note**: The endpoint requires **berlioz/http-core** (declared as a `suggest` dependency of the package). In
> CLI-only projects the HTTP wiring stays inert.

Configure it under the `berlioz.queues.metrics` key:

```json
{
    "berlioz": {
        "queues": {
            "metrics": {
                "enable": true,
                "path": "/metrics/queues",
                "format": "prometheus",
                "ip": ["127.0.0.1", "10.0.0.0/8"],
                "token": "{env: QUEUE_METRICS_TOKEN}",
                "prometheus_labels": {
                    "env": "production",
                    "host": "web1"
                },
                "total": true
            }
        }
    }
}
```

| Key | Description | Default |
|---|---|---|
| `enable` | Enable the HTTP endpoint | `false` |
| `path` | Path served by the middleware | `/metrics/queues` |
| `format` | Output format: `prometheus` or `json` | `prometheus` |
| `ip` | Client IP allow-list (exact IPs, CIDR, or hostnames); empty means no IP restriction | `[]` |
| `token` | Optional bearer token required in the `Authorization` header | `null` |
| `prometheus_labels` | Extra Prometheus labels (object of name/value) appended to every metric | `{}` |
| `total` | Append the aggregated `job_queue_length_total` metric | `false` |

Access control is **layered and cumulative**:

- **IP allow-list** (`ip`): when set, only these client IPs may reach the endpoint. The client IP is resolved exactly
  like the debug console — `X-Forwarded-For` is only trusted behind a configured trusted proxy
  (`berlioz.proxies.trusted`), so an allow-listed IP cannot be spoofed.
- **Bearer token** (`token`): when set, requests must send `Authorization: Bearer <token>` **in addition** to passing
  the IP check. Prefer resolving it from the environment (`{env: QUEUE_METRICS_TOKEN}`) to keep the secret out of your
  versioned configuration.

The endpoint is read-only: only `GET` (and its `HEAD` counterpart) requests are served. A `HEAD` request returns the
same status and `Content-Type` as `GET` with an empty body, and does not query the queue backends. Any unmet condition
(wrong HTTP method, disabled, IP not allowed, missing/invalid token) results in a `404`, without disclosing the
endpoint.

Example Prometheus scrape configuration:

```yaml
scrape_configs:
  - job_name: berlioz-queues
    metrics_path: /metrics/queues
    authorization:
      type: Bearer
      credentials: <your-token>   # only if a token is configured
    static_configs:
      - targets: ['app.example.com']
```

The exposed metrics are the same as those of the CLI; see the
[Queue CLI commands](../cli/queues.md#queuesize) page for output examples.

## Custom queue factories

The package includes factories for all the queues of the Queue Manager component (Memory, Database, AWS SQS, Redis,
AMQP, RabbitMQ and Null). To use your own queue implementation, you can write a custom factory.

A factory must implement the `QueueFactory` interface:

```php
use App\Queue\MyQueue;
use App\Queue\MyQueueClient;
use Berlioz\Package\QueueManager\Factory\QueueFactory;
use Berlioz\Package\QueueManager\Factory\QueueFactoryTrait;
use Generator;

class MyQueueFactory implements QueueFactory
{
    use QueueFactoryTrait;

    public static function getQueueClass(): string
    {
        return MyQueue::class;
    }

    public static function createFromConfig(array $config): Generator
    {
        $client = new MyQueueClient(
            host: $config['client']['host'] ?? '127.0.0.1',
            port: (int)($config['client']['port'] ?? 1234),
        );

        foreach ((array)($config['name'] ?? []) as $queue) {
            !is_array($queue) && $queue = ['name' => (string)$queue];

            yield new MyQueue(
                client: $client,
                name: $queue['name'] ?? 'default',
                limiter: self::getRateLimiterFromConfig($queue['rate_limit'] ?? null),
            );
        }
    }
}
```

Register it in your configuration:

```json
{
    "berlioz": {
        "queues": {
            "factories": [
                "App\\Queue\\MyQueueFactory"
            ],
            "queues": [
                {
                    "type": "App\\Queue\\MyQueue",
                    "client": {
                        "host": "127.0.0.1",
                        "port": 1234
                    },
                    "name": ["emails", "notifications"]
                }
            ]
        }
    }
}
```

The `QueueFactoryTrait` provides a `getRateLimiterFromConfig()` helper that parses rate limit strings from configuration
into `RateLimiterInterface` instances.

> ℹ️ **Note**: Custom factories are **merged** with the default ones. You don't need to re-declare the built-in
> factories. A custom factory for the same queue class as a built-in factory (for example your own factory for
> `RedisQueue`, written before version 3.4) replaces the built-in one.

## Further reading

- [Queue Manager component](../components/queue-manager.md) — Full API reference (queues, jobs, workers, payloads,
  rate limiters)
- [Queue CLI commands](../cli/queues.md) — Running workers, purging queues, monitoring sizes, production deployment
- [After the response](../http/terminate.md) — Terminate lifecycle and its caveats
- [Packages](packages.md) — How Berlioz packages work
