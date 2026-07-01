---
breadcrumb:
  - CLI Project
  - Queue Commands
summary-order: 3;4
keywords:
  - queue
  - worker
  - command
---

# Queue Commands

The **berlioz/queue-manager-package** registers three CLI commands for managing queues. These commands are available
when the package is installed alongside **berlioz/cli-core**.

For queue configuration and job dispatching, see the [Queues guide](../guides/queues.md).

## `queue:worker`

Start a worker to process queue jobs:

```bash
$ vendor/bin/berlioz queue:worker -q emails -q notifications --limit 100 --memory 128 --time 3600
```

Parameters:

| Parameter | Short | Description | Default |
|---|---|---|---|
| `--name` | `-n` | Worker name | — |
| `--queue` | `-q` | Queue name (repeatable, supports wildcards) | all queues |
| `--limit` | — | Maximum number of jobs to process | unlimited |
| `--delay` | — | Delay between two consumptions (seconds) | `0` |
| `--delay-no-job` | — | Delay when no job is available (seconds) | `1` |
| `--memory` | — | Memory limit in MB | unlimited |
| `--time` | — | Time limit in seconds | unlimited |
| `--rate` | — | Rate limit string (repeatable, e.g. `"100/1m"`) | — |
| `--backoff` | — | Backoff time on failure (seconds) | `0` |
| `--backoff-multiplier` | — | Exponential backoff multiplier | `1` |
| `--kill-file` | — | Path to a kill file for remote stop | — |
| — | `-v` | Verbose output (debug level logging) | — |

The worker processes jobs from the specified queues in a loop. It exits when one of the following conditions is met:

- The job **limit** is reached
- The **memory** limit is exceeded
- The **time** limit is exceeded
- A **SIGTERM** or **SIGQUIT** signal is received (graceful shutdown)
- The **kill file** is created

### Queue filtering

The `-q` option supports wildcards. For example, `-q "notifications.*"` will match queues named `notifications.sms`,
`notifications.push`, etc.

### Backoff strategy

When a job fails, the worker can apply an exponential backoff before retrying. The delay for retry **n** is:

```
delay = backoff × (backoff-multiplier ^ (n - 1))
```

For example, with `--backoff 10 --backoff-multiplier 2`: the first retry waits 10s, the second 20s, the third 40s, etc.

## `queue:purge`

Purge all jobs from queues:

```bash
$ vendor/bin/berlioz queue:purge -q emails
```

Parameters:

| Parameter | Short | Description |
|---|---|---|
| `--queue` | `-q` | Queue name (repeatable, supports wildcards) |

A confirmation prompt is shown before purging. Only queues implementing `PurgeableQueueInterface` can be purged
(Database, Memory, AWS SQS, AMQP). Non-purgeable queues (like Redis) are skipped with a warning.

## `queue:size`

> 🆕 **Info**: *Since version 3.1*

Display queue monitoring metrics:

```bash
$ vendor/bin/berlioz queue:size --format json --total
```

Parameters:

| Parameter | Short | Description | Default |
|---|---|---|---|
| `--queue` | `-q` | Queue name (repeatable, supports wildcards) | all queues |
| `--format` | `-f` | Output format: `json`, `prometheus`, or table | table |
| `--total` | — | Include total count | `false` |
| `--prometheus-labels` | — | Additional Prometheus labels | — |

The command always reports queue `size`, and can also expose `waitTime` and `delayed` metrics when the selected
backend implements `MonitorableQueueInterface`.

Output examples:

**Table (default):**

```
emails           12 (wait: 34s, delayed: 2)
notifications     3 (wait: n/a, delayed: n/a)
```

**JSON** (`--format json --total`):

```json
{
    "queues": {
        "emails": {
            "size": 12,
            "waitTime": 34,
            "delayed": 2
        },
        "notifications": {
            "size": 3,
            "waitTime": null,
            "delayed": null
        }
    },
    "total": 15
}
```

**Prometheus** (`--format prometheus --prometheus-labels 'env="production"' --total`):

```
# HELP job_queue_length Number of jobs waiting in the queue.
# TYPE job_queue_length gauge
job_queue_length{queue_name="emails",env="production"} 12
job_queue_length{queue_name="notifications",env="production"} 3
# HELP job_queue_wait_time_seconds Age in seconds of the oldest consumable job in the queue.
# TYPE job_queue_wait_time_seconds gauge
job_queue_wait_time_seconds{queue_name="emails",env="production"} 34
# HELP job_queue_delayed Number of delayed jobs in the queue.
# TYPE job_queue_delayed gauge
job_queue_delayed{queue_name="emails",env="production"} 2
# HELP job_queue_length_total Total number of jobs waiting across all queues.
# TYPE job_queue_length_total gauge
job_queue_length_total{env="production"} 15
```

> 🆕 **Info**: *Since version 3.2*
>
> The Prometheus output now includes `# HELP` / `# TYPE` metadata lines. Label values are escaped per the exposition
> format.

Metrics with unavailable values are omitted in Prometheus output.

These metrics can also be served over HTTP for a Prometheus server to scrape directly — see the
[Monitoring and metrics](../guides/queues.md#monitoring-and-metrics) section of the Queues guide.

## Production deployment

### Supervisor

In production, workers should be managed by a process supervisor like
[Supervisor](http://supervisord.org/). Example configuration:

```ini
[program:queue-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/myapp/vendor/bin/berlioz queue:worker -q emails --memory 128 --time 3600 --backoff 10 --backoff-multiplier 2
autostart=true
autorestart=true
numprocs=2
redirect_stderr=true
stdout_logfile=/var/log/myapp/queue-worker.log
stopwaitsecs=60
```

Key settings:

- **`numprocs`**: Number of parallel worker processes
- **`stopwaitsecs`**: Time to wait for graceful shutdown before killing (should be longer than your longest job)
- **`autorestart=true`**: Restart workers that exit (due to memory/time limits)

### Memory and time limits

Always set `--memory` and `--time` limits in production. Workers exit cleanly when limits are reached, and Supervisor
automatically restarts them. This prevents memory leaks and ensures workers pick up code changes after deployments:

```bash
$ vendor/bin/berlioz queue:worker -q "*" --memory 256 --time 3600
```

### Graceful shutdown

The worker handles `SIGTERM` and `SIGQUIT` signals. When a signal is received, the worker finishes processing the
current job and then exits cleanly.

> ℹ️ **Note**: The `pcntl` PHP extension must be enabled for signal handling. Without it, workers can only be stopped
> via the kill file mechanism or by reaching their configured limits.

### Remote stop (kill file)

You can stop a worker remotely by creating a file at a specified path:

```bash
$ vendor/bin/berlioz queue:worker -q emails --kill-file /tmp/stop-email-worker
```

To stop the worker:

```bash
$ touch /tmp/stop-email-worker
```

The worker checks for the file's existence after each job and exits if found. This is useful when `pcntl` is not
available or for deployment scripts that need to stop workers before code updates.

## Further reading

- [Queues guide](../guides/queues.md) — Configuration, dispatching jobs, creating handlers
- [Queue Manager component](../components/queue-manager.md) — Full API reference
