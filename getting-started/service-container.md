```index
breadcrumb: Getting started; Service container
keywords: service, container
```

# Service container

The default service container of Berlioz Framework is the package [**berlioz/service-container**](https://github.com/BerliozFramework/ServiceContainer).

**Berlioz Service Container** is a PHP library to manage your services with dependencies injection, respecting PSR-11 (Container interface) standard.

## Configuration

You can define services in the configuration file.

Example:

```json
{
    "services": {
        "myServiceAlias": {
            "class": "MyService\\ClassName",
            "arguments": {
                "foo": "value",
                "bar": "value"
            },
            "calls": [
                {
                    "method": "myMethodName",
                    "arguments": {
                        "baz": "value",
                        "qux": "value"
                    }
                }
            ]
        },
        ...
    }
}
```

In the example, when you got service `myServiceAlias` or `MyService\ClassName::class`, the instantiator will take configuration, and do:

- Instantiate class with constructor arguments `foo` and `bar`
- Call method `myMethodName` with arguments `baz` and `qux`, after instantiation


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

Others methods of **Service Container** in many cases, are not necessaries for projects. If you want to know more, go on [repository documentation](https://github.com/BerliozFramework/ServiceContainer).

> **Tips**:
>
> You can get service quickly from `Controller` classes with method: `Controller::get($id)`.

## Instantiator

**Berlioz Framework** give you an **Instantiator** class that you can use to do dependency injection with object, methods or functions.

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