---
breadcrumb:
  - Guides
  - ORM
  - Hector
summary-order: ;;1
---

# Hector ORM

**Hector ORM** is the default and recommended ORM for Berlioz Framework. The **berlioz/hector-package** provides
seamless integration between the framework and the ORM: automatic configuration, event subscription, cache management,
debug console page, and CLI commands.

## Installation

Use Composer to install the package:

```bash
composer require berlioz/hector-package
```

For more details on package installation, see the [package description](../packages.md) page.

## Configuration

Create a `hector.json` file in your [configuration directory](../../getting-started/directories.md), with this content:

```json
{
    "hector": {
        "dsn": "mysql:dbname=mydbname;host=127.0.0.1;port=3306;charset=UTF8",
        "username": "username",
        "password": "password",
        "schemas": [
            "mydbname"
        ]
    }
}
```

The default configuration is:

```json
{
    "hector": {
        "dsn": null,
        "read_dsn": null,
        "username": null,
        "password": null,
        "options": {},
        "schemas": [],
        "dynamic_events": true,
        "types": []
    }
}
```

> ⚠️ **Warning**: Ignore this configuration file in your `.gitignore` file. It is likely to contain passwords and MUST
> NOT be pushed to the Git repository!

### PDO options

> 🆕 **Info**: *Since version 3.4*, `berlioz/hector-package` requires HectorORM **1.5 or later**.
> HectorORM 1.5 introduced PDO connection options; earlier versions do not forward them to PDO.

PDO connection options are given under `hector.options`. They are passed to the PDO constructor and shared by the
read/write and read-only (`read_dsn`) connections.

Since configuration files cannot reference PHP constants in keys, PDO attributes can be written as integers:

```json
{
  "hector": {
    "options": {
      "2": 5,
      "20": false
    }
  }
}
```

> 🆕 **Info**: *Since version 3.4*
>
> Attributes can also be written with **constant names**, resolved to their integer value:
>
> ```json
> {
>   "hector": {
>     "options": {
>       "PDO::ATTR_TIMEOUT": 5,
>       "Pdo\\Mysql::ATTR_INIT_COMMAND": "SET time_zone = '+00:00'",
>       "20": false
>     }
>   }
> }
> ```
>
> Options can also be given as a **list** of `attribute` / `value` entries. `attribute` accepts an integer, a constant
> name or the `{constant: ...}` configuration function:
>
> ```json
> {
>   "hector": {
>     "options": [
>       {"attribute": "PDO::ATTR_TIMEOUT", "value": 5},
>       {"attribute": "{constant: PDO::ATTR_EMULATE_PREPARES}", "value": false}
>     ]
>   }
> }
> ```

Rules:

- When the same attribute is defined several times, **the last definition wins**. Across configuration files, the
  definition of the file with the highest priority wins.
- An unknown constant, a constant that is not an integer, or a list entry without `attribute` and `value` keys throws
  a `HectorException`.

> ⚠️ **Warning**:
>
> - Do not mix the map and list formats across configuration files (e.g. a map in `hector.json.dist` and a list in
>   `hector.json`): when one side is a list, the configuration merge renumbers integer keys.
> - Options like `PDO::ATTR_AUTOCOMMIT` or `PDO::ATTR_ERRMODE` may interfere with the internal transaction handling of
>   Hector ORM.
> - Since PHP 8.4, driver-specific constants are available on driver classes (e.g. `Pdo\Mysql::ATTR_INIT_COMMAND`);
>   `PDO::MYSQL_ATTR_*` constants are deprecated since PHP 8.5.

## Package additions

- Subscription of ORM events with the framework event manager
- Debug page in the console
- An entity not found exception generates a not found HTTP error
- Usage of the cache system of the framework
- Magic methods in entities to manage events simply

## Usage

To learn more about using the ORM, see the [official documentation of Hector ORM](https://gethectororm.com).

## Event magic methods

Save magic methods:

- `Entity::onSave(): void` called after save (insert/update)
- `Entity::onBeforeSave(): void` called before save (insert/update)
- `Entity::onAfterSave(): void` called after save (insert/update)

Insert magic methods:

- `Entity::onInsert(): void` called after insert
- `Entity::onBeforeInsert(): void` called before insert
- `Entity::onAfterInsert(): void` called after insert

Update magic methods:

- `Entity::onUpdate(): void` called after update
- `Entity::onBeforeUpdate(): void` called before update
- `Entity::onAfterUpdate(): void` called after update

Delete magic methods:

- `Entity::onDelete(): void` called after delete
- `Entity::onBeforeDelete(): void` called before delete
- `Entity::onAfterDelete(): void` called after delete

All methods are called from the service container, so dependency injection is enabled :).

## Debug console

When debug mode is enabled, the Hector section of the debug console lists every executed SQL query with its timing.

> 🆕 **Info**: *Since version 3.2*
>
> The debug console now highlights **slow queries** and detects **duplicate queries** (identical statements executed
> several times in the same request, a common N+1 symptom). Copy-to-clipboard buttons are available on each query
> (raw and with interpolated values).

Thresholds are configurable under `hector.debug`:

```json
{
  "hector": {
    "debug": {
      "slow_query": 50,
      "very_slow_query": 100,
      "duplicate_threshold": 2
    }
  }
}
```

- **`slow_query`** / **`very_slow_query`** — durations in milliseconds above which a query is flagged as slow / very slow.
- **`duplicate_threshold`** — number of identical executions from which a query is reported as duplicate.

## CLI commands

The Hector package registers the following CLI commands:

### `hector:cache-clear`

Clear the Hector ORM cache.

```bash
$ vendor/bin/berlioz hector:cache-clear
```

### `hector:generate-schema`

Generate the schema for Hector ORM entities. This command initializes the ORM and triggers schema generation.

```bash
$ vendor/bin/berlioz hector:generate-schema
```

### Database migrations

> 🆕 **Info**: *Since version 3.2*
>
> Requires `hectororm/hectororm` `^1.4`.

The package exposes Hector ORM database migrations through three commands:

```bash
# Apply all pending migrations
$ vendor/bin/berlioz hector:migrate

# Roll back migrations
$ vendor/bin/berlioz hector:migrate:down

# Show the status of migrations (applied / pending)
$ vendor/bin/berlioz hector:migrate:status
```

Both `hector:migrate` and `hector:migrate:down` accept:

- `--dry-run` — show what would be executed without touching the database (delegates to the runner's native dry-run)
- `--interactive` / `-i` — ask for confirmation before each migration

Migrations are configured under `hector.migration`:

```json
{
  "hector": {
    "migration": {
      "provider": {
        "type": "directory",
        "directory": "{config: berlioz.directories.app}/migrations",
        "namespace": null,
        "pattern": "*.php",
        "depth": 0
      },
      "tracker": {
        "type": "db",
        "table": "hector_migrations",
        "file": "{config: berlioz.directories.var}/hector.migrations.json"
      },
      "schema": null
    }
  }
}
```

- **`provider`** — where migrations are discovered. `type` can be `directory` (default), `psr4`, or a custom provider service.
- **`tracker`** — how applied migrations are recorded. `type` can be `db` (default, table `hector_migrations`), `file` (JSON file), or a custom tracker service.
