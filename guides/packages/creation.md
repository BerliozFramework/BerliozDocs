```index
breadcrumb: Guides; Packages; Creation
summary-visible: false
```

# Creation of package

## Package class

First step is to create a class who implements `Berlioz\Core\Package\PackageInterface` interface.
To simplify your creation, you can extend `Berlioz\Core\Package\AbstractPackage` class.

Representation of interface:

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

Second step is to declare in your `composer.json` file, that the project is a Berlioz package.

So add `"type": "berlioz-package"` in your `composer.json` file. And add in "config" section the declaration of your package class.

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