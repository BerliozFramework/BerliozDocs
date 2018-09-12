# Packages

## Installation of a package

1. Use composer to install package:

    ```bash
    composer require berlioz/my-package
    ```

2. If you have a `packages.json` file in your [configuration directory](../directories.md), go to 4.
   Else, create a file named `packages.json` in your [configuration directory](../directories.md) and add the default content:

   ```json
   {
       "packages": []
   }
   ```

3. Add package to the `packages.json` file of your project, looks like:

   ```json
   {
       "packages": [
           "\\Berlioz\\Package\\MyPackage\\ClassOfMyPackage"
       ]
   }
   ```

4. Finished!