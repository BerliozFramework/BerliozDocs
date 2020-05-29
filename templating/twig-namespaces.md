```index
breadcrumb: Templating; Namespaces
keywords: templating
```

# Twig namespaces

You can use the twig namespace to separate templates or parts of your project.

## Declaration

Paths need to be declared in your configuration file like this:

```json
{
    "twig": {
        "paths": {
            "foo": "%berlioz.directories.templates%/foo",
            "bar": "%berlioz.directories.templates%/bar"
        }
    }
}
```

Namespaces can be accessed by this notation:

```php
$this->render('@foo/index.html', []);
$this->render('@bar/index.html', []);
```
