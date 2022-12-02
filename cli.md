```index
breadcrumb: CLI Project
summary-order: 3
description: Berlioz Framework allows you to manage your CLI projects with commands and other extras
keywords: command
```

# CLI Project

**Berlioz Framework** allows you to manage your CLI projects.

In some case, you don't have any rendering functionality or HTTP support. The **Berlioz/CliCore** project is do for you!

## Installation

Installation of **Berlioz/CliCore** must be done by [Composer](https://getcomposer.org/), it's the recommended installation.

First, create your classical project with composer support. And executes this command:

```bash
composer require berlioz/cli-core
```

An executable will be created in the binary directory of **Composer** (default: **vendor/bin**).

## Commands

CLI commands are available on the framework.
You can use some default commands or create your owns.

For execution of commands:

```bash
$ vendor/bin/berlioz myproject:foo
Printed result!
```

For more details of commands usage, referrer you to the [Commands page](cli/commands.md).

## Default commands

### `berlioz:config`

To get the beautified configuration.

Parameters:

- `-f ...` `--filter ...`
  To filter the configuration key to shown

### `berlioz:cache-clear`

To clear the cache.

Parameters (version `^2.2`):

- `-all`
  To clear all contents of cache directory (except hidden items)
- `...`
  To specify directories name to clear.
