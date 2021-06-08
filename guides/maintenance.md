```index
breadcrumb: Guides; Maintenance mode
summary-order: ; 4
```

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

All options are optionals.

The default handler class: `\Berlioz\Http\Core\Http\Handler\MaintenanceHandler`.
Like [HTTP errors](../http/error-handler.md), you can set your own handler to display a maintenance page.

The maintenance object is accessible in the application object: `\Berlioz\Http\Core\App\HttpApp::getMaintenance()`