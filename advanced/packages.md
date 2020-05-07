```index
breadcrumb: Advanced usage; Package creation
summary-order: 3
description: Create package to make implementation of functionnalities for others peoples
```

# Creation of package

## Package class

First step is to create a class who implements `Berlioz\Core\Package\PackageInterface` interface.
To simplify your creation, you can extends `Berlioz\Core\Package\AbstractPackage` class.

Representation of interface:

```php
/**
 * Interface PackageInterface.
 *
 * @package Berlioz\Core\Package
 */
interface PackageInterface extends CoreAwareInterface
{
    /**
     * Register package.
     *
     * Method called for the registration of all packages.
     * Do not use this method to do any actions on framework, only configuration and registration of services.
     *
     * @return mixed
     */
    public function register();

    /**
     * Init package.
     *
     * Method called after creation of all packages.
     *
     * @return mixed
     */
    public function init();
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