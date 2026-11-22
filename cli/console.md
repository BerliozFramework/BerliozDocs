---
breadcrumb:
  - CLI Project
  - Console
summary-order: 3;3
keywords:
  - command
---

# Console

Berlioz uses the [**league/climate**](https://climate.thephpleague.com/) package to manage output.

> CLImate allows you to easily output colored text, special formatting, and more. It makes output to the terminal
> clearer and debugging a lot simpler.

The console is accessible from the environment object passed as an argument to the
`CommandInterface::run(Environment $env)` method.

## Output

Example of colorized output:

```php
// ...

$env->console()->red('Red output');
$env->console()->output('Normal output');
```

For all the capabilities of the console, see the [official documentation](https://climate.thephpleague.com/).
