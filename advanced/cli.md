<meta name="docparser-index" content="Advanced usage; CLI Project" />
<meta name="docparser-index-order" content="0" />
<meta name="docparser-description" content="Berlioz Framework allows you to manage your CLI projects with commands and other extras" />

# CLI Project

**Berlioz Framework** allows you to manage your CLI projects.

In some case, you don't have any rendering functionality or HTTP support. The **Berlioz/CliCode** project is do for you!

## Installation

Installation of **Berlioz/CliCore** must be done by [Composer](https://getcomposer.org/), it's the recommended installation.

First, create your classical project with composer support. And executes this command:

```bash
composer require berlioz/cli-core
```

An executable will be create in binary directory of **Composer** (default: **vendor/bin**).

## Commands

CLI commands are available on framework.
You can use some default commands or create your owns.

For execution of commands:

```bash
$ vendor/bin/berlioz myproject:foo
Printed result!
```

For more details of commands usage, referrer you to the [Commands page](./commands.md).

## Default commands

### `berlioz:config`

To see the beautify configuration.

Parameters:

- `-f ...` `--filter ...`
  To filter the configuration key to shown

### `berlioz:cache-clear`

To clear the cache.