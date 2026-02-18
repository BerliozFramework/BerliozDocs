---
breadcrumb:
  - Getting started
  - Upgrading
keywords:
  - installation
  - upgrade
  - migration
summary-order: 2
---

# Upgrading

## Upgrading from version 2

Version 3.0 marks the transition to a **monorepo** structure. All Berlioz components are now developed, tested and
released together from a single repository.

### What changed

#### Monorepo structure

All Berlioz packages are now maintained in a single repository:
[github.com/BerliozFramework/Berlioz](https://github.com/BerliozFramework/Berlioz).

- The individual component repositories (Config, Router, ServiceContainer, etc.) are now **read-only mirrors**
  automatically synchronized via subsplits.
- **Issues and pull requests** should be submitted to the main monorepo.
- Composer packages remain the same (`berlioz/config`, `berlioz/router`, etc.) and can still be required individually.

#### PHP 8.2 minimum

All components now require **PHP ^8.2** (previously ^8.0 or ^7.1 for some).

#### Updated PSR dependencies

| Package              | Version 2.x  | Version 3.x      |
|----------------------|---------------|-------------------|
| `psr/http-message`   | ^1.0          | ^2.0              |
| `psr/container`      | ^1.0 \|\| ^2.0 | ^2.0            |
| `psr/log`            | ^1.0 \|\| ^2.0 | ^2.0 \|\| ^3.0 |
| `psr/http-factory`   | ^1.0          | ^1.0 \|\| ^2.0   |
| `psr/simple-cache`   | ^1.0 \|\| ^2.0 | ^2.0 \|\| ^3.0 |

> **Note:** The `psr/http-message` upgrade from v1 to v2 introduces return types on interface methods. If you have
> custom implementations of PSR-7 interfaces, you will need to add the appropriate return types.

#### Removed package: Atlas ORM

The `berlioz/atlas-package` has been removed in version 3.0. If you were using Atlas ORM, you will need to migrate
to [Hector ORM](../guides/orm/hector.md) or manage your ORM integration manually.

#### No other API breaking changes

Beyond the dependency updates and Atlas removal above, version 3.0 does **not** introduce any breaking changes in the
Berlioz API itself. All namespaces, class names, method signatures and behaviors remain the same as in version 2.x.

### Steps to upgrade

1. **Update your `composer.json`** to require version `^3.0` for Berlioz packages, or require the monorepo directly:

   ```json
   {
       "require": {
           "berlioz/berlioz": "^3.0"
       }
   }
   ```

2. **Update PHP** to version 8.2 or higher if not already done.

3. **Run `composer update`** and fix any dependency conflicts.

4. **Check your PSR implementations**: if you have custom classes implementing PSR-7 (`MessageInterface`,
   `RequestInterface`, etc.), add the required return types introduced in `psr/http-message` v2.

5. **Test your application** thoroughly. No other code changes should be required.

---

## Upgrading from version 1

### Breaking changes

#### Breaking changes for Core

- Packages methods signature changed

#### Breaking changes for HttpCore

- Namespace `Berlioz\HttpCore` moved to `Berlioz\Http\Core`
- Namespace `Berlioz\Router` moved to `Berlioz\Http\Router`
- Magic methods `_b_pre()` and `_b_post()` of controllers are removed in favor of [middlewares](../http/middleware.md)
- Remove usage of PhpDoc annotations in favor of [PHP 8 attributes](https://www.php.net/manual/language.attributes.php),
  like [routes annotations](../http/routing.md)
- Method `AbstractController::getService()` renamed to `AbstractController::get()`
- `ErrorHandler`

#### Breaking changes for CliCore

- Namespace `Berlioz\CliCore` moved to `Berlioz\Cli\Core`
- Signature of `CommandInterface` changed

### Steps to upgrade

1. Update versions into your composer.json
2. Run command `composer update`
3. Create a directory "resources" at root project directory
4. Move directories "assets" and "templates" into new "resources" directory
5. Update your webpack config if necessary
6. Fixes breaking changes
7. Update files "public/index.php"
   like [WebsiteSkeleton](https://github.com/BerliozFramework/WebsiteSkeleton/tree/2.x/public) (do not forget .htaccess
   file)
