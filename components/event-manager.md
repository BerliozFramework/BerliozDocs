---
breadcrumb:
  - Components
  - Event Manager
keywords:
  - event
  - dispatcher
  - listener
  - subscriber
  - psr-14
---

# Event Manager

> ℹ️ **Note**: The event manager is part of the **Berlioz** ecosystem but is available as a standalone package:
> [`berlioz/event-manager`](https://github.com/BerliozFramework/EventManager).
> You can find it on [Packagist](https://packagist.org/packages/berlioz/event-manager).
> You can use it independently of the framework, in any PHP application.

**Berlioz Event Manager** is a PHP event manager/dispatcher, respecting PSR-14 (Event Dispatcher) standard.

## Dispatcher

To initialize the event dispatcher:

```php
use Berlioz\EventManager\EventDispatcher;

$dispatcher = new EventDispatcher();
```

To listen an event:

```php
use Berlioz\EventManager\EventDispatcher;

$callback = function($event) {
    // Do something
    return $event;
};

/** @var EventDispatcher $dispatcher */

// A named event
$dispatcher->addEventListener('event.name', $callback);

// Your event object
$dispatcher->addEventListener(MyEvent::class, $callback);
```

To dispatch an event:

```php
/** @var EventDispatcher $dispatcher */
use Berlioz\EventManager\Event\CustomEvent;
use Berlioz\EventManager\EventDispatcher;

// A named event
$dispatcher->dispatch(new CustomEvent('event.name'));

// Your event object
$dispatcher->dispatch(new MyEvent());
```

## Priority

You can define a priority in your listeners. The highest priority is in the first executions.

```php
use Berlioz\EventManager\Listener\ListenerInterface;

/** ... */

// Normal priority (0)
$dispatcher->addEventListener('event.name', $callback, ListenerInterface::PRIORITY_NORMAL);
// High priority (100)
$dispatcher->addEventListener('event.name', $callback, ListenerInterface::PRIORITY_HIGH);
// Low priority (-100)
$dispatcher->addEventListener('event.name', $callback, ListenerInterface::PRIORITY_LOW);
```

The priority argument is an integer ; you can so define your priority with integer value instead of constant.

## Add delegate dispatcher

You can delegate dispatch to another dispatcher who respects PSR-14. The delegated dispatchers are called after, only if
event isn't stopped.

```php
use Berlioz\EventManager\EventDispatcher;

$dispatcher = new EventDispatcher();
$dispatcher->addEventDispatcher(new MyCustomDispatcher());
```

## Add listener provider

You can add listener providers. Providers are called in the order of addition.

```php
use Berlioz\EventManager\EventDispatcher;

$dispatcher = new EventDispatcher();
$dispatcher->addListenerProvider(new MyListenerProvider());
```

## Default listener

The default listener is `\Berlioz\EventManager\Listener\Listener`. You can define your own default provider, he must
implement `\Berlioz\EventManager\Listener\ListenerInterface` interface.

To declare this into the dispatcher:

```php
use Berlioz\EventManager\EventDispatcher;
use Berlioz\EventManager\Provider\ListenerProvider;

$myDefaultProvider = new ListenerProvider();

$dispatcher = new EventDispatcher(defaultProvider: $myDefaultProvider);
```

## Trigger convenience method

The `trigger()` method is a shortcut to dispatch a string-named event without manually constructing a `CustomEvent`:

```php
/** @var EventDispatcher $dispatcher */
$event = $dispatcher->trigger('user.registered', ['userId' => 42]);

// Equivalent to:
$event = $dispatcher->dispatch(new CustomEvent('user.registered', ['userId' => 42]));

// Access data from the event:
$event->getData(); // ['userId' => 42]
```

## Subscribers

Subscribers group multiple listeners in a single class. They are lazily activated: listeners are registered only when a
matching event is dispatched.

Create a subscriber by implementing `SubscriberInterface` or extending `AbstractSubscriber`:

```php
use Berlioz\EventManager\Provider\ListenerProviderInterface;
use Berlioz\EventManager\Subscriber\AbstractSubscriber;

class UserSubscriber extends AbstractSubscriber
{
    protected array $listens = ['user.registered', 'user.deleted'];

    public function subscribe(ListenerProviderInterface $provider): void
    {
        $provider->addEventListener('user.registered', [$this, 'onUserRegistered']);
        $provider->addEventListener('user.deleted', [$this, 'onUserDeleted']);
    }

    public function onUserRegistered($event): void
    {
        // Handle user registration
    }

    public function onUserDeleted($event): void
    {
        // Handle user deletion
    }
}
```

Register a subscriber:

```php
/** @var EventDispatcher $dispatcher */
$dispatcher->addSubscriber(new UserSubscriber());
```

## Advanced dispatch behaviors

Some advanced behaviors are available during dispatch:

- If a listener returns `false`, the propagation is stopped immediately.
- If a listener returns a different event object, it is recursively dispatched and its result is returned.
- If a listener returns the same event object (mutated), subsequent listeners receive the mutated event.
