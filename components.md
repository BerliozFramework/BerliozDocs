---
breadcrumb:
  - Components
summary-order: 6
description: Standalone components of Berlioz Framework, usable independently
keywords:
  - components
  - standalone
  - libraries
---

# Components

**Berlioz Framework** is built on top of standalone PHP components. Each component can be used **independently** in any
PHP project, without the full framework.

They are all distributed as separate [Composer](https://getcomposer.org/) packages via
[Packagist](https://packagist.org/?query=berlioz).

> All components require **PHP ^8.2** and follow [Semantic Versioning](https://semver.org/).

## Configuration & Services

- [**Config**](components/config.md)

  A library to manage your configuration files with support for JSON, JSON5, YAML and INI formats.

- [**Service Container**](components/service-container.md)

  A dependency injection container respecting the PSR-11 (Container interface) standard.

- [**Event Manager**](components/event-manager.md)

  An event manager/dispatcher respecting the PSR-14 (Event Dispatcher) standard.

- [**Flash Bag**](components/flash-bag.md)

  A lightweight library to manage flash messages stored in the PHP session.

## HTTP

- [**HTTP Message**](components/http-message.md)

  Implementation of PSR-7 (HTTP message interfaces) and PSR-17 (HTTP Factories) standards.

- [**HTTP Client**](components/http-client.md)

  An HTTP client with continuous navigation support (cookies, sessions, history), implementing PSR-18.

- [**Router**](components/router.md)

  A library to manage HTTP routes with attribute-based definitions, respecting PSR-7.

## Web

- [**Form**](components/form.md)

  A library to manage HTML forms: types, groups, collections, validators and transformers.

- [**HTML Selector**](components/html-selector.md)

  A library to query HTML documents with CSS selectors, like jQuery on the DOM.

- [**Mailer**](components/mailer.md)

  A library for sending emails via SMTP or PHP's built-in `mail()` function.

## Async

- [**Queue Manager**](components/queue-manager.md)

  A queue and worker system supporting multiple backends (Database, Redis, AMQP, AWS SQS, Memory).
