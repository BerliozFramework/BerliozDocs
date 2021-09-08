```index
breadcrumb: Getting started; Upgrading
keywords: installation, upgrade
summary-order: 2
```

# Upgrading from version 1

## Breaking changes

### Breaking changes for Core

- Packages methods signature changed

### Breaking changes for HttpCore

- Namespace `Berlioz\HttpCore` moved to `Berlioz\Http\Core`
- Namespace `Berlioz\Router` moved to `Berlioz\Http\Router`
- Magic methods `_b_pre()` and `_b_post()` of controllers are removed in favor of [middlewares](../http/middleware.md)
- Remove usage of PhpDoc annotations in favor of [PHP 8 attributes](https://www.php.net/manual/language.attributes.php), like [routes annotations](../http/routing.md)
- Method `AbstractController::getService()` renamed to `AbstractController::get()`
- `ErrorHandler`

### Breaking changes for CliCore

- Namespace `Berlioz\CliCore` moved to `Berlioz\Cli\Core`
- Signature of `CommandInterface` changed

## Steps to upgrade

1. Update versions into your composer.json
2. Run command `composer update`
3. Create a directory "resources" at root project directory
4. Move directories "assets" and "templates" into new "resources" directory
5. Update your webpack config if necessary
6. Fixes breaking changes
7. Update files "public/index.php" like [WebsiteSkeleton](https://github.com/BerliozFramework/WebsiteSkeleton/tree/2.x/public) (do not forget .htaccess file)