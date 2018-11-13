<meta name="docparser-index" content="Basic uses; Service container" />
<meta name="docparser-index-order" content="2" />

# Service container

The default service container of Berlioz Framework is the package [**berlioz/service-container**](https://github.com/BerliozFramework/ServiceContainer).

**Berlioz Service Container** is a PHP library to manage your services with dependencies injection, respecting PSR-11 (Container interface) standard.

## Usage

Service container is accessible from the application object with method `getServiceContainer()`.

Remember the 2 main methods:

- `get($id)`

  > PSR-11: Finds an entry of the container by its identifier and returns it.
  >
  > **Accept a class name directly.**

- `has($id)`

  > PSR-11: Returns true if the container can return an entry for the given identifier.
  > Returns false otherwise.

- `getInstantiator()`

  > Returns an `Instantiator` class with dependencies injection functionality.
  > Go next on the page to know more.

Others methods of **Service Container** in many cases, are not necessaries for projects. If you want know more, go on [repository documentation](https://github.com/BerliozFramework/ServiceContainer).

> **Tips**:
>
> You can get service quickly from `Controller` classes with method: `Controller::getService($id)`.

## Instantiator

**Berlioz Framework** give you an **Instantiator** class that you can use to do dependency injection with object, methods or functions.

In all next examples cases, the last argument is an array of parameters to give to the constructor, method or function.
The order of arguments is not important.

In case of a parameter is an object, the system get the classes implemented to inject in good parameter.

### New instance of a class or object

```php
$instantiator = $this->getApp()->getServiceContainer()->getInstantiator();
$object = $instantiator->newInstanceOf(MyClass::class,
                                       ['argument1' => 'Value',
                                        'argument3' => 'Value',
                                        'argument2' => 'Value']);
```

### Invoke a method

```php
$instantiator = $this->getApp()->getServiceContainer()->getInstantiator();
$result = $instantiator->invokeMethod($myObject,
                                      'myMethodName',
                                      ['argument1' => 'Value',
                                       'argument3' => 'Value',
                                       'argument2' => 'Value']);
```

### Invoke a function

```php
$instantiator = $this->getApp()->getServiceContainer()->getInstantiator();
$result = $instantiator->invokeFunction('myFunctionName',
                                        ['argument1' => 'Value',
                                         'argument3' => 'Value',
                                         'argument2' => 'Value']);
```