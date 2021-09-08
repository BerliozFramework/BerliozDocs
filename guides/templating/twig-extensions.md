```index
breadcrumb: Guides; Templating; Extensions
summary-order: ; ; 2
keywords: templating, template, twig
```

# Twig extensions

3 extensions are provided with Twig Package :

- `Berlioz\Package\Twig\Extension\AssetExtension`
- `Berlioz\Package\Twig\Extension\DefaultExtension`
- `Berlioz\Package\Twig\Extension\RouterExtension`

## Extension `AssetExtension`

Functions provided:

- `asset()`
- `entrypoints()`
- `entrypoints_list()`
- `preload()`

## Extension `DefaultExtension`

Filters provided:

- `date_format`
- `truncate`
- `nl2p`
- `human_file_size`
- `json_decode`
- `basename`

Tests provided:

- `instance of`

## Extension `RouterExtension`

Functions provided:

- `path()`
- `path_exists()`