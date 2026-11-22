---
breadcrumb:
  - Getting started
  - Directories
---

# Hierarchy of directories

We recommend using the default hierarchy of directories for your projects.

## Default hierarchy

Default hierarchy of directories:

 Path                   | Description                                          
------------------------|------------------------------------------------------
 `/config`              | Configuration of the project                         
 `/public`              | Public directory your web server serves to visitors  
 `/src`                 | Your project sources                                 
 `/resources`           | Resources of the project                             
 `/resources/assets`    | Assets of the project                                
 `/resources/templates` | Your template files                                  
 `/var`                 | Var files                                            
 `/var/cache`           | Cache directory                                      
 `/var/debug`           | Debug reports directory                              
 `/var/tmp`             | Temporary directory for your project                 
 `/vendor`              | Vendors (for Composer)                               

It's a basic hierarchy that a lot of frameworks use. You won't be lost if you come from any of them :).

The `/var/cache` directory is used by the default file cache. See the [Cache guide](../guides/cache.md) to learn how to
configure other cache drivers (memory, Redis, fallback…).

## Custom hierarchy

You can replace the default hierarchy with your own.
To do that, you have two choices:

- Implement the `Berlioz\Core\Directories\DirectoriesInterface` interface
- Extend the `Berlioz\Core\Directories\DefaultDirectories` class

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

Pass your object as the first argument of the `Berlioz\Core\Core` object.
