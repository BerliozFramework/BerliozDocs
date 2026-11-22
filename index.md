---
breadcrumb:
  - Getting started
summary-order: 1
---

# Getting started

## Introduction

**Berlioz Framework** is a slim PHP framework to easily publish your online projects like websites, intranets, APIs...
or all your creations!
**Berlioz** is a simple and effective alternative to the PHP frameworks on the market.

## Quick start

Do you want to quickly install a Berlioz project?

```bash
composer create-project berlioz/website-skeleton my-project --remove-vcs
```

*The `--remove-vcs` option removes the default `.git` directory.*

It's done! You are ready to develop your project!

[Need help to configure your HTTP server?](getting-started/installation.md)

## Concepts and basic usage

You are now ready to work on your project!

You must understand some concepts of the **Berlioz Framework**:

- [Hierarchy of directories](getting-started/directories.md)
- [Configuration](getting-started/config.md)
- [Service Container](getting-started/service-container.md)
- [Packages](guides/packages.md)
- [Cache](guides/cache.md)
- [Controllers](http/controllers.md)
- [Middlewares](http/middleware.md)
- [Routing](http/routing.md)
- [After the response](http/terminate.md)
- [HTTP exceptions](http/exceptions.md)
- [Error handler](http/error-handler.md)
- [Templates](guides/templating.md)
- [ORM](guides/orm.md)
- [Forms](guides/forms.md)

## Standalone components

**Berlioz Framework** is built on top of standalone PHP components. Each component can be used independently in any PHP
project:

- [Components overview](components.md)

## Advanced usage

If you want to go further with **Berlioz Framework**, you can consult:

- [CLI Project](cli.md)
- [Commands](cli/commands.md)
- [Console](cli/console.md)
- [Package creation](guides/packages/creation.md)
- [Upgrading](getting-started/upgrading.md)
