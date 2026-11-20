---
breadcrumb:
  - Components
  - Mailer
keywords:
  - mail
  - email
  - smtp
  - starttls
  - tls
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
use Berlioz\Mailer\Transport\SmtpEncryption;

$smtp = new Smtp(
    host: 'smtp.test.com',
    username: 'user@test.com',
    password: 'password',
    port: 587,
    encryption: SmtpEncryption::STARTTLS,
    timeout: 5,
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
            'host' => 'smtp.test.com',
            'username' => 'user@test.com',
            'password' => 'password',
            'port' => 587,
            'encryption' => 'starttls',
            'timeout' => 5,
        ]
    ]
]);
```

SMTP transport arguments:

| Argument     | Default     | Description                                                                         |
|--------------|-------------|-------------------------------------------------------------------------------------|
| `host`       | `localhost` | SMTP server host                                                                    |
| `username`   | `null`      | Username, authentication is done only if username and password are given           |
| `password`   | `null`      | Password                                                                            |
| `port`       | `25`        | SMTP server port                                                                    |
| `encryption` | `null`      | Connection mode: `null`, `starttls` or `tls` (see [connection modes](#smtp-connection-modes)) |
| `context`    | `null`      | Stream context options (array) or stream context, see [TLS options](#tls-options)   |
| `timeout`    | `10`        | Connection and read timeout, in seconds                                             |

> 🆕 **Info**: *Since version 3.4*
>
> Arguments `encryption`, `context` and `timeout` are available. The `options` argument (`['timeout' => 5]`) is
> deprecated and emits an `E_USER_DEPRECATED` notice: use the `timeout` argument instead. It will be removed in v4.

#### SMTP connection modes

> 🆕 **Info**: *Since version 3.4*

The `encryption` argument accepts a `Berlioz\Mailer\Transport\SmtpEncryption` enum case or its string value:

| Mode                     | Value      | Usual port | Description                                                                    |
|--------------------------|------------|------------|--------------------------------------------------------------------------------|
| Plain (default)          | `null`     | 25         | No encryption: credentials and messages are sent in clear text                 |
| STARTTLS                 | `starttls` | 587        | Plain connection upgraded to TLS with the `STARTTLS` command, before login     |
| Implicit TLS             | `tls`      | 465        | Connection encrypted from the first byte                                       |

With `starttls`, encryption is **mandatory**: if the server does not advertise `STARTTLS`, refuses it, or if the TLS
negotiation fails, sending fails with a `TransportException`. The transport never falls back to a plain connection,
and authentication and message delivery only happen over the encrypted connection.

TLS 1.2 and TLS 1.3 are used by default. The server certificate and the host name are validated against the system
certificate authorities.

> ⚠️ **Warning**: The plain mode sends credentials and message contents unencrypted. Use it only with a local server
> or a trusted network, e.g. a development mail catcher (Mailpit, MailHog...).

Prefixing the host with a scheme (`ssl://smtp.test.com`, `tls://smtp.test.com`) is still supported for implicit TLS
when `encryption` is `null`, but prefer the `encryption` argument. Combining a host scheme with `encryption` throws an
`InvalidArgumentException`.

#### TLS options

The `context` argument accepts the [PHP SSL context options](https://www.php.net/manual/context.ssl.php), as an array
or as a stream context created with `stream_context_create()`. By default, `peer_name` is the host.

To trust a private certificate authority or a self-signed certificate, without disabling validation:

```php
use Berlioz\Mailer\Transport\Smtp;
use Berlioz\Mailer\Transport\SmtpEncryption;

$smtp = new Smtp(
    host: '10.0.0.12',
    port: 587,
    encryption: SmtpEncryption::STARTTLS,
    context: [
        'ssl' => [
            'cafile' => '/path/to/ca.pem', // CA or self-signed certificate
            'peer_name' => 'smtp.internal', // Name in the certificate, if different from host
        ],
    ],
);
```

You can also pin the server certificate with the `peer_fingerprint` option.

> ⚠️ **Warning**: Setting `verify_peer` or `verify_peer_name` to `false`, or `allow_self_signed` to `true`, disables
> protection against man-in-the-middle attacks. The transport logs a warning in this case. Prefer `cafile` or
> `peer_fingerprint`.

#### Create a new transport

It's possible to create new transport for various reasons.
To do that, you need to create class who implements **\Berlioz\Mailer\Transport\TransportInterface** interface.
