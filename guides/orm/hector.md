```index
breadcrumb: Guides; ORM; Hector
summary-order: ; ; 1
```

# Hector ORM

A Berlioz package exist for **Hector ORM** library to easier the interactions and configuration between them.

## Installation

Use composer to install package:

```bash
composer require berlioz/hector-package
```

For more detail package installation, referred to the [package description](packages.md) page.

## Configuration

Create a `hector.json` file in your [configuration directory](../../getting-started/directories.md), with this content:

```json
{
    "hector": {
        "dsn": "mysql:dbname=mydbname;host=127.0.0.1;port=3306;charset=UTF8;user=username;password=password",
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
        "schemas": [],
        "dynamic_events": true,
        "types": []
    }
}
```

> **Warning**:
>
> Ignore this configuration file in your `.gitignore` file. It should contain passwords... and MUST NOT push on GIT repository!

## Package additions

- Subscription of ORM events with framework event manager
- Debug page in console
- Not found entities exception generate a not found http error
- Usage of cache system of framework
- Magics methods in entities to manage events simply

## Usage

To know more on usage with the ORM, referrer you to the [official documentation of Hector ORM](https://gethectororm.com).

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