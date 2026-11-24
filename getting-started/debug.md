---
breadcrumb:
  - Getting started
  - Debug
---

# Debug

Berlioz provides a debug toolbar and console to facilitate your development.

## Toolbar

The toolbar is shown at the bottom left of your page, and indicates the execution time of the page.

![Debug toolbar](../_assets/debug-toolbar.png)

> ℹ️ **Note**: If an error occurs during script execution, the check icon turns into a cross icon ;)

Two icons appear on hover. The first one hides the toolbar, and the second one moves the toolbar to the right (this
action is saved in a cookie to restore the position).

### Response time

> 🆕 **Info**: *Since version 3.4*

The toolbar displays the **response time**: the time until the response has been sent to the client. Work done
[after the response](../http/terminate.md) is not included. The console displays both the response time and the
**total time** of the request (dashboard and performances), and the activities timeline marks the moment the response
has been sent.

### Snapshot in progress

> 🆕 **Info**: *Since version 3.4*

The snapshot is saved at the very end of the request, after the work done after the response. Meanwhile, the toolbar
shows a clock icon with the response time, and the console a "Snapshot in progress" page: both reload automatically
once the snapshot is saved.

If the snapshot is never saved before a delay, because the process has been interrupted (timeout, memory limit, killed
worker...), the console displays "Snapshot incomplete" and the toolbar a cross icon. This delay is configured with
`berlioz.debug.pending_ttl`, in seconds; by default, the PHP `max_execution_time`, or 300 seconds if unlimited:

```json
{
  "berlioz": {
    "debug": {
      "pending_ttl": 60
    }
  }
}
```

The pending TTL only controls when the console labels an unfinished snapshot as incomplete; it does not stop PHP
or cancel deferred jobs. Choose a TTL that covers your expected after-response work. A snapshot saved later still
becomes available normally.

The console answers with the HTTP status `202` while the snapshot is in progress, `410` when it is incomplete, and
`404` when the snapshot is unknown.

## Console

To open the console, click on the toolbar.

![Debug Console](../_assets/debug-console.png)

### HTTP response

> 🆕 **Info**: *Since version 3.3*

The **HTTP / Router** section also displays the response status code, reason phrase, HTTP protocol version
and headers, allowing you to inspect the response alongside the request and routing information.

## Enable/Disable debug mode

By default, debug mode is not enabled. You need to enable it in your configuration file.

```json
{
  "berlioz": {
    "debug": {
      "enable": true
    }
  }
}
```

## Restrict access

Debug mode exposes sensitive information (configuration, environment, PHP info, queries...). Only allowed clients
get the debug toolbar, the snapshots and the console. Other clients get a `404` error on the console.

### Default access

> 🆕 **Info**: *Since version 3.4*

When `berlioz.debug.ip` is not configured, only local clients are allowed: `127.0.0.1` and `::1`. CLI commands are
considered local too, as long as the list allows `127.0.0.1` or `::1`.

To allow other clients, declare the list of allowed addresses:

```json
{
  "berlioz": {
    "debug": {
      "enable": true,
      "ip": [
        "127.0.0.1",
        "::1",
        "203.0.113.10",
        "198.51.100.0/24"
      ]
    }
  }
}
```

The list replaces the default one: add `127.0.0.1` and `::1` if you still need local access. An empty list
(`"ip": []`) denies every client.

> ⚠️ **Warning**: if your list does not allow `127.0.0.1` or `::1` (for example only remote addresses), CLI commands
> run without debug mode.

Each entry of the list can be:

- an IP address: `203.0.113.10`, `::1`;
- a CIDR range: `198.51.100.0/24`, `2001:db8::/32`;
- an alias: `loopback` (`127.0.0.0/8` and `::1`), `private` (private and reserved ranges), `public`;
- `*` to allow every client, including behind a reverse proxy.

> ⚠️ **Warning**: `*` exposes debug information to everyone who can reach your application. Never use it on a
> public server.

#### Docker and virtual machines

With Docker, a virtual machine or a local network, your browser does not reach the application from `127.0.0.1`
but from a private address, for example the Docker gateway `172.17.0.1`. Allow the private ranges, or only the
address of your network:

```json
{
  "berlioz": {
    "debug": {
      "enable": true,
      "ip": ["loopback", "private"]
    }
  }
}
```

#### Hosts

> ⚠️ **Warning**: *Deprecated since version 3.4*
>
> Hosts trigger an `E_USER_DEPRECATED` notice and will be removed in version 4. Use IP addresses, CIDR ranges or
> aliases instead.

An entry that is not an address, a range or an alias is compared to the host name of the client IP (reverse DNS).

### Client IP resolution behind a proxy

> 🆕 **Info**: *Since version 3.2*
>
> The client IP checked against the allow-list is now taken from `REMOTE_ADDR` and the `X-Forwarded-For` header is
> **no longer trusted by default**. Previously, a spoofed `X-Forwarded-For` header could bypass the IP restriction and
> enable the debug mode from any origin.

If your application runs behind a trusted reverse proxy or load balancer, declare the proxy addresses (single IPs,
CIDR ranges or aliases) under `berlioz.proxies.trusted`. Only then is the forwarded client IP taken into account:

```json
{
  "berlioz": {
    "proxies": {
      "trusted": [
        "10.0.0.0/8",
        "192.168.1.10"
      ]
    },
    "debug": {
      "enable": true,
      "ip": [
        "203.0.113.10"
      ]
    }
  }
}
```

The client IP is read from the `X-Forwarded-For` header sent by the trusted proxy.

> 🆕 **Info**: *Since version 3.4*
>
> When the client cannot be determined reliably, debug access is denied:
>
> - the request has a `Forwarded`, `X-Forwarded-For` or `X-Real-IP` header, but comes from a proxy that is not
>   declared in `berlioz.proxies.trusted`;
> - the request comes from a trusted proxy that does not send the `X-Forwarded-For` header.
>
> A reverse proxy running on the same server (`127.0.0.1`) therefore no longer makes every remote client local.

A proxy that sends no forwarding header at all cannot be detected: all its clients have the address of the proxy.
Configure your proxy to send the `X-Forwarded-For` header, and declare it in `berlioz.proxies.trusted`.

## Snapshot retention

> 🆕 **Info**: *Since version 3.2*

When debug mode is enabled, Berlioz records a snapshot (debug report) for each request. To prevent the debug directory
from growing indefinitely, old snapshots are garbage collected **probabilistically on write**, according to a
retention policy configured under `berlioz.debug.gc`:

```json
{
  "berlioz": {
    "debug": {
      "enable": true,
      "gc": {
        "probability": 1,
        "divisor": 100,
        "max_age": 7,
        "max_files": 100
      }
    }
  }
}
```

- **`probability`** / **`divisor`**: the garbage collector runs with a probability of `probability / divisor` on each
  snapshot write (default `1 / 100`, i.e. ~1% of requests). Set `probability` to `0` to disable automatic collection.
- **`max_age`**: delete snapshots older than this number of days (`null` to disable).
- **`max_files`**: keep at most this number of snapshots, deleting the oldest ones beyond the limit (`null` to
  disable).

You can also trigger the cleanup manually with the [`berlioz:debug-clear`](../cli.md#berliozdebug-clear) CLI command.
