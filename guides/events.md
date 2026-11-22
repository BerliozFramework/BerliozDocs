---
breadcrumb:
  - Guides
  - Events
summary-order: ;4
---

# Events

**Berlioz framework** provides an event dispatcher. It is accessible with the `Core::getEventDispatcher()` method or
from the container.

## Listen to an event

### With Subscriber

You can create a subscriber to listen to some events. The subscriber class must implement the
`\Berlioz\EventManager\Subscriber\SubscriberInterface` interface.

To help you, you can use the `\Berlioz\EventManager\Subscriber\AbstractSubscriber` abstract class. This class provides
a dynamic `listens()` method.

Example:

```php
use Berlioz\EventManager\Provider\ListenerProviderInterface;
use Berlioz\EventManager\Subscriber\AbstractSubscriber;

class MySubscriber extends AbstractSubscriber
{
    protected array $listens = [
        'anEvent',        
        'aSecondEvent',        
    ];

    /**
     * @inheritDoc
     */
    protected function subscribe(ListenerProviderInterface $provider): void
    {
        $provider->addEventListener('anEvent', [$this, 'doSomething']);
        $provider->addEventListener('aSecondEvent', [$this, 'doSomethingElse']);
    }

    public function doSomething($event): void
    {
        // ...
    }

    public function doSomethingElse($event): void
    {
        // ...
    }
}
```

Subscribers must be declared in the configuration:

```json
{
    "events": {
        "subscribers": [
            "MySubscriber"
        ]
    }
}
```

> ℹ️ **Note**: It's recommended to use subscribers to listen to events, and to split them by task.
> In this case, only the necessary listeners are declared to the dispatcher.

### Without Subscriber

If you want to listen to an event without a subscriber, you need to declare the event and the callback in the
configuration:

```json
{
    "events": {
        "listeners": {
            "AnEvent": "MyWebsite\\Event\\MyListener::doSomething",
            "ASecondEvent": [
                "MyWebsite\\Event\\MyListener::doSomethingElse",
                {
                    "callback": "MyWebsite\\Event\\MyListener::doSomethingElse",
                    "priority": 10
                }
            ]
        }
    }
}
```

> ℹ️ **Note**: Prefer subscribers to get the best performance.

## Framework events

### Terminate

> 🆕 **Info**: *Since version 3.4*

A `Berlioz\Core\Event\TerminateEvent` is dispatched once, when the application is terminated:

- `Berlioz\Http\Core\Event\HttpTerminateEvent`, after the HTTP response has been sent (gives the request and the
  response metadata);
- `Berlioz\Cli\Core\Event\CliTerminateEvent`, at the end of a CLI command (gives the returned exit code, or `null` if
  command handling did not complete normally).

Listen to `TerminateEvent` to be called in both contexts. See [After the response](../http/terminate.md).

## Trigger an event

You can trigger a specific event with the `trigger` method. It dispatches a
`\Berlioz\EventManager\Event\CustomEvent` with the given name and data, and returns it.

```php
/** @var \Berlioz\Core\Event\EventDispatcher $eventDispatcher */
$eventDispatcher->trigger('myEvent', []);
```

## Dispatch an event

Your application can dispatch events with the `dispatch` method of the event dispatcher.

You can use the `\Berlioz\EventManager\Event\CustomEvent` class to generate simple events.

Example:

```php
use Berlioz\EventManager\Event\CustomEvent;

$myEvent = new CustomEvent('myEventName', ['foo' => 'bar']);

/** @var \Berlioz\Core\Event\EventDispatcher $eventDispatcher */
$eventDispatcher->dispatch($myEvent);

// ...all listeners whose listens 'myEventName' will be triggered
```
