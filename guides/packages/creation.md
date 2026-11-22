---
breadcrumb:
  - Guides
  - Packages
  - Creation
summary-visible: false
---

# Creating a package

## Package class

The first step is to create a class that implements the `Berlioz\Core\Package\PackageInterface` interface.
To simplify the creation, you can extend the `Berlioz\Core\Package\AbstractPackage` class.

Representation of the interface:

```php
use Berlioz\Config\ConfigInterface;
use Berlioz\Config\Exception\ConfigException;
use Berlioz\Core\Core;
use Berlioz\ServiceContainer\Container;

/**
 * Interface PackageInterface.
 */
interface PackageInterface
{
    /**
     * Package configuration.
     *
     * Method called for the configuration of package.
     * Do not use this method to do any actions on framework, only configuration of package.
     *
     * @return ConfigInterface|null
     * @throws ConfigException
     */
    public static function config(): ?ConfigInterface;

    /**
     * Register package.
     *
     * Method called for the registration of services associated to the package.
     * Do not use this method to do any actions on framework, only registration of services.
     *
     * @param Container $container
     *
     * @return void
     */
    public static function register(Container $container): void;

    /**
     * Boot package.
     *
     * Method called after creation of all packages.
     *
     * @param Core $core
     *
     * @return void
     */
    public static function boot(Core $core): void;
}
```

## Declaration in `composer.json` file

The second step is to declare in your `composer.json` file that the project is a Berlioz package.

So add `"type": "berlioz-package"` to your `composer.json` file. Then add the declaration of your package class in the
"config" section.

Full example of your `composer.json` file:

```json
{
    "name": "project/my-berlioz-package",
    "type": "berlioz-package",
    ...
    "config": {
        "berlioz": {
            "package": "My\\Project\\BerliozPackage"
        }
    }
}
```
