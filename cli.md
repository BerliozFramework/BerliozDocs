---
breadcrumb:
  - CLI Project
summary-order: 3
description: Berlioz Framework allows you to manage your CLI projects with commands and other extras
keywords:
  - command
---

# CLI Project

**Berlioz Framework** allows you to manage your CLI projects.

In some cases, you don't have any rendering functionality or HTTP support. The **Berlioz/CliCore** project is made for
you!

## Installation

Install **Berlioz/CliCore** with [Composer](https://getcomposer.org/); it's the recommended installation method.

First, create a standard project with Composer support. Then execute this command:

```bash
composer require berlioz/cli-core
```

An executable will be created in the binary directory of **Composer** (default: **vendor/bin**).

## Commands

CLI commands are available in the framework.
You can use some default commands or create your own.

To execute a command:

```bash
$ vendor/bin/berlioz myproject:foo
Printed result!
```

For more details on commands, see the [Commands page](cli/commands.md).

> 🆕 **Info**: *Since version 3.4*
>
> After each command, even a failed one, the `berlioz` binary terminates the application: a `CliTerminateEvent` is
> dispatched and the [deferred queues](guides/queues.md#deferred-queues) are consumed. The event exposes the exit code
> returned by command handling; it is `null` if handling did not complete normally, even though the binary exits
> with code `1` when it catches an exception. See
> [Framework events](guides/events.md#terminate).
>
> If you start `CliApp` from your own script instead of the `berlioz` binary, call `$app->terminate()` yourself after
> `$app->handle()`.

## Default commands

### `berlioz:config`

To get the beautified configuration.

Parameters:

- `-f ...` `--filter ...`
  To filter the configuration key to show

### `berlioz:cache-clear`

To clear the cache.

Parameters:

- `--all`
  To clear all contents of the cache directory (except hidden items)
- `...`
  To specify the names of the directories to clear. When specific names are provided, all matching directories are
  cleared, including hidden ones.

### `berlioz:debug-clear`

> 🆕 **Info**: *Since version 3.2*

To clear the recorded debug reports (snapshots).

Parameters:

- `--all`
  To delete **all** debug reports
- `--days=N`
  To delete reports older than `N` days

When called without any parameter, the command applies the configured retention policy
(`berlioz.debug.gc.max_age` and `berlioz.debug.gc.max_files`, see [Debug](getting-started/debug.md#snapshot-retention)).

> 🆕 **Info**: *Since version 3.4*
>
> The command also removes the markers of [snapshots in progress](getting-started/debug.md#snapshot-in-progress):
> all of them with `--all`, only the expired ones otherwise.

## Package commands

Additional CLI commands are available when optional packages are installed:

- **[Queue commands](cli/queues.md)** — `queue:worker`, `queue:purge`, `queue:size` (provided by
  **berlioz/queue-manager-package**)
- **Hector ORM commands** — `hector:cache-clear`, `hector:generate-schema` (provided by **berlioz/hector-package**,
  see [Hector ORM](guides/orm/hector.md#cli-commands))
