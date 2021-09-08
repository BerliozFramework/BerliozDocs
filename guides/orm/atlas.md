```index
breadcrumb: Guides; ORM; Atlas
summary-order: ; ; 2
```

# Atlas

A Berlioz package exist for **Atlas ORM** library to easier the interactions and configuration between them.

## Installation

Use composer to install package:

```bash
composer require berlioz/atlas-package
```

For more detail package installation, referred to the [package description](packages.md) page.

## Configuration

Create a `atlas.json` file in your [configuration directory](../../getting-started/directories.md), with this content:

```json
{
    "atlas": {
        "pdo": {
            "connection_locator": {
                "default": {
                    "dsn": "mysql:dbname=mydbname;host=127.0.0.1;port=3306",
                    "username": "username",
                    "password": "password"
                }
            }
        }
    }
}
```

> **Warning**:
>
> Ignore this configuration file in your `.gitignore` file. It should contain passwords... and MUST NOT push on GIT repository!

## Usage

You can call service container in controllers to get Atlas object:

```php
/** @var \Atlas\Orm\Atlas $atlas */
$atlas = $this->getService('atlas');
$atlas = $this->getService(\Atlas\Orm\Atlas::class);
```

To know more on usage with the ORM, referrer you to the [official documentation of Atlas ORM](http://atlasphp.io).