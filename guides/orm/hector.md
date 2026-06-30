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

Use composer to install package:

```bash
composer require berlioz/hector-package
```

For more detail package installation, referred to the [package description](../packages.md) page.

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

Default configuration is:

```json
{
    "hector": {
        "dsn": null,
        "read_dsn": null,
        "username": null,
        "password": null,
        "schemas": [],
        "dynamic_events": true,
        "types": []
    }
}
```

> **Warning**:
>
> Ignore this configuration file in your `.gitignore` file. It should contain passwords... and MUST NOT push on GIT
> repository!

## Package additions

- Subscription of ORM events with framework event manager
- Debug page in console
- Not found entities exception generate a not found http error
- Usage of cache system of framework
- Magics methods in entities to manage events simply

## Usage

To know more on usage with the ORM, referrer you to
the [official documentation of Hector ORM](https://gethectororm.com).

## Events magic methods

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

All methods are called from service container, so the dependency injection is enabled :).

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