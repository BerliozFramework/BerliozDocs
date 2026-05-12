---
breadcrumb:
  - Guides
  - Maintenance mode
keywords:
  - maintenance
summary-order: ;7
---

# Maintenance mode

A middleware treat the maintenance of websites.

## Enable / Disable

Maintenance is disabled by default.
To enable it, set config parameter `berlioz.maintenance` to `true`:

```json
{
  "berlioz": {
    "maintenance": true
  }
}
```

## Plan a maintenance

To plan a maintenance, define the configuration like this:

```json
{
  "berlioz": {
    "maintenance": {
      "start": "2021-06-08 00:00:00",
      "end": "2021-06-08 12:00:00",
      "message": "My maintenance message",
      "handler": "MyHandlerClass"
    }
  }
}
```

All options are optional.

> 🆕 **Info**: *Since version 3.1*
>
> When `start` and/or `end` are specified, maintenance mode is only active **between** those dates. Before `start` or
> after `end`, the application continues to serve requests normally.
>
> - If only `start` is set, maintenance begins at that time and continues indefinitely.
> - If only `end` is set, maintenance is active immediately until that time.
> - If neither is set (or `maintenance` is simply `true`), maintenance is always active.

The default handler class: `\Berlioz\Http\Core\Http\Handler\MaintenanceHandler`.
Like [HTTP errors](../http/error-handler.md), you can set your own handler to display a maintenance page.

The maintenance object is accessible in the application object: `\Berlioz\Http\Core\App\HttpApp::getMaintenance()`