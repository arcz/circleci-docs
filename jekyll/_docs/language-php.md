---
layout: classic-docs
title: Continuous Integration and Continuous Deployment with PHP
short-title: PHP
categories: [languages]
description: Continuous Integration and Continuous Deployment with PHP
---

Circle works seamlessly with all the tools and package managersthat
help PHP developers test and deploy their code. We can generally infer
most of your dependencies and test commands, but we also provide custom
configuration via a `circle.yml` checked into your repo's root directory.

### Version

We have many versions of PHP pre-installed on [Ubuntu 12.04]({{ site.baseurl }}/build-image-precise/#php) and [Ubuntu 14.04]({{ site.baseurl }}/build-image-trusty/#php) build images.

If you don't want to use the default, you can specify your version in `circle.yml`:

```
machine:
  php:
    version: 5.5.11
```

Please [contact us](mailto:sayhi@circleci.com)
if you need a different version; we'll be happy to install it for you.

### Dependencies

Circle has the composer, pear, and pecl package managers installed.
If we find a composer.json file, then we'll automatically run `composer install`.

To install your dependencies with either `pear` or `pecl`,
you have to include [dependency commands]({{site.baseurl}}/configuration/#dependencies)
in your `circle.yml` file.
The following example shows how to install the MongoDB extension using `pecl`.

```
dependencies:
  pre:
    - pecl install mongo
```

You can also edit your PHP configuration from your `circle.yml`. For example, if you have a custom configuration file checked in to your repo, then you could do:

```
dependencies:
  pre:
    - cp config/custom.ini ~/.phpenv/versions/$(phpenv global)/etc/conf.d/
```

<span class='label label-info'>Note:</span>
`phpenv global` returns the PHP version that has been
specified in your `circle.yml` file.

Here's another example showing how you could adjust PHP settings in
a `.ini` file.

```
dependencies:
  pre:
    - echo "memory_limit = 64M" > ~/.phpenv/versions/$(phpenv global)/etc/conf.d/memory.ini
```

<span class='label label-info'>Note:</span>
you'll have to specify your PHP version in your `circle.yml` in order to edit PHP's configuration files.

### Databases

We have pre-installed more than a dozen databases and queues,
including PostgreSQL and MySQL. If needed, you have the option of
[manually setting up your test database]({{site.baseurl}}/manually/#dependencies).

<h3 id="php-apache">Using the Apache Webserver</h3>

Apache2 is already installed on CircleCI containers but it needs to be
configured to host your PHP application.

To enable your site check a file containing your site configuration into your
repository and copy it to `/etc/apache2/sites-available/` during
your build.
Then enable the site with `a2ensite` and restart Apache.

An example configuration that sets up Apache to serve the PHP site from
`/home/ubuntu/MY-PROJECT/server-root` is:

```
Listen 8080

<VirtualHost *:8080>
  LoadModule php5_module /home/ubuntu/.phpenv/versions/PHP_VERSION/libexec/apache2/libphp5.so

  DocumentRoot /home/ubuntu/MY-PROJECT/server-root
  ServerName host.example.com
  <FilesMatch \.php$>
    SetHandler application/x-httpd-php
  </FilesMatch>
</VirtualHost>
```

Replace `MY-SITE` in with the name of your site configuration
file and `PHP_VERSION` with the version of PHP you configured
in your `circle.yml`.

Then enable your site and restart Apache by adding the following to your `circle.yml`

```
dependencies:
  post:
    - cp ~/MY-PROJECT/MY-SITE /etc/apache2/sites-available
    - a2ensite MY-SITE
    - sudo service apache2 restart
```

### Testing

Circle always runs your tests on a fresh machine. If we find a `phpunit.xml` file in your repo, then we'll run `phpunit` for you. You can add custom test commands to the test section of your `circle.yml`:

```
test:
  override:
    - ./my_testing_script.sh
```

If you want CircleCI to show a test summary of your build see
[Metadata collection in custom test steps for PHPUnit]({{ site.baseurl }}/test-metadata/#phpunit).

<h3 id="xdebug">Enable Xdebug</h3>

Xdebug is installed for all versions of PHP, but is disabled (for performance reasons) by
default. It is simple to enable this tool by adding the following to your
`circle.yml` file:

```
dependencies:
  pre:
    - sed -i 's/^;//' ~/.phpenv/versions/$(phpenv global)/etc/conf.d/xdebug.ini
```

### Deployment

Circle offers first-class support for [deployment]({{site.baseurl}}/configuration/#deployment).
When a build is green, Circle will deploy your project as directed
in your `circle.yml` file. We can deploy to PaaS providers as well as to
physical servers under your control.

### Troubleshooting for PHP

If you run into problems, check out our [PHP troubleshooting]({{site.baseurl}}/troubleshooting-php/)
write-ups about these issues:

*   [Adding memcached with pecl on CircleCI]({{site.baseurl}}/php-memcached-compile-error/)
*   [Composer hitting GitHub API rate limits]({{site.baseurl}}/composer-api-rate-limit/)

If you are still having trouble, please
[contact us](mailto:sayhi@circleci.com)
and we will be happy to help.
