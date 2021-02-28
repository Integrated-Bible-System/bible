# Server Info for the Integrated-Bible-System

## Server

- Kagoya VPS
  - Specifications
    - Open VZ
    - 6 Core
    - 2GB Memory
    - CentOS 7
  - IP Address
    - 133.18.25.204

## Domain

- Registerer
  - お名前.com
  - Domain name
    - bible-hymnal-tools.faith
    - A Record only
  - Users
    - root
    - admin _(wheel)_

### アカウント

- FTP
  - ftpuser
- Webadmin
  - webadmin@bible-hymnal-tools.faith
    - まだ`postfix`も`dovecot`もインストールしていない
    - && webadminも`useradd`していない

## 進捗

- 2021/02/22
  - vsftpdインストール
    - 初期設定
    - ユーザー`ftpuser`作成
      - ftp接続できず
  - apacheインストール
- 2021/02/23
  - vsftpd調整
    - `ftpuser`でのftp接続成功
  - httpd
    - 以下の設定ファイルを修正
      - `/etc/httpd/conf`以下
      - `/etc/httpd/conf.d`以下
      - `/etc/httpd/conf.modules.d`以下
    - iptablesでパケットフィルタリング解除
    - httpでの接続に成功
  - MariaDB
    - `sudo yum -y install mariadb mariadb-server`
    - `curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash`
    - まで…
- 2021/02/24
  - htpd
    - vhostの設定完了
      - www
      - hymns
      - locations
      - reader
      - wiki
    - php 8.4
      - インストール完了
      - [www](http://www.bible-hymnal-tools.faith)に`phpinfo();`のみの`index.php`を設置して動作確認済み
      - `composer`インストール済み
    - MariaDB
      - 本体の設定は文字コードの変更含めて完了
    - git
      - インストール完了
- 2021/02/25
  - phpMyAdmin
    - インストールしたが、`php8`に未対応
    - `yum`で`remi-80`を有効にして`yum upgrade`するも依存関係のエラー多発

TODO:

- MariaDB
  - phpMyAdminのアップデートエラーの解決
- httpd
  - `Node.js`と`Ruby on Rails`の検討
- postfix
  - **インストールせいやぁ！**
- dovecot
  - **インストールせいやぁ！**

### Downgrade to PHP 7.4

#### 2021/02/08

```bash
$ yum search --enablerepo=remi,remi-php74  php

php74-build.x86_64 : Package shipping basic build configuration
php74-php-ast.x86_64 : Abstract Syntax Tree
php74-php-ffi.x86_64 : Foreign Function Interface
php74-php-horde-horde-lz4.x86_64 : Horde LZ4 Compression Extension
php74-php-ioncube-loader.x86_64 : Loader for ionCube Encoded Files with ionCube
                                : 24 support
php74-php-maxminddb.x86_64 : MaxMind DB Reader extension
php74-php-opcache.x86_64 : The Zend OPcache
php74-php-pdo-dblib.x86_64 : PDO driver for Microsoft SQL Server and Sybase
                           : databases
php74-php-pdo-firebird.x86_64 : PDO driver for Interbase/Firebird databases
php74-php-pecl-ahocorasick.x86_64 : Effective Aho-Corasick string pattern
                                  : matching algorithm
php74-php-pecl-amqp.x86_64 : Communicate with any AMQP compliant server
php74-php-pecl-apcu.x86_64 : APC User Cache
php74-php-pecl-apcu-bc.x86_64 : APCu Backwards Compatibility Module
php74-php-pecl-apcu-devel.x86_64 : APCu developer files (header)
php74-php-pecl-apfd.x86_64 : Always Populate Form Data
php74-php-pecl-base58.x86_64 : Encode and decode data with base58
php74-php-pecl-bitset.x86_64 : BITSET library
php74-php-pecl-cmark.x86_64 : CommonMark extension
php74-php-pecl-crypto.x86_64 : Wrapper for OpenSSL Crypto Library
php74-php-pecl-dbase.x86_64 : dBase database file access functions
php74-php-pecl-decimal.x86_64 : Arbitrary-precision floating-point decimal
php74-php-pecl-dio.x86_64 : Direct I/O functions
php74-php-pecl-eio.x86_64 : Provides interface to the libeio library
php74-php-pecl-env.x86_64 : Load environment variables
php74-php-pecl-ev.x86_64 : Provides interface to libev library
php74-php-pecl-event.x86_64 : Provides interface to libevent library
php74-php-pecl-fann.x86_64 : Wrapper for FANN Library
php74-php-pecl-gender.x86_64 : Gender Extension
php74-php-pecl-geoip.x86_64 : Extension to map IP addresses to geographic places
php74-php-pecl-gmagick.x86_64 : Provides a wrapper to the GraphicsMagick library
php74-php-pecl-gnupg.x86_64 : Wrapper around the gpgme library
php74-php-pecl-grpc.x86_64 : General RPC framework
php74-php-pecl-handlebars.x86_64 : Handlebars templating language
php74-php-pecl-hrtime.x86_64 : High resolution timing
php74-php-pecl-http.x86_64 : Extended HTTP support
php74-php-pecl-http-devel.x86_64 : Extended HTTP support developer files
                                 : (header)
php74-php-pecl-http-message.x86_64 : PSR-7 HTTP Message implementation
php74-php-pecl-igbinary-devel.x86_64 : Igbinary developer files (header)
php74-php-pecl-imagick.x86_64 : Extension to create and modify images using
                              : ImageMagick
php74-php-pecl-imagick-devel.x86_64 : imagick extension developer files (header)
php74-php-pecl-inotify.x86_64 : Inotify
php74-php-pecl-interbase.x86_64 : InterBase/FireBird extension
php74-php-pecl-ip2location.x86_64 : Get geo location information of an IP
                                  : address
php74-php-pecl-ip2proxy.x86_64 : Get proxy information of an IP address
php74-php-pecl-json-post.x86_64 : JSON POST handler
php74-php-pecl-krb5.x86_64 : Kerberos authentification extension
php74-php-pecl-krb5-devel.x86_64 : Kerberos extension developer files (header)
php74-php-pecl-lua.x86_64 : Embedded lua interpreter
php74-php-pecl-luasandbox.x86_64 : Lua interpreter with limits and safe
                                 : environment
php74-php-pecl-lzf.x86_64 : Extension to handle LZF de/compression
php74-php-pecl-mcrypt.x86_64 : Bindings for the libmcrypt library
php74-php-pecl-memcache.x86_64 : Extension to work with the Memcached caching
                               : daemon
php74-php-pecl-memcached.x86_64 : Extension to work with the Memcached caching
                                : daemon
php74-php-pecl-memprof.x86_64 : Memory usage profiler
php74-php-pecl-mosquitto.x86_64 : Extension for libmosquitto
php74-php-pecl-msgpack.x86_64 : API for communicating with MessagePack
                              : serialization
php74-php-pecl-msgpack-devel.x86_64 : MessagePack developer files (header)
php74-php-pecl-mustache.x86_64 : Mustache templating language
php74-php-pecl-mysql.x86_64 : MySQL database access functions
php74-php-pecl-mysql-xdevapi.x86_64 : MySQL database access functions
php74-php-pecl-mysqlnd-azure.x86_64 : Redirection plugin for mysqlnd
php74-php-pecl-opencensus.x86_64 : A stats collection and distributed tracing
                                 : framework
php74-php-pecl-orng.x86_64 : Object scoped PRNG Extension
php74-php-pecl-parle.x86_64 : Parsing and lexing
php74-php-pecl-pcov.x86_64 : Code coverage driver
php74-php-pecl-pdflib.x86_64 : Package for generating PDF files
php74-php-pecl-pq.x86_64 : PostgreSQL client library (libpq) binding
php74-php-pecl-propro.x86_64 : Property proxy
php74-php-pecl-protobuf.x86_64 : Mechanism for serializing structured data
php74-php-pecl-psr.x86_64 : PSR interfaces
php74-php-pecl-radius.x86_64 : Radius client library
php74-php-pecl-raphf.x86_64 : Resource and persistent handles factory
php74-php-pecl-rdkafka.x86_64 : Kafka client based on librdkafka
php74-php-pecl-rdkafka4.x86_64 : Kafka client based on librdkafka
php74-php-pecl-rdkafka5.x86_64 : Kafka client based on librdkafka
php74-php-pecl-redis4.x86_64 : Extension for communicating with the Redis
                             : key-value store
php74-php-pecl-redis5.x86_64 : Extension for communicating with the Redis
                             : key-value store
php74-php-pecl-request.x86_64 : Server-side request and response objects
php74-php-pecl-rpminfo.x86_64 : RPM information
php74-php-pecl-runkit7.x86_64 : For all those things you... shouldn't have been
                              : doing anyway... but surely do!
php74-php-pecl-scrypt.x86_64 : Scrypt hashing function
php74-php-pecl-solr2.x86_64 : API orientée objet pour Apache Solr
php74-php-pecl-ssdeep.x86_64 : Wrapper for libfuzzy library
php74-php-pecl-ssh2.x86_64 : Bindings for the libssh2 library
php74-php-pecl-stats.x86_64 : Routines for statistical computation
php74-php-pecl-stomp.x86_64 : Stomp client extension
php74-php-pecl-svm.x86_64 : Support Vector Machine Library
php74-php-pecl-sync.x86_64 : Named and unnamed synchronization objects
php74-php-pecl-taint.x86_64 : XSS code sniffer
php74-php-pecl-tcpwrap.x86_64 : Tcpwrappers binding
php74-php-pecl-timecop.x86_64 : Time travel and freezing extension
php74-php-pecl-trader.x86_64 : Technical Analysis for traders
php74-php-pecl-translit.x86_64 : Transliterates non-latin character sets to
                               : latin
php74-php-pecl-uopz.x86_64 : User Operations for Zend
php74-php-pecl-uploadprogress.x86_64 : An extension to track progress of a file
                                     : upload
php74-php-pecl-uv.x86_64 : Libuv wrapper
php74-php-pecl-varnish.x86_64 : Varnish Cache bindings
php74-php-pecl-wddx.x86_64 : Web Distributed Data Exchange
php74-php-pecl-xattr.x86_64 : Extended attributes
php74-php-pecl-xdebug3.x86_64 : Provides functions for function traces and
                              : profiling
php74-php-pecl-xdiff.x86_64 : File differences/patches
php74-php-pecl-xlswriter.x86_64 : An efficient and fast xlsx file export
                                : extension
php74-php-pecl-xmldiff.x86_64 : XML diff and merge
php74-php-pecl-yac.x86_64 : Lockless user data cache
php74-php-pecl-yaconf.x86_64 : Yet Another Configurations Container
php74-php-pecl-yaf.x86_64 : Yet Another Framework
php74-php-pecl-yar.x86_64 : Light, concurrent RPC framework
php74-php-pecl-yaz.x86_64 : Z39.50/SRU client
php74-php-pecl-zip.x86_64 : Une extension de gestion des ZIP
php74-php-pecl-zmq.x86_64 : ZeroMQ messaging
php74-php-pggi.x86_64 : GTK bindings
php74-php-phalcon4.x86_64 : Phalcon Framework
php74-php-phpiredis.x86_64 : Client extension for Redis
php74-php-pinba.x86_64 : Client extension for Pinba statistics server
php74-php-realpath-turbo.x86_64 : Use realpath cache despite open_basedir
                                : restriction
php74-php-sodium.x86_64 : Wrapper for the Sodium cryptographic library
php74-php-wkhtmltox.x86_64 : HTML Converter
php74-php-zephir-parser.x86_64 : Zephir parser extension
 php74-php-zstd.x86_64 : Zstandard extension
```