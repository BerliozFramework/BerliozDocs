```index
breadcrumb: CLI Project; Commands
summary-order: 3; 2
keywords: command
```

# Commands

You can create some commands to call your functions or services to automate some jobs in crontab for example.

## Create a command

You need to implement `\Berlioz\Cli\Core\Command\CommandInterface` interface or extends `\Berlioz\CliCore\Command\AbstractCommand` abstract class.

Representation of the interface:

```php
use Berlioz\Cli\Core\Console\Environment;

/**
 * Interface CommandInterface.
 */
interface CommandInterface
{
    /**
     * Get description.
     *
     * @return string|null
     */
    public static function getDescription(): ?string;

    /**
     * Get help.
     *
     * @return string|null
     */
    public static function getHelp(): ?string;

    /**
     * Run command.
     *
     * @param Environment $env
     *
     * @return int
     */
    public function run(Environment $env): int;
}
```

## Arguments

Arguments must be declared with PHP 8 attributes, example:

```php
use Berlioz\Cli\Core\Command\Argument;
use Berlioz\Cli\Core\Command\CommandInterface;
use Berlioz\Cli\Core\Console\Environment;

#[Argument('argument1', prefix: 'a', longPrefix: 'arg')]
#[Argument('argument2', longPrefix: 'arg2')]
class MyCommand implements CommandInterface
{
    // ...

    public function run(Environment $env): int
    {
        $env->getArgument('argument1'); // Get the argument 1
        $env->getArgument('argument2'); // Get the argument 2
        
        // ...
    }
}
```

Options for arguments:

Name | Type |Description
-----|------|------------
name | string | Name of argument
prefix | string (default: null) | Prefix (one character)
longPrefix | string (default: null) | Long prefix
description | string (default: null) | Description, used for --help
defaultValue | mixed (default: null) | Default value
required | bool (default: false) | If argument is required
noValue | bool (default: false) | If argument has no value
castTo | string (default: null) | Cast to a specified type

## Declare a command

You need to declare each commands in a `commands.json` file in your [configuration directory](../getting-started/config.md).

Example of file:

```json
{
    "commands": {
        "myproject:foo": "\\MyProject\\Command\\FooCommand",
        "myproject:bar": "\\MyProject\\Command\\BarCommand"
    }
}
```

## Call a command

An executable is available in the binary directory of **Composer** to call commands.

Examples:

```bash
$ cd /my/project/directory
$ vendor/bin/berlioz myproject:foo
Printed result!
$ vendor/bin/berlioz myproject:bar --arg1 "Test" -aze
Printed result of second command!
```