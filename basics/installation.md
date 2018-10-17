<meta name="docparser-index" content="Installation" />
<meta name="docparser-index-order" content="1" />

# Installation

This page describe the default installation for the most of HTTP projects.
But you can use **Berlioz Framework** for CLI projects for example, go to [advanced usage page](../advanced.md) to know more.

## Berlioz Framework

Installation of Berlioz Framework must be done by [Composer](https://getcomposer.org/), it's the recommended installation.

```bash
composer create-project berlioz/berlioz --remove-vcs
```

You can specify the version to composer:

```bash
composer create-project berlioz/berlioz 1.0 --remove-vcs
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
    ServerName www.berlioz-framework.com
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
    ServerName www.berlioz-framework.com
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

If you want increase the performances of your server, it's to disable support of `.htaccess` files.

For that, you must specify to Apache the fallback resource and disable option `AllowOverride`:

```apache
<Directory "/path/to/my-project/public">
    AllowOverride None
    FallbackResource /app.php
    # ...
</Directory>
```  

### Nginx

[Nginx](https://www.nginx.com) is a high performance web server.

To run PHP application with [Nginx](https://www.nginx.com), you need to use [PHP FPM](http://php.net/manual/book.fpm.php) (FastCGI Process Manager).

```nginx
server {
    server_name www.berlioz-framework.com;
    root /path/to/my-project/public;

    # Test existent of file before redirect to fallback file: app.php
    location / {
        try_files $uri /app.php$is_args$args;
    }

    # Configuration for app.php file calls
    location ~ ^/app\.php(/|$) {
        # FastCGI configuration
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        fastcgi_split_path_info ^(.+\.php)(/.*)$;
        include fastcgi_params;

        # FastCGI parameters to give directory and script infos
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        fastcgi_param DOCUMENT_ROOT $realpath_root;

        # To do not use app.php file for routes
        internal;
    }

    # To deny all others PHP files
    location ~ \.php$ {
        return 404;
    }
}
```

### PHP HTTP server

If you want test application without Web Server to installed, you can use [built-in web server](http://php.net/manual/features.commandline.webserver.php).

To start server:

```bash
cd /path/of/project
cd public
php -S localhost:8000 app.php
```

In your web browser, go to the url: `http://localhost:8000`, you will see your project!

> **Warning**:
>
> The PHP built-in web server is only for **development environment**.
> Do not use it in production!

## Test first page

Go to in your favorite browser, and try to go to your page, in your example: `www.berlioz-framework.com`.

If everything ok, you will see:
TODO IMAGE