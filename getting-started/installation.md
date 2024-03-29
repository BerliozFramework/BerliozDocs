```index
breadcrumb: Getting started; Installation
keywords: installation
summary-order: 1
```

# Installation

This page describes the default installation for the most of HTTP projects.
If you want use **Berlioz Framework** for CLI projects, go to [CLI Project](../cli.md) to know more.

## Berlioz Framework

Installation of Berlioz Framework must be done by [Composer](https://getcomposer.org/), it's the recommended installation.

```bash
composer create-project berlioz/website-skeleton my-project --remove-vcs
```

You can specify the version to composer:

```bash
composer create-project berlioz/website-skeleton my-project "2.0" --remove-vcs
```

> **Info**:
>
> Option `--remove-vcs` remove automatically the `.git` directory from the default project.

## HTTP server

To run the application, your HTTP server must be pointed on `public` directory.

### Apache server

[Apache](https://httpd.apache.org/) is the most popular web server.

For **Apache** version >= 2.4:

```apache
<VirtualHost *:80>
    ServerName getberlioz.com
    DocumentRoot "/path/to/my-project/public"

    <Directory "/path/to/my-project/public">
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

For **Apache** version < 2.4:

```apache
<VirtualHost *:80>
    ServerName getberlioz.com
    DocumentRoot "/path/to/my-project/public"

    <Directory "/path/to/my-project/public">
        AllowOverride All
        Allow from all
    </Directory>
</VirtualHost>
```

> **Warning**:
>
> Needs to enable the module **mod_rewrite** on Apache.
>
> Cf. [Apache documentation](https://httpd.apache.org/docs/current/fr/mod/mod_rewrite.html) for installation.

If you want increase the performances of your server, disables support of `.htaccess` files.

For that, you must specify to Apache the fallback resource and disable `AllowOverride` option:

```apache
<Directory "/path/to/my-project/public">
    AllowOverride None
    FallbackResource /index.php
    # ...
</Directory>
```  

### Nginx

[Nginx](https://www.nginx.com) is a high performance web server.

To run PHP application with [Nginx](https://www.nginx.com), you need to use [PHP FPM](http://php.net/manual/book.fpm.php) (FastCGI Process Manager).

```nginx
server {
    server_name getberlioz.com;
    root /path/to/my-project/public;

    # Test existent of file before redirect to fallback file: index.php
    location / {
        try_files $uri /index.php$is_args$args;
    }

    # Configuration for index.php file calls
    location ~ ^/index\.php(/|$) {
        # FastCGI configuration
        fastcgi_pass unix:/var/run/php/php-fpm.sock;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;

        # FastCGI parameters to give directory and script infos
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        fastcgi_param DOCUMENT_ROOT $realpath_root;

        # To do not use index.php file for routes
        internal;
    }

    # To deny all others PHP files
    location ~ \.php$ {
        return 404;
    }
}
```

### PHP HTTP server

If you want test application without Web Server to install, you can use [built-in web server](http://php.net/manual/features.commandline.webserver.php).

To start server:

```bash
cd /path/of/project
cd public
php -S localhost:8000 index.php
```

In your web browser, go to the url: `http://localhost:8000`, you will see your project!

> **Warning**:
>
> The PHP built-in web server is only for **development environment**.
> Do not use it in production!

## Test first page

Go to in your favorite browser, and try to go to your page, in your example: `getberlioz.com`.

If everything ok, you will see this screen:

![First page](../_assets/first-page.png)

You can also see on this first page, the debug console of **Berlioz**. If you click on, you will see the debug detail:

![Debug](../_assets/first-page-debug.png)
