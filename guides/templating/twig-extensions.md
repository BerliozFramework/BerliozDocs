---
breadcrumb:
  - Guides
  - Templating
  - Extensions
summary-order: ;;2
keywords:
  - templating
  - template
  - twig
---

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

### Preload

`preload()` sends a `Link: <...>; rel=preload` response header for the given link, and returns the finalized link.
Parameters are added to the header (`true` adds the parameter without value, `false` and `null` are ignored).
A link is sent only once per response, and nothing is sent once headers are already sent.

```twig
<link rel="stylesheet" href="{{ preload(asset('website.css'), {as: 'style'}) }}">
<script src="{{ preload('/assets/js/app.js', {as: 'script', crossorigin: true}) }}"></script>
```

The `preload` option of `entrypoints()` preloads every rendered file, with `as=style` for CSS files and `as=script`
for JS files. An array can be given to add parameters to each preload header:

```twig
{{ entrypoints('website', 'css', {preload: true}) }}
{{ entrypoints('website', 'js', {preload: {crossorigin: true}, defer: true}) }}
```

> 🆕 **Info**: *Since version 3.3.3*
>
> Every file rendered by `entrypoints()` is preloaded (previously only the first one), and the `Link` header is sent on
> every response. The HTTP/2 Server Push cache (`h2pushes` cookies) has been removed.

> ⚠️ **Deprecated**: *Since version 3.3.3*
>
> The `nopush` parameter is deprecated, HTTP/2 Server Push is no longer supported by browsers. It will be removed in
> version 4.

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
- `finalize_path()`