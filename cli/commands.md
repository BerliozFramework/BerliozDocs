```index
breadcrumb: CLI Commands; Commands
```

# Commands

You can create some commands to call your functions or services to automate some jobs in crontab for example.

## Create a command

You need to implement `\Berlioz\CliCore\Command\CommandInterface` interface or extends `\Berlioz\CliCore\Command\AbstractCommand` abstract class.

Representation of interface:

```php
/**
 * Interface CommandInterface.
 *
 * @package Berlioz\CliCore\Command
 */
interface CommandInterface
{
    /**
     * Get short description.
     *
     * @return string|null
     */
    public static function getShortDescription(): ?string;

    /**
     * Get description.
     *
     * @return string|null
     */
    public static function getDescription(): ?string;

    /**
     * Get options.
     *
     * Must return an array of options.
     *
     * @return \GetOpt\Option[]
     * @see http://getopt-php.github.io/getopt-php/options.html
     */
    public static function getOptions(): array;

    /**
     * Get operands.
     *
     * Must return an array of operands.
     *
     * @return \GetOpt\Operand[]
     * @see http://getopt-php.github.io/getopt-php/operands.html
     */
    public static function getOperands(): array;

    /**
     * Run command.
     *
     * @param \GetOpt\GetOpt $getOpt
     *
     * @return void
     */
    public function run(GetOpt $getOpt);
}
```

## Use arguments

Berlioz commands uses `ulrichsg/getopt-php` composer package to manage CLI arguments. So to use arguments, `\Berlioz\CliCore\Command\CommandInterface::getOptions()` method must return an array of `\GetOpt\Option` objects.

Example:

```php
public static function getOptions(): array
{
    return [
        (new Option('f', 'filter', GetOpt::OPTIONAL_ARGUMENT))
            ->setDescription('Filter')
            ->setValidation('is_string'),
        (new Option(null, 'nb', GetOpt::OPTIONAL_ARGUMENT))
            ->setDescription('Number of results')
            ->setValidation('is_numeric')
    ];
}
```

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