```index
breadcrumb: Guides; Events
summary-order: ; 4
```

# Events

**Berlioz framework** provides an event dispatcher. He's accessible by method `Core::getEventDispatcher()` or by
container.

## Listen an event

### With Subscriber

You can create a subscriber to listen some events. Subscriber class must
implement `\Berlioz\EventManager\Subscriber\SubscriberInterface` interface.

You can help you of `\Berlioz\EventManager\Subscriber\AbstractSubscriber` abstract class. This class provides
dynamic `listens()` method.

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

Subscribers must be declared in configuration:

```json
{
  "events": {
    "subscribers": [
      "MySubscriber"
    ]
  }
}
```

> Info!
>
> It's recommended to use subscribers to listen events, and explode by work.
> In this case, only necessaries listeners are declared to the dispatcher.

### Without Subscriber

If you want listen event without subscriber, you need to declare event and callback into the configuration:

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

> Info!
>
> Prefers subscribers to have the best performances.

## Trigger an event

You can trigger a specific event with `trigger` method.

```php
/** @var \Berlioz\Core\Event\EventDispatcher $eventDispatcher */
$eventDispatcher->trigger('myEvent', []);
```

> Warning!
>
> The original event can contain some necessary data.

## Dispatch an event

Your application can dispatch events with method `dispatch` of event dispatcher.

You can use `\Berlioz\EventManager\Event\CustomEvent` class to generate simple events.

Example:

```php
use Berlioz\EventManager\Event\CustomEvent;

$myEvent = new CustomEvent('myEventName', ['foo' => 'bar']);

/** @var \Berlioz\Core\Event\EventDispatcher $eventDispatcher */
$eventDispatcher->dispatch($myEvent);

// ...all listeners whose listens 'myEventName' will be triggered
```