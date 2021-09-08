```index
breadcrumb: Guides; Packages
summary-order: ; 5
```

# Packages

Packages are available to simplify the integration of some functionality in project without fastidious configuration.

They are also the specificity to be able to used instead of middleware with implementation of **PSR-15**.

## Installation of a package

If the package respect [our recommendations](packages/creation.md), the installation is automatic ; but you can also declare package manually.

## Manual installation of a package

1. Use composer to install package:

    ```bash
    composer require berlioz/my-package
    ```

2. If you have a `packages.json` file in your [configuration directory](../getting-started/directories.md), go to 4.
   Else, create a file named `packages.json` in your [configuration directory](../getting-started/directories.md) and add the default content:

   ```json
   {
       "packages": []
   }
   ```

3. Add package to the `packages.json` file of your project, looks like:

   ```json
   {
       "packages": [
           "Berlioz\\Package\\MyPackage\\ClassOfMyPackage"
       ]
   }
   ```

4. Finished!