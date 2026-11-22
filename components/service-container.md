---
breadcrumb:
  - Components
  - Service Container
keywords:
  - service
  - container
  - dependency injection
  - psr-11
  - instantiator
---

# Service Container

> ℹ️ **Note**: The service container is part of the **Berlioz** ecosystem but is available as a standalone package:
> [`berlioz/service-container`](https://github.com/BerliozFramework/ServiceContainer).
> You can find it on [Packagist](https://packagist.org/packages/berlioz/service-container).
> You can use it independently of the framework, in any PHP application.

**Berlioz Service Container** is a PHP library to manage your services with dependency injection, respecting the
PSR-11 (Container interface) standard.

## Container

Methods available from PSR-11:

- `get($id)`

  > PSR-11: Finds an entry of the container by its identifier and returns it.
  >
  > **Accept a class name.**

- `has($id)`

  > PSR-11: Returns true if the container can return an entry for the given identifier.
  > Returns false otherwise.

## Add a service

You can add a service with the `Container::add()` method.

```php
use Berlioz\ServiceContainer\Container;

$container = new Container();
$service = $container->add(MyService::class, 'alias'); // Returns a Berlioz\ServiceContainer\Service\Service object
```

Or with the `Container::addService()` method, which accepts a `Service` object.

```php
use Berlioz\ServiceContainer\Container;
use Berlioz\ServiceContainer\Service\Service;

$container = new Container();
$service = new Service(MyService::class, 'alias');
$container->addService($service);
```

## Service object

- ```
  Service::public function __construct(
      string|object $class,
      ?string $alias = null,
      callable|string|null $factory = null,
      ?CacheStrategy $cacheStrategy = null,
  )
  ```

  > Constructor of the Service object.

- `Service::setNullable(bool $nullable = true): Service`

  > The service can be null after factory execution (false by default).

- `Service::setShared(bool $shared = true): Service`

  > Share a service: the same instance is always returned for a shared service.

- `Service::addProvide(string ...$provide): Service`

  > Add classes, interfaces or aliases provided by the service.

- `Service::addArgument(string $name, mixed $value): Service`

  > Add an argument used to make an instance of the service class.

- `Service::addArguments(array $arguments): Service`

  > An array of arguments: the key must be the name of the argument, and the value must be the argument value.

- `Service::addCall(string $method, array $arguments = []): Service`

  > Method name (and arguments) called just after the construction of the object.

- `Service::addCalls(array $calls = []): void`

  > An array of calls: the key must be the name of the called method, and the value an array of arguments.

- `Service::setFactory(string $factory): Service`

  > The static factory method used to make the object.
  >
  > Example: `MyProject\Name\Space\MyFactory::service`

## Auto-wiring

Auto-wiring allows the container to automatically resolve and instantiate classes without explicit registration:

```php
use Berlioz\ServiceContainer\Container;

$container = new Container();
$container->autoWiring(); // Enable auto-wiring

// Now you can get any instantiable class without registering it:
$myService = $container->get(MyService::class);
```

Auto-wired instances are cached: the same instance is returned for any class or interface it implements. Auto-wiring has
the lowest priority — explicitly registered services and providers are always preferred.

> 🆕 **Info**: *Since version 3.1*
>
> **Union types** are supported: when a constructor parameter has a union type (e.g., `FooInterface|BarClass`), the
> container tries each type in order until one resolves successfully.
>
> **Intersection types** and **DNF types** (e.g., `Countable&Iterator`) are gracefully skipped by auto-wiring; if the
> parameter has a default value, that value is used instead.

To disable auto-wiring:

```php
$container->autoWiring(false);
```

## Container-aware interface

Any service implementing `ContainerAwareInterface` will automatically receive the container via an inflector:

```php
use Berlioz\ServiceContainer\ContainerAwareInterface;
use Berlioz\ServiceContainer\ContainerAwareTrait;

class MyService implements ContainerAwareInterface
{
    use ContainerAwareTrait;

    public function doSomething(): void
    {
        $other = $this->getContainer()->get(OtherService::class);
    }
}
```

Similarly, `InstantiatorAwareInterface` provides access to the `Instantiator`.

Both interfaces are automatically inflected — you never need to call `setContainer()` or `setInstantiator()` manually.

## Service argument references

When defining service arguments, string values starting with `@` are resolved as service references:

```php
$service = $container->add(MyService::class);
$service->addArgument('logger', '@Psr\Log\LoggerInterface');
// Resolves to $container->get('Psr\Log\LoggerInterface') at injection time
```

## Instantiator

New instance of a class or object:

```php
use Berlioz\ServiceContainer\Instantiator;

$instantiator = new Instantiator();
$object = $instantiator->newInstanceOf(
    MyClass::class,
    [
        'argument1' => 'Value',
        'argument3' => 'Value',
        'argument2' => 'Value'
    ]
);
```

Invoke a method:

```php
use Berlioz\ServiceContainer\Instantiator;

$instantiator = new Instantiator();
$instantiator->invokeMethod(
    $myObject,
    'myMethodName',
    [
        'argument1' => 'Value',
        'argument3' => 'Value',
        'argument2' => 'Value'
    ]
);
```

Invoke a function:

```php
use Berlioz\ServiceContainer\Instantiator;

$instantiator = new Instantiator();
$instantiator->invokeFunction(
    'myFunctionName',
    [
        'argument1' => 'Value',
        'argument3' => 'Value',
        'argument2' => 'Value'
    ]
);
```

In all these examples, the last argument is an array of parameters to give to the constructor, method or function. The
order of arguments is not important.

If a parameter is an object, the system gets it from the container or tries to instantiate the class.

The `Container::call()` method calls the right method of the instantiator according to the given value:

```php
use Berlioz\ServiceContainer\Container;

$container = new Container();
$container->call(fn() => 'test'); // Call closure
$container->call(MyClass::class); // Instantiate the class
$container->call(MyClass::method); // Call static method or instantiate the class and call method 
```

## Inflector

The inflector is useful if you want to inject some dependencies through methods declared by an interface.

```php
use Berlioz\ServiceContainer\Container;
use Berlioz\ServiceContainer\Inflector\Inflector;

$inflector = new Inflector(
    MyInterface::class, // Interface implemented by object
    'setFoo', // Method to call
    [/*...*/] // Arguments
);
$container = new Container();
$container->addInflector($inflector);
```

## Service provider

In some cases, such as performance constraints, you need to add a service provider.

A service provider needs to implement the `\Berlioz\ServiceContainer\Provider\ServiceProviderInterface` interface.
An abstract class `\Berlioz\ServiceContainer\Provider\AbstractServiceProvider` can help you.

Example of a service provider:

```php
use Berlioz\ServiceContainer\Container;
use Berlioz\ServiceContainer\Provider\AbstractServiceProvider;

class MyServiceProvider extends AbstractServiceProvider
{
    // Declare services class and alias
    protected array $provides = [stdClass::class, 'service'];
    
    public function boot(Container $container) : void
    {
        // This method is called when provider is added to container.
        // Add inflectors here.
    }
    
    public function register(Container $container) : void
    {
         // Add services here
         $container->add(stdClass::class, 'service');
    }
}
```

Add your service provider:

```php
use Berlioz\ServiceContainer\Container;

$container = new Container();
$container->addProvider(new MyServiceProvider());

$container->has('service'); // Returns TRUE
$container->get('service'); // Returns an `stdClass` instance
```
