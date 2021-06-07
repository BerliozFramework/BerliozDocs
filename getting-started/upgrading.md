```index
breadcrumb: Getting started; Upgrading
keywords: installation, upgrade
summary-order: 2
```

# Upgrading from version 1

## Breaking changes for Core

- Packages methods signature changed

## Breaking changes for HttpCore

- Namespace `Berlioz\HttpCore` moved to `Berlioz\Http\Core`
- Namespace `Berlioz\Router` moved to `Berlioz\Http\Router`
- Magic methods `_b_pre()` and `_b_post()` of controllers are removed in favor of [middlewares](../http/middleware.md)
- Remove usage of PhpDoc annotations in favor of [PHP 8 attributes](https://www.php.net/manual/language.attributes.php), like [routes annotations](../http/routing.md)
- `ErrorHandler`

## Breaking changes for CliCore

- Namespace `Berlioz\CliCore` moved to `Berlioz\Cli\Core`
- Signature of `CommandInterface` changed