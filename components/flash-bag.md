---
breadcrumb:
  - Components
  - Flash Bag
keywords:
  - flash
  - messages
  - session
---

# Flash Bag

> ℹ️ **Note**: The flash bag is part of the **Berlioz** ecosystem but is available as a standalone package:
> [`berlioz/flash-bag`](https://github.com/BerliozFramework/FlashBag).
> You can find it on [Packagist](https://packagist.org/packages/berlioz/flash-bag).
> You can use it independently of the framework, in any PHP application.

**Berlioz FlashBag** is a PHP library to manage flash messages to show to the user.

All messages are stored in session of user. So you be able to get the messages after a reload of page or redirect.
When you got the messages, they are deleted on the stack and no longer available.

## Add message

It's very simple to add messages:

```php
$flashBag = new FlashBag;
$flashBag
    ->add(FlashBag::TYPE_SUCCESS, 'Message success')
    ->add(FlashBag::TYPE_INFO, 'Second message')
    ->add(FlashBag::TYPE_INFO, 'Third message for %d %s', 3, 'persons');
```

Some default types are available in constants:

```php
FlashBag::TYPE_INFO = 'info';
FlashBag::TYPE_SUCCESS = 'success';
FlashBag::TYPE_WARNING = 'warning';
FlashBag::TYPE_ERROR = 'error';
```

## Get message

To get message, it's also simple then add:

```php
$flashBag = new FlashBag;
$successMessages = $flashBag->get('success');

foreach ($successMessages as $msg) {
    print $msg;
}
```

## Get all messages

You can also get all messages in one time:

```php
$flashBag = new FlashBag;
$allMessages = $flashBag->all();

foreach ($allMessages as $type => $messages) {
    foreach ($messages as $msg) {
        print sprintf('%s: %s', $type, $msg);
    }
}
```

> **Note:** Both `get()` and `all()` are destructive — messages are removed from the bag after retrieval.

## Clear messages

You can clear messages without retrieving them:

```php
$flashBag = new FlashBag;
$flashBag->clear();           // Clear all messages
$flashBag->clear('success');  // Clear only success messages
```

## Count messages

`FlashBag` implements `Countable`, so you can count the total number of messages:

```php
$flashBag = new FlashBag;
$total = count($flashBag); // Total number of messages across all types
```
