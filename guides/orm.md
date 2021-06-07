```index
breadcrumb: Guides; ORM
summary-order: ; 1
```

# Object-Relational Mapping

## Definition

> **Object-relational mapping** (**ORM**, **O/RM**, and **O/R mapping tool**) in computer science is a programming technique for converting data between incompatible type systems using object-oriented programming languages. This creates, in effect, a "virtual object database" that can be used from within the programming language. There are both free and commercial packages available that perform object-relational mapping, although some programmers opt to construct their own ORM tools.
>
> In object-oriented programming, data-management tasks act on object-oriented (OO) objects that are almost always non-scalar values. For example, an address book entry that represents a single person along with zero or more phone numbers and zero or more addresses. This could be modeled in an object-oriented implementation by a "Person object" with attributes/fields to hold each data item that the entry comprises: the person's name, a list of phone numbers, and a list of addresses. The list of phone numbers would itself contain "PhoneNumber objects" and so on. The address-book entry is treated as a single object by the programming language (it can be referenced by a single variable containing a pointer to the object, for instance). Various methods can be associated with the object, such as a method to return the preferred phone number, the home address, and so on.
>
> However, many popular database products such as SQL database management systems (DBMS) can only store and manipulate scalar values such as integers and strings organized within tables. The programmer must either convert the object values into groups of simpler values for storage in the database (and convert them back upon retrieval), or only use simple scalar values within the program. Object-relational mapping implements the first approach.
>
> The heart of the problem involves translating the logical representation of the objects into an atomized form that is capable of being stored in the database while preserving the properties of the objects and their relationships so that they can be reloaded as objects when needed. If this storage and retrieval functionality is implemented, the objects are said to be persistent.
>
> Source: [Wikipedia](https://en.wikipedia.org/wiki/Object-relational_mapping)

## Atlas

Berlioz purposes you to use **Atlas ORM**. We have create a package to easier the interactions and configuration between them.

### Installation

Use composer to install package:

```bash
composer require berlioz/atlas-package
```

For more detail package installation, referred to the [package description](packages.md) page.

### Configuration

Create a `atlas.json` file in your [configuration directory](../getting-started/directories.md), with this content:

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
> Ignore this configuration file in your `.gitignore` file. It should contains passwords... and MUST NOT push on GIT repository!

### Usage

You can call service container in controllers to get Atlas object:

```php
/** @var \Atlas\Orm\Atlas $atlas */
$atlas = $this->getService('atlas');
$atlas = $this->getService(\Atlas\Orm\Atlas::class);
```

To know more on usage with the ORM, referrer you to the [official documentation or Atlas ORM](http://atlasphp.io).