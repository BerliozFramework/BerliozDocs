---
breadcrumb:
  - Getting started
  - Service container
keywords:
  - service
  - container
---

# Service container

The default service container of Berlioz Framework is the package
[**berlioz/service-container**](../components/service-container.md).

**Berlioz Service Container** is a PHP library to manage your services with dependencies injection, respecting PSR-11 (
Container interface) standard.

## Configuration

Declare your application services in a [service provider](../components/service-container.md#service-provider), and
register the provider in the configuration with the `container.providers` key.

```json
{
  "container": {
    "providers": [
      "App\\Container\\AppServiceProvider"
    ]
  }
}
```

```php
namespace App\Container;

use Berlioz\ServiceContainer\Container;
use Berlioz\ServiceContainer\Provider\AbstractServiceProvider;
use Berlioz\ServiceContainer\Service\Service;
use MyService\ClassName;

class AppServiceProvider extends AbstractServiceProvider
{
    // Classes and aliases provided by this provider
    protected array $provides = [ClassName::class, 'myServiceAlias'];

    public function register(Container $container): void
    {
        $service = new Service(ClassName::class, 'myServiceAlias');
        $service->addArguments(['foo' => 'value', 'bar' => 'value']);
        $service->addCall('myMethodName', ['baz' => 'value', 'qux' => 'value']);

        $container->addService($service);
    }
}
```

In the example, when you get service `myServiceAlias` or `MyService\ClassName::class`, the instantiator will:

- Instantiate class with constructor arguments `foo` and `bar`
- Call method `myMethodName` with arguments `baz` and `qux`, after instantiation

Providers listed in `container.providers` are instantiated by the container, so their constructor can receive
services like `Berlioz\Core\Core` by auto-wiring.

### Migrating from `container.services`

> ⚠️ **Warning**: Declaring services in the `container.services` configuration key is deprecated since version 3.4
> and triggers an `E_USER_DEPRECATED` notice. It will be removed in version 4. `container.providers` is not
> deprecated.

Each entry of `container.services` becomes a service declared in the `register()` method of a provider. The class
and the alias of each service, and its `provides` values, must also be listed in the `$provides` property of the
provider.

The examples below show the configuration before, then the `register()` method after migration.

#### Class or alias

```json
{
  "container": {
    "services": {
      "mailer": "App\\Service\\Mailer"
    }
  }
}
```

```php
public function register(Container $container): void
{
    $container->add(Mailer::class, 'mailer');
}
```

#### Factory, arguments and calls

Argument values keep the same syntax, including references to other services like `@alias`.

```json
{
  "container": {
    "services": {
      "client": {
        "class": "App\\Service\\Client",
        "factory": "App\\Service\\ClientFactory::create",
        "arguments": {
          "baseUri": "https://example.com",
          "logger": "@logger"
        },
        "calls": [
          {
            "method": "setTimeout",
            "arguments": {
              "timeout": 30
            }
          }
        ]
      }
    }
  }
}
```

```php
public function register(Container $container): void
{
    $service = new Service(
        class: Client::class,
        alias: 'client',
        factory: ClientFactory::class . '::create',
    );
    $service->addArguments(['baseUri' => 'https://example.com', 'logger' => '@logger']);
    $service->addCall('setTimeout', ['timeout' => 30]);

    $container->addService($service);
}
```

#### Provides

```json
{
  "container": {
    "services": {
      "storage": {
        "class": "App\\Storage\\LocalStorage",
        "provides": ["App\\Storage\\StorageInterface"]
      }
    }
  }
}
```

```php
protected array $provides = [LocalStorage::class, 'storage', StorageInterface::class];

public function register(Container $container): void
{
    $container
        ->add(LocalStorage::class, 'storage')
        ->addProvide(StorageInterface::class);
}
```

#### Cache

The `cache` option used the Core cache with a TTL: an ISO 8601 duration string, a number of seconds, or `true`
for no expiration. Create a `CacheStrategy` with the Core cache, injected in the provider constructor.

```json
{
  "container": {
    "services": {
      "catalog": {
        "class": "App\\Catalog\\Catalog",
        "cache": "PT1H"
      }
    }
  }
}
```

```php
use Berlioz\Core\Core;
use Berlioz\ServiceContainer\Service\CacheStrategy;
use DateInterval;

class AppServiceProvider extends AbstractServiceProvider
{
    protected array $provides = [Catalog::class, 'catalog'];

    public function __construct(private Core $core)
    {
    }

    public function register(Container $container): void
    {
        $container->addService(
            new Service(
                class: Catalog::class,
                alias: 'catalog',
                cacheStrategy: new CacheStrategy($this->core->getCache(), new DateInterval('PT1H')),
            )
        );
    }
}
```

#### Per-environment declarations

If a configuration file of a given environment (like a local `config.json` not committed) overrides
`container.services`, use one of these options:

- Create a dedicated provider, and list it in `container.providers` of that configuration file only.
- Check the environment in the provider with `Core::getEnv()`, which returns the `berlioz.environment` value:

  ```php
  public function register(Container $container): void
  {
      if (Core::ENV_DEV === $this->core->getEnv()) {
          $container->add(FakeMailer::class, 'mailer');
          return;
      }

      $container->add(Mailer::class, 'mailer');
  }
  ```

## Usage

Service container is accessible from the application object with method `getContainer()`.

Remember the 3 main methods:

- `get($id)`

  > PSR-11: Finds an entry of the container by its identifier and returns it.
  >
  > **Accept a class name directly.**

- `has($id)`

  > PSR-11: Returns true if the container can return an entry for the given identifier.
  > Returns false otherwise.

- `call(string|array|Closure $subject, array $arguments = [])`

  > Returns the subject with dependencies injection functionality.
  > Go next on the page to know more.

Others methods of **Service Container** in many cases, are not necessaries for projects. If you want to know more, go
on [component documentation](../components/service-container.md).

> 💡 **Tip**: You can get service quickly from `Controller` classes with method: `Controller::get($id)`.

## Instantiator

**Berlioz Framework** give you an **Instantiator** class that you can use to do dependency injection with object,
methods or functions.

In all next examples cases, the last argument is an array of parameters to give to the constructor, method or function.
The order of arguments is not important.

In case of a parameter is an object, the system get the classes implemented to inject in good parameter.

### New instance of a class or object

You can create a new instance of a class.

```php
$object = $this->getCore()->getContainer()->call(
    MyClass::class,
    [
        'argument1' => 'Value',
        'argument3' => 'Value',
        'argument2' => 'Value'
    ]
);
```

### Invoke a method

You can invoke a specific method of an object.

```php
$result = $this->getCore()->getContainer()->call(
    [
        $myObject,
        'myMethodName'
    ],
    [
        'argument1' => 'Value',
        'argument3' => 'Value',
        'argument2' => 'Value'
    ]
);
```

### Invoke a function

You can invoke a function.

```php
$result = $this->getCore()->getContainer()->call(
    'myFunctionName',
    [
        'argument1' => 'Value',
        'argument3' => 'Value',
        'argument2' => 'Value'
    ]
);
```

### Invoke a Closure

You can invoke a closure.

```php
$result = $this->getCore()->getContainer()->call(
    fn(Berlioz\Core\Core $core) => true
);
```