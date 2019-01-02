<meta name="docparser-index" content="Basic uses; Hierarchy of directories" />
<meta name="docparser-index-order" content="0" />
<meta name="docparser-index-visible" content="false" />

# Hierarchy of directories

We recommend to uses the default hierarchy of directories for your projects.

## Default hierarchy

Default hierarchy of directories:

- `/config`: configurations of project
- `/public`: public directory where your web server link visitors
- `/src`: your project sources
- `/templates`: your templates files
- `/var`
  - `/var/cache`: cache directory
  - `/var/debug`: debug reports directory
  - `/var/tmp`: temporary directory for your project
- `/vendor`: vendors (for composer)

It's a basic hierarchy that lot of frameworks used. You will not loose if you come for one of them.

## Custom hierarchy

You are able to change default hierarchy by your own.
To do that, you have two choices:

- Implements `Berlioz\Core\Directories\DirectoriesInterface` interface
- Extends `Berlioz\Core\Directories\DefaultDirectories` class

Representation of interface:

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

Give your object in first argument of `Berlioz\Core\Core` object.