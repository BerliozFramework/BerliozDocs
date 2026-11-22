---
breadcrumb:
  - Components
  - Config
keywords:
  - configuration
  - json
  - json5
  - yaml
  - ini
---

# Configuration

> ℹ️ **Note**: The configuration component is part of the **Berlioz** ecosystem but is available as a standalone
> package: [`berlioz/config`](https://github.com/BerliozFramework/Config).
> You can find it on
> [Packagist](https://packagist.org/packages/berlioz/config).
> You can use it independently of the framework, in any PHP application. 🎉

**Berlioz Configuration** is a PHP library to manage your configuration files.

## Create a configuration object

You can create the configuration with adapters. Four default adapters are available:

- `ArrayAdapter`: accepts a PHP array or a PHP file returning an array
- `IniAdapter`: accepts an INI string or file
- `JsonAdapter`: accepts a JSON/JSON5 string or file
- `YamlAdapter`: accepts a YAML string or file (requires `ext-yaml` or `symfony/yaml`)

Example:

```php
use Berlioz\Config\Adapter;
use Berlioz\Config\Config;

$arrayAdapter = new Adapter\ArrayAdapter([/*...*/]);
$iniAdapter = new Adapter\IniAdapter('/path/of-project/config/config.ini', true);
$jsonAdapter = new Adapter\JsonAdapter('/path/of-project/config/config.json', true);

$yamlAdapter = new Adapter\YamlAdapter('/path/of-project/config/config.yml', true);

$config = new Config([$arrayAdapter, $jsonAdapter, $iniAdapter, $yamlAdapter]);
print $config->get('foo.bar.qux'); // Print value of configuration
```

The second parameter of the `IniAdapter`, `JsonAdapter` and `YamlAdapter` constructors tells the adapter that the first
parameter is the path of a file, not the content itself.

The order of the adapters is important: the first one has priority. The value returned by the `get` method comes from
the first adapter that has the key. If the value is an array, it is merged with the values of all adapters.

For more flexibility, you can define the priority as an integer in the constructor of the adapters, with the
`priority` parameter.

## Get a value

To get a value, you must call the `get` method:

```php
$config = new \Berlioz\Config\Config(/* ... */);

$config->get('foo'); // Returns value of key 'foo'
$config->get('foo.bar'); // Returns value of nested key 'foo.bar'
$config->get('baz', true); // Returns value of key 'baz' or TRUE default value if key does not exist
$config->get(); // Returns the full compiled configuration array
```

The second parameter of the `ConfigInterface::get()` method is the default value if the key does not exist. The default
value of this parameter is `NULL`.

> 🆕 **Info**: *Since version 3.1*
>
> Calling `get()` without a key returns the full compiled configuration array.

You can also test if a key exists:

```php
$config = new \Berlioz\Config\Config(/* ... */);

$exists = $config->has('foo'); // Returns boolean
```

You can get a value or throw an exception if the key is missing or if the resolved value is `null`:

```php
$config = new \Berlioz\Config\Config(/* ... */);

$value = $config->getOrFail('database.host'); // Throws ConfigException if key is missing or value is null
```

> ℹ️ **Note**: `getOrFail()` does not reject valid falsy values such as `false`, `0`, `""`, or `[]`. This method is only
> available on the concrete `Config` class, not on `ConfigInterface`.

## Functions

The `Config` object accepts a set of functions. The syntax to call a function is: `{functionName:value}`.

A value can combine function calls with text, for example `{config: berlioz.directories.var}/tmp`.

Default functions:

- `config`: replaces the value with another part of the config
- `constant`: replaces the value with a constant
- `env`: replaces the value with an environment variable
- `var`: replaces the value with a variable value
- `file`: replaces the value with the contents of a file

Examples:

```php
use Berlioz\Config\Adapter;
use Berlioz\Config\Config;

define('FOO', 'foo constant value');

$arrayAdapter = new Adapter\ArrayAdapter([
    'foo' => '{constant:FOO}',
    'bar' => [
        'foo' => 'value2',
    ],
    'baz' => '{config: bar.foo}',
    'qux' => '{var: BAR}'
]);
$config = new Config([$arrayAdapter], ['BAR' => 'bar value']);

print $config->get('foo'); // Print "foo constant value"
print $config->get('baz'); // Print "value2"
print $config->get('qux'); // Print "bar value"
print_r($config->get('bar')); // Print array "['foo' => 'value2']"
```

> 🆕 **Info**: *Since version 3.1*
>
> Circular references in `{config: ...}` expressions are detected and throw a `ConfigException` with the resolution
> chain.

## Variables

You can define variables usable in the configuration with the `var` function.

Define variables in the constructor:

```php
// Define variable in an array
$variables = [
    'foo' => 'foo value',
    'bar' => 'bar value',
];

$config = new \Berlioz\Config\Config(variables: $variables);
```

You can also manipulate the variables after the instantiation of the config. Variables are stored in an `ArrayObject`
object, accessible with the `Config::getVariables()` method:

```php
$config = new \Berlioz\Config\Config();

// Set variables
$config->getVariables()['foo'] = 'foo value';
$config->getVariables()['bar'] = 'bar value';

// Unset a variable
unset($config->getVariables()['bar']);
```

## Extend the library

### Create an adapter

You can create your own adapter. You only need to implement the `\Berlioz\Config\Adapter\AdapterInterface` interface.

This interface has 5 methods:

- `AdapterInterface::getPriority(): int`
- `AdapterInterface::setPriority(int $priority): void`
- `ConfigInterface::get(?string $key = null, mixed $default = null): mixed`
- `ConfigInterface::has(string $key): bool`
- `ConfigInterface::getArrayCopy(): array`

The `\Berlioz\Config\Adapter\AbstractAdapter` class implements them all from an array of configuration.

Look at the existing adapters in the source code of the library for inspiration.

### Create a function

You can create your own functions. You only need to implement the
`\Berlioz\Config\ConfigFunction\ConfigFunctionInterface` interface.

This interface has only 2 methods:

- `ConfigFunctionInterface::getName(): string`
- `ConfigFunctionInterface::execute(string $str): mixed`

Look at the existing functions in the source code of the library for inspiration.

To add a custom function to your config instance:

```php
$config->addFunction(new MyCustomFunction());
```

If you add a function with the same name as a built-in one, it will override it.
