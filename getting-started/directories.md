```index
breadcrumb: Getting started; Directories
```

# Hierarchy of directories

We recommend using the default hierarchy of directories for your projects.

## Default hierarchy

Default hierarchy of directories:

Path | Description
-----|------------
`/config` | Configurations of project
`/public` | Public directory where your web server link visitors
`/src` | Your project sources
`/resources` | Resources of project
`/resources/assets` | Assets of project
`/resources/templates` | Your templates files
`/var` | Var files
`/var/cache` | Cache directory
`/var/debug` | Debug reports directory
`/var/tmp` | Temporary directory for your project
`/vendor` | Vendors (for composer)

It's a basic hierarchy that lot of frameworks used. You won't be lost if you come for any of them :).

## Custom hierarchy

You are able to change default hierarchy by your own.
To do that, you have two choices:

- Implements `Berlioz\Core\Directories\DirectoriesInterface` interface
- Extends `Berlioz\Core\Directories\DefaultDirectories` class

Representation of the interface:

```php
/**
 * Interface DirectoriesInterface.
 *
 * @package Berlioz\Core\Directories
 */
interface DirectoriesInterface
{
    /**
     * Get working directory.
     *
     * @return string
     */
    public function getWorkingDir(): string;

    /**
     * Get app directory.
     *
     * Find last composer.json file.
     *
     * @return string
     */
    public function getAppDir(): string;

    /**
     * Get config directory.
     *
     * @return string
     */
    public function getConfigDir(): string;

    /**
     * Get var directory.
     *
     * @return string
     */
    public function getVarDir(): string;

    /**
     * Get cache directory.
     *
     * @return string
     */
    public function getCacheDir(): string;

    /**
     * Get log directory.
     *
     * @return string
     */
    public function getLogDir(): string;

    /**
     * Get debug directory.
     *
     * @return string
     */
    public function getDebugDir(): string;

    /**
     * Get vendor directory.
     *
     * @return string
     */
    public function getVendorDir(): string;
}
```

Pass your object in first argument of `Berlioz\Core\Core` object.