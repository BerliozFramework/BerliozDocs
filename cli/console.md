```index
breadcrumb: CLI Project; Console
summary-order: 3; 3
keywords: command
```

# Console

Berlioz uses [**league/climate**](https://climate.thephpleague.com/) package to manage output.

> CLImate allows you to easily output colored text, special formatting, and more. It makes output to the terminal clearer and debugging a lot simpler.

The console is accessible from environment variable passed in argument of `CommandInterface::run(Environment $env)` method.

## Output

Example to colorize output:

```php
// ...

$env->console()->red('Red output');
$env->console()->output('Normal output');
```

For all capabilities of console, refers you to the [official documentation](https://climate.thephpleague.com/).