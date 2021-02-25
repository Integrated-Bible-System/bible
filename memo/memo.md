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
