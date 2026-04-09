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

- Configuration-based queue setup (Database, AWS SQS, Memory)
- Automatic service container registration (`QueueManager`, `JobHandlerManager`, `Worker`)
- Built-in job handlers for running Berlioz commands and system commands asynchronously
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
                "berlioz:command": "Berlioz\\Package\\QueueManager\\Handler\\BerliozCommandJobHandler",
                "berlioz:system": "Berlioz\\Package\\QueueManager\\Handler\\BerliozSystemJobHandler"
            },
            "factories": [
                "Berlioz\\Package\\QueueManager\\Factory\\MemoryQueueFactory",
                "Berlioz\\Package\\QueueManager\\Factory\\AwsSqsQueueFactory",
                "Berlioz\\Package\\QueueManager\\Factory\\DbQueueFactory"
            ]
        }
    }
}
```

- **`queues`**: Array of queue definitions, each with a `type` key matching a queue class
- **`handlers`**: Map of job name to handler class
- **`factories`**: Array of factory classes that know how to create queues from configuration

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

### Per-queue overrides

Each queue name can be an object with its own `retry_time`, `max_attempts`, and `rate_limit`:

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

Executes a system command via `passthru()`. The payload `command` key is an array of arguments joined with spaces:

```php
use Berlioz\QueueManager\Job\JobDescriptor;

// Run a system command asynchronously
$job = new JobDescriptor('berlioz:system', [
    'command' => ['php', '/path/to/script.php', '--option=value'],
]);
$queueManager->push($job);
```

> ⚠️ **Warning**: The `berlioz:system` handler executes arbitrary system commands. Ensure job payloads are never built
> from untrusted user input.

## CLI commands

The package registers three CLI commands for managing queues. See the dedicated
[Queue CLI commands](../cli/queues.md) page for full usage details and production deployment tips.

- **`queue:worker`** — Start a worker to process queue jobs
- **`queue:purge`** — Purge all jobs from queues
- **`queue:size`** — Display queue monitoring metrics

### Queue monitoring

> 🆕 **Info**: *Since version 3.1*

The `queue:size` command can now expose more than queue length. Depending on the backend, it can report:

- `size`: number of jobs ready to be consumed
- `waitTime`: age in seconds of the oldest consumable job
- `delayed`: number of delayed jobs waiting to become available

See the [Queue CLI commands](../cli/queues.md#queuesize) page for output formats and Prometheus examples.

## Custom queue factories

The package includes factories for Database, AWS SQS, and Memory queues. To use other backends (Redis, RabbitMQ/AMQP,
or your own), you can write a custom factory.

A factory must implement the `QueueFactory` interface:

```php
use Berlioz\Package\QueueManager\Factory\QueueFactory;
use Berlioz\Package\QueueManager\Factory\QueueFactoryTrait;
use Berlioz\QueueManager\Queue\RedisQueue;
use Generator;
use Redis;

class RedisQueueFactory implements QueueFactory
{
    use QueueFactoryTrait;

    public static function getQueueClass(): string
    {
        return RedisQueue::class;
    }

    public static function createFromConfig(array $config): Generator
    {
        $redis = new Redis();
        $redis->connect(
            $config['redis']['host'] ?? '127.0.0.1',
            (int)($config['redis']['port'] ?? 6379),
        );

        foreach ((array)($config['name'] ?? []) as $queue) {
            !is_array($queue) && $queue = ['name' => (string)$queue];

            yield new RedisQueue(
                redis: $redis,
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
                "App\\Queue\\RedisQueueFactory"
            ],
            "queues": [
                {
                    "type": "Berlioz\\QueueManager\\Queue\\RedisQueue",
                    "redis": {
                        "host": "127.0.0.1",
                        "port": 6379
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
> factories.

## Further reading

- [Queue Manager component](../components/queue-manager.md) — Full API reference (queues, jobs, workers, payloads,
  rate limiters)
- [Queue CLI commands](../cli/queues.md) — Running workers, purging queues, monitoring sizes, production deployment
- [Packages](packages.md) — How Berlioz packages work
