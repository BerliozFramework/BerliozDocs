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

> ℹ️ **Note**: While configuration are part of the **Berlioz** ecosystem, they are available as a standalone package:
> [`berlioz/config`](https://github.com/BerliozFramework/Config).
> You can find it on
> [Packagist](https://packagist.org/packages/berlioz/config).
> You can use them independently of the framework, in any PHP application. 🎉

**Berlioz Configuration** is a PHP library to manage your configuration files.

## Create configuration object

You can create the configuration with adapters. 4 default adapters are available:

- `ArrayAdapter`: accept a PHP array or a PHP file returning an array
- `IniAdapter`: accept a INI string or file
- `JsonAdapter`: accept a JSON/JSON5 string or file
- `YamlAdapter`: accept a YAML string or file (requires `ext-yaml` or `symfony/yaml`)

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

Second parameter of `IniAdapter`, `JsonAdapter` and `YamlAdapter` constructors define that the first parameter is an url.

The order of adapter is important, the first have priority... So the value returned by `get` method is the first adapter
to respond at key. If the value is an array, it will be merged with all adapters.

For more flexibility, you can define the priority by an integer in the constructor of adapters, withe the
parameter `priority`.

## Get value

To get value, you must call `get` method:

```php
$config = new \Berlioz\Config\Config(/* ... */);

$config->get('foo'); // Returns value of key 'foo'
$config->get('foo.bar'); // Returns value of nested key 'foo.bar'
$config->get('baz', true); // Returns value of key 'baz' or TRUE default value if key does not exist
$config->get(); // Returns the full compiled configuration array
```

The second parameter of `ConfigInterface::get()` method is the default value if key does not exist. Default value of
this parameter is `NULL`.

> 🆕 **Info**: *Since version 3.1*
>
> Calling `get()` without a key returns the full compiled configuration array.

You can also test if a key exist:

```php
$config = new \Berlioz\Config\Config(/* ... */);

$exists = $config->has('foo'); // Returns boolean
```

You can get a value or throw an exception if the key is missing or if the resolved value is `null`:

```php
$config = new \Berlioz\Config\Config(/* ... */);

$value = $config->getOrFail('database.host'); // Throws ConfigException if key is missing or value is null
```

> **Note:** `getOrFail()` does not reject valid falsy values such as `false`, `0`, `""`, or `[]`. This method is only
> available on the concrete `Config` class, not on `ConfigInterface`.

## Functions

`Config` object accept a set of functions. The syntax to call a function is: `{functionName:value}`.

A function call must be alone in value of configuration key.

Defaults functions:

- `config`: replace value by another part of config
- `constant`: replace value by a constant
- `env`: replace value by environment variable
- `var`: replace value by variable value
- `file`: replace value by file contents

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

You can define variables usable in configuration with function `var`.

Define variables in the constructor:

```php
// Define variable in an array
$variables = [
    'foo' => 'foo value',
    'bar' => 'bar value',
];

$config = new \Berlioz\Config\Config(variables: $variables);
```

You can also manipulate the variables after instantiation of config. Variables are stored in a `ArrayObject` object,
accessible with `Config::getVariables()` method:

```php
$config = new \Berlioz\Config\Config();

// Set variables
$config->getVariables()['foo'] = 'foo value';
$config->getVariables()['bar'] = 'bar value';

// Unset a variable
unset($config->getVariables()['bar']);
```

## Extend library

### Create an adapter

You can create your own adapter. Only implements `\Berlioz\Config\Adapter\AdapterInterface` interface.

This interface has only 3 methods:

- `AdapterInterface::getPriority(): int`
- `ConfigInterface::get(string $key, mixed $default = null): mixed`
- `ConfigInterface::has(string $key): bool`

Look at the existent adapters in the source code of the library for inspiration.

### Create a function

You can create your owns functions. Only implements `\Berlioz\Config\ConfigFunction\ConfigFunctionInterface` interface.

This interface has only 2 methods:

- `ConfigFunctionInterface::getName(): string`
- `ConfigFunctionInterface::execute(string $str): mixed`

Look at the existent functions in the source code of the library for inspiration.

To add a custom function to your config instance:

```php
$config->addFunction(new MyCustomFunction());
```

If you add a function with the same name as a built-in one, it will override it.
