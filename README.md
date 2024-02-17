DNMP (Docker + Nginx/Openresty + MySQL5,8 + PHP7,8 + Redis + ElasticSearch + MongoDB + RabbitMQ) is a full-featured model**LNMP one-click installer with Support for Arm CPUs**。

> It is best to read it in advance before use[directory](#目录), in order to get started quickly, and problems can be eliminated in time.

**[\[中文\]](README-zh.md)** -
[**\[GitHub Address\]**](https://github.com/weimjay/dnmp) -

DNMP Project Features:

1.  `100%`open source
2.  `100%`Follow docker standards
3.  Supports the coexistence of **multiple versions of PHP**, can be switched arbitrarily (PHP7.0, PHP7.1, PHP7.2, PHP7.3, PHP7.4, PHP8.0)
4.  Supports binding **Any number of domain names**
5.  Supports **HTTPS and HTTP/2**
6.  **PHP source code, MySQL data, configuration files, log files**All can be directly modified in Host
7.  Built-in **Full PHP extension installation** command
8.  Supports `pdo_mysql`、`mysqli`、`mbstring`、`gd`、`curl`、`opcache` by default and other commonly used and popular extensions, flexibly configured according to the environment
9.  One-click selection of common services:
    *   Multiple PHP versions: PHP7.0-7.4, PHP8.0
    *   Web services: Nginx, Openresty
    *   Databases: MySQL8, Redis, MongoDB, ElasticSearch
    *   Message Queuing: RabbitMQ
    *   Accessibility: Kibana, Logstash, phpMyAdmin, phpRedisAdmin, AdminMongo
10. Apply in real projects, ensure`100%`available
11. All mirrors originate from[Docker official repository](https://hub.docker.com), safe and reliable
12. One configuration,**Windows、Linux、MacOs**All available
13. Supports quick installation extension commands `install-php-extensions apcu`
14. Supports installing CERTBOT to obtain SSL certificates for free https

# directory

*   [1. Directory structure](#1目录结构)
*   [2. Quick to use](#2快速使用)
*   [3.PHP and extensions](#3PHP和扩展)
    *   [3.1 Switch the PHP version used by Nginx](#31-切换Nginx使用的PHP版本)
    *   [3.2 Install PHP extensions](#32-安装PHP扩展)
    *   [3.3 Quickly install php extensions](#33-快速安装php扩展)
    *   [3.4 Using php command line in Host (php-cli)](#34-host中使用php命令行php-cli)
    *   [3.5 Use commoser](#35-使用composer)
*   [4. Administrative commands](#4管理命令)
    *   [4.1 Server Startup and Build Commands](#41-服务器启动和构建命令)
    *   [4.2 Add shortcut commands](#42-添加快捷命令)
*   [5. Use Log](#5使用log)
    *   [5.1 Nginx logs](#51-nginx日志)
    *   [5.2 PHP-FPM logs](#52-php-fpm日志)
    *   [5.3 MySQL logs](#53-mysql日志)
*   [6. Database management](#6数据库管理)
    *   [6.1 phpMyAdmin](#61-phpmyadmin)
    *   [6.2 phpRedisAdmin](#62-phpredisadmin)
*   [7. Safe to use in a formal environment](#7在正式环境中安全使用)
*   [8. Frequently Asked Questions](#8常见问题)
    *   [8.1 How to use curl in PHP code?](#81-如何在php代码中使用curl)
    *   [8.2 Docker uses cron to time tasks](#82-Docker使用cron定时任务)
    *   [8.3 Docker container time](#83-Docker容器时间)
    *   [8.4 How to connect to MySQL and Redis servers](#84-如何连接MySQL和Redis服务器)

## 1. Directory structure

    /
    ├── data                        data directory
    │   ├── esdata                  ElasticSearch data
    │   ├── mongo                   MongoDB data
    │   ├── mysql                   MySQL8 data
    ├── services                    Service build file and configuration file directory
    │   ├── elasticsearch           ElasticSearch configuration
    │   ├── mysql                   MySQL8 configuration
    │   ├── nginx                   Nginx configuration
    │   ├── php                     PHP7.4 configuration
    │   ├── php80                   PHP8.0 configuration
    │   └── redis                   Redis configuration
    ├── logs                        Log directory
    ├── docker-compose.sample.yml   Docker compose configuration sample file
    ├── env.smaple                  Environment configuration sample file
    └── www                         PHP codebase

## 2. Quick to use

1.  Local installation
    *   `git`
    *   `Docker`(The system needs to be Linux, Windows 10 Build 15063+, or MacOS 10.12+, and must be.)`64`bit)
    *   `docker-compose 1.7.0+`
2.  `clone`Project:
    ```
        $ git clone https://github.com/weimjay/dnmp.git
    ```
3.  If the host is a Linux system and the current user is not`root`Users, you also need to join the current user`docker`User Groups:
        $ sudo gpasswd -a ${USER} docker
4.  Copy and name the configuration file (for Windows systems.)`copy`command), start:
    ```
        $ cd dnmp                                           # Enter the project directory
        $ cp env.sample .env                                # Copy the environment variable file
        $ cp docker-compose.sample.yml docker-compose.yml   # Copy the docker-compose configuration file. 3 services are started by default:
                                                            # Nginx、PHP7和MySQL8. To enable more other services, such as redis, such as Redis, 
                                                            # PHP8.0, MongoDB, ElasticSearch，Please remove the comment before the service block
        $ docker-compose up                                 # start up
    ```
5.  Access in a browser:`http://localhost`or`https://localhost`(Self-signed HTTPS demo) can see the effect of PHP code in the file`./www/localhost/index.php`。

## 3.PHP and extensions

### 3.1 Switch the PHP version used by Nginx

First, you need to start another version of PHP, such as PHP 5.4, so that's the first step`docker-compose.yml`Delete the comments that preceded PHP5.4 from the file and start the PHP5.4 container.

After PHP5.4 starts, open Nginx Configuration and modify it`fastcgi_pass`The host address of the company, by `php`to`php54`As follows:

        fastcgi_pass   php:9000;

For:

        fastcgi_pass   php54:9000;

thereinto `php` and `php54` be`docker-compose.yml`The name of the server in the file.

At last**Reboot Nginx** Take effect.

```bash
$ docker exec -it nginx nginx -s reload
```

Here are two`nginx`, the first is the container name, and the second is in the container`nginx`Procedure.

### 3.2 Install PHP extensions

Many of php's features are implemented through extensions, and installing extensions is a slightly time-consuming process.
So, in addition to the PHP built-in extension, in`env.sample`In the file we only install a small number of extensions by default,
If you want to install more extensions, open yours`.env`Modify the following PHP configuration,
Add the required PHP extensions:

```bash
PHP_EXTENSIONS=pdo_mysql,opcache,redis       # PHP 要安装的扩展列表，英文逗号隔开
PHP54_EXTENSIONS=opcache,redis                 # PHP 5.4要安装的扩展列表，英文逗号隔开
```

Then re-build the PHP image.

```bash
docker-compose build php
```

See the available extensions in the same file`env.sample`Comment block description.

### 3.3 Quickly install php extensions

1\. Enter the container:

```sh
docker exec -it php /bin/sh

install-php-extensions apcu 
```

2\. Support quick installation extension list

<!-- START OF EXTENSIONS TABLE -->

<!-- ########################################################### -->

<!-- #                                                         # -->

<!-- #  DO NOT EDIT THIS TABLE: IT IS GENERATED AUTOMATICALLY  # -->

<!-- #                                                         # -->

<!-- #  EDIT THE data/supported-extensions FILE INSTEAD        # -->

<!-- #                                                         # -->

<!-- ########################################################### -->

| Extension | PHP 5.5 | PHP 5.6 | PHP 7.0 | PHP 7.1 | PHP 7.2 | PHP 7.3 | PHP 7.4 | PHP 8.0 | PHP 8.1 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| amqp | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| apcu | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| apcu_bc |  |  | ✓ | ✓ | ✓ | ✓ | ✓ |  |  |
| ast |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| bcmath | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| blackfire | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| bz2 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| calendar | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| cmark |  |  | ✓ | ✓ | ✓ | ✓ | ✓ |  |  |
| csv |  |  |  |  |  | ✓ | ✓ | ✓ | ✓ |
| dba | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| decimal |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| ds |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| enchant | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| ev | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| event | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| excimer |  |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| exif | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| ffi |  |  |  |  |  |  | ✓ | ✓ | ✓ |
| gd | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| gearman | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| geoip | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |  |
| geospatial | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| gettext | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| gmagick | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| gmp | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| gnupg | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| grpc | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| http | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| igbinary | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| imagick | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| imap | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| inotify | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| interbase | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |  |  |
| intl | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| ioncube_loader | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |  |
| jsmin | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |  |
| json_post | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| ldap | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| lzf | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| mailparse | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| maxminddb |  |  |  |  | ✓ | ✓ | ✓ | ✓ | ✓ |
| mcrypt | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| memcache | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| memcached | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| mongo | ✓ | ✓ |  |  |  |  |  |  |  |
| mongodb | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| mosquitto | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |  |
| msgpack | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| mssql | ✓ | ✓ |  |  |  |  |  |  |  |
| mysql | ✓ | ✓ |  |  |  |  |  |  |  |
| mysqli | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| oauth | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| oci8 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| odbc | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| opcache | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| opencensus |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| openswoole |  |  |  |  | ✓ | ✓ | ✓ | ✓ | ✓ |
| parallel[\*](#special-requirements-for-parallel) |  |  |  | ✓ | ✓ | ✓ | ✓ |  |  |
| pcntl | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pcov |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pdo_dblib | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| pdo_firebird | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pdo_mysql | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pdo_oci |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pdo_odbc | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pdo_pgsql | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pdo_sqlsrv[\*](#special-requirements-for-pdo_sqlsrv) |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pgsql | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| propro | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |  |
| protobuf | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| pspell | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| pthreads[\*](#special-requirements-for-pthreads) | ✓ | ✓ | ✓ |  |  |  |  |  |  |
| raphf | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| rdkafka | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| recode | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |  |  |
| redis | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| seaslog | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| shmop | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| smbclient | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| snmp | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| snuffleupagus |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| soap | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| sockets | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| solr | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| sourceguardian | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |
| spx |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| sqlsrv[\*](#special-requirements-for-sqlsrv) |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| ssh2 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| stomp | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |  |
| swoole | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| sybase_ct | ✓ | ✓ |  |  |  |  |  |  |  |
| sysvmsg | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| sysvsem | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| sysvshm | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| tensor[\*](#special-requirements-for-tensor) |  |  |  |  | ✓ | ✓ | ✓ | ✓ |  |
| tidy | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| timezonedb | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| uopz | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| uploadprogress | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| uuid | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| vips[\*](#special-requirements-for-vips) |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| wddx | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |  |  |
| xdebug | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| xhprof | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| xlswriter |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| xmldiff | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| xmlrpc | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| xsl | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| yac |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| yaml | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| yar | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| zephir_parser |  |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| zip | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| zookeeper | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| zstd | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

*Number of supported extensions: 116*

This extension comes from<https://github.com/mlocati/docker-php-extension-installer>
Refer to the sample files

### 3.4 Using php command line in Host (php-cli)

1.  reference[bash.alias.sample](bash.alias.sample)Sample file, copy the corresponding php cli function to the host `~/.bashrc`File.
2.  To make a file work:
    ```bash
    source ~/.bashrc
    ```
3.  You can then execute php commands in the host:
    ```bash
    ~ php -v
    PHP 7.2.13 (cli) (built: Dec 21 2018 02:22:47) ( NTS )
    Copyright (c) 1997-2018 The PHP Group
    Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
        with Zend OPcache v7.2.13, Copyright (c) 1999-2018, by Zend Technologies
        with Xdebug v2.6.1, Copyright (c) 2002-2018, by Derick Rethans
    ```

### 3.5 Use commoser

**Method 1: Use the compare command in the host**

1.  Determine the path to the compiler cache. For example, my dnmp download is in`~/dnmp`directory, that compiler's cache path is`~/dnmp/data/composer`。
2.  reference[bash.alias.sample](bash.alias.sample)A sample file that copies the corresponding php composer function to the host `~/.bashrc`File.
    > It is important to note here that the sample file is in`~/dnmp/data/composer`The directory needs to be the directory identified in the first step.
3.  To make a file work:
    ```bash
    source ~/.bashrc
    ```
4.  You can use a composer in any directory on the host:
    ```bash
    cd ~/dnmp/www/
    composer create-project yeszao/fastphp project --no-dev
    ```
5.  Optionally, the first time you use composer, it will be `~/dnmp/data/composer` Generate one under the directory**config.json**file, in which you can specify a domestic repository, for example:
    ```json
    {
        "config": {},
        "repositories": {
            "packagist": {
                "type": "composer",
                "url": "https://mirrors.aliyun.com/composer/"
            }
        }
    }

    ```

**Method 2: Use the composer command inside the container**

There is another way, which is to go into the container and execute`composer`Command, using the PHP7 container as an example:

```bash
docker exec -it php /bin/sh
cd /www/localhost
composer update
```

## 4. Administrative commands

### 4.1 Server Startup and Build Commands

To manage services, follow the command with the server name, for example:

```bash
$ docker-compose up                         # Create and start all containers
$ docker-compose up -d                      # Create and start all containers in background mode
$ docker-compose up nginx php mysql         # Create and start multiple containers of nginx, php, and mysql
$ docker-compose up -d nginx php  mysql     # Create and start nginx, php, and mysql containers running in the background


$ docker-compose start php                  # Start service
$ docker-compose stop php                   # Stop service
$ docker-compose restart php                # Restart service
$ docker-compose build php                  # Build or rebuild the service

$ docker-compose rm php                     # Delete and stop the php container
$ docker-compose down                       # Stop and delete containers, networks, images and mounted volumes
```

### 4.2 Add shortcut commands

At the time of development, we may use it often`docker exec -it`Entering the container and aliasing the commands commonly used is a convenient way to do so.

First, review the available containers in the host:

```bash
$ docker ps           # show all running containers
$ docker ps -a        # show all containers
```

Output`NAMES`That column is the name of the container, or if the default configuration is used, then the name is`nginx`、`php`、`php56`、`mysql`Wait.

Then, open`~/.bashrc`or`~/.zshrc`file, plus:

```bash
alias dnginx='docker exec -it nginx /bin/sh'
alias dphp='docker exec -it php /bin/sh'
alias dphp56='docker exec -it php56 /bin/sh'
alias dphp54='docker exec -it php54 /bin/sh'
alias dmysql='docker exec -it mysql /bin/bash'
alias dredis='docker exec -it redis /bin/sh'
```

The next time you enter the container, it is very fast, such as entering the php container:

```bash
$ dphp
```

### 4.3 View docker networks

```sh
ifconfig docker0
```

For filling in`extra_hosts`The container accesses the host`hosts`address

## 5. Use Log

The location where the log file is generated depends on the value of each log configuration under conf.

### 5.1 Nginx logs

Nginx logs are the logs we use the most, so we put them separately in the root directory`log`Under.

`log`The directory will be mapped for the Nginx container`/var/log/nginx`directory, so in the Nginx configuration file, the location of the output log needs to be configured`/var/log/nginx`Directories, such as:

    error_log  /var/log/nginx/nginx.localhost.error.log  warn;

### 5.2 PHP-FPM logs

In most cases, PHP-FPM logs are output to Nginx's logs, so no additional configuration is required.

In addition, it is recommended to open the error log directly in PHP:

```php
error_reporting(E_ALL);
ini_set('error_reporting', 'on');
ini_set('display_errors', 'on');
```

If you really need it, you can open it (in a container) by following the steps below.

1.  Go to the container, create a log file, and modify the permissions:
    ```bash
    $ docker exec -it php /bin/sh
    $ mkdir /var/log/php
    $ cd /var/log/php
    $ touch php-fpm.error.log
    $ chmod a+w php-fpm.error.log
    ```
2.  Open and modify the configuration file for PHP-FPM on the host`conf/php-fpm.conf`, find the following line, delete the comment, and change the value to:
        php_admin_value[error_log] = /var/log/php/php-fpm.error.log
3.  Restart the PHP-FPM container.

### 5.3 MySQL logs

Because MySQL in the MySQL container is used`mysql`The user starts, it cannot be self-contained`/var/log`Add log files under Add log files. So, we put mySQL logs in the same directory as data, i.e. projects`mysql`directory, corresponding to the container`/var/log/mysql/`Directory.

```bash
slow-query-log-file     = /var/log/mysql/mysql.slow.log
log-error               = /var/log/mysql/mysql.error.log
```

The above is the configuration of the log file in mysql.conf.

## 6. Database management

This project defaults to `docker-compose.yml`For MySQL online management is not turned on*phpMyAdmin*, and for redis online management*phpRedisAdmin*, which can be modified or deleted as needed.

### 6.1 phpMyAdmin

The port address of the phpMyAdmin container mapped to the host is:`8080`, so the address on the host to access phpMyAdmin is:

    http://localhost:8080

MySQL connection information:

*   host: (MySQL container network for this project)
*   port：`3306`
*   username:(Manually entered in the phpmyadmin interface)
*   password:(Manually entered in the phpmyadmin interface)

### 6.2 phpRedisAdmin

The port address of the phpRedisAdmin container mapped to the host is:`8081`, so the address on the host to access phpMyAdmin is:

    http://localhost:8081

The Redis connection information is as follows:

*   host: (Redis Container Network for this project)
*   port: `6379`

## 7. Safe to use in a formal environment

To use in a formal environment, please:

1.  Turn off XDebug debugging in php .ini
2.  Security policies to enhance Access to MySQL databases
3.  Enhanced security policies for redis access

## 8 Frequently Asked Questions

### 8.1 How to use curl in PHP code?

Refer to this issue:<https://github.com/yeszao/dnmp/issues/91>

### 8.2 Docker uses cron to time tasks

[Docker uses cron to time tasks](https://www.awaimai.com/2615.html)

### 8.3 Docker container time

The container time is configured in an .env file`TZ`For variables, see all supported time zones[List of time zones on Wikipedia](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)or[List of time zones supported by PHP· PHP official website](https://www.php.net/manual/zh/timezones.php)。

### 8.4 How to connect to MySQL and Redis servers

There are two cases of this.

The first case, in**PHP code**。

```php
// Connect to MySQL
$dbh = new PDO('mysql:host=mysql;dbname=mysql', 'root', '123456');

// Connect to Redis
$redis = new Redis();
$redis->connect('redis', 6379);
```

Because containers are containers`expose`The ports are connected, and they are in the same one`networks`down, so connected`host`Parameters directly with the container name,`port`The parameter is the port inside the container. Please refer to it for more[The Difference Between Docker-compose Ports and Expo](https://www.awaimai.com/2138.html)。

In the second case,**In the host**Pass**command line**or**Navicat**and other tools connected. For the host to connect mysql and redis, the container must pass through`ports`The port is mapped to the host. Take mysql as an example.`docker-compose.yml`There is such a thing in the document`ports`Disposition:`3306:3306`, that is, the 3306 port of the host and the 3306 port of the container form a map, so we can connect like this:

```bash
$ mysql -h127.0.0.1 -uroot -p123456 -P3306
$ redis-cli -h127.0.0.1
```

Over here`host`The parameter cannot be used localhost because it communicates with mysql through the sock file by default, and the container is isolated from the host file system, so it needs to be connected via TCP, so you need to specify the IP.

### 8.5 How php in a container connects to host MySQL

1\. Host execution`ifconfig docker0`get`inet`It's about connecting`ip`address

```sh
$ ifconfig docker0
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ...
```

2\. Run the host Mysql command line

```mysql
 mysql>GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
 mysql>flush privileges;
// The meaning of each character:
// *.* Valid for any table in any database
// "root" "123456" are the database username and password
// '%' means allowing any IP address to access the database, you can also specify the IP address
// flush privileges Refresh permission information
```

3\. Then use the php container directly`172.0.17.1:3306`Just connect

### 8.6 SQLSTATE\[HY000] \[1130] Host '172.19.0.2' is not allowed to connect to this MySQL server

1.  Currently using mysql-server `8.0.28`The above version, php version is required`7.4.7`The above can only be connected

## License

MIT
