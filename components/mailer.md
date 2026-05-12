---
breadcrumb:
  - Components
  - Mailer
keywords:
  - mail
  - email
  - smtp
  - transport
---

# Mailer

> ℹ️ **Note**: The mailer is part of the **Berlioz** ecosystem but is available as a standalone package:
> [`berlioz/mailer`](https://github.com/BerliozFramework/Mailer).
> You can find it on [Packagist](https://packagist.org/packages/berlioz/mailer).
> You can use it independently of the framework, in any PHP application.

**Berlioz Mailer** is a PHP library for sending mail, with or without local server.

## Example

You can send simply the like this:

```php
use Berlioz\Mailer\Address;
use Berlioz\Mailer\Mail;
use Berlioz\Mailer\Mailer;

$mail = (new Mail())
            ->setSubject('Test of Berlioz/Mailer')
            ->setText('Text plain of my mail')
            ->setHtml('<p>Html text of my mail</p>')
            ->setFrom(new Address('sender@test.com', 'Me the sender'))
            ->setTo([new Address('recipient@test.com', 'The recipient')]); 

$mailer = new Mailer();
$mailer->send($mail);
```

## Mail

**\Berlioz\Mailer\Mail** it's the object representation of a mail.

#### Basic

```php
use Berlioz\Mailer\Address;
use Berlioz\Mailer\Mail;

$mail = new Mail();
$mail->setSubject('Subject of my mail')
     ->setText('Text plain of my mail')
     ->setHtml('<p>Html text of my mail</p>')
     ->setFrom(new Address('sender@test.com', 'Me the sender'))
     ->setTo([new Address('recipient@test.com', 'The recipient')]);
```

#### Attachments

To add downloadable attachment:

```php
use Berlioz\Mailer\Attachment;
use Berlioz\Mailer\Mail;

$attachment = new Attachment('/path/of/my/file.pdf');
$mail = new Mail();
$mail->addAttachment($attachment);
```

To attach an attachment to HTML content:

```php
use Berlioz\Mailer\Attachment;
use Berlioz\Mailer\Mail;

$attachment = new Attachment('/path/of/my/img.jpg');
$mail = new Mail();
$mail->addAttachment($attachment);

$html = '<p>Html content 1</p>';
$html .= '<img src="cid:' . $attachment->getId() . '">';
$html .= '<p>Html content 2</p>';

$mail->setHtml($html);
```

**WARNING:** call `$attachment->getId()` method, does that the attachment will be in inline disposition. Only uses this method for inline attachments.

#### CC and BCC

You can set carbon copy and blind carbon copy recipients:

```php
$mail->setCc([new Address('cc@test.com', 'CC recipient')]);
$mail->setBcc([new Address('bcc@test.com', 'BCC recipient')]);
```

#### Custom headers

You can add custom headers to a mail:

```php
$mail->addHeader('X-Custom-Header', 'value');
$mail->addHeader('X-Tag', 'first');
$mail->addHeader('X-Tag', 'second'); // Multi-value: both values are sent
$mail->addHeader('X-Tag', 'replaced', replace: true); // Replaces all previous values
```

Reserved headers (`Subject`, `From`, `To`, `Cc`, `Bcc`) cannot be set this way — the check is case-insensitive.

> 🆕 **Info**: *Since version 3.1*
>
> Header names and values must not contain CR (`\r`) or LF (`\n`) characters; an `InvalidArgumentException` is thrown
> otherwise.

> **Note:** header values must be MIME-encoded by the caller if they contain non-ASCII characters.

#### Mass sending

You can send the same mail to multiple recipients individually:

```php
$addresses = [
    new Address('user1@test.com'),
    new Address('user2@test.com'),
    new Address('user3@test.com'),
];

$mailer->massSend($mail, $addresses, function ($address, $index) {
    echo "Sent to recipient #$index\n";
});
```

Each recipient receives an individual mail. The callback is optional and receives the address and the zero-based index.

## Transports

> 🆕 **Info**: *Since version 3.1*
>
> A sender address (`setFrom()`) is now required. Both transports throw a `TransportException` if no sender is defined
> when `send()` is called.

#### Defaults transports

Default transport is **\Berlioz\Mailer\Transport\PhpMail** uses internal **mail()** of PHP.

You can uses another available transport for direct communication with SMTP server: **\Berlioz\Mailer\Transport\Smtp**.

```php
use Berlioz\Mailer\Mailer;
use Berlioz\Mailer\Transport\Smtp;

$smtp = new Smtp(
    'smpt.test.com',
    'user@test.com',
    'password',
    25,
    ['timeout' => 5]
);
$mailer = new Mailer();
$mailer->setTransport($smtp);
```

```php
use Berlioz\Mailer\Mailer;
$mailer = new Mailer([
    'transport' => [
        'name' => 'smtp',
        'arguments' => [
            'host' => 'smpt.test.com',
            'username' => 'user@test.com',
            'password' => 'password',
            'port' => 25,
            'options' => ['timeout' => 5]
        ]
    ]
]);
```

#### Create a new transport

It's possible to create new transport for various reasons.
To do that, you need to create class who implements **\Berlioz\Mailer\Transport\TransportInterface** interface.
