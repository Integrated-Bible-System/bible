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

`PHP8.0`関連のものを全て削除。その後`phpMyAdmin`も削除。

改めて*-enablerepo=remi-php74*を指定して`php`と関連モジュールをインストール。

```bash
$ which php
/usr/bin/php
$ php -v
PHP 7.4.15 (cli) (built: Feb  2 2021 14:19:57) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
```

を確認。

同様に*-enablerepo=remi-php74*を指定して`phpMyAdmin`をインストール。

`/etc/phpMyAdmin/config.inc.php`は一応修正済み

`/etc/php.ini`は未着手。

を記述して、*MariaDB*にも設定sqlを投入して管理DB User作成済み。

### 2021/03/01

```bash
$ yum install --enablerepo=remi-php74 -y php-fpm
```

で、全てが`PHP80*`になってしまったので、一旦

```
$ yum remove --enablerepo=remi-php74 php*
```

を実行して全削除した後に、

```bash
$ yum install -y --enablerepo=remi-php74 php74 php74-php-devel php74-php-ffi php74-php-gd php74-php-intl php74-php-mbstring php74-php-mysqlnd php74-php-mcrypt php74-php-xml php74-php-bcmath php74-php-tokenizer php74-php-zip php74-php-pecl-xdebug
```

で再インストール。

現在は

```bash
$ yum list installed --enablerepo=remi-php74 php*
php74.x86_64
php74-php-bcmath.x86_64
php74-php-cli.x86_64
php74-php-common.x86_64
php74-php-devel.x86_64
php74-php-ffi.x86_64
php74-php-gd.x86_64
php74-php-intl.x86_64
php74-php-json.x86_64
php74-php-mbstring.x86_64
php74-php-mysqlnd.x86_64
php74-php-pdo.x86_64
php74-php-pecl-mcrypt.x86_64
php74-php-pecl-xdebug.x86_64
php74-php-pecl-zip.x86_64
php74-php-xml.x86_64
php74-runtime.x86_64
```

までインストール済み。

インストール・未インストールを問わずに収集した`php74*`は以下の通り

```bash
$ yum search --enablerepo=remi-php74 php |grep "^php74"
php74.x86_64 : Package that installs PHP 7.4
php74-php.x86_64 : PHP scripting language for creating dynamic web sites
php74-php-bcmath.x86_64 : A module for PHP applications for using the bcmath
php74-php-brotli.x86_64 : Brotli Extension for PHP
php74-php-cli.x86_64 : Command-line interface for PHP
php74-php-common.x86_64 : Common files for PHP
php74-php-componere.x86_64 : Composing PHP classes at runtime
php74-php-dba.x86_64 : A database abstraction layer module for PHP applications
php74-php-dbg.x86_64 : The interactive PHP debugger
php74-php-devel.x86_64 : Files needed for building PHP extensions
php74-php-embedded.x86_64 : PHP library for embedding in applications
php74-php-enchant.x86_64 : Enchant spelling extension for PHP applications
php74-php-fpm.x86_64 : PHP FastCGI Process Manager
php74-php-gd.x86_64 : A module for PHP applications for using the gd graphics
php74-php-geos.x86_64 : PHP module for GEOS
php74-php-gmp.x86_64 : A module for PHP applications for using the GNU MP
php74-php-imap.x86_64 : A module for PHP applications that use IMAP
php74-php-intl.x86_64 : Internationalization extension for PHP applications
php74-php-json.x86_64 : JavaScript Object Notation extension for PHP
php74-php-ldap.x86_64 : A module for PHP applications that use LDAP
php74-php-libvirt.x86_64 : PHP language binding for Libvirt
php74-php-libvirt-doc.noarch : Document of php-libvirt
php74-php-litespeed.x86_64 : LiteSpeed Web Server PHP support
php74-php-lz4.x86_64 : LZ4 Extension for PHP
php74-php-mbstring.x86_64 : A module for PHP applications which need multi-byte
php74-php-mysqlnd.x86_64 : A module for PHP applications that use MySQL
php74-php-oci8.x86_64 : A module for PHP applications that use OCI8 databases
php74-php-odbc.x86_64 : A module for PHP applications that use ODBC databases
php74-php-pdlib.x86_64 : A PHP extension for Dlib
php74-php-pdo.x86_64 : A database access abstraction module for PHP applications
php74-php-pear.noarch : PHP Extension and Application Repository framework
php74-php-pecl-cassandra.x86_64 : DataStax PHP Driver for Apache Cassandra
php74-php-pecl-couchbase2.x86_64 : Couchbase Server PHP extension
php74-php-pecl-couchbase3.x86_64 : Couchbase Server PHP extension
php74-php-pecl-csv.x86_64 : CSV PHP extension
php74-php-pecl-datadog-trace.x86_64 : APM and distributed tracing for PHP
php74-php-pecl-druid.x86_64 : A Druid driver for PHP
php74-php-pecl-ds.x86_64 : Data Structures for PHP
php74-php-pecl-gearman.x86_64 : PHP wrapper to libgearman
php74-php-pecl-geospatial.x86_64 : PHP Extension to handle common geospatial
php74-php-pecl-hdr-histogram.x86_64 : PHP extension wrapper for the C
php74-php-pecl-hprose.x86_64 : Hprose for PHP
php74-php-pecl-http-message-devel.x86_64 : php74-php-pecl-http-message developer
php74-php-pecl-ice.x86_64 : Simple and fast PHP framework
php74-php-pecl-igbinary.x86_64 : Replacement for the standard PHP serializer
php74-php-pecl-leveldb.x86_64 : LevelDB PHP bindings
php74-php-pecl-mailparse.x86_64 : PHP PECL package for parsing and working with
php74-php-pecl-mogilefs.x86_64 : PHP client library to communicate with the
php74-php-pecl-mongodb.x86_64 : MongoDB driver for PHP
php74-php-pecl-nsq.x86_64 : PHP extension for NSQ client
php74-php-pecl-oauth.x86_64 : PHP OAuth consumer extension
php74-php-pecl-pcs.x86_64 : PHP Code Service
php74-php-pecl-pcs-devel.x86_64 : PHP Code Service (header)
php74-php-pecl-pcsc.x86_64 : An extension for PHP using the winscard PC/SC API
php74-php-pecl-pcsc-devel.x86_64 : php74-php-pecl-pcsc developer files (header)
php74-php-pecl-pkcs11.x86_64 : PHP Bindings for PKCS11 modules
php74-php-pecl-propro-devel.x86_64 : php74-php-pecl-propro developer files
php74-php-pecl-psr-devel.x86_64 : php74-php-pecl-psr developer files (header)
php74-php-pecl-raphf-devel.x86_64 : php74-php-pecl-raphf developer files
php74-php-pecl-rar.x86_64 : PHP extension for reading RAR archives
php74-php-pecl-recode.x86_64 : A module for PHP applications for using the
php74-php-pecl-rrd.x86_64 : PHP Bindings for rrdtool
php74-php-pecl-scoutapm.x86_64 : Native Extension Component for ScoutAPM's PHP
php74-php-pecl-sdl.x86_64 : Simple DirectMedia Layer for PHP
php74-php-pecl-seasclick.x86_64 : An Yandex ClickHouse client driven extension
php74-php-pecl-seaslog.x86_64 : An effective, fast, stable log extension for PHP
php74-php-pecl-selinux.x86_64 : SELinux binding for PHP scripting language
php74-php-pecl-skywalking.x86_64 : The PHP instrument agent for Apache
php74-php-pecl-svn.x86_64 : PHP Bindings for the Subversion Revision control
php74-php-pecl-swoole4.x86_64 : PHP's asynchronous concurrent distributed
php74-php-pecl-tensor.x86_64 : Objects for scientific computing in PHP
php74-php-pecl-termbox.x86_64 : A termbox wrapper for PHP
php74-php-pecl-trie.x86_64 : PHP Trie extension
php74-php-pecl-uuid.x86_64 : Universally Unique Identifier extension for PHP
php74-php-pecl-vips.x86_64 : PHP extension for interfacing with libvips
php74-php-pecl-vld.x86_64 : Dump the internal representation of PHP scripts
php74-php-pecl-xdebug.x86_64 : PECL package for debugging PHP scripts
php74-php-pecl-xhprof.x86_64 : PHP extension for XHProf, a Hierarchical Profiler
php74-php-pecl-xmldiff-devel.x86_64 : php74-php-pecl-xmldiff developer files
php74-php-pecl-xxtea.x86_64 : XXTEA encryption algorithm extension for PHP
php74-php-pecl-yaconf-devel.x86_64 : php74-php-pecl-yaconf developer files
php74-php-pecl-yaml.x86_64 : PHP Bindings for yaml
php74-php-pgsql.x86_64 : A PostgreSQL database module for PHP
php74-php-process.x86_64 : Modules for PHP script using system process
php74-php-pspell.x86_64 : A module for PHP applications for using pspell
php74-php-smbclient.x86_64 : PHP wrapper for libsmbclient
php74-php-snappy.x86_64 : Snappy Extension for PHP
php74-php-snmp.x86_64 : A module for PHP applications that query SNMP-managed
php74-php-snuffleupagus.x86_64 : Security module for PHP
php74-php-soap.x86_64 : A module for PHP applications that use the SOAP protocol
php74-php-sqlsrv.x86_64 : Microsoft Drivers for PHP for SQL Server
php74-php-tidy.x86_64 : Standard PHP module provides tidy library support
php74-php-xml.x86_64 : A module for PHP applications which use XML
php74-php-xmlrpc.x86_64 : A module for PHP applications which use the XML-RPC
php74-php-zephir-parser-devel.x86_64 : php74-php-zephir-parser developer files
php74-php-zstd-devel.x86_64 : php74-php-zstd developer files (header)
php74-runtime.x86_64 : Package that handles php74 Software Collection.
php74-scldevel.x86_64 : Package shipping development files for php74
php74-unit-php.x86_64 : PHP module for NGINX Unit
php74-uwsgi-plugin-php.x86_64 : uWSGI - Plugin for PHP support
php74-xhprof.noarch : A Hierarchical Profiler for PHP - Web interface
php74-zephir.noarch : Zephir language for creation of extensions for PHP.
php74-build.x86_64 : Package shipping basic build configuration
php74-php-ast.x86_64 : Abstract Syntax Tree
php74-php-ffi.x86_64 : Foreign Function Interface
php74-php-horde-horde-lz4.x86_64 : Horde LZ4 Compression Extension
php74-php-ioncube-loader.x86_64 : Loader for ionCube Encoded Files with ionCube
php74-php-maxminddb.x86_64 : MaxMind DB Reader extension
php74-php-opcache.x86_64 : The Zend OPcache
php74-php-pdo-dblib.x86_64 : PDO driver for Microsoft SQL Server and Sybase
php74-php-pdo-firebird.x86_64 : PDO driver for Interbase/Firebird databases
php74-php-pecl-ahocorasick.x86_64 : Effective Aho-Corasick string pattern
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
php74-php-pecl-http-message.x86_64 : PSR-7 HTTP Message implementation
php74-php-pecl-igbinary-devel.x86_64 : Igbinary developer files (header)
php74-php-pecl-imagick.x86_64 : Extension to create and modify images using
php74-php-pecl-imagick-devel.x86_64 : imagick extension developer files (header)
php74-php-pecl-inotify.x86_64 : Inotify
php74-php-pecl-interbase.x86_64 : InterBase/FireBird extension
php74-php-pecl-ip2location.x86_64 : Get geo location information of an IP
php74-php-pecl-ip2proxy.x86_64 : Get proxy information of an IP address
php74-php-pecl-json-post.x86_64 : JSON POST handler
php74-php-pecl-krb5.x86_64 : Kerberos authentification extension
php74-php-pecl-krb5-devel.x86_64 : Kerberos extension developer files (header)
php74-php-pecl-lua.x86_64 : Embedded lua interpreter
php74-php-pecl-luasandbox.x86_64 : Lua interpreter with limits and safe
php74-php-pecl-lzf.x86_64 : Extension to handle LZF de/compression
php74-php-pecl-mcrypt.x86_64 : Bindings for the libmcrypt library
php74-php-pecl-memcache.x86_64 : Extension to work with the Memcached caching
php74-php-pecl-memcached.x86_64 : Extension to work with the Memcached caching
php74-php-pecl-memprof.x86_64 : Memory usage profiler
php74-php-pecl-mosquitto.x86_64 : Extension for libmosquitto
php74-php-pecl-msgpack.x86_64 : API for communicating with MessagePack
php74-php-pecl-msgpack-devel.x86_64 : MessagePack developer files (header)
php74-php-pecl-mustache.x86_64 : Mustache templating language
php74-php-pecl-mysql.x86_64 : MySQL database access functions
php74-php-pecl-mysql-xdevapi.x86_64 : MySQL database access functions
php74-php-pecl-mysqlnd-azure.x86_64 : Redirection plugin for mysqlnd
php74-php-pecl-opencensus.x86_64 : A stats collection and distributed tracing
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
php74-php-pecl-redis5.x86_64 : Extension for communicating with the Redis
php74-php-pecl-request.x86_64 : Server-side request and response objects
php74-php-pecl-rpminfo.x86_64 : RPM information
php74-php-pecl-runkit7.x86_64 : For all those things you... shouldn't have been
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
php74-php-pecl-uopz.x86_64 : User Operations for Zend
php74-php-pecl-uploadprogress.x86_64 : An extension to track progress of a file
php74-php-pecl-uv.x86_64 : Libuv wrapper
php74-php-pecl-varnish.x86_64 : Varnish Cache bindings
php74-php-pecl-wddx.x86_64 : Web Distributed Data Exchange
php74-php-pecl-xattr.x86_64 : Extended attributes
php74-php-pecl-xdebug3.x86_64 : Provides functions for function traces and
php74-php-pecl-xdiff.x86_64 : File differences/patches
php74-php-pecl-xlswriter.x86_64 : An efficient and fast xlsx file export
php74-php-pecl-xmldiff.x86_64 : XML diff and merge
php74-php-pecl-yac.x86_64 : Lockless user data cache
php74-php-pecl-yaconf.x86_64 : Yet Another Configurations Container
php74-php-pecl-yaf.x86_64 : Yet Another Framework
php74-php-pecl-yar.x86_64 : Light, concurrent RPC framework
php74-php-pecl-yaz.x86_64 : Z39.50/SRU client
php74-php-pecl-zip.x86_64 : A ZIP archive management extension
php74-php-pecl-zmq.x86_64 : ZeroMQ messaging
php74-php-pggi.x86_64 : GTK bindings
php74-php-phalcon4.x86_64 : Phalcon Framework
php74-php-phpiredis.x86_64 : Client extension for Redis
php74-php-pinba.x86_64 : Client extension for Pinba statistics server
php74-php-realpath-turbo.x86_64 : Use realpath cache despite open_basedir
php74-php-sodium.x86_64 : Wrapper for the Sodium cryptographic library
php74-php-wkhtmltox.x86_64 : HTML Converter
php74-php-zephir-parser.x86_64 : Zephir parser extension
php74-php-zstd.x86_64 : Zstandard extension
```

まだ`phpMyAdmin`までインストールしていない。

また現時点では`php74-php-fm`をインストールしていないので、`httpd`の設定ファイルを修正するついでに`/etc/httpd/conf.modules.d`以下のファイルの整合性を確認し始めたところ。

今日は`00-base.conf`で`LoadModule`の対象を修正した。

続きは明日！