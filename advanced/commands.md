# Commands

You can create some commands to call your functions or services.

## Create a command

You need to implements `\Berlioz\CliCore\Command\CommandInterface` interface or extends `\Berlioz\CliCore\Command\AbstractCommand` abstract class.

Representation of interface:

```php
interface CommandInterface
{
    /**
     * Get args.
     *
     * Must return an array of arguments.
     *
     * @return \Berlioz\CliCore\Command\CommandArg[]
     */
    public function getArgs(): array;

    /**
     * Run command.
     *
     * @param \Berlioz\CliCore\App\CliArgs $args
     *
     * @return void
     */
    public function run(CliArgs $args);
}
```

Methods:

- `CommandInterface::getArgs()`: to your arguments declaration
- `CommandInterface::run()`: to the command running code (with dependencies injection)

## Declare a command

You need to declare each commands in a `commands.json` file in you configuration directory.

Example of file:

```json
{
    "json": {
        "myproject:foo": "\\MyProject\\Command\\FooCommand",
        "myproject:bar": "\\MyProject\\Command\\BarCommand"
    }
}
```

## Call a command

An executable is available in binary directory of **Composer** to call commands.

Examples:

```bash
$ cd /my/project/directory
$ vendor/bin/berlioz myproject:foo
Printed result!
$ vendor/bin/berlioz myproject:bar --arg1 "Test" -aze
Printed result of second command!
```