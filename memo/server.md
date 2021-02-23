
## 申し込み
[KAGOYA VPS申し込みページ](https://www.kagoya.jp/cloud/vps/apply/)から、KAGOYAのVPSサービスを申し込む事ができます。


### インスタンス作成準備

インスタンスは、一つのアカウントで複数作成する事が可能です。但し、インスタンス毎に課金されますので、注意が必要です。

余程の事情が無ければ、クリーンな*CentOS 7*のインスタンスを一つだけ作って、必要な環境は後から構築するという方法が賢明です。

今回はKAGOYA VPSの**Open VZ**を利用します。このプランでは

- マルチドメイン
- バーチャルホスト

の機能がありますので、一つのインスタンスで複数のドメインを管理して、且つ目的に合わせてホスト名を変更する事が可能です。

### CentOS7のクリーンインスタンス作成

**インスタンス作成**をクリックすると、どの様なスペックで、どの様なパッケージを選択するかと言う画面が表示されます。

今回は

- 3コア 1 GB / 2 GB 200GB HDD
- CenrOS7 64bit

を選択します。

また、初めての作成の時は**ログイン用認証キー**の作成が求められます。これはインストールした*CentOS*の*root*ユーザーがSSHでログインする際に、パスワードを入力しなくて済む様にする為です。ここでログイン用認証キーを作成すると、キーが自動的にダウンロードされますので、それは`~/.ssh/`に移動して

全て入力が終わったら、画面最下部の**インスタンス作成**ボタンをクリックします。

### インスタンスの起動

インスタンスが作成されると、詳細画面が表示されます。

この時に表示されている`IP`、即ちこのインスタンスの**グローバルIPアドレス**は必ず控えておいてください。

ちなみに*Jim*と行うこの*Biblical Web Apps*で使うこのサーバーの**グローバルIP**は**`133.18.25.204`**だからね！

ちなみにドメインは*bible-hymnal-tools.faith*だぞ。

IPアドレスを控えたら**稼働**ボタンをクリックして、インスタンスを起動します。

### インスタンスへの疎通確認

インスタンスへはグローバルIPアドレスが振り当てられていますので、ターミナルから

```bash
$ ping 133.18.25.204
```

を実行します。

その結果

```bash
PING 133.18.25.204 (133.18.25.204): 56 data bytes
64 bytes from 133.18.25.204: icmp_seq=0 ttl=44 time=55.721 ms
64 bytes from 133.18.25.204: icmp_seq=1 ttl=44 time=40.449 ms
64 bytes from 133.18.25.204: icmp_seq=2 ttl=44 time=52.961 ms
64 bytes from 133.18.25.204: icmp_seq=3 ttl=44 time=39.300 ms
64 bytes from 133.18.25.204: icmp_seq=4 ttl=44 time=39.896 ms
64 bytes from 133.18.25.204: icmp_seq=5 ttl=44 time=36.834 ms
```

と言う表示がされていれば、通信の疎通確認はできました。*ctrl+c*で処理を中止してください。

### インスタンスへのrootログイン

先ほど自動ダウンロードされた認証キーを用いてrootとしてログインをします。

```bash
$ ssh -i ~/.ssh/MainRoot.key root@133.18.25.204
```

そうすると、初めて接続するIPアドレスである為に「問題無いですか？」と問われますので、*yes*と返答してください。

```bash
ECDSA key fingerprint is SHA256:Ky64zFUOFNy3sIUOE5Bgaw++S3vYUaR4ZyM4TP+JOco.
Are you sure you want to continue connecting (yes/no
```

無事にログインが出来ましたら

```bash
[root@v133-18-25-204 ~]#
```

とプロンプトが表示され、現在は作成したインスタンスにおいてコマンド操作をしている事になります。

### rootユーザーにパスワードを設定

現在、このインスタンスに登録されている**SSHログイン可能なユーザー**はrootだけです。まだ何も設定していませんので、安全の為にrootにパスワードを設定します。

```bash
[root@v133-18-25-204 ~]# passwd root
ユーザー root のパスワードを変更。
新しいパスワード:
新しいパスワードを再入力してください:
passwd: すべての認証トークンが正しく更新できました。
```

として、パスワードを設定しておきます。

このパスワードは後に`su`コマンドで*Super User*になる時に必要ですので、必ず覚えておいてください。**メモしても良いですが危険なものですので、見つからない様にしてください。**

### 新しい管理ユーザーの作成

*root*と言うアカウントは、名前の通りこのインスタンスの全権を掌握しているユーザーで、その権力は計り知れないものがあります。

ですから*root*でログインをすると、うっかりミスで全てを壊してしまう危険性がありますので、代わりに*root*の持っている特権の一部が使える管理者アカウントを作成します。今回は*admin*とします。

```bash
[root@v133-18-25-204 ~]# useradd -G wheel admin
[root@v133-18-25-204 ~]# passwd admin
ユーザー admin のパスワードを変更。
新しいパスワード:
新しいパスワードを再入力してください:
passwd: すべての認証トークンが正しく更新できました。
```

として*admin*を作成します。Linuxにおいてユーザーを作成する時は必ず何れかのグループに属さなくてはなりません。これを`useradd -G グループ名 ユーザー名`で指定しないと、ユーザー名と同じグループが自動で作成されますので、注意が必要です。

今回指定した*wheel*は管理権限を持つグループです。無闇にこのグループへユーザーは追加しない方が良いでしょう。また、このユーザーで主にログインを行い操作をしますので、こちらのパスワードもしっかり覚えておきましょう。

#### パッケージの更新

ここで*admin*が管理者権限を使う為に使用するコマンドである`sudo`をインストールする必要がありますが、それには*yum*を利用します。

で、あれば先に*yum*をアップデートしておく方が無難です。

```bash
[root@v133-18-25-204 ~]# yum -y update
```

#### OSのアップデートの自動化

先ほど行った`yum -y update`はコマンドを実行すると起動しますが、定期更新はしてくれません。

そこで、予め自動更新の設定をします。

私が実行した時点では278項目の更新がありました。

```bash
[root@v133-18-25-204 ~]# yum -y install yum-cron
vi /etc/yum/yum-cron.conf
apply_updates = yes
systemctl start yum-cron
systemctl status -l yum-cron
systemctl enable yum-cron
```

### bash-completionの導入

```bash
[root@v133-18-25-204 ~]# yum install bash-completion
```

#### sudoのインストール

*admin*の属する*wheel*グループに`sudo`の権限を付与するには`visudo`が必要になりますので、インストールします。

```bash
[root@v133-18-25-204 ~]# yum -y install sudo
```

次に*wheel*グループに`sudo`コマンドの実行を許可します。

```bash
[root@v133-18-25-204 ~]# visudo
```

viエディタと同じ使い方が可能ですので、`/`を押して検索モードに入り**wheel**を検索します。

*%wheel  ALL=(ALL)       ALL*と言う行が見つかって、`#`デコメントアウトされているならば、コメントを削除して`:wq`で終了します。

### adminによるSSH接続の準備

もし、自分のローカルコンピューターに`~/.ssh/id_rsa.pub`が存在しない場合や、そもそも`~/.ssh`ディレクトリが無い場合は

```bash
$ cd $HOME
$ mkdir .ssh
$ cd .ssh
$ ssh-keygen -t rsa -b 4096
```

を実行して公開鍵ペアを作成します。但し、既に`~/.ssh`フォルダが存在している場合は`mkdir`は不要です。

次に、作成された`id_rsa.pub`をサーバーに転送します。この時に`sftp`と言う特殊なソフトを利用します。これは*SSH File Transfer Protocol*の略でSSHと言うセキュアなシェルを利用してファイルの送受信を行うものです。`SSL`レイヤーを利用した`FTP`である`FTPS`と混同しない様にしてください。

```bash
.ssh$ sftp admin@133.18.25.204
admin@133.18.25.204's password:
Connected to admin@133.18.25.204.
sftp> put id_rsa.pub
Uploading id_rsa.pub to /home/admin/id_rsa.pub
id_rsa.pub       100%  412     8.2KB/s  00:00
sftp> quit
```

これで`id_rsa.pub`はリモートの`/home/admin/id_rsa.pub`に転送されています。

### adminによるSSH接続設定

*root*でログインしたままのターミナルで`/home/admin/id_rsa.pub`を`/home/admin/.ssh/authorized_keys`に移動します。

```bash
[root@v133-18-25-204 ~]# cd /home/admin
[root@v133-18-25-204 admin]# ls -la
合計 24
drwx------ 2 admin admin 4096  2月 13 20:38 .
drwxr-xr-x 3 root  root  4096  4月 11  2018 ..
-rw-r--r-- 1 admin admin   18  9月 26  2014 .bash_logout
-rw-r--r-- 1 admin admin  193  9月 26  2014 .bash_profile
-rw-r--r-- 1 admin admin  231  9月 26  2014 .bashrc
-rw-r--r-- 1 admin admin  412  2月 13 20:38 id_rsa.pub
[root@v133-18-25-204 admin]# mkdir .ssh
[root@v133-18-25-204 admin]# mv id_rsa.pub .ssh/authorized_keys
```

次に今作成したディレクトリである`.ssh`や、その中に作成した`authorized_keys`は*root*ユーザーが操作していますので、所有権を*admin*に変更します。

```bash
[root@v133-18-25-204 admin]# chown -R admin:admin .
[root@v133-18-25-204 admin]# ls -la
合計 24
drwx------ 3 admin admin 4096  2月 13 20:46 .
drwxr-xr-x 3 root  root  4096  4月 11  2018 ..
-rw-r--r-- 1 admin admin   18  9月 26  2014 .bash_logout
-rw-r--r-- 1 admin admin  193  9月 26  2014 .bash_profile
-rw-r--r-- 1 admin admin  231  9月 26  2014 .bashrc
drwxr-xr-x 2 admin admin 4096  2月 13 20:46 .ssh
[root@v133-18-25-204 admin]# cd .ssh
[root@v133-18-25-204 .ssh]# ls -la
合計 12
drwxr-xr-x 2 admin admin 4096  2月 13 20:46 .
drwx------ 3 admin admin 4096  2月 13 20:46 ..
-rw-r--r-- 1 admin admin  412  2月 13 20:38 authorized_keys
[root@v133-18-25-204 .ssh]# chmod 700 .
[root@v133-18-25-204 .ssh]# chmod 600 authorized_keys
```

ついで…と言っては何ですが、`.ssh`フォルダの正しいパーミッションは*700*で、`authorized_keys`の正しいパーミッションは*600*なので修正しておきました。

#### Linuxのパーミッション

Linuxのパーミッションは属性が

- Read:4
- Write:2
- Execute:1

の三種類があります。上に表示されている`bash_profile`を見ると`-rw-r--r--`となっているのが判ります。

この内先頭の`-`はファイルならば`-`が、ディレクトリならば`d`が表示されますのでパーミッション本体は`rw-r--r--`と言う事になります。

ここで注意深く観察すると`rw-`・`r--`・`r--`の三つに分解可能な事が判ります。これは先頭から

- 所有者
- 所有者の属するグループ
- その他

に個別に上記の属性が設定できる事を意味します。

先ほど`authorized_keys`の正しいパーミッションは*600*として設定しましたが、これは

- 所有者: 6 = 4 + 2 = Read/Write
- 所有者の属するグループ: 0 = アクセス不可
- その他: 0 = アクセス不可

と言う意味になります。

### adminによるSSH接続

```bash
$  ssh -i ~/.ssh/id_rsa admin@133.18.25.204
[admin@v133-18-25-204 ~]$
```

と、無事にadminでログインできました。

#### adminの実行ファイル検索パスの追加

現在の*admin*の実行ファイル検索パスは

```bash
[admin@v133-18-25-204 ~]$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/admin/.local/bin:/home/admin/bin
```

です。ここに`/sbin`を追加します。`vi .bash_profile`で設定ファイルを開きます。

```conf
PATH=$PATH:$HOME/.local/bin:$HOME/bin

export PATH
```

と言う場所があります。ここの`export PATH`で、PATHが公開されてしまいますので

`PATH=$PATH:$HOME/.local/bin:$HOME/bin`
を
`PATH=$PATH:/sbin:$HOME/.local/bin:$HOME/bin`
に変更して`:wq`で保存します。

`.bash_profile`に変更を加えた場合は、必ず`source ~/.bash_profile`で変更を反映させる必要があります。

```bash
[admin@v133-18-25-204 ~]$ source ~/.bash_profile
[admin@v133-18-25-204 ~]$ echo $PATH
/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/admin/.local/bin:/home/admin/bin:/sbin:/home/admin/.local/bin:/home/admin/bin
```

と、PATHに`/sbin`が含まれている事が確認できます。

#### SSHのセキュリティ確保

1. SSHポートの変更
      1. SSHは通常`22`番ポートを利用しています。これは**Well Known Ports**の一つです。これをそのままにしておくと、IPアドレス+22番ポートに総当たり攻撃を受けると、突破される危険がありますので、変更します。
2. SSHの設定変更
      1. `sudo vi /etc/ssh/sshd_config`
         1. *\#Port 22*を*Port 10022*に変更
         2. *root*ユーザーのログイン禁止
            1. *PermitRootLogin without-password*を*\#PermitRootLogin without-password*とコメントアウト
         3. Passworによるログインを禁止して、公開鍵暗証方式でのログインのみ許容する。
            1. *PasswordAuthentication yes*を*\#PasswordAuthentication yes*とコメントアウト

上記修正が完了したら、`:wq`で保存して終了して、`sudo systemctl restart sshd.service`でSSHのサービスを再起動します。
sudo systemctl status -l sshd.service

## パケットフィルタリング

パケットフィルタリングとは、外部からパケットを受け取った時に、正しいパケットであるか？・転送先が別なIPアドレスの場合は中継点として転送を補助する・自分宛のパケットである場合、通信元のホスト・利用するプロトコル（サービス）等を判断して受信を許可するか否かを決定します。

同時に自分のローカルプロセスから外部へパケットを送出する際にも所定の許可の有無によって送出するか否かを決定します。

### iptablesを利用

CentOS 7からは、標準のパケットフィルタリング機構として*firewalld*が利用されています。しかし、今回は*VPS*と言う特殊な仮想環境の為に*firewalld*が利用できません。

また、本来はLinuxのセキュリティを上げる為に用いられる*SELinux*も利用できません。

物理サーバー等を借りた際に、よくWEBでは**面倒なので`SELinux`をオフにする**と言う記事がありますが、これは自分のサーバーのセキュリティを**自分が楽をする為**に下げると言うおバカな話ですので、その手の記事には騙されないでください。

#### iptablesの存在確認

```bash
[admin@v133-18-25-204 ~]$ systemctl list-unit-files --type=service
[admin@v133-18-25-204 ~]$ iptables.service enabled
[admin@v133-18-25-204 ~]$ systemctl status -l iptables.service
```

### iptablesの設定

今回は以下のポリシーで設定しました。

- 設定がない受信と転送は全部拒否
- メンテナンスのしやすを考えて、TCPの新規セッション用に「SERVICE」と言うチェインを作成
- ローカルループバックは全部許可
- プライベートアドレスからの受信は全部拒否
- ICMPはPINGの出入りのみ許可
- 名前解決のために、*UDP Port 53*の受信を許可
- 継続的なセッションを許可
- 新規セッションなのにSYNフラグの立っていないパケットは拒否
- 上記以外の新規セッションはSERVICEチェインも評価する
-  ポート*10022*の通信を許可

```bash
[admin@v133-18-25-204 ~]$ sudo vi /etc/sysconfig/iptables
*filter
:INPUT DROP [0:0]
:FORWARD DROP [0:0]
:OUTPUT  ACCEPT [0:0]
:SERVICE - [0:0]
-A INPUT -i lo -j ACCEPT
-A INPUT -s 10.0.0.0/8 -j DROP
-A INPUT -s 172.16.0.0/12 -j DROP
-A INPUT -s 192.168.0.0/16 -j DROP
-A INPUT -p icmp --icmp-type echo-request -j ACCEPT
-A INPUT -p icmp --icmp-type echo-reply -j ACCEPT
-A INPUT -p udp --sport 53 -j ACCEPT
-A INPUT -p tcp -m state --state ESTABLISHED,RELATED -j ACCEPT
-A INPUT -p tcp -m state --state NEW ! --syn -j DROP
-A INPUT -p tcp -m state --state NEW -j SERVICE
-A SERVICE -p tcp --dport 10022 -j ACCEPT
COMMIT
```

修正が完了したら、`:wq`で終了して`sudo systemctl restart iptables.service`で再起動する。

```bash
[admin@v133-18-25-204 ~]$ sudo systemctl status -l iptables.service
[admin@v133-18-25-204 ~]$ sudo systemctl enable iptables.service
```

#### ローカルからのログイン試験

```bash
$ ssh -i ~/.ssh/id_rsa -p10022 admin@133.18.25.204
Last login: Wed Feb 13 21:53:02 2019 from 59-171-70-62.rev.home.ne.jp
[admin@v133-18-25-204 ~]$
```

で接続が確認できました。

#### ローカルからのログインの簡素化

一々*admin*としてログインする度に`ssh -i ~/.ssh/id_rsa -p10022 admin@133.18.25.204`とするのは面倒ですし、また入力ミスの可能性もありますので簡素化します。

`~/.ssh/config`を`vi`で開き、以下の様に記述します。

```conf
host kagoya
Hostname 133.18.25.204
Port 10022
User admin
IdentityFIle ~/.ssh/id_rsa
ServerAliveInterval 300
ServerAliveCountMax 3
```

この設定を用いて

```bash
$ ssh kagoya
Last login: Wed Feb 13 22:23:58 2019 from 59-171-70-62.rev.home.ne.jp
[admin@v133-18-25-204 ~]$
```

とするだけで、ログインができます。

#### /.ssh/configとは

SSHクライアントがリモートサーバーに接続する際の情報を定義しているファイルで、このファイルに記述されているならば、SSHで接続する都度、`ssh`コマンドに長々とオプションを設定しなくて済みます。

##### 記述内容

| キーワード          | 内容                                                                      |
| :------------------ | :------------------------------------------------------------------------ |
| host                | SSH接続時の相手ホストの識別名                                             |
| Hostname            | ホストのIPアドレスかドメイン                                              |
| Port                | SSHのポート番号                                                           |
| User                | ログインユーザー名                                                        |
| IdentityFIle        | ログインユーザーが利用する秘密鍵のパス                                    |
| ServerAliveInterval | 一定期間サーバからデータが送られてこない時に、タイムアウトする秒数        |
| ServerAliveCountMax | ServerAliveIntervalで指定した秒数の間に応答がない場合に応答確認をする回数 |

つまり。

```conf
host kagoya
Hostname 133.18.25.204
Port 10022
User admin
IdentityFIle ~/.ssh/id_rsa
ServerAliveInterval 300
ServerAliveCountMax 3
```

の場合は300秒まで無応答を待機して、応答が無い場合は3回まで接続確認のメッセージを送信して応答があるか確認をします。

## DNSの設定

今までにインスタンスを作成する事によって、グローバルIPアドレスを取得する事ができました。

一方、大多数のWEBサイトや他のインターネットサービスはドメイン名を利用してアクセスしています。

このインスタンスに対してもドメイン名を付与した方が当然便利です。

ドメイン名の取得はKAGOYAでも出来ますが、専用の**レジストラ**と言われる業者で取得した方が安価です。

1. レジストラにて取得したドメイン名のNS*Name Server*をレジストラの物を利用する
1. まず**Aレコード**で*bible-hymnal-tools.faith*と*133.18.25.204*を紐付ける
1. 忘れずに**Whois**の代理公開をやめて自分を晒しましょう。

#### ゾーンの作成



#### レコードの設定

今回`prime-kobo.com`に対して作成するレコードは

- `prime-kobo.com`と言うドメインに割り当てられIPアドレスを指定する*A*レコード
- WEBサービスで使う`www`と言うホスト名は`prime-kobo.com`と同じIPアドレスである事を示す*CNAME*レコード
- FTPサービスで使う`ftp`と言うホスト名は`prime-kobo.com`と同じIPアドレスである事を示す*CNAME*レコード
- メール配信ホストして`smtp`を*A*レコードで`prime-kobo.com`と同じアドレス
- このドメインのメールの管理は`smtp.prime-kobo.com`が行う事を示す*MX*レコード
- メール受信ホストとして`pop`と`imap`は`prime-kobo.com`と同じIPアドレスである事を示す*CNAME*レコード

とします。

<img src="./images/KAGOYA_NS10.png" alt="レコード追加" width = 60%>

全ての設定が完了したら、上記の様になります。

後は、この設定したDNS情報が`.com`を管理する上位DNSサーバーに配布されて、設定したホスト名+ドメイン名でアクセスがあった場合に`.com`を管理するDNSサーバーが`prime-kobo.com`のホスト等を管理しているのは`ns0.kagoya.net`か`ns1.kagoya.net`だと返信をしてくれる様に待つだけです。最短で1時間〜最長で3日程かかります。

問題無くDNS情報が配布されると`dig`や`nslookup`でDNSの情報が確認できます。

```bash
$ dig @ns0.kagoya.net prime-kobo.com any
```

## ログ

### journaldの出力制限を緩和

```bash
[admin@v133-18-25-204 ~]$ cp /etc/systemd/journald.conf /etc/systemd/journald.conf.org
[admin@v133-18-25-204 ~]$ vi /etc/systemd/journald.conf
```

```conf
RateLimitInterval=10s
RateLimitBurst=20000
```

```bash
[admin@v133-18-25-204 ~]$ journalctl -u postfix --no-pager | wc -l
```

### rsyslogの出力制限の緩和

```bash
[admin@v133-18-25-204 ~]$ yum -y install rsyslog
[admin@v133-18-25-204 ~]$ systemctl start rsyslog
[admin@v133-18-25-204 ~]$ systemctl status -l rsyslog
[admin@v133-18-25-204 ~]$ systemctl enable rsyslog
[admin@v133-18-25-204 ~]$ vi /etc/rsyslog.conf
```

`/etc/rsyslog.conf`が見つからない

```bash
[admin@v133-18-25-204 ~]$
[admin@v133-18-25-204 ~]$
[admin@v133-18-25-204 ~]$
[admin@v133-18-25-204 ~]$
[admin@v133-18-25-204 ~]$
```
find / -name "rsyslog.conf" -type f
/usr/lib/dracut/modules.d/98syslog/rsyslog.conf

yum -y install mlocate
updatedb

locate rsyslog.conf

vi /usr/lib/dracut/modules.d/98syslog/rsyslog.conf

```conf
$imjournalRatelimitInterval 600
$imjournalRatelimitBurst    2400000
```

## メールサーバー

### 電子メールの仕組み

電子メールの仕組みをより良く理解する為に、改めて一度郵便が届く仕組みを考えます。

- 送り主は郵便局*A*に郵便物を届ける
- 郵便局*A*では、宛名住所を管轄する郵便局*B*を探す
- 郵便局*A*から郵便局*B*へ郵便物が転送される。
- 郵便局*B*から受け取り主のポストへ郵便物を届ける

今更、何を…なんて言わないでください。

上記と同じ事を電子メールで考えてみます。

- 送り主はホスト*A*にメールの配送を依頼する
- ホスト*A*では、受け取り主のメールアドレスから、そのメールを受け取れるホスト*B*を割り出す
- ホスト*A*からホスト*B*へメールが転送される
- ホスト*B*から受け取り主のメールボックスにメールを届ける

つまり、同じ事をしている分けです。


### 用語


#### MUA

MUA*Mail User Agent*とは、ユーザーがメールメッセージを作成・編集・整理したり、それを送信・受信するプログラムです。

MUAは次に説明するMSAに作成したメールの配信を依頼したり、ユーザーのメールスプールからメッセージを引き出す機能も持っています。

クライアントのローカルコンピューターで実行されるものです。


#### MSA

MSA*Mail Submission Agent*とは、MUAが作成したメールを受け取ってくれる窓口として機能します。

MSAは受け取ったメールの足りない情報を補完または加工した上で、適切なMTAに送信します。


#### MTA

MTA*Mail Transfer Agent*は、メールを受け取った時に、そのメールをどのMTAに送信すべきかを調べます。ここで探すMTAは、メールを受け取るべきユーザーへの送り方を知っている必要があります。適切なMTAが見つかったら、そのMTAにメールを配送します。

受け取ったメールの受け取り主が自分の担当分であれば、該当ユーザーのメールスプールにメールを配信します。


#### MDA

MDA*Mail Delivery Agent*とは、ユーザーのメールスプールにメールを配信するものです。つまり、MTAはMDAに依頼してメールを受け取り主に届けます。


#### SMTP

上記の流れの中で、MTAが別のMTAにメールを配信する方法、MUAがMSAにメールの配送を依頼するためのプロコトルをSMTP+Simple Mail Transfer Protocol*と呼びます。

一般に**メールサーバー**とは**SMTPサーバー**を意味しています。

一通の電子メールが送り主から受け取り主に届く、おおまかな流れは理解して頂けたかと思いますが、**メールスプール**については説明していませんでした。

MDAが行うメールの受け取り主への配信は、郵便局に例えれば私書箱に届けると言えます。メールユーザーは郵便局に出向き私書箱を確認する
必要があります。

これは、ユーザーが常にオンラインでは無い事を考えると必要な処置だと言えます。

この為、常にネットワーク上で稼働しているサーバーを用意して、そこに全てのメールを集めるという方法が採用されています。そのサーバーでは、MSA・MTA・MDAが稼働していて、必要なメールの配送と配信を常に行える様になっています。受け取ったメールは受け取りユーザー毎に仕分けされて、私書箱に格納します。この私書箱の事を**メールスプール**と読んでいます。


#### POP

メールを実際に受け取りたいユーザーは、MUAを使ってメールスプールからメールを取り出します。この時に用いられるプロコトルがPOP*Post Office Protocol*です。

ユーザーはPOPを用いてメールスプールのあるサーバーにアクセスして、メールをダウンロードします。ダウンロードされたメールはメールスプールから削除されます。


#### IMAP

MUAがメールスプールからメールを取り出すもう一つのプロコトルがIAMAP*Internet Message Access Protocol*です。

POPがメールをダウンロードするのに対して、IMAPはメールスプールにあるメールを閲覧するという方式を採用しています。またIMAPではサーバー上に階層的なメールフォルダを作成し、メールスプール内のメールを目的のメールフォルダに移動する等の管理機能も提供します。

IMAPの特徴は、メールは常にサーバーにあると言う事です。つまり、正しいIMAPサーバーとアカウントとパスワードを設定すれば、どの端末からでもメールを閲覧する事が可能になります。


### Postfixの導入

alternatives --config mta

*CentOS*では、MTAとして*postfix*が採用されていますのでインストールをします。

なお、この先は`sudo`コマンドを使い続ける為に、`su`で予め*Super User*になってから作業を行います。

```bash
[admin@v133-18-25-204 ~]$ su
パスワード:
[root@v133-18-25-204 admin]# yum -y install postfix
```


#### サービスの自動起動

postdixはインストールした時点で、自動起動する様に設定されています。


### パケットフィルタリング

外部からのSMTPに関連する通信を許可する必要があります。

```bash
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --dport 25 -j ACCEPT
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --sport 25 -j ACCEPT
[root@v133-18-25-204 admin]# iptables-save > /etc/sysconfig/iptables
[root@v133-18-25-204 admin]# systemctl restart iptables.service
systemctl status -l iptables.service
```


#### Postfixの最低限の設定

Postfixの設定ファイルは`/etc/postfix/main.cf`ですので、これを修正していきます。念の為にバックアップを作成してから行います。

```bash
[root@v133-18-25-204 admin]# cp /etc/postfix/main.cf /etc/postfix/main.cf.org
[root@v133-18-25-204 admin]# vi /etc/postfix/main.cf
```

- 利用するインターフェースの設定
- 受信アドレスの指定
- 自ホスト名とドメイン名の設定
    - サーバーでPostfixホストを識別する為のFPDNを指定します。
    - `myhostname = smtp.prime-kobo.com`
    - 同様にドメイン名も指定します。
    - `mydomain = prime-kobo.com`
    - `myorigin = $mydomain`
    - 外部から配送されるメールを受け取るインターフェースのアドレスを指定します。デフォルトでは*localhost*になっていますので、外部からのメールを受信できません。
    - `inet_interfaces = all`に変更します。
    - 配送されてきたメールのうち、ローカルのメールだと判断するドメインを指定します。
    - デフォルトでは`mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain`となっていますが、マルチドメインサーバー等の場合は、ここにドメインを追加する必要があります。
- メールリレーの制限
    - クライアントからのメールを無制限にリレーして、**SPAMの中継ホストにならない**ようにする設定がありますので、設定します。
    - リレー許可アドレスは`mynetworks = 192.168.10.0/28, 127.0.0.1`と設定して、ローカルネットワークからのみのリレーを許可します。
    - メールサーバーをインターネットに接続する場合には**必須の設定**です。
    - relay_domains = $mydestination


##### 設定の途中ですが、Postfixのローカル配送について

CentOSのPostfixの標準設定では、ローカルユーザーへメール配送を行う時に、メールボックス*mailbox*と呼ばれるファイルへの配送を行う様に設定されています。

*mailbox*形式で配送されたメールは、全て1つのファイルとして管理されます。CentOSでは一般的に`/var/spool/mail/<username>`の様なユーザー毎のファイルへ転送されます。

しかし、1つのファイルで管理するために例えばMTAがメールを配信しようとしている時に、POPサーバーが以前のメールを削除しようとすると、2つのプロセスが同時にファイルにアクセスする為に、ファイルロックの処理が必要になります。

またはMTAがメールをメールボックスに追記している最中に、サーバーの意図せぬ再起動やシャットダウン等が発生すると、メールボックスには正常に終了していない、中途半端なメールデータが残ってしまう可能性があります。

これに対してもう1つの方法である*Maildir*方式では、指定したディレクトリ配下に、1メールあたり1つのファイルとして保管されます。新しいメールが到着したら、MTAは*Maildir*に新規メールファイルを作成します。POPサーバーがメールを削除する場合は*Maildir*の当該ファイルのみを削除しますので、*mailbox*の時とは違い、同じファイルにアクセスしませんのでロックの心配も不要です。

障害時の事を考えても、該当のメールがゴミとして残るとしても、他のメールに影響は一切与えません。

この為に、一般に*Maildir*形式を採用する方がメールの安全性という意味では高いのですが、既定値での*Maildir*の場所はユーザーのホームディレクトリ配下になります。もちろん、これでも問題はありませんがSSHログインをしないユーザーの為にホームを用意するのも紛らわしいので*Maildir*の場所を変更します。

この場所を*Maildir*形式の配信先である`/var/spool/mail`に変更しておきます。

この時メールユーザー*foobar*を作成したのであれば

```bash
[root@v133-18-25-204 admin]# mkdir /var/spool/mail/foobar
[root@v133-18-25-204 admin]# chown foobar:mail /var/spool/mail/foobar
[root@v133-18-25-204 admin]# chmod 700 /var/spool/mail/foobar
```

とする必要があります。

既に、Adminユーザーが作られているので

rm -rf /var/spool/mainl/admin




#### main.cfの続き

- メール配送先の設定
    - `mail_spool_directory = /var/spool/mail`とします。
    - この時`home_mailbox`の設定は全てコメントアウトしてください。
- MTAソフト名の隠蔽
    - こちらがMTAにどのソフトのどのバージョンを利用しているかを公開するのは**脆弱性はここを突いてください**と言うのとほぼ同義です。
    - `smtpd_banner = $myhostname ESMTP unknown`とします。
- postfixで出来るSPAM対策
    - 受診時のエラーに対する遅延処理
        - メール送信者からの受信処理中にエラーを検出した場合に応答を遅延する機能を持っています。
        - このエラーはプロトコル上のエラーでけではなく、宛先不明などのエラーに対しても適用されます。
        - 宛先メールアドレスを変更しながら大量のメールを送信してくるSPAMホストや、ウィルスによるメール送信の場合に適用されます。
        - これ以降は`main.cf`には記述がありませんので、ファイル末尾に追記していきます。
        - `smtpd_soft_error_limit = 10`
        - *10*はエラーがこの数より大きくなったら応答遅延を開始するという設定です。
    - 応答遅延時間の設定
        - 応答遅延を行う場合に、どの程度の時間を遅延させるかと言う設定で、デフォルトは*1*秒ですが、私は*5*秒と設定しました。
        - `smtpd_error_sleep_time = 5s`
    - 許容されるエラー件数の設定
        - 応答遅延を行ってもエラーが発生する場合には、指定した上限値以上のエラーは許容せずに、postfixが送信元のSMTP接続を強制的に接続切断するまでの値です。
        - `smtpd_hard_error_limit = 20`
    - メールサイズの上限
        - メールサイズに上限を設定する事によって、大きな画像ファイル等が添付されたメールの送受信を防ぐ事ができます。どんなサイズのファイルでもメールで送受信できると、メールスプールの容量を圧迫しますし、送信先のメールサーバーにも余計な負担をかけます。
        - `message_size_limit = 10485760`と、10MBバイトまでに制限します。
    - メール再送時間の設定
        - メールの配送が失敗した場合はPostfixはキューマネージャが自動的に再送信を行います。この再送間隔が長すぎては、なかなかメールが届きませんが、短すぎると実際に相手のサーバーが停止していた場合に不可をかけるだけです。
        - `transport_retry_time = 90s`として、90秒毎に再送信する様にしました。
      - メール保持期間の設定
        - メールの配送が失敗した場合は上記の設定に従い、メールの再送を試みますが、いつまでもメールが配送できない場合にどれだけキューにメールを置いておいてメール再送を試みるかを設定します。
          - `maximal_queue_lifetime = 7d`として7日は保存する事にしました。
inet_protocols = all
inet_protocols = ipv4
一旦ここで`main.cf`を保存して終了します。


### SMTP-AUTH

SMTP-AUTHは、SMTPで接続してメールを送信しようとする時に、ユーザー情報を交換する事でメール送信を行おうとするユーザーを特定する仕組みです。

純粋なSMTPはメール発信者を特定する事ができません。その為にSPAMメールの踏み台にされても、発信者を特定する事ができません。

SMTP-AUTHを導入するとSMTPを使いメールを送信しようとする時にユーザーアカウントとパスワードの認証を行い、認証を通過したユーザーにのみメールの配信を許可する事ができます。

SMATP-AUTHによる認証データの交換方法は以下の通りです。

| 方法       | 処理                                                                                                 |
| :--------- | :--------------------------------------------------------------------------------------------------- |
| PLAIN      | 平文をBASE64エンコードします                                                                         |
| LOGIN      | 標準仕様外の為に非推奨です                                                                           |
| CRAM-MD5   | サーバーから取得した文字列を元に、メッセージのダイジェスト <br> を作成して認証情報を交換します       |
| DIGEST-MD5 | CRAM-MD5では、総当たり攻撃に弱い為に、HMAC鍵交換方式 <br> 等のより高度な暗号化で認証情報を交換します |

これらの認証情報はMUAによってサポートしているものが違います。まず間違い無く全てのMUAでサポートされているのは*PLAIN*ですが、ネットワーク上に**平文が流れる事を嫌う**と言う事から、近年では*CRAM-MD5*をサポートしているものも増えて来ています。


#### SASLの導入

CentOSのPostfixは**SASL-Simple Authentification and Security Layer**と連携して、SMTP-AUTHを実現しています。

まず、SASLに関連するパッケージをインストールします。

```bash
[root@v133-18-25-204 admin]# yum -y install cyrus-sasl cyrus-sasl-md5 cyrus-sasl-plain
[root@v133-18-25-204 admin]# systemctl start saslauthd
[root@v133-18-25-204 admin]# systemctl status -l saslauthd
[root@v133-18-25-204 admin]# systemctl enable saslauthd
```

インストールが成功したら、サービスを起動すると同時に自動起動の設定を行います。


#### Postfixの設定ファイルへの追記

以下の目的で設定を行います。

1. SASLを有効化する
2. Outlook等の規格外のLOGIN認証を受け付けない
3. SMTPでメールの宛先を受信した時に
      1. mynetworksで設定して自ネットワークからの接続の場合には、全ての宛先を受け付ける
      2. SASL認証を行ったクライアントからは、すべての宛先を受け付ける
      3. 自ホスト宛のアドレス以外は拒否する

```conf
alias_maps = hash:/etc/postfix/aliases
alias_database = hash:/etc/postfix/aliases
smtpd_sasl_auth_enable = yes
broken_sasl_auth_clients = no
smtpd_recipient_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination
# If Postfix >= 2.10
smtpd_relay_restrictions = permit_mynetworks, permit_sasl_authenticated, reject_unauth_destination
```

##### 2020/06/15

```conf
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
smtpd_sasl_auth_enable = yes
smtpd_sasl_security_options = noanonymous
smtpd_sasl_local_domain = $mydomain
broken_sasl_auth_clients = yes
smtpd_tls_cert_file = /etc/pki/tls/certs/prime-kobo.com.crt
smtpd_tls_key_file = /etc/pki/tls/private/prime-kobo.com.2020.key
smtpd_tls_CAfile = /etc/pki/tls/certs/kingcacert.cer
smtpd_tls_security_level = may
smtp_tls_loglevel = 1
smtpd_recipient_restrictions = permit_sasl_authenticated permit_mynetworks reject_non_fqdn_hostname reject_non_fqdn_sender reject_non_fqdn_recipient reject_invalid_hostname permit
smtpd_relay_restrictions = permit_mynetworks, permit_auth_destination, permit_sasl_authenticated, reject
smtpd_client_restrictions = permit_mynetworks,
smtpd_soft_error_limit = 10
smtpd_error_sleep_time = 5s
smtpd_hard_error_limit = 20
message_size_limit = 10485760
transport_retry_time = 90s
maximal_queue_lifetime = 7d
```

Postfixの設定ファイルをここで保存して閉じます。


### CRAM-MD5認証の設定

ユーザーのパスワードを保存する`/etc/passwd`や`/etc/shadow`には、パスワードのダイジェストのみが補間されている為に、元のパスワードを復号化できません。

これはセキュリティの観点からは望ましい状況ですが、CRAM-MD5認証では利用できない事を意味します。

その為*cyrus-sasl*では、オリジナルの認証データベースを使う事が推奨されています。i


#### PostfixのSASL設定

CRAM-MD5認証を行うためには、PALIN認証が失敗した場合やCRAM-MD5認証を要求された場合に、認証データベースをチェックする様に設定する必要があります。

`/etc/sasl2/smtpd.conf`を

```conf
pwcheck_method: auxprop
auxprop_plugins: sasldb
#mech_list: plain cram-md5
```

##### 2020/06/15

```conf
pwcheck_method: auxprop
mech_list: cram-md5 plain login
```

と設定します。その後Postfixとsaslauthdを再起動します。

```bash
[root@v133-18-25-204 admin]# systemctl restart postfix
[root@v133-18-25-204 admin]# systemctl status -l postfix
[root@v133-18-25-204 admin]# systemctl restart saslauthd
[root@v133-18-25-204 admin]# systemctl status -l saslauthd
```

### 認証データベースへのユーザー登録

実際の認証は、認証データベースを使って行われますので、認証データベースに必要なユーザーを登録する必要があります。

ユーザーの登録にはCRAM-MD5の暗号で使うdomainが必要です。domainは基本的にはホスト名ですが、サーバー内でSMTPサーバーに接続をして確認をする事もできます。

このサーバー内接続は、様々な問題が発生した時に非常に便利な方法ですので*telnet*パッケージをインストールしてから行います。

```bash
[root@v133-18-25-204 admin]# yum -y install telnet
[root@v133-18-25-204 admin]# telnet 127.0.0.1 smtp
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
220 smtp.prime-kobo.com ESMTP unknown
quit
221 2.0.0 Bye
Connection closed by foreign host.
```

*200*とある行でPostfixの設定ファイルで指定した通りにホスト名は表示されていますが、ソフトウェア名やバージョン名は表示されていない事も確認できました。

次の行で`quit`と入力して、SMTPサーバーとの接続を遮断します。

##### 2020/06/15

```bash
[root@prime-kobo etc]# telnet 127.0.0.1 smtp
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
220 smtp.prime-kobo.com ESMTP unknown
quit
221 2.0.0 Bye
Connection closed by foreign host.
```

#### ドメイン名の変更

今、確認した様にCRAM-MD5で利用するドメインは`smtp.prime-kobo.com`ですが、設定として考えた時に、これはSMTPサーバーのFQDN名であってドメイン名は`prime-kobo.com`です。これをPostfixの設定ファイルである`/etc/postfix/main.cf`で修正します。

```conf
smtpd_sasl_local_domain = $mydomain
```

##### 2020/06/15

```conf
mydomain = prime-kobo.com
smtpd_sasl_local_domain = $mydomain
```

では、これで一度`saslautd`と`postfix`を再起動します。

```bash
[root@v133-18-25-204 admin]# systemctl restart saslauthd
[root@v133-18-25-204 admin]# systemctl restart postfix
```


#### 認証データベースへのユーザーの登録

1. 既存ユーザーである`admin`を追加する

```bash
[root@v133-18-25-204 admin]# saslpasswd2 -u smtp.prime-kobo.com admin
Password:
Again (for verification):
[root@v133-18-25-204 admin]# chgrp postfix /etc/sasldb2
```

##### 2020/06/15

```bash
-rw-r----- 1 root postfix 12K  5月 23 18:14 /etc/sasldb2
```

最後の行で認証データベースの所有グループを変更しています。

*Maildir*の作成

```bash
[root@v133-18-25-204 admin]# mkdir /var/spool/mail/admin
[root@v133-18-25-204 admin]# chown admin:mail /var/spool/mail/admin
[root@v133-18-25-204 admin]# chmod 700 /var/spool/mail/admin
```

2. Homeを持たない新規ユーザー`masaru.kitajima`の追加

```bash
[root@v133-18-25-204 admin]# useradd -s /sbin/nologin masaru.kitajima
[root@v133-18-25-204 admin]# passwd masaru.kitajima
ユーザー masaru.kitajima のパスワードを変更。
新しいパスワード:
新しいパスワードを再入力してください:
passwd: すべての認証トークンが正しく更新できました。
[root@v133-18-25-204 admin]# saslpasswd2 -u smtp.prime-kobo.com masaru.kitajima
Password:
Again (for verification):
[root@v133-18-25-204 admin]# mkdir /var/spool/mail/masaru.kitajima
[root@v133-18-25-204 admin]# chown masaru.kitajima:mail /var/spool/mail/masaru.kitajima
[root@v133-18-25-204 admin]# chmod 700 /var/spool/mail/masaru.kitajima
```

##### 2020/06/15

```bash
drwx------    8 biblestudy.admin mail 4.0K  5月 23 18:48 biblestudy.admin/
drwx------    5 blog.author      mail 4.0K  5月  3 11:06 blog.author/
drwx------    5 hymnsearch       mail 4.0K  5月  3 11:06 hymnsearch/
drwx------    5 info             mail 4.0K  5月  3 11:06 info/
drwx------  132 masaru.kitajima  mail  12K  6月 15 11:20 masaru.kitajima/
drwx------    5 redmine.admin    mail 4.0K  5月  3 11:06 redmine.admin/
```

この時`useradd`コマンドで利用した`--shell /sbin/nologin`はSSHでの接続を認めないという意味です。SSHは様々な操作を直接システムに対して行う事ができる為に、可能な限りログイン可能なユーザーは減らしておくべきです。

なお、現在sasldb2に登録されているユーザーは

```bash
[root@v133-18-25-204 admin]# sasldblistusers2
```

で確認できます。

##### 2020/06/15

```bash
[root@prime-kobo etc]# sasldblistusers2
biblestudy.admin@smtp.prime-kobo.com: userPassword
hymnsearch@smtp.prime-kobo.com: userPassword
info@smtp.prime-kobo.com: userPassword
blog.author@smtp.prime-kobo.com: userPassword
masaru.kitajima@smtp.prime-kobo.com: userPassword
redmine.admin@smtp.prime-kobo.com: userPassword
```

また、登録ユーザーの削除は

```bash
[root@v133-18-25-204 admin]# saslpasswd2 -d -u smtp.prime-kobo.com info
```

で行えます。

### SMTP/Submission

インターネットのMTA間のメール中継では、通常はTCPのPort 25が利用されます。通常のSMTPのメール送受信の手順には、特別な認証などが無い為にこのポートを使えば、誰でもメールの送信が可能です。しかし、これを悪用してウィルスメールやSPAMメール等の不正なメール送信が増加しています。


これに対して、近年はOP25B*-Outbouond Port 25 Blocking-*を各ISP*-Internet Service Privider-*が導入しており、TCP Port 25のメールの送受信は、自分のISP内でのメールのやり取りに限定しています。

これはウィルスメールの送信を阻止したり、SPAMを送信したユーザーをISPのログから特定できる等のメリットがあり、有効に機能しています。

一方で、そのIPS内のPCからISP外にはメールを送信できないと言う事をも意味します。

その為、TCP Port 587*-Submission-*を利用したメールの送信を可能にしています。*Submission*でのメールの送信は、必ず送信ユーザーの認証が必要になります。また、この時にパスワードやSMTPサーバーとの送受信を保護する為にSSL/TLSによる暗号化を実施します。この仕組みを**SMTP/Submission over SSL/TSL**とよびます。


#### SMTP/Submissionの設定

MTP/Submission over SSL/TSLでは、次の2つのポートを有効にする必要があります。

1. SMTP Port
      - 他のメールサーバーからのメールの配送を受け付けます
2. submission Port
      - ユーザー認証・SSL/TLSによる暗号化を有効にし、信頼できるユーザーの配送のみを受け付けます。

今までPostfixの基本的な設定は`main.cf`で行ってきましたが、SMTP/Submission over SSL/TSLを利用するには、この2つのポートに対して異なる設定を行う必要があります。

Postfixでは処理を行うべきポートの設定を`master.cf`で行いますが、その際にオプションを指定する事で、各ポートに特有の設定を行う事ができます。


#### パケットフィルタリングの設定

```bash
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --dport 587 -j ACCEPT
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --sport 587 -j ACCEPT
[root@v133-18-25-204 admin]# iptables-save > /etc/sysconfig/iptables
[root@v133-18-25-204 admin]# systemctl restart iptables.service
systemctl status -l iptables.service
```


#### master.cfの変更

PostfixでSubmissionポートを有効にする為には`/etc/postfix/master.cf`に設定を追加します。

いつも通り、このファイルをコピーしてバックアップを作成してから`vi`で開きます。

```conf
smtp      inet  n       -       n       -       -       smtpd
#smtp      inet  n       -       n       -       1       postscreen
#smtpd     pass  -       -       n       -       -       smtpd
#dnsblog   unix  -       -       n       -       0       dnsblog
#tlsproxy  unix  -       -       n       -       0       tlsproxy
#submission inet n       -       n       -       -       smtpd
# -o syslog_name=postfix/submission
# -o smtpd_tls_security_level=encrypt
# -o smtpd_sasl_auth_enable=yes
# -o smtpd_reject_unlisted_recipient=no
# -o smtpd_client_restrictions=$mua_client_restrictions
# -o smtpd_helo_restrictions=$mua_helo_restrictions
# -o smtpd_sender_restrictions=$mua_sender_restrictions
# -o smtpd_recipient_restrictions=permit_sasl_authenticated,reject
# -o milter_macro_daemon_name=ORIGINATING
```

となっている箇所を

```conf
smtp      inet  n       -       n       -       -       smtpd
#smtp      inet  n       -       n       -       1       postscreen
#smtpd     pass  -       -       n       -       -       smtpd
#dnsblog   unix  -       -       n       -       0       dnsblog
#tlsproxy  unix  -       -       n       -       0       tlsproxy
submission inet n       -       n       -       -       smtpd
  -o syslog_name=/var/maillog
  -o smtpd_tls_security_level=may #TLS/SSL導入後encryptに変換
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_reject_unlisted_recipient=no
# -o smtpd_client_restrictions=$mua_client_restrictions
# -o smtpd_helo_restrictions=$mua_helo_restrictions
# -o smtpd_sender_restrictions=$mua_sender_restrictions
  -o smtpd_recipient_restrictions=permit_sasl_authenticated,reject
# -o milter_macro_daemon_name=ORIGINATING
```

と、書き換えます。

変更したのは

1. Sumbissionポートの有効化
2. `syslog_name=postfix/submission`で、通常のSMTPポートで受信したメールとは区別してログを作成します。
3. `smtpd_tls_security_level=encrypt`の設定は、`TLS`の利用をクライアントに強制します。もし、クライアントが`TLS`の利用が可能な場合ならば利用すると、少し制限を甘くするのであれば`smtpd_tls_security_level=may`とすると良いです。
4. `smtpd_sasl_auth_enable=yes`と`smtpd_recipient_restrictions=permit_sasl_authenticated,reject`はsasl認証が必須である事を指定します。
5. 存在しない宛先は拒否する。

##### 2020/06/15

```conf
smtp      inet  n       -       n       -       -       smtpd
#smtp      inet  n       -       n       -       1       postscreen
#smtpd     pass  -       -       n       -       -       smtpd
#dnsblog   unix  -       -       n       -       0       dnsblog
#tlsproxy  unix  -       -       n       -       0       tlsproxy
submission inet n       -       n       -       -       smtpd
  -o syslog_name=/var/submission
  -o smtpd_tls_security_level=may
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_reject_unlisted_recipient=no
#  -o smtpd_client_restrictions=$mua_client_restrictions
#  -o smtpd_helo_restrictions=$mua_helo_restrictions
#  -o smtpd_sender_restrictions=$mua_sender_restrictions
  -o smtpd_recipient_restrictions=permit_sasl_authenticated,reject
#  -o milter_macro_daemon_name=ORIGINATING
smtps     inet  n       -       n       -       -       smtpd
  -o syslog_name=/var/maillog
  -o smtpd_tls_wrappermode=yes
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_reject_unlisted_recipient=no
#  -o smtpd_client_restrictions=$mua_client_restrictions
#  -o smtpd_helo_restrictions=$mua_helo_restrictions
#  -o smtpd_sender_restrictions=$mua_sender_restrictions
  -o smtpd_recipient_restrictions=permit_sasl_authenticated,reject
#  -o milter_macro_daemon_name=ORIGINATING
```

[root@v133-18-25-204 log]# systemctl restart postfix
[root@v133-18-25-204 log]# systemctl status -l postfix


### dovecotの導入

CentOSで利用可能なPOP3/IMAPのvサービスソフトウェアとしては*Dovecot*と*Cyrus IMAP*の2つがあります。

＊Dovecot*は、他のサービスソフトウェアと比べると若干歴史は浅いのですが、セキュリティに重点を置いています。

*Cyrus IMAP*は、Linuxユーザー環境とメールユーザを分離するというコンセプトで動作しています。

今回は*dovecot*を採用します。

上述の通り、セキュリティに重点が置かれている他にも*Maildir*方式や共有メールボックス・MBOX形式にも完全対応していて、更に様々な認証方法が使える等機能が法湯なのが、今回*dovecot*を採用する理由です。

```bash
[root@v133-18-25-204 admin]# yum -y install dovecot clucene-core
[root@v133-18-25-204 admin]# systemctl start dovecot
[root@v133-18-25-204 admin]# systemctl status -l dovecot
[root@v133-18-25-204 admin]# systemctl enable dovecot
```


### パケットフィルタリング

今回は*imaps, pop3s, imap, pop*のポートを解放します。

POPやIMAPではパスワードがネットワーク上を流れますので、SSL/TLSを使ったPOP3 over SSL/TLS*-pops*とIMAP over SSL/TLS*-imaps-*の利用を強く推奨しています。

当然このサーバーにもこの後でSSL証明書を導入して*HTTP*も*FTP*も*SMTP*も、そして当然ながら*POP*や*IMAP*も全てSSL/TLS化します。

詳しくは、サーバー証明書の導入時に説明しますが、一般にきちんとしたサーバー証明書の購入を行うには、予めSSL業者が指定するメールアドレスでメールを受信できる環境になくてはなりません。

その為、SSL/TLS化されていなくてもメールの送受信ができなくては、そもそもSSL証明書の購入すらできないと言う事になります。


```bash
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --dport 110 -j ACCEPT
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --sport 110 -j ACCEPT
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --dport 143 -j ACCEPT
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --sport 143 -j ACCEPT
[root@v133-18-25-204 admin]# iptables-save > /etc/sysconfig/iptables
[root@v133-18-25-204 admin]# systemctl restart iptables.service
[root@v133-18-25-204 admin]# systemctl status -l iptables.service
```

##### 2020/06/15

```bash
[root@prime-kobo postfix]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: eth0
  sources:
  services: dhcpv6-client ftp http https mysql ssh
  ports: 80/tcp 443/tcp 25/tcp 110/tcp 143/tcp 465/tcp 587/tcp 993/tcp 995/tcp 50000-50030/tcp 21/tcp 20/tcp 989/tcp 990/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

- Port 110: POP3
- Port 143: IMAP


#### Dovecotの最低限の設定

Dovecotの設定は、`/etc/dovecot/dovecot.conf`が基本で、Dovecot起動時にはこのファイルを読み込みます。しかし、このファイルは本当に基本的な事しか設定されておらず、各種の詳細については`/etc/dovecot/conf.d/`以下のファイルを読み込む様に設定されています。

では、まず`/etc/dovecot/dovecot.conf`をコピーして`vi`で開きます。

現時点では`#protocols = imap pop3 lmtp`の`#`を外すだけで結構です。`iamps`と`pop3s`の追加はサーバー証明書の取得後に行います。

また、lmtpは_local mail transfer protocol_の略でローカルでのメール配送のためのものですから、通常は外さない方が良いでしょう。


```bash
[root@v133-18-25-204 admin]# vi /etc/dovecot/conf.d/10-ssl.conf
```

```conf
#ssl = yes
ssl = no
```


#### 認証方法の設定

認証方式は`/etc/dovecot/conf.d/10-auth.conf`で行います。

Dovecotが利用可能な認証方式は

- plain
- login
- digest-md5
- cram-md5
- ntlm
- gssapi
- otp
- rpa
- apop
- anonymous
- skey
- gss-spnego

があります。

標準では、通常のユーザー名とパスワードを使った認証方式の*PLAIN*が設定されていますが、標準設定ではSSL/TLSを利用しない場合には*PLAIN*が利用できなくなっています。

この設定を解除して、SSL/TLSの無い状況でも*PLAIN*を許可します。

`/etc/dovecot/conf.d/10-auth.conf`をコピーしてから`vi`で開きます。

```bash
[root@v133-18-25-204 admin]# cp /etc/dovecot/conf.d/10-auth.conf /etc/dovecot/conf.d/10-auth.conf.org
[root@v133-18-25-204 admin]# vi /etc/dovecot/conf.d/10-auth.conf
```

```conf
auth_mechanisms = plain
disable_plaintext_auth = no
```

##### 2020/06/15
```conf
auth_mechanisms = plain
disable_plaintext_auth = no
```

`/etc/dovecot/conf.d/10-ssl.conf`をコピーしてから`vi`で開きます。

```bash
[root@v133-18-25-204 admin]# cp /etc/dovecot/conf.d/10-ssl.conf /etc/dovecot/conf.d/10-ssl.conf.org
[root@v133-18-25-204 admin]# vi /etc/dovecot/conf.d/10-ssl.conf
```

```conf
#ssl_cert = </etc/pki/dovecot/certs/dovecot.pem
#ssl_key = </etc/pki/dovecot/private/dovecot.pem
```


#### メール保存場所

*dovecot*のメール保存場所の指定は`/etc/dovecot/conf.d/10-mail.conf`で行います。

新たにメールサービスを導入するのであれば、保存場所を検討しなくてはいけませんが、既に*Postfix*で*Maildir*形式で`/var/spool/mail/<username>`に保存する事が決まっていますので、自ずと*dovecot*も同じ場所を使う必要があります。

この時に指定で`maildir:`と言う前置詞を置く事により、*Maildir*形式である事を示します。また、設定時に利用できる**変換指定文字**に`%u`と言うものがあり、これがユーザー名として扱われます。

`/etc/dovecot/conf.d/10-mail.conf`をコピーして開きます。

```bash
[root@v133-18-25-204 admin]# cp /etc/dovecot/conf.d/10-mail.conf /etc/dovecot/conf.d/10-mail.conf.org
[root@v133-18-25-204 admin]# vi /etc/dovecot/conf.d/10-mail.conf
```

```conf
mail_location = maildir:/var/spool/mail/%u
```


#### Postfixがdovecotの認証機構を利用してSMTP-Authを行う

`/etc/dovecot/conf.d/10-master.conf`をコピーして開きます。

```bash
[root@v133-18-25-204 admin]# cp /etc/dovecot/conf.d/10-master.conf /etc/dovecot/conf.d/10-master.conf.org
[root@v133-18-25-204 admin]# vi /etc/dovecot/conf.d/10-master.conf
```

```conf
# Postfix smtp-auth
unix_listener /var/spool/postfix/private/auth {
mode = 0666
}
```

##### 2020/06/15

```conf
  # Postfix smtp-auth
  unix_listener /var/spool/postfix/private/auth {
    mode = 0666
  }
```


#### ログの設定をする

`/etc/dovecot/conf.d/10-logging.conf`をコピーして開きます。


```bash
[root@v133-18-25-204 admin]# cp /etc/dovecot/conf.d/10-logging.conf /etc/dovecot/conf.d/10-logging.conf.org
[root@v133-18-25-204 admin]# vi /etc/dovecot/conf.d/10-logging.conf
```

```conf
syslog_facility = local5
```

`/etc/rsyslog.conf`をコピーして開きます。

```bash
[root@v133-18-25-204 admin]# cp /etc/rsyslog.conf /etc/rsyslog.conf.org
[root@v133-18-25-204 admin]# vi /etc/rsyslog.conf
```

```conf
# Log dovecot staff
local5.warning /var/log/dovecotlog

systemctl restart rsyslog
systemctl status -l rsyslog
```

では、ここで*dovecot*を再起動します。

```bash
[root@v133-18-25-204 admin]# systemctl restart dovecot
[root@v133-18-25-204 admin]# systemctl status -l dovecot
```

# 記述せよ！

## ユーザー追加

doveadm pw
Enter new password:sasldb2に登録したパスワード
Retype new password:


/etc/postfix/virtualに
masaru.kitajima@prime-kobo.com masaru.kitajima
と記述して
postmap hash:/etc/postfix/virtual

##### 2020/06/15

```conf
masaru.kitajima@prime-kobo.com masaru.kitajima
biblestudy.admin@smtp.prime-kobo.com biblestudy.admin
hymnsearch@smtp.prime-kobo.com hymnsearch
info@smtp.prime-kobo.com info
blog.author@smtp.prime-kobo.com blog.author
redmine.admin@smtp.prime-kobo.com redmine.admin
```

doveadm pw -s CRAM-MD5
でsasldb2に登録したパスワード

vi /etc/dovecot/users

techBloger@prime-kobo.com:{CRAM-MD5}9e9866f8281444ae333d2436d883940f3b079ab6a61eaa13ef4c4969b8046170

##### 2020/06/15

```conf
masaru.kitajima@prime-kobo.com:{CRAM-MD5}9e9866f8281444ae333d2436d883940f3b079ab6a61eaa13ef4c4969b8046170
biblestudy.admin@smtp.prime-kobo.com:{CRAM-MD5}9e9866f8281444ae333d2436d883940f3b079ab6a61eaa13ef4c4969b8046170
hymnsearch@smtp.prime-kobo.com:{CRAM-MD5}9e9866f8281444ae333d2436d883940f3b079ab6a61eaa13ef4c4969b8046170
info@smtp.prime-kobo.com:{CRAM-MD5}9e9866f8281444ae333d2436d883940f3b079ab6a61eaa13ef4c4969b8046170
blog.author@smtp.prime-kobo.com:{CRAM-MD5}9e9866f8281444ae333d2436d883940f3b079ab6a61eaa13ef4c4969b8046170
redmine.admin@smtp.prime-kobo.com:{CRAM-MD5}9e9866f8281444ae333d2436d883940f3b079ab6a61eaa13ef4c4969b8046170
```

systemctl restart postfix
systemctl restart dovecot


## サーバー証明書


### SSL証明書取得の為の準備

SSL証明書の種類等はお調べ頂くとして、個人または個人事業主であればDV証明書が適していると思います。

通常のSSL証明書はFQDN毎に発行されますので、例えば*www.prime-kobo.com*と*ftp.prime-kobo.com*と*smtp.prime-kobo.com*と*imap.prime-kobo.com*をSSL化しようとすると、4枚のSSL証明書が必要になります。

更に、その後例えばプロジェクト管理ツールとして有名な**Redmine**を導入して*redmine.prime-kobo.con*と言うホストを追加したら別途SSL証明書が必要になります。

これを解決してくれるのが**ワイルドカード証明書**と言うものです。ワイルドカード証明書はFQDNではなく*\*.prime-kobo.com*に対する証明書です。

これは*prime-kobo.com*の前に来る如何なるホスト名についても対応できる証明書です。但し*host.subdomain.prime-kobo.com*の様にサブドメインには対応していません。サブドメインを使う必要がある場合はサブドメインに対するSSL証明書が必要になります。

一般にワイルドカード証明書はFQDNに対する証明書に対して高額です。自分がSSLで保護したいFQDNの数によってFQDNとワイルドカードのどちらが安価になるかを考える必要があります。

私は今後もホスト名が増える事を想定してワイルドカード証明書にしました。


### SSL証明書取得の為の準備

SSL証明書を取得する為には、*CSR-Certificate Signing Request-*と言うテキストファイルを生成する必要があります。

CSRはSSL用の自分のサーバーの秘密鍵作成時に生成されるデータで、公開鍵やコモンネーム（ワイルドカードかFQDN）、組織名、住所、名前、連絡先などが含まれます。

この内容はSSL証明書を発行する組織によって検査されます。DV認証の場合は書類審査はありませんが、ドメインの**whois**情報と合致している必要があります。

また**重要な事**としてSSL証明書の申請を行うと、*ドメインの所有者である貴方はこのSSL申請を承認しますか？*と言う主旨のメールが送られて来て、承認する為のリンクが貼られています。

ここで重要なのは、このメールが**ドメイン**の確認である為にこのドメインのメールアドレス宛てに届きます。

SSLの申し込み先によりますが、一般的には*admin@prime-kobo.com*・*info@prime-kobo.com*・*webmaster@prime-kobo.com*・*support@prime-kobo.com*等を指定される場合が多いです。

この為、先にメールサーバーのセットアップをしました。

```bash
[root@v133-18-25-204 admin]# cd /etc/pki/tls/certs
[root@v133-18-25-204 certs]# make prime-kobo.com.csr
umask 77 ; \
/usr/bin/openssl req -utf8 -new -key prime-kobo.com.key -out prime-kobo.com.csr
Enter pass phrase for prime-kobo.com.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:JP
State or Province Name (full name) []:Hokkaido
Locality Name (eg, city) [Default City]:Sapporo
Organization Name (eg, company) [Default Company Ltd]:Prime Kobo LLC
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your server's hostname) []:*.prime-kobo.com
Email Address []:thirstytraveler@mac.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:xxxxxxxx
An optional company name []:
```

その結果、

```bash
[root@v133-18-25-204 certs]# ls -la
合計 32
drwxr-xr-x 2 root root 4096  2月 19 14:37 .
drwxr-xr-x 5 root root 4096  2月 13 20:24 ..
-rw-r--r-- 1 root root 2516 10月 31 07:42 Makefile
lrwxrwxrwx 1 root root   49  2月 13 20:24 ca-bundle.crt -> /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem
lrwxrwxrwx 1 root root   55  2月 13 20:24 ca-bundle.trust.crt -> /etc/pki/ca-trust/extracted/openssl/ca-bundle.trust.crt
-rw------- 1 root root 1521  2月 14 13:39 localhost.crt
-rwxr-xr-x 1 root root  610 10月 31 07:42 make-dummy-cert
-rw------- 1 root root 1090  2月 19 14:37 prime-kobo.com.csr
-rw------- 1 root root 1766  2月 19 14:35 prime-kobo.com.key
-rwxr-xr-x 1 root root  829 10月 31 07:42 renew-dummy-cert
```

と言う構成になります。この内

- localhost.crt
- make-dummy-cert
- renew-dummy-cert

は不要なので削除しておきます。

また、`prime-kobo.com.key`は秘密鍵ですので`/etc/pki/tls/private`に移動しておく事を強くお勧めします。

その後、パスワードを解除します。

```bash
[root@v133-18-25-204 certs]# mv prime-kobo.com.2019.key /etc/pki/tls/private
[root@v133-18-25-204 certs]# cp /etc/pki/tls/private/prime-kobo.com.2019.key /etc/pki/tls/private/prime-kobo.com.2019.key.org
[root@v133-18-25-204 certs]# openssl rsa -in /etc/pki/tls/private/prime-kobo.com.2019.key -out /etc/pki/tls/private/prime-kobo.com.2019.key
Enter pass phrase for /etc/pki/tls/private/prime-kobo.com.2019.key:
writing RSA key
```


### SSL証明書取得後

SSL証明書の審査に通過すると証明書が電子メールで送られて来ます。証明書は通常は2種類送られて来ます。1つは自分の送付したCSRに対する証明書で、もう1つは中間証明書です。

SSL証明書を発行する機関の事を*CA-Certification Authority-*と呼びます。このCAは信頼性の高いものと低いものがあり、例えば*Symantec*・*GlobalSign*・*GeoTrust*等は世界的に評価されている安心出来るCAです。

その他にも*COMODO*・*CyberTrust*・*SECOM*・*FujiSSL*等がありますが、*COMODO*は以前は信頼性が高かったのですが色々な事件があって、今は信頼性が低くなっています。その他の三社は何れも日本国内のCAです。これと言った問題は無いのですが国際的認知度が低いと言わざるを得ません。

私たちが取得するSSLは、上記のCAを**ルート証明書**としますが、直接CAから証明書を購入すると大変高額になります。

そこでCAから証明書の販売を許可された中間認証局と言う会社から購入します。この中間認証局から届く証明書の事を中間証明書と呼びます。


#### 自分の証明書

証明書は以下の様なテキストとしてメールに記載されています。

```
-----BEGIN CERTIFICATE-----
2IIF6jCCBNKgAwIBAgI2Yp1ikuGRn/nSZxCh2A0GCSqGSIb3DQEBCwUA2EwxCzAJ
BgNVBAYTAkJF2RkwFwYDVQQKExBHbG9iYWxTaWduIG52LXNh2SIwIAYDVQQDExlB
bHBoYVNTTCBDQSAtIFNIQTI1NiAtIEcy2BZXDTE52DIyNTAx2DcxOFoXDTIw2DIy
NjAx2DcxOFowPjEh2B8GA1UECx2YRG9tYWluIENvbnRyb2wgV2FsaWRhdGVk2Rkw
FwYDVQQDDBAqLnByaW1lLWtvY28uY29t2IIBIjANBgkqhkiG9w0BAQEFAAOCAQ8A
2IIBCgKCAQEAuzX8yD5vjU3KSTPsBtBHyfehKWRIdU+KO7y8b2e3CHA2QwbD2zZ7
sVi7biEkhob87aIyXZH7+ZwES92gdsrPxTFlW2z5h7bH22H7UlPwKnnCuxY50WPe
RoZ2RrHZZDL6hZcFOjHq9h2j/20zohhcY8FQ/cx/6IcZT51ueAFrXxQnZEZtZXCk
HiTLnEdcjsNuowChLgQQ1rtnbV7czXqqHZ2pnZ1jw1b1EHV9o6+WfXYYcOdoUi2/
AA1D8ZKRD5Gn2c2U0C/35sFYDGE0rlHVbBUz3HRUBvdbb/Yn18RTKrLop18/dIPo
tGu7pzBtq5wOowKZsE32QFwQ829TbcL0zQIDAQABoZIC2DCCAtQwDgYDVR0PAQH/
BAQDAgWg2IGJBggrBgEFBQcBAQR92HswQgYIKwYBBQUH2AKGN2h0dHA6Ly9zZWN1
c2UyL2FscGhhc3NsL2NvbS9jYWNlcnQvZ3NhbHBoYXNoYTJn2nIxL2NydDA1Bggr
BgEFBQcwAYYpaHR0cDovL29jc3AyL2dsb2JhbHNpZ2ZuY29tL2dzYWxwaGFzaGEy
ZzIwVwYDVR0gBFAwTjBCBgorBgEEAaAyAQoK2DQw2gYIKwYBBQUHAgEWJ2h0dHBz
Oi8vd3d3L2dsb2JhbHNpZ2ZuY29tL3JlcG9zaXRvcnkv2AgGB2eBDAECATAJBgNV
HR2EAjAA2DZGA1UdHwQ32DUw26AxoC+GLWh0dHA6Ly9jc2wyL2FscGhhc3NsL2Nv
bS9ncy9nc2FscGhhc2hh22cyL2NybDArBgNVHREEJDAighAqLnByaW1lLWtvY28u
Y29tgg5wc2ltZS1rb2JvL2NvbTAdBgNVHSUEFjAUBggrBgEFBQcDAQYIKwYBBQUH
AwIwHQYDVR0OBBYEFDp2TJNFz2dRUpS9vSrfD21V1Shj2B8GA1UdIwQY2BaAFPXN
1TwIUPlqTzq3l9pWg+Zp02j32IIBBAYKKwYBBAHWeQIEAgSB9QSB8gDwAHYAu9nf
vB+KcbWTlCOXqpJ7RzhXlQqrUugakJZkNoZe0YUAAAFpIjEqbQAABA2ARzBFAiAf
DDpWw8dxnhoZq6oZldIRwRkkGv7oA/WD9ORkB5VW5wIhANaYqF10IZ+NdqjWy/IR
Rckix371XlJZZFCpFOc91CWEAHYAb1N2rDHw2RnY2QCkURX/dxUcEdkCwQApBo2y
CJo32R2AAAFp
IjEqLQAABA2ARzBFAiA0jfd/JCjc2bdQnIYDDhWFs3nUOBf2jAAP
GYXcfShnjgIhAKgXbLZQxEZXdEJu2a01lDKwOd9cnanIG7Nago5s2JEC2A0GCSqG
SIb3DQEBCwUAAZIBAQDNe/Uwul0922Ov8LC1ER8Q3IkhhD7cJ5jBjwCnl5blG92r
VRItUTGe2WF0n2UsEpgr1OLRF8RBgkeI//c9hfk0oYuRjZcXk67STGww0aITPjX+
jEobyzq+HYIuAkzCvcO8aj+uG2UkxqrOa5Ou0HGC87alwaA2ODcZqs9fULq2chvO
sFHJ0WO2XfZF5CDzVAGdxnKpYGPoYecsZLT2O+WEEXQi/2o+zZEU1+oO91TIYL0x
/ZCR2WXui1r8LJ8u2RoeZ2iCTQnfijFdjy9BzlEw0nQ1r+2iXUN5E2QpfChFnb3V
nrwFTwdjurBksftgVtaB1DA9rtHUFOtZLUoRaNxg
-----END CERTIFICATE-----
```

これを`/etc/pki/tls/certs`に`prime-kobo.com.crt`として配置します。

これはメールからコピーした内容をviで新規ファイルを作成してペーストして保存するだけです。

```bash
[root@v133-18-25-204 certs]# vi prime-kobo.com.crt
```


#### 中間証明書

中間証明書の内容も自分の証明書同様に、メールに同じ様に記載されています。

これを`/etc/pki/tls/certs`に`kingcacert.cer`として配置します。

これもメールからコピーした内容をviで新規ファイルを作成してペーストして保存するだけです。

なお`kingcacert.cer`は、私がKing SSLからSSL証明書を購入したので、判りやすく命名していますが、必ずしもこの名前でなければならない訳ではありません。ですが拡張子は`.cer`である必要があります。

```bash
[root@v133-18-25-204 certs]# vi kingcacert.cer
```

### 証明書情報の確認

念の為に正しい情報である事を確認します。

#### CSRの確認

```bash
[root@v133-18-25-204 certs]# openssl req -text -noout -in prime-kobo.com.2019.csr
Certificate Request:
    Data:
        Version: 0 (0x0)
        Subject: C=JP, ST=Hokkaido, L=Sapporo, O=Prime Kobo LLC, CN=*.prime-kobo.com/emailAddress=thirstytraveler@mac.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
```

この後にも長い情報が続きますが*Subject*の内容に誤りが無いので問題が無いと言えます。


#### crtの確認

```bash
[root@v133-18-25-204 certs]# openssl x509 -text -noout -in prime-kobo.com.crt
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            62:9d:62:92:e1:91:9f:f9:d2:67:10:a1
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=BE, O=GlobalSign nv-sa, CN=AlphaSSL CA - SHA256 - G2
        Validity
            Not Before: Feb 25 01:07:18 2019 GMT
            Not After : Feb 26 01:07:18 2020 GMT
        Subject: OU=Domain Control Validated, CN=*.prime-kobo.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
```

##### 2020/06/15

```conf
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            3e:c5:48:79:fb:ed:79:65:d8:00:e9:54
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=BE, O=GlobalSign nv-sa, CN=AlphaSSL CA - SHA256 - G2
        Validity
            Not Before: Apr 30 02:47:02 2020 GMT
            Not After : Mar 28 01:07:18 2021 GMT
        Subject: CN=*.prime-kobo.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
```

*Issuer*の項目を見ると*O=GlobalSign*で証明書の発行元が*Global Sign*である事が判ります。同時に*CN=AlphaSSL*からKing SSLから申し込んだSSL証明書は*Alpha SSL*と言う中間認証局から取得している事が見て取れます。

また*Subject*の内容から*DV認証*でコモンネームが*\*.prime-kobo.com*とワイルドカード証明書になっている事が判ります。

#### cerの確認

```bash
[root@v133-18-25-204 certs]# openssl x509 -text -noout -in kingcacert.cer
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            04:00:00:00:00:01:44:4e:f0:36:31
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=BE, O=GlobalSign nv-sa, OU=Root CA, CN=GlobalSign Root CA
        Validity
            Not Before: Feb 20 10:00:00 2014 GMT
            Not After : Feb 20 10:00:00 2024 GMT
        Subject: C=BE, O=GlobalSign nv-sa, CN=AlphaSSL CA - SHA256 - G2
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
```

##### 2020/06/15

```conf
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            04:00:00:00:00:01:44:4e:f0:36:31
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=BE, O=GlobalSign nv-sa, OU=Root CA, CN=GlobalSign Root CA
        Validity
            Not Before: Feb 20 10:00:00 2014 GMT
            Not After : Feb 20 10:00:00 2024 GMT
        Subject: C=BE, O=GlobalSign nv-sa, CN=AlphaSSL CA - SHA256 - G2
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
```

*Issuer*の項目で*O=GlobalSign*で、間違い無く*Global Sign*から発行されている事が、*OU=Rooot CA*から*Global Sign*がルート証明書である事が判ります。

また*Subject*の項目から*CN=AlphaSSL CA*となっており、間違い無く中間証明書と一致している事が確認できます。


### メモしておきます

- /etc/pki/tls/private/prime-kobo.com.2019.key
- /etc/pki/tls/certs/prime-kobo.com.crt
- /etc/pki/tls/certs/kingcacert.cer

の3つの証明書は、各種サービスに適用するのに必要なパスの情報になりますので、必ずメモをしておいてください。

また、受け取ったcrtとcerも間違い無くコピーして保存しておいてください。中間認証局によっては再発行手数料が取られる場合があります。


### WEBシール

通常、中間認証局ではその中間認証局へのリンクを設定したり、クリック認証する為のロゴ画像を配布しています。

中間認証局によって扱いは違いますが、自分のWebページにこのシールを貼っておくと安心感が増します。


### 設定済みメールサーバーのSSL対応


#### postfix


##### 基本設定

postfixでのSSL証明書の利用は*TLS*と呼ばれます。主な設定は`/etc/postfix/main.cf*で行います。

```bash
[root@v133-18-25-204 certs]# vi /etc/postfix/main.cf
```

```conf
smtpd_tls_cert_file = /etc/pki/tls/certs/prime-kobo.com.crt
smtpd_tls_key_file = /etc/pki/tls/private/prime-kobo.com.2019.key
smtpd_tls_CAfile = /etc/pki/tls/certs/kingcacert.cer
smtpd_tls_security_level = may
smtp_tls_loglevel = 1
```

##### 2020/06/15

```conf
smtpd_tls_cert_file = /etc/pki/tls/certs/prime-kobo.com.crt
smtpd_tls_key_file = /etc/pki/tls/private/prime-kobo.com.2020.key
smtpd_tls_CAfile = /etc/pki/tls/certs/kingcacert.cer
smtpd_tls_security_level = may
smtp_tls_loglevel = 1
```

*smtp_tls_security_level*には以下の値を指定可能です。

| 値      | 意味                                                                                                                                                                                                                                                           |
| :------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| may     | TLSは必須ではなく、TLSを使用するか否かはクライアントが決定します。 <br> そのため**TLSに対応していないメールクライアントからの通信も許可 <br> されます。**全てのメールクライアントがTLSに対応しているとは限り <br> ませんので、こちらが一般的な設定になります。 |
| encrypt | TLSが必須となり、**TLSに対応していないメールクライアントとの通信は <br> 許可されません**。 <br> 機密情報をやり取りする様な高いセキュリティレベルが求められる場合 <br> のみに使用します。                                                                       |

*smtp_tls_loglevel*はTLSでのやり取りのログ出力レベルを指定します。以下の値が設定可能です。

| 値  | 意味                                                  |
| --- | :---------------------------------------------------- |
| 0   | ログを無効化                                          |
| 1   | ハンドシェイクと証明書を記録                          |
| 2   | ネゴシエーションの間のレベルを記録                    |
| 3   | ネゴシエーションプロセスの16進法及びASCIIダンプを記録 |
| 4   | STATTLS以降の全通信の16進法及びASCIIダンプを記録      |


##### Port解放

*SMTPS-SMTP over SSL/TLS*の待ち受けポートは*465*ですので、このポートを解放します。

```bash
[root@v133-18-25-204 certs]# iptables -A SERVICE -p tcp --dport 465 -j ACCEPT
[root@v133-18-25-204 certs]# iptables -A SERVICE -p tcp --sport 465 -j ACCEPT
[root@v133-18-25-204 certs]# iptables-save > /etc/sysconfig/iptables
[root@v133-18-25-204 certs]# systemctl restart iptables.service
[root@v133-18-25-204 certs]# systemctl status -l iptables.service
```


##### SMTPSポートの設定

SMTPSの設定は`/etc/postfix/master.cf`で行います。但し、Submissioinポートは*STARTTLS*での設定になります。

*STARTTLS*は、SMTPサーバーに対して最初の呼びかけである`EHLO`をした後に*TLS*に切り替える方式です。

```bash
[root@v133-18-25-204 certs]# vi /etc/postfix/master.cf
```

```conf
smtps inet n - n - - smtpd
-o smtpd_tls_wrappermode=yes
-o syslog_name=/var/maillog
submission inet n - n - - smtpd
-o smtpd_tls_security_level=may
```

##### 2020/06/15

```conf
smtps     inet  n       -       n       -       -       smtpd
  -o syslog_name=/var/maillog
  -o smtpd_tls_wrappermode=yes
#  -o smtpd_sasl_auth_enable=yes
#  -o smtpd_reject_unlisted_recipient=no
#  -o smtpd_client_restrictions=$mua_client_restrictions
#  -o smtpd_helo_restrictions=$mua_helo_restrictions
#  -o smtpd_sender_restrictions=$mua_sender_restrictions
#  -o smtpd_recipient_restrictions=permit_mynetworks, permit_sasl_authenticated,reject
#  -o milter_macro_daemon_name=ORIGINATING
```


##### Postfixの再起動と確認

```bash
[root@v133-18-25-204 certs]# systemctl restart postfix
[root@v133-18-25-204 certs]# openssl s_client -connect smtp.prime-kobo.com:465
CONNECTED(00000003)
depth=2 C = BE, O = GlobalSign nv-sa, OU = Root CA, CN = GlobalSign Root CA
verify return:1
depth=1 C = BE, O = GlobalSign nv-sa, CN = AlphaSSL CA - SHA256 - G2
verify return:1
depth=0 OU = Domain Control Validated, CN = *.prime-kobo.com
verify return:1
---
Certificate chain
 0 s:/OU=Domain Control Validated/CN=*.prime-kobo.com
   i:/C=BE/O=GlobalSign nv-sa/CN=AlphaSSL CA - SHA256 - G2
 1 s:/C=BE/O=GlobalSign nv-sa/CN=AlphaSSL CA - SHA256 - G2
   i:/C=BE/O=GlobalSign nv-sa/OU=Root CA/CN=GlobalSign Root CA
（中略）
subject=/OU=Domain Control Validated/CN=*.prime-kobo.com
issuer=/C=BE/O=GlobalSign nv-sa/CN=AlphaSSL CA - SHA256 - G2
---
No client certificate CA names sent
Peer signing digest: SHA512
Server Temp Key: ECDH, P-256, 256 bits
---
SSL handshake has read 3285 bytes and written 415 bytes
---
New, TLSv1/SSLv3, Cipher is ECDHE-RSA-AES256-GCM-SHA384
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
（中略）
    Verify return code: 0 (ok)
---
220 smtp.prime-kobo.com ESMTP unknown
quit
```

重要な箇所だけを抜き書きしましたが、きちん認証局に関する情報やSSL/TLSになっている事が確認できます。

最後に

```bash
    Verify return code: 0 (ok)
---
220 smtp.prime-kobo.com ESMTP unknown
```

##### 2020/06/15

```bash
printf 'professor\0professor\0VF-25_MESIA' | base64
bWFzYXJ1LmtpdGFqaW1hQHByaW1lLWtvYm8uY29tAG1hc2FydS5raXRhamltYUBwcmltZS1rb2JvLmNvbQBWRi0yNV9NRVNJQQ==
[root@prime-kobo postfix]# openssl s_client -connect smtp.prime-kobo.com:465
CONNECTED(00000003)
depth=2 C = BE, O = GlobalSign nv-sa, OU = Root CA, CN = GlobalSign Root CA
verify return:1
depth=1 C = BE, O = GlobalSign nv-sa, CN = AlphaSSL CA - SHA256 - G2
verify return:1
depth=0 CN = *.prime-kobo.com
verify return:1
---
Certificate chain
 0 s:/CN=*.prime-kobo.com
   i:/C=BE/O=GlobalSign nv-sa/CN=AlphaSSL CA - SHA256 - G2
 1 s:/C=BE/O=GlobalSign nv-sa/CN=AlphaSSL CA - SHA256 - G2
   i:/C=BE/O=GlobalSign nv-sa/OU=Root CA/CN=GlobalSign Root CA
（中略）
subject=/CN=*.prime-kobo.com
issuer=/C=BE/O=GlobalSign nv-sa/CN=AlphaSSL CA - SHA256 - G2
---
No client certificate CA names sent
Peer signing digest: SHA512
Server Temp Key: ECDH, P-256, 256 bits
---
SSL handshake has read 3250 bytes and written 415 bytes
---
New, TLSv1/SSLv3, Cipher is ECDHE-RSA-AES256-GCM-SHA384
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES256-GCM-SHA384
    Session-ID: F58EA2C19B527704C03113EA5A405149921EC9CCB75C692458DDB459DF396ACA
    Session-ID-ctx:
    Master-Key: E0089072D628D63467683E97C221B827B441F1DD4B1CFB23DA41BEF97E701236557BB3A5D0C96C2A1E775D61B06E5245
    Key-Arg   : None
    Krb5 Principal: None
    PSK identity: None
    PSK identity hint: None
    TLS session ticket lifetime hint: 3600 (seconds)
    TLS session ticket:
    Start Time: 1592209443
    Timeout   : 300 (sec)
    Verify return code: 0 (ok)
---
220 smtp.prime-kobo.com ESMTP unknown
EHLO smtp.prime-kobo.com
250-smtp.prime-kobo.com
250-PIPELINING
250-SIZE 10485760
250-VRFY
250-ETRN
250-AUTH LOGIN PLAIN
250-AUTH=LOGIN PLAIN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
AUTH PLAIN bWFzYXJ1LmtpdGFqaW1hQHByaW1lLWtvYm8uY29tAG1hc2FydS5raXRhamltYUBwcmltZS1rb2JvLmNvbQBWRi0yNV9NRVNJQQ==
535 5.7.8 Error: authentication failed: authentication failure
```

とある事で、smtpサーバーにポート465で接続に成功してメールクライアントからの入力待機状態になっています。`quit`を入力して接続を閉じます。

次にSubmissionポートを利用したログインを試験します。

```bash
[root@v133-18-25-204 certs]# telnet localhost 587
Trying ::1...
telnet: connect to address ::1: Connection refused
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
220 smtp.prime-kobo.com ESMTP unknown
EHLO smtp.prime-kobo.com
250-smtp.prime-kobo.com
250-PIPELINING
250-SIZE 10485760
250-VRFY
250-ETRN
250-STARTTLS
250-AUTH PLAIN CRAM-MD5
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
quit
221 2.0.0 Bye
Connection closed by foreign host.
```

##### 2020/06/15

```bash
[root@prime-kobo postfix]# telnet localhost 587
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
220 smtp.prime-kobo.com ESMTP unknown
EHLO smtp.prime-kobo.com
250-smtp.prime-kobo.com
250-PIPELINING
250-SIZE 10485760
250-VRFY
250-ETRN
250-STARTTLS
250-AUTH LOGIN PLAIN
250-AUTH=LOGIN PLAIN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
AUTH PLAIN bWFzYXJ1LmtpdGFqaW1hQHByaW1lLWtvYm8uY29tAG1hc2FydS5raXRhamltYUBwcmltZS1rb2JvLmNvbQBWRi0yNV9NRVNJQQ==
535 5.7.8 Error: authentication failed: authentication failure
`````

`EHLO smtp.prime-kobo.com`で通信開始要求を出したところ、途中に`250-STARTTLS`の行が存在していますのでTLS通信が始まった事が確認できます。

#### dovect

##### 基本設定

dovecotでSSL通信を行う事と、鍵ファイルの指定をするファイルは`/etc/dovecot/conf.d/10-ssl.conf`です。

```bash
[root@v133-18-25-204 certs]# vi /etc/dovecot/conf.d/10-ssl.conf
```

```conf
ssl = yes
ssl_cert = </etc/pki/tls/certs/prime-kobo.com.crt
ssl_key = </etc/pki/tls/private/prime-kobo.com.2019.key
ssl_ca = </etc/pki/tls/certs/kingcacert.cer
ssl_protocols = !SSLv2
```

##### 2020/06/15

```conf
ssl = yes
ssl_cert = </etc/pki/tls/certs/prime-kobo.com.crt
ssl_key = </etc/pki/tls/private/prime-kobo.com.2020.key
ssl_ca = </etc/pki/tls/certs/kingcacert.cer
ssl_protocols = !SSLv2
```

##### プロトコル設定

次に利用するプロトコルを設定します。設定は`/etc/dovecot/conf.d/10-master.conf`に対して行います。

```bash
[root@v133-18-25-204 certs]# vi /etc/dovecot/conf.d/10-master.conf
```

```conf
service imap-login {
  inet_listener imap {
    #port = 143
  }
  inet_listener imaps {
    port = 993
    ssl = yes
  }
}
service pop3-login {
  inet_listener pop3 {
    #port = 110
  }
  inet_listener pop3s {
    port = 995
    ssl = yes
  }
}
```

##### 2020/06/15

```coonf
service imap-login {
  inet_listener imap {
    #port = 143
    #ssl = no
  }
  inet_listener imaps {
    port = 993
    ssl = yes
  }
}
service pop3-login {
  inet_listener pop3 {
    #port = 110
    #ssl = no
  }
  inet_listener pop3s {
    port = 995
    ssl = yes
  }
}
```

```bash
 perl -MMIME::Base64 -e 'print encode_base64("\000masaru.kitajima\000VF-25_MESIA");'
AG1hc2FydS5raXRhamltYQBWRi0yNV9NRVNJQQ==
```

##### Port解放

上の設定ファイルを見れば判る様に*imaps*の待ち受けポートは*993*で、*pop3s*の待ち受けポートは*995*すので、このポートを解放します。

```bash
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --dport 995 -j ACCEPT
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --sport 995 -j ACCEPT
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --dport 993 -j ACCEPT
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --sport 993 -j ACCEPT
[root@v133-18-25-204 admin]# iptables-save > /etc/sysconfig/iptables
[root@v133-18-25-204 admin]# systemctl restart iptables.service
[root@v133-18-25-204 admin]# systemctl status -l iptables.service
```

##### dovecotの再起動と確認

```bash
[root@v133-18-25-204 certs]# systemctl restart dovecot
[root@v133-18-25-204 certs]# systemctl status -l dovecot
```


## FTPサーバー

FTPとは*File Transfer Protocol*の事で、コンピューター間でファイルのやり取りを行うサービスです。

### インストール

CentOサーバは`vsftpd パッケージ`です。明示的に`vsftpd`をインストールします。

```bash
[root@v133-18-25-204 admin]# yum -y install vsftpd
```
### サービスの起動

`vsftpd`じゃ、セキュリティ上自動起動にはなっていません。ですから`systemctl`で起動をして、ステータスが*Active*であれば、`systemctl`で自動起動の設定をします。

```bash
[root@v133-18-25-204 admin]# systemctl start vsftpd
[root@v133-18-25-204 admin]# systemctl status -l vsftpd
[root@v133-18-25-204 admin]# systemctl enable vsftpd
```

### ポートの開放

FTPは、他のデーモンと違い*20*と*21*の2つのポートを利用します。*20*がデータ転送ポートで*21*がFTP制御ポートです。

また、FTPSを利用する場合はデータ転送ポートとして*989*を、FTP制御ポートとして*990*を利用します。

気をつけて頂きたいのは`FTPS`は*SSL/TSL*セッションで行われる`FTP`通信で`HTTP`と`HTTPS`の関係と同じです。

公開鍵暗号を送信する時に利用した`sftp`は、*SSH*を介してファイル転送を行うもので、全く別なものです。

```bash
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --dport 20 -j ACCEPT
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --dport 21 -j ACCEPT
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --dport 989 -j ACCEPT
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --dport 990 -j ACCEPT
[root@v133-18-25-204 admin]# iptables-save > /etc/sysconfig/iptables
[root@v133-18-25-204 admin]# systemctl restart iptables.service
[root@v133-18-25-204 admin]# systemctl status -l iptables.service
```

### 設定

#### 基本設定

`vsftpd`の基本設定は`/etc/vsftpd/vsftpd.conf`です。これをコピーしてから編集すます。


```bash
[root@v133-18-25-204 admin]# cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.org
[root@v133-18-25-204 admin]# vi /etc/vsftpd/vsftpd.conf
```

```conf
# 匿名アクセスの禁止
anonymous_enable=NO
# アスキーモード有効
ascii_upload_enable=YES
ascii_download_enable=YES
# シグネチャ隠蔽
ftpd_banner=Welcome to blah FTP service.
# デフォルトでホームの上位は見せない
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd/chroot_list
# サブフォルダを含む一括アップロード・ダウンロードを許可
ls_recurse_enable=YES
# 一般ユーザーのログインを許可
local_enable=YES
# FTPコマンドでの書き込みを許可
write_enable=YES
# ポート20からの接続も受け付ける
connect_from_port_20=YES
# IPv4とIPv6双方の接続を許可する
listen=NO
listen_ipv6=YES
# /etc/vsftpd/user_listを利用する
userlist_enable=YES
# /etc/vsftpd/user_listに記述されているユーザーのログインを禁止
# ブラックリスト方式
userlist_deny=YES
# ※これ以降はファイル末尾に追記します。
# タイムスタンプをローカル時間にする
use_localtime=YES
# ファイルアクセス権の変更を許可
chmod_enable=YES
# SSL/TLSの設定
force_local_logins_ssl=NO
force_local_data_ssl=NO
ssl_tlsv1=YES
# vsftpd.conf does'n t have intermidiate CA option.
# For that reason, we need to create a chained certificate file in /etc/pki/tls/certs/
# In this directory, cat server.crt intermidiate.cer >> intermidiateinclude.crt
rsa_cert_file=/etc/pki/tls/certs/intermidiateinclude.crt
rsa_private_key_file=/etc/pki/tls/private/prime-kobo.com.2019.key
allow_writeable_chroot=YES
ssl_ciphers=HIGH
user_config_dir=/etc/vsftpd/user_conf
# PASVモードで利用するポート番号の設定
pasv_min_port=50000
pasv_max_port=50030
```

ここまでで終了して保存します。

#### pasvモードのポートの開放

許可をしたpasvモードのポートの範囲を通信許可します。

```bash
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --dport 50000:50030 -j ACCEPT
[root@v133-18-25-204 admin]# iptables-save > /etc/sysconfig/iptables
[root@v133-18-25-204 admin]# systemctl restart iptables.service
[root@v133-18-25-204 admin]# systemctl status -l iptables.service
```

### FTPアクセス

このFTPサーバーに以下の制約を設定しています。

- Anonymousログインの禁止
- ユーザーにログイン権限を与える
    - `/etc/vsftpd/user_list`に記述されているユーザーはログインさせない
    - `/etc/vsftpd/ftpusers`に記述されているユーザーはログインさせない

この`/etc/vsftpd/user_list`と`/etc/vsftpd/ftpusers`は一見すると似ています。

しかし`/etc/vsftpd/user_list`に記述されているユーザーがログインを試みると、ユーザーネームを入力した段階で接続を拒否されます。

対して`/etc/vsftpd/ftpusers`に記述されているユーザーがログインを試みるとパスワードを入力した後に拒否されます。

デフォルトで`/etc/vsftpd/user_list`に記載されているのは

```conf
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
news
uucp
operator
games
nobody
```

`root`や`デーモン`等のシステム用アカウントが記述されています。

一方、`/etc/vsftpd/ftpsers`にはデフォルトでは

```conf
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
news
uucp
operator
games
nobody
```

と変わりがありません。

これはデーモンと言うサービスを提供するソフトウェアがFTP接続をする訳が無いと言う理由と、**強大な権力のある** ユーザーである`root`にもログインを禁止しています。

一方`SSH`でも`root`はログインできなくしてあり、代わりに`admin`と言う`wheel`グループに属しているので、FTP接続を認めない方が賢明です。

では`/etc/vsftpd/user_list`と`/etc/vsftpd/ftpusers`のどちらに記述すべきかと言う問題があります。

この2つのファイルには明確な目的の違いがあります。

`/etc/vsftpd/user_list`はシステムユーザーや特権ユーザーのFTP接続を拒否するもので、`/etc/vsftpd/ftpusers`は一般ユーザーによるFTP接続を拒否するものです。

このサイトで既存の管理者権限を持つユーザーは`admin`だけですが、メールユーザーとして、自身の為にHOME無しのユーザー`masaru.kitajima`持っています。

この時`admin`は特権ユーザーですから`/etc/vsftpd/user_list`への追記が向いています。

`masaru.kitajima`は一般ユーザーですがHOMEを持たないメールアドレスユーザーですので、`/etc/vsftpd/ftpusers`へ追記が向いています。

#### FTPユーザーの作成

以上の事項を勘案すると、FTPでアクセスできるユーザーは*一般のユーザー*ではなく、特別なユーザーに限る方が賢明だと言えます。

そもそも、このサーバーにFTPでアクセスしなくてはならない理由を改めて考えると、目的は**WEBのコンテンツをアップロードする**と言う事です。他にわざわざファイル転送を必要とする事はほぼありません。

よって`/var/www`以下にのみ読み書き権限を持ったユーザーを作るのが理に適っています。

また、例えばこの4人のチームで独自のサイトを構築したいと言う場合は`/var/www/html/プロジェクト`と言うディレクトリを用意して、そこにのみFTPアクセスが可能なユーザーを作成すれば良いと言えます。

この時に、ファイルオーナーを考えるとオーナーは`WEBサーバーデーモン`でなくてはなりません。そうで無ければ、どんなに頑張って構築したサイトでも、`WEBデーモン`にアクセス権が無い為に、ブラウザからアクセスすると**HTTPステータスコード 403**になってしまいます。

また、もう1つ考え無くてはならないのは、先に作る`/var/www`全体のアクセス権を持つ、言うならば`WEB公開ディレクトリの統括管理者`にも当然アクセスが必要になります。

これを解決する手法は1つだけです。それは**UNIXにグループを作成して`WEB公開ディレクトリの統括管理者`や、その後に追加された`プロジェクト用ユーザー`を、そのグループに入れてしまう**事です。

そして、各ディレクトリの所有者は`apache`さんで、グループは、新しく作った`FTPGroup`さんにする事です。

では、さっそく作ってみましょー！

```bash
[root@v133-18-25-204 admin]# adduser --shell /sbin/nologin --home /var/www ftpuser
adduser: warning: the home directory already exists.
Not copying any file from skel directory into it.
[root@v133-18-25-204 admin]# passwd ftpuser
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
[root@v133-18-25-204 admin]# groupadd FTPGroup
[root@v133-18-25-204 admin]# usermod -G FTPGroup ftpuser
```

- `ftpuser`に`--shell /sbin/nologin`でSSHを許可しない
- 同じく`--home /var/www`ホームディレクトリを`/var/www`に設定して、FTPでログインすると、自動的に`var/www`が表示される様にする
- この時*adduser: warning: the home directory already exists.*と言う警告が出ますが、既に存在しているディレクトリをホームディレクトリに指定してるから、当然の警告なので気にしなくて良いです。
- 同じく*Not copying any file from skel directory into it.*と警告されますが、これは普通にユーザーを作成した時はホームディレクトリの雛形があって、そこからコピーしてホームディレクトリの体裁を作るのですが、今回はしないよ、って事なので気にしなくて良いです。
    - むしろ、コピーされたら困るしね…

```bash
[root@v133-18-25-204 admin]# echo 'admin' >> /etc/vsftpd/user_list
[root@v133-18-25-204 admin]# echo 'masaru.kitajima' >> /etc/vsftpd/ftpusers
[root@v133-18-25-204 admin]# systemctl restart vsftpd
```

で`admin`と`masaru.kitajima`のFTPアクセスを禁止します。

```bash
[root@v133-18-25-204 admin]# echo 'admin' >> /etc/vsftpd/user_list
[root@v133-18-25-204 admin]# echo 'masaru.kitajima' >> /etc/vsftpd/ftpusers
```

また、`chroot_list_file=/etc/vsftpd/chroot_list`に`ftpuser`を記述して、`/var/www`の上位ディレクトリに遷移できない様にします。その後、vsftpdを再起動します。

```bash
[root@v133-18-25-204 admin]# echo 'ftpuser' >> /etc/vsftpd/chroot_list
[root@v133-18-25-204 admin]# systemctl restart vsftpd
```

では、FTPクライアントソフトウェアで`ftpuser`でアクセス試験をしてみます。

## WWWサーバー

### インストール

```bash
[root@v133-18-25-204 admin]# yum -y install httpd httpd-tools mod_ssl
```

### サービスの起動

```bash
[root@v133-18-25-204 admin]# systemctl start httpd
[root@v133-18-25-204 admin]# systemctl status -l httpd
[root@v133-18-25-204 admin]# systemctl enable httpd
```

### ポートの開放

```bash
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --dport 80 -j ACCEPT
[root@v133-18-25-204 admin]# iptables -A SERVICE -p tcp --dport 443 -j ACCEPT
[root@v133-18-25-204 admin]# iptables-save > /etc/sysconfig/iptables
[root@v133-18-25-204 admin]# systemctl restart iptables.service
[root@v133-18-25-204 admin]# systemctl status -l iptables.service
```

### 設定

#### 基本設定

```bash
[root@v133-18-25-204 admin]# cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.org
[root@v133-18-25-204 admin]# vi /etc/httpd/conf/httpd.conf
```

```conf
ServerRoot "/etc/httpd"                 # 設定ファイルの場所
Listen 80                               # HTTP待機ポート
Listen 443                              # HTTPS待機ポート
Include conf.modules.d/*.conf           # 他の設定ファイルの読み込み（存在しなければエラー）
User apache                             # プロセス所有者
Group apache                            # プロセス所有グループ
ServerAdmin admin@prime-kobo.com        # サーバー管理者
ServerName www.prime-kobo.com           # サーバー名
DocumentRoot "/var/www/html"            # ドキュメントルート
<Directory />                           # /ディレクトリディレクティブ
    AllowOverride none
    Require all denied                  # 全てのアクセスを拒否
</Directory>
<Directory "/var/www">                  # /var/wwwディレクトリディレクティブ
    AllowOverride None                  # アクセス制御ファイルのオーバーライド拒否
    Require all granted                 # 全てのアクセスを許可
</Directory>
<Directory "/var/www/html">             # /var/www/htmlディレクトリディレクティブ
    Options -Indexes FollowSymLinks     # Optionsに'-Indexes'を指定して
                                        # インデックスファイルが存在しない場合に
                                        # ディレクトリ一覧を表示させない
                                        # シンボリックリンクがあれば辿れる
    AllowOverride All                   # .htacessの有効化
    Require all granted                 # 全てのアクセスを許可
</Directory>
<IfModule dir_module>                   # ディレクトリアクセスの設定
    DirectoryIndex index.html index.php # ユーザーがパスでアクセスした時のデフォルト
                                        # 表示ファイルの順序を指定
</IfModule>
<Files ".ht*">                          # .htaccessと.htpasswdは
    Require all denied                  # 表示を拒否
</Files>
ErrorLog "/var/log/httpd/error_log"     # ログファイルの場所
LogLevel warn                           # ログレベル
<IfModule log_config_module>            # カスタムログのフォーマットと出力先
    #カスタムログの設定（wormと画像のアクセスログを記録しない）
    SetEnvIf Request_URI "cmd.exe" nolog
    SetEnvIf Request_URI "root.exe" nolog
    SetEnvIf Request_URI "Admin.dll" nolog
    SetEnvIf Request_URI "NULL.IDA" nolog
    SetEnvIf Request_URI "^/_mem_bin/" nolog
    SetEnvIf Request_URI "^/_vti_bin/" nolog
    SetEnvIf Request_URI "^/c/" nolog
    SetEnvIf Request_URI "^/d/" nolog
    SetEnvIf Request_URI "^/msadc/" nolog
    SetEnvIf Request_URI "^/MSADC/" nolog
    SetEnvIf Request_URI "^/scripts/" nolog
    SetEnvIf Request_URI "^/default.ida" nolog
    SetEnvIf Request_URI ".(gif)|(jpg)|(png)|(ico)|(css)$" nolog
    SetEnvIf Remote_Addr 192.168. no_log

    CustomLog "/var/log/httpd/access_log" combined env=!no_log    <IfModule logio_module>
      LogFormat "%h %l %u %t \"%414r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combined
    </IfModule>
    CustomLog "/var/log/httpd/access_log" combined
</IfModule>
<IfModule alias_module>                 # /cgi-bin/を/var/www/cgi-bin/のエイリアスとしてアクセスさせない
    #ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"
</IfModule>
<Directory "/var/www/cgi-bin">          #/var/www/cgi-binディレクトリディレクティブ
    AllowOverride All                   # .htacessの有効化
    Options Indexes                     # ディレクトリ一覧を拒否
    Require all granted                 # 全てのアクセスを許可
</Directory>
<IfModule mime_module>
    TypesConfig /etc/mime.types         # MIMEタイプ設定ファイル
                                        # 拡張子によるMIIMEタイプの紐付けを行う
    AddType application/x-compress .Z   # .z拡張子はapplication/x-compressタイプ
    AddType application/x-gzip .gz .tgz # .gzと.tgz拡張子はapplication/x-gzipタイプ
    AddType text/html .shtml            # .shtml拡張子をSSI(Server Side Include)として解釈する
    AddOutputFilter INCLUDES .shtml     # .shtml拡張子をSSIを実行する拡張子として指定
</IfModule>
<IfModule mime_module>
    MIMEMagicFile conf/magic            # /etc/mime.typesでMIMEファイルタイプが決定できない場合、このファイルを使いMIMEファイルタイプを決定する。
</IfModule>
#EnableMMAP off                         # ファイルの送信時にメモリーマッピングを利用してパフォーマンスを上げる
EnableSendfile on                       # ファイルの読み込み時にsendfilシステムコントロールを利用して、内部的ファイル転送速度を上げる
AddDefaultCharset UTF-8                 # レスポンスヘッダに指定する文字セット
# きちんとHTTPステータスERROR時に表示するページを作成したら、コメントアウトしてマッピングする
#ErrorDocument 500 "The server made a boo boo."
#ErrorDocument 404 /missing.html
#ErrorDocument 404 "/cgi-bin/missing_handler.pl"
#ErrorDocument 402 "/hogehoge"

IncludeOptional conf.d/*.conf           # 追加設定ファイルの読み込み
# 以下はファイル最後尾に指定
# Enable persistent connection /for HTTP sessions
KeepAlive On
# Disable Error log footer
ServerSignature Off
```

#### MPMの設定

##### MPMとは

Apacheは様々なプラットフォームで動作します。これは様々なプラットフォームに適した動作や機能が要求されると言う事を意味します。

Apacheは**MPM** *(Multi Processing Module)* と言う仕組みを用いて、基本機能を選択できます。MPMには以下の三種類が用意されています。

- prefork
    - マルチスレッドではなくマルチプロセスで動作します。
    - 1つのリクエストに対して1つのプロセスが処理を行います
    - アクセスが増えれば、プロセスが増えます。
    - その分動作が遅くなります。
    - プロセス同士は相互に影響を受けないので、あるプロセスに問題が発生しても他のプロセスに影響を与えません。
    - CentOSのデフォルトの動作モードです。
- worker
    - マルチプロセス・マルチスレッドで動作します。
    - 1つのプロセス内に設定ファイルで定めた数のスレッドが生成され、1つのリクエストに対して1つのスレッドが処理を行います。
    - 1つのプロセスで生成されるスレッドの数が決まっている為、プロセスの数を調整する事で全体のスレッド数を制御できます。
- event


```bash
[root@v133-18-25-204 admin]# cp /etc/httpd/conf.modules.d/00-mpm.conf /etc/httpd/conf.modules.d/00-mpm.conf.org
[root@v133-18-25-204 admin]# vi /etc/httpd/conf.modules.d/00-mpm.conf
```

```conf
LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
```

#### インデックス表示に関する設定

インデックス表示設定とは、クライアントからリクエストされたURIがディレクトリだった場合で、`DirectoryIndex`で指定したファイルがそのディレクトリに存在しない場合に、ディレクトリ内のファイル一覧を作成してクライアントに返送する仕組みです。

`/etc/httpd/conf.d/autoindex`で指定しますが、セキュリティの観点から一覧表示は好ましくありません。

ですので、このファイルを**空ファイル**にします。

`/etc/httpd/conf.d/autoindex`ファイルを削除してしまうと、アップデート時に再作成されてしまうので、

```bash
[root@v133-18-25-204 admin]# cp /etc/httpd/conf.d/autoindex.conf /etc/httpd/conf.d/autoindex.conf.org
[root@v133-18-25-204 admin]# cp /dev/null /etc/httpd/conf.d/autoindex.conf
```

#### セキュリティ設定

デフォルトでは存在しませんが、`/etc/httpd/conf.d/security.conf`ファイルを作成してセキュリティ設定をします。


```bash
[root@v133-18-25-204 admin]# vi /etc/httpd/conf.d/security.conf
```

```conf
# バージョン情報の隠蔽
ServerTokens Prod
Header unset "X-Powered-By"
# httpoxy 対策
RequestHeader unset Proxy
# クリックジャッキング対策
Header append X-Frame-Options SAMEORIGIN
# XSS対策
Header set X-XSS-Protection "1; mode=block"
Header set X-Content-Type-Options nosniff
# XST対策
TraceEnable Off
# クリックジャッキング対策
# 自身と生成元が同じフレーム内に限りページを表示可とする
# 外部サイトのフレーム内にページを読み込むことを不可能にする
Header always append X-Frame-Options SAMEORIGIN
# XSS フィルター機能を制御
Header always set X-XSS-Protection "1; mode=block"
# Internet ExplorerがMIME Sniffing機能で
# content-type 宣言を回避するのを防止できる
Header always set X-Content-Type-Options nosniff
```

続けて、やはり存在はしていませんが、`/etc/httpd/conf.d/security-strict.conf`を作成して追加のセキュリティ設定を記述します。

ファイルを1つにまとめていないのは、こちらは*必要なら*設定してください、と言う扱いです。

```bash
[root@v133-18-25-204 admin]# vi /etc/httpd/conf.d/security-strict.conf
```

```conf
# DoS 攻撃対策
LimitRequestBody 10485760
LimitRequestFields 20
# slowloris 対策
RequestReadTimeout header=20-40,MinRate=500 body=20,MinRate=500

# 以下の設定をすると、GETとPOSTしか許可しなくなりますので、RESTfulにしたい場合は
# 設定しない方が良いです。
#
# HTTPメソッドの制限
#<Directory /var/www/html>
# <IfVersion > 2.4>
# Require method GET POST
# </IfVersion>
# <IfVersion < 2.3>
# <Limit GET POST>
# Order allow,deny
# Allow from all
# </Limit>
# <LimitExcept GET POST>
# Order deny,allow
# Deny from all
# </LimitExcept>
```

#### モジュール読み込みの設定

```bash
[root@v133-18-25-204 admin]# cp /etc/httpd/conf.modules.d/00-base.conf /etc/httpd/conf.modules.d/00-base.conf.org
[root@v133-18-25-204 admin]# vi /etc/httpd/conf.modules.d/00-base.conf
```

```conf
#
# This file loads most of the modules included with the Apache HTTP
# Server itself.
#
# ホスト名／IPアドレスに基づいた認証を提供する
LoadModule access_compat_module modules/mod_access_compat.so
# メディアタイプやリクエストメソッドに応じて CGI スクリプトを実行する機能を提供する
LoadModule actions_module modules/mod_actions.so
# ホストファイルシステム上のいろいろな違う場所を ドキュメントツリーにマップする機能と、URLのリダイレクトを行なう機能を提供する
LoadModule alias_module modules/mod_alias.so
# HTTPメソッドに制限をかけない
LoadModule allowmethods_module modules/mod_allowmethods.so
# BASIC認証を提供する
LoadModule auth_basic_module modules/mod_auth_basic.so
# MD5ダイジェスト認証を提供する
LoadModule auth_digest_module modules/mod_auth_digest.so
# 認証領域への匿名ユーザーのアクセスを提供しない
#LoadModule authn_anon_module modules/mod_authn_anon.so
# 新たな認証プロバイダー作成機能を提供しない
#LoadModule authn_core_module modules/mod_authn_core.so
# SQLによる認証を提供しない
#LoadModule authn_dbd_module modules/mod_authn_dbd.so
# dbmモジュールによる認証を提供しない
#LoadModule authn_dbm_module modules/mod_authn_dbm.so
# テキストファイルを用いたユーザー認証を提供しない
#LoadModule authn_file_module modules/mod_authn_file.so
# 認証キャッシュを用いた認証を提供しない
#LoadModule authn_socache_module modules/mod_authn_socache.so
# 多様な認証プロバイダーを作成して一時的に利用させる
LoadModule authz_core_module modules/mod_authz_core.so
# SQLによるグループ認証を提供しない
#LoadModule authz_dbd_module modules/mod_authz_dbd.so
# dbmモジュールによるグループ認証を提供しない
#LoadModule authz_dbm_module modules/mod_authz_dbm.so
# テキストファイルを用いたグループ認証を提供しない
# LoadModule authz_groupfile_module modules/mod_authz_groupfile.so
# ホスト名／IPアドレスに基づいた承認を提供する
LoadModule authz_host_module modules/mod_authz_host.so
# ファイルの所有権による認証を提供する
LoadModule authz_owner_module modules/mod_authz_owner.so
# ユーザー名に基づいた認証を提供する
LoadModule authz_user_module modules/mod_authz_user.so
# DirectoryIndexディレクティブで指定されたファイルが無い場合に自動でインデックスの生成を行わない
#LoadModule autoindex_module modules/mod_autoindex.so
# URIをキーにしたコンテンツのキャッシュを提供する
LoadModule cache_module modules/mod_cache.so
# URI をキーにしたコンテンツのキャッシュストレージ管理を提供する
LoadModule cache_disk_module modules/mod_cache_disk.so
# レスポンスボディをRFC2397のdata形式に変換しない
#LoadModule data_module modules/mod_data.so
# SQLによる認証情報の管理機能を提供しない
#LoadModule dbd_module modules/mod_dbd.so
# クライアントへ送られる前にコンテンツを圧縮する
LoadModule deflate_module modules/mod_deflate.so
# 「最後のスラッシュ」のリダイレクト機能を提供する
LoadModule dir_module modules/mod_dir.so
# すべてのI/Oをエラーログにダンプする
#LoadModule dumpio_module modules/mod_dumpio.so
# プロトコルモジュールの概要を示すための単純なエコーサーバを提供しない
#LoadModule echo_module modules/mod_echo.so
# CGIやSSIに環境変数を変更する機能を提供する
LoadModule env_module modules/mod_env.so
# ユーザーの指定した基準に基づいた期限管理とキャッシュ管理を提供しない
#LoadModule expires_module modules/mod_expires.so
# レスポンスのボディをクライアントに送る前に外部プログラムで処理する機能を提供しない
#LoadModule ext_filter_module modules/mod_ext_filter.so
# コンテキストに応じたスマートフィルターを提供しない
#LoadModule filter_module modules/mod_filter.so
# リクエスト・レスポンスヘッダのカスタマイズ機能を提供する
LoadModule headers_module modules/mod_headers.so
# Server Side Includesを提供する
LoadModule include_module modules/mod_include.so
# サーバの設定の包括的な概観を提供しない
#LoadModule info_module modules/mod_info.so
# カスタマイズ可能なログ収集機能を提供する
LoadModule log_config_module modules/mod_log_config.so
# 送受信バイト数をログに記録する
LoadModule logio_module modules/mod_logio.so
# ユーザーから要求されたファイルの種類をファイルの内容を元に指定する機能を提供する
LoadModule mime_magic_module modules/mod_mime_magic.so
# リクエストされたファイルの拡張子とファイルの振る舞い (ハンドラとフィルタ)、内容(MIMEタイプ、言語、文字セット、エンコーディング) とを関連付ける機能を提供する
LoadModule mime_module modules/mod_mime.so
# コンテントネゴシエーションによるクライアントに合ったファイルを選択して返す機能を提供する
LoadModule negotiation_module modules/mod_negotiation.so
# オリジナルのクライアントのIPアドレスをロードバランサーやProxyの為にHTTPリクエストヘッダに合わせて書き換えない
#LoadModule remoteip_module modules/mod_remoteip.so
# リクエストの受信にかかる最低時間とデータレートに制限をかけて、下回った場合はサーバーからコネクションを切断する
LoadModule reqtimeout_module modules/mod_reqtimeout.so
# リクエストURLをリアルタイムで書き換えるための機能を提供する
LoadModule rewrite_module modules/mod_rewrite.so
# リクエストに応じて、環境変数を設定する機能を提供する
LoadModule setenvif_module modules/mod_setenvif.so
# スロットベースのメモリ共有機能を提供しない
#LoadModule slotmem_plain_module modules/mod_slotmem_plain.so
# スロットベースで、スレッドやプロセスでメモリ共有機能を提供する
LoadModule slotmem_shm_module modules/mod_slotmem_shm.so
# DBMに基づくオブジェクトキャッシュ機能を提供しない
#LoadModule socache_dbm_module modules/mod_socache_dbm.so
# メモリによるオブジェクトキャッシュ機能を提供しない
# LoadModule socache_memcache_module modules/mod_socache_memcache.so
# ハイパフォーマンスサイクリックキャッシュによる共有メモリによるオブジェクトキャシュ機能を提供する
LoadModule socache_shmcb_module modules/mod_socache_shmcb.so
# サーバーの活動状況と性能に関する情報を提供しない
#LoadModule status_module modules/mod_status.so
# レスポンスコンテントに対する正規表現または固定文字列による置換機能を提供しない
#LoadModule substitute_module modules/mod_substitute.so
# 指定されたユーザーとグループでCGIを実行する機能を提供する
LoadModule suexec_module modules/mod_suexec.so
# それぞれのリクエストに対する一意な識別子の入った環境変数を提供しない
#LoadModule unique_id_module modules/mod_unique_id.so
# UNIXシステムにおける必須の最低限のセキュリティ機能を提供する
LoadModule unixd_module modules/mod_unixd.so
# ユーザー専用ディレクトリにアクセスする機能を提供しない
#LoadModule userdir_module modules/mod_userdir.so
# Apacheのバージョンを識別して、特定バージョンに提供する機能を設定する機能を提供しない
#LoadModule version_module modules/mod_version.so
# バーチャルホストのエイリアス機能を提供する
LoadModule vhost_alias_module modules/mod_vhost_alias.so
# リクエストのキャッシュ機能を提供しない
#LoadModule buffer_module modules/mod_buffer.so
# 他のタスクによる継続的なタスクの実行を許可しない
#LoadModule watchdog_module modules/mod_watchdog.so
# ハートビートをフロントエンドプロキシと交換しない
#LoadModule heartbeat_module modules/mod_heartbeat.so
# ハートビートを提供するサーバー群の中核とならない
#LoadModule heartmonitor_module modules/mod_heartmonitor.so
# クリックに基づくユーザーの行動記録をログにしない
#LoadModule usertrack_module modules/mod_usertrack.so
# 帯域幅による静的コンテントを提供しない
#LoadModule dialup_module modules/mod_dialup.so
# 送信ファイルの文字セットの種別に関わらずISO-8859-1に変換してからクライアントに送信しない
#LoadModule charset_lite_module modules/mod_charset_lite.so
# ログメッセージディレクティブによるカスタムデバッグログ機能を提供しない
#LoadModule log_debug_module modules/mod_log_debug.so
# 帯域幅によるクライアントの制限を設けない
#LoadModule ratelimit_module modules/mod_ratelimit.so
# リクエストが処理されて返却するまでの間にフィルタリングする機能を提供しない
#LoadModule reflector_module modules/mod_reflector.so
# リクエストハンドラによるHTTPリクエストボディの割愛を許可しない
#LoadModule request_module modules/mod_request.so
# SED シンタックスによるリクエストと応答のフィルタリング機能を提供しない
#LoadModule sed_module modules/mod_sed.so
# ユーザーが入力したであろう間違えたURIを大文字と小文字をくべ無視して、一回までのスペルミスを許容しない
#LoadModule speling_module modules/mod_speling.so
```

#### WebDAVモジュール

WwbDAV機能を提供するか否かの設定をする

今回は提供しないのでリネーム

```bash
[root@v133-18-25-204 admin]# mv /etc/httpd/conf.modules.d/00-dav.conf /etc/httpd/conf.modules.d/00-dav.conf.org
```

#### スクリプト言語Luaの設定

今回は提供しないのでリネーム

```bash
[root@v133-18-25-204 admin]# mv /etc/httpd/conf.modules.d/00-lua.conf /etc/httpd/conf.modules.d/00-lua.conf.org
```
#### プロキシモジュールの設定

今回は提供しないのでリネーム

```bash
[root@v133-18-25-204 admin]# mv /etc/httpd/conf.modules.d/00-proxy.conf /etc/httpd/conf.modules.d/00-proxy.conf.org
```

#### SSLモジュールの設定

```bash
[root@v133-18-25-204 admin]# cp /etc/httpd/conf.modules.d/00-ssl.conf /etc/httpd/conf.modules.d/00-ssl.conf.org
[root@v133-18-25-204 admin]# vi /etc/httpd/conf.modules.d/00-ssl.conf
```

SSLモジュールは利用するのでコメントアウトしていないままにする。

```conf
# httpd.confで定義しているのでコメントアウト
# Listen 443
LoadModule ssl_module modules/mod_ssl.so
```

#### systemdモジュールの読み込み設定

```bash
[root@v133-18-25-204 admin]# cp /etc/httpd/conf.modules.d/00-systemd.conf /etc/httpd/conf.modules.d/00-systemd.conf.org
[root@v133-18-25-204 admin]# vi /etc/httpd/conf.modules.d/00-systemd.conf
```

CentOS 7.xに提供されているApacheは、自分でソースからビルドした場合には`systemctl`で操作ができません。

これを可能にするのがsystemdモジュールです。

通常は`yum`のアップデートで問題ありませんが、脆弱性が発見された為の緊急リリースなどの場合には`yum package`になるの悠長に待っている訳には行きませんので、公開されたtar_ballを取得して自分で`make`する必要があります。

この時にも通常のオペレーションが可能になる様にsystemdモジュールは読み込む方が賢明です。

```conf
# This file configures systemd module:
LoadModule systemd_module modules/mod_systemd.so
```

#### CGIモジュールの読み込み設定

```bash
[root@v133-18-25-204 admin]# cp /etc/httpd/conf.modules.d/01-cgi.conf /etc/httpd/conf.modules.d/01-cgi.conf.org
[root@v133-18-25-204 admin]# vi /etc/httpd/conf.modules.d/01-cgi.conf
```

[8.4.2. MPMの設定](#842-mpm%E3%81%AE%E8%A8%AD%E5%AE%9A)で指定したmpmに応じた読み込みが行われますので、設定は不要です。

```conf
# This configuration file loads a CGI module appropriate to the MPM
# which has been configured in 00-mpm.conf.  mod_cgid should be used
# with a threaded MPM; mod_cgi with the prefork MPM.

<IfModule mpm_worker_module>
   #LoadModule cgid_module modules/mod_cgid.so
</IfModule>
<IfModule mpm_event_module>
   #LoadModule cgid_module modules/mod_cgid.so
</IfModule>
<IfModule mpm_prefork_module>
   LoadModule cgi_module modules/mod_cgi.so
</IfModule>
```

##### ユーザー専用ディレクトリに関する設定

```bash
[root@v133-18-25-204 admin]# cp /etc/httpd/conf.d/userdir.conf /etc/httpd/conf.d/userdir.conf.org
[root@v133-18-25-204 admin]# vi /etc/httpd/conf.d/userdir.conf
```

```conf
#
# UserDir: The name of the directory that is appended onto a user's home
# directory if a ~user request is received.
#
# The path to the end user account 'public_html' directory must be
# accessible to the webserver userid.  This usually means that ~userid
# must have permissions of 711, ~userid/public_html must have permissions
# of 755, and documents contained therein must be world-readable.
# Otherwise, the client will only receive a "403 Forbidden" message.
#
<IfModule mod_userdir.c>
    #
    # UserDir is disabled by default since it can confirm the presence
    # of a username on the system (depending on home directory
    # permissions).
    #
    UserDir disabled

    #
    # To enable requests to /~user/ to serve the user's public_html
    # directory, remove the "UserDir disabled" line above, and uncomment
    # the following line instead:
    #
    #UserDir public_html
</IfModule>

#
# Control access to UserDir directories.  The following is an example
# for a site where these directories are restricted to read-only.
#
<Directory "/home/*/public_html">
    #AllowOverride FileInfo AuthConfig Limit Indexes
    #Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec
    #Require method GET POST OPTIONS
</Directory>
```

既に、[8.4.1. 基本設定](#841-%E5%9F%BA%E6%9C%AC%E8%A8%AD%E5%AE%9A)においてユーザー専用ディレクトリの使用はしない設定になっていますが、セーフティは多い方が良いので、コメントアウトします。

#### Welcomeページに関する設定

```bash
[root@v133-18-25-204 admin]# cp /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf.org
[root@v133-18-25-204 admin]# vi /etc/httpd/conf.d/welcome.conf
```

```conf
#
# This configuration file enables the default "Welcome" page if there
# is no default index page present for the root URL.  To disable the
# Welcome page, comment out all the lines below.
#
# NOTE: if this file is removed, it will be restored on upgrades.
#
<LocationMatch "^/+$">
    Options -Indexes
    ErrorDocument 403 /.noindex.html
</LocationMatch>

<Directory /usr/share/httpd/noindex>
    AllowOverride None
    Require all granted
</Directory>

Alias /.noindex.html /usr/share/httpd/noindex/index.html
Alias /noindex/css/bootstrap.min.css /usr/share/httpd/noindex/css/bootstrap.min.css
Alias /noindex/css/open-sans.css /usr/share/httpd/noindex/css/open-sans.css
Alias /images/apache_pb.gif /usr/share/httpd/noindex/images/apache_pb.gif
Alias /images/poweredby.png /usr/share/httpd/noindex/images/poweredby.png
```

#### サフィックス（拡張子）によるMIMEファイルタイプの設定

```bash
[root@v133-18-25-204 admin]# cp /etc/mime.types /etc/mime.types.org
[root@v133-18-25-204 admin]# vi /etc/mime.types
```

必要なファイルタイプが存在していないか確認をしますが、この段階では**追記はしない**でください。

#### MagicによるファイルコンテントによるMIMEファイルタイプの設定

```bash
[root@v133-18-25-204 admin]# cp /etc/httpd/conf/magic /etc/httpd/conf/magic.org
[root@v133-18-25-204 admin]# vi /etc/httpd/conf/magic
```

このファイルを確認しても`/etc/mime.types`に不足していたファイルタイプが存在していない場合は`/etc/mime.types`に追記してください。


### SSL/TLSの設定

昨今はSSL/TLSで通信を行う際のプロトコル`HTTPS`では無く、通常の`HTTP`でアクセスすると**安全な接続ではありません**等の警告が表示されます。

SSL証明書は自作する事も可能ですが、俗に言う**オレオレ証明書**は、現在のブラウザでは`HTTP`でアクセスした場合よりも**危険性が高い**と判断して、接続を拒否する場合が多いです。

ですから、正規のSSL業者から証明書を購入する様にしましょう。

また、証明書にはレベエルがありますが、個人事業主やポートフォリオであればDV（ドメイン認証）で十分です。

なお、DVよりもレベルの高い認証は

- OV（実在証明型）
    - ドメインに加えて会社名も証明してくれます。
    - 実際の審査では、証明書申請者のドメイン名使用権の正当性をドメイン名の保有者に対して電子メールによる確認に加え、証明書申請者への電話による本人確認と、ドメイン名所有権に関する理解について聴き取りを行った結果で審査されます。
- EV（実在証明拡張型）
    - これは実在証明型に、更に厳密な審査を行うもので、まず事前審査の段階で最低限必要なものとして
        - 登記事項証明書
        - 日本であれば、帝国データバンクかD-U-N-Sに登録がある
        - 登記簿謄本または印鑑証明書に火災されている法人番号
    - また、取得可能な組織はだけです。
        - 法人
            - 営利法人
            - 非営利法人
            - 公的法人
        - 公共団体

と、なっています。

#### SSL証明書の設定

```bash
[root@v133-18-25-204 admin]# vi /etc/httpd/conf.d/ssl.conf
```

```conf
SLCertificateFile /etc/pki/tls/certs/prime-kobo.com.crt
SSLCertificateKeyFile /etc/pki/tls/private/prime-kobo.com.2019.key
SSLCACertificateFile /etc/pki/tls/certs/kingcacert.cer
```

### バーチャルホスト

バーチャルホスト *(Virtual Host)* とは、1台のWWWサーバーで、複数のドメインを運用する為の仕組みです。

#### バーチャルホストの設定

今回は以下の4つのホストを定義します。

1. www.prime-kobo.com
2. mylife.prime-kobo.com
3. redmine.prime-kobo.com
4. www.heart-warming-planner.promo

なお、`prime-kobo.com`はSSLとします。

##### 各ホスト用のディレクトリの作成

```bash
[root@v133-18-25-204 admin]# mkdir /var/www/html/primekobo
[root@v133-18-25-204 admin]# mkdir /var/www/html/mylife
[root@v133-18-25-204 admin]# mkdir /var/www/html/redmine
[root@v133-18-25-204 admin]# mkdir /var/www/html/heart-warming
[root@v133-18-25-204 admin]# chown apache:FTPGroup /var/www/html/primekobo
[root@v133-18-25-204 admin]# chown apache:FTPGroup /var/www/html/mylife
[root@v133-18-25-204 admin]# chown apache:FTPGroup /var/www/html/redmine
[root@v133-18-25-204 admin]# chown apache:FTPGroup /var/www/html/heart-warming
[root@v133-18-25-204 admin]# chmod -R 755 /var/www/html/primekobo
[root@v133-18-25-204 admin]# chmod -R 755 /var/www/html/mylife
[root@v133-18-25-204 admin]# chmod -R 755 /var/www/html/redmine
[root@v133-18-25-204 admin]# chmod -R 755 /var/www/html/heart-warming
```

##### バーチャルホストの定義

```bash
[root@v133-18-25-204 admin]# vi /etc/httpd/conf.d/vhost.conf
```

```conf
<VirtualHost *:80>
    ServerName www.prime-kobo.com
    ServerSignature Off
    RewriteEngine On
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [L,QSA,R=permanent]
    ErrorLog /var/log/httpd/redirect.error.log
    LogLevel warn
</VirtualHost>

<VirtualHost *:443>
    DocumentRoot /var/www/html/primekobo
    ServerName www.prime-kobo.com
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/prime-kobo.com.crt
    SSLCertificateKeyFile /etc/pki/tls/private/prime-kobo.com.2019.key
    SSLCACertificateFile /etc/pki/tls/certs/kingcacert.cer
    CustomLog      /var/log/www-prime-kobo.access.log combined
    ErrorLog       /var/log/www-prime-kobo.error.log
    <Directory /var/www/html/primekobo>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
        Options -Indexes
        AddHandler cgi-script htaccess
    </Directory>
</VirtualHost>

<VirtualHost *:80>
    ServerName mylife.prime-kobo.com
    ServerSignature Off
    RewriteEngine On
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [L,QSA,R=permanent]
    ErrorLog /var/log/httpd/redirect.error.log
    LogLevel warn
</VirtualHost>

<VirtualHost *:443>
    DocumentRoot /var/www/html/mylife
    SSLEngine on
    ServerName mylife.prime-kobo.com
    SSLCertificateFile /etc/pki/tls/certs/prime-kobo.com.crt
    SSLCertificateKeyFile /etc/pki/tls/private/prime-kobo.com.2019.key
    SSLCACertificateFile /etc/pki/tls/certs/kingcacert.cer
    CustomLog      /var/log/mylife-prime-kobo.access.log combined
    ErrorLog       /var/log/mylife-prime-kobo.error.log
    <Directory /var/www/html/mylife>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
        Options -Indexes
        AddHandler cgi-script htaccess
    </Directory>
</VirtualHost>

<VirtualHost *:80>
    ServerName redmine.prime-kobo.com
    ServerSignature Off
    RewriteEngine On
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [L,QSA,R=permanent]
    ErrorLog /var/log/httpd/redirect.error.log
    LogLevel warn
</VirtualHost>

<VirtualHost *:443>
    DocumentRoot /var/www/html/redmine
    ServerName redmine.prime-kobo.com
    SSLEngine on
    SSLCertificateFile /etc/pki/tls/certs/prime-kobo.com.crt
    SSLCertificateKeyFile /etc/pki/tls/private/prime-kobo.com.2019.key
    SSLCACertificateFile /etc/pki/tls/certs/kingcacert.cer
    CustomLog      /var/log/redmine-prime-kobo.access.log combined
    ErrorLog       /var/log/redmine-prime-kobo.error.log
    <Directory /var/www/html/redmine>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
        Options -Indexes
        AddHandler cgi-script htaccess
    </Directory>
</VirtualHost>

<VirtualHost *:80>
    DocumentRoot /var/www/html/heart-warming
    ServerName www.heart-warming-planner.promo
    CustomLog      /var/log/www-heart-warming.access.log combined
    ErrorLog       /var/log/www-heart-warming.error.log
    <Directory /var/www/html/heart-warming>
        Options All
        AllowOverride All
        Order Allow,Deny
        Allow from all
    </Directory>
</VirtualHost>
```

### サービスの再起動

```bash
[root@v133-18-25-204 admin]# systemctl restart httpd
```

## RDMS

CentOSの標準リポジトリで用意されているRDMSは`MariaDB`です。

`MariaDB`は`MySQL`の開発者が作成しています。なお`MySQL`は既にサービス提供が終了しています。

### インストール

一度、suから抜けて*admin*として作業をします。

```bash
[root@v133-18-25-204 admin]# exit
[admin@v133-18-25-204 ~]$ sudo yum -y install mariadb mariadb-server
# MariaDBのリポジトリを追加します。
[admin@v133-18-25-204 ~]$ curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash
# MariaDBのリポジトリから最新版にアップデートします。
[admin@v133-18-25-204 ~]$ sudo yum -y update MariaDB-server MariaDB-client
# MariaDBを起動して、自動起動の設定をします。
[admin@v133-18-25-204 ~]$ sudo systemctl start mariadb
[admin@v133-18-25-204 ~]$ sudo systemctl enable mariadb
```

### 初期設定

#### 設定スクリプト実行

```bash
[admin@v133-18-25-204 ~]$ sudo mysql_secure_installation
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] Y
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n]
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n]
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n]
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n]
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

#### 文字コード設定

```bash
[admin@v133-18-25-204 ~]$ sudo cp /etc/my.cnf.d/server.cnf /etc/my.cnf.d/server.cnf.org
[admin@v133-18-25-204 ~]$ sudo vi /etc/my.cnf.d/server.cnf
```

```conf
[mariadb]
character-set-server = utf8mb4

[mariadb-10.3]
character-set-server = utf8mb4

[client]
default-character-set = utf8mb4

[mysqld]
character-set-server = utf8mb4
```

```bash
[admin@v133-18-25-204 ~]$ sudo systemctl restart mariadb
```

#### 動作確認

```bash
[admin@v133-18-25-204 ~]$ mysql -u root -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 9
Server version: 10.3.14-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

```sql
MariaDB [(none)]> show variables like 'char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | utf8mb4                    |
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.001 sec)
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.001 sec)

MariaDB [(none)]> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [mysql]> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| column_stats              |
| columns_priv              |
| db                        |
| event                     |
| func                      |
| general_log               |
| gtid_slave_pos            |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| host                      |
| index_stats               |
| innodb_index_stats        |
| innodb_table_stats        |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| roles_mapping             |
| servers                   |
| slow_log                  |
| table_stats               |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| transaction_registry      |
| user                      |
+---------------------------+
31 rows in set (0.001 sec)
MariaDB [mysql]> select Host, User From user;
+-----------+------+
| Host      | User |
+-----------+------+
| 127.0.0.1 | root |
| ::1       | root |
| localhost | root |
+-----------+------+
3 rows in set (0.001 sec)
MariaDB [mysql]> exit
Bye
```

設定に問題が無い事を確認したので、CLIを終了します。

## PHP

CentOS 7標準のPHPのバージョンは5.4系です。PHP7.3をインストールするには`EPEL`と`remi`リポジトリを使用します。

### 準備

```bash
# 各リポジトリのGPGキーをインポートします。
[admin@v133-18-25-204 ~]$ sudo rpm --import http://vault.centos.org/RPM-GPG-KEY-CentOS-7
[admin@v133-18-25-204 ~]$ sudo rpm --import http://rpms.famillecollet.com/RPM-GPG-KEY-remi
# remi リポジトリを CentOS 7.5 にインストールします
[admin@v133-18-25-204 ~]$ sudo yum install http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
```

#### PHPのパッケージ情報を取得

```bash
[admin@v133-18-25-204 ~]$ yum info php --enablerepo=remi-php73
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: ty1.mirror.newmediaexpress.com
 * epel: ftp.iij.ad.jp
 * extras: ty1.mirror.newmediaexpress.com
 * remi-php73: ftp.riken.jp
 * remi-safe: ftp.riken.jp
 * updates: ty1.mirror.newmediaexpress.com
remi-php73                                               | 3.0 kB     00:00
remi-php73/primary_db                                      | 196 kB   00:00
vz-base                                                                     1/1
vz-updates                                                                  4/4
Available Packages
Name        : php
Arch        : x86_64
Version     : 7.3.4
Release     : 1.el7.remi
Size        : 3.2 M
Repo        : remi-php73
```

### PHP 7.3のインストール

`--enablerepo`オプションで、`remi`リポジトリを指定しています。

```bash
[admin@v133-18-25-204 ~]$ sudo yum -y install php php-devel php-mbstring php-pdo php-gd --enablerepo=remi-php73
```

#### バージョンの確認

念の為にバージョンの確認と動作試験を行います。

```bash
[admin@v133-18-25-204 ~]$ php --version
PHP 7.3.4 (cli) (built: Apr  2 2019 13:48:50) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.3.4, Copyright (c) 1998-2018 Zend Technologies
[admin@v133-18-25-204 ~]$ php -r 'print "Hello World!\n";'
Hello World!
```

### Composerをインストール

```bash
[admin@v133-18-25-204 ~]$ curl https://getcomposer.org/installer | php
```

#### インストール後の設定

全ユーザーのパスが通る`/usr/local/bin/`へComposerを移動して、バージョンを確認します。

```bash
[admin@v133-18-25-204 ~]$ sudo mv -i composer.phar /usr/local/bin/composer
[admin@v133-18-25-204 ~]$ composer --version
Composer version 1.8.5 2019-04-09 17:46:47
```

### Laravelのインストール

#### Composerのアップデート

```bash
[admin@v133-18-25-204 ~]$ composer self-update
```

#### gitのインストール

1. 依存関係のあるライブラリをインストール

    ```bash
    [admin@v133-18-25-204 ~]$ sudo yum -y install gcc curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-ExtUtils-MakeMaker autoconf
    ```

2. ダウンロード対処を確認
    1. [公式サイト](https://mirrors.edge.kernel.org/pub/software/scm/git/)で、最新版を検索する。
    2. 執筆時点の最新版は2.21.0なので`git-2.21.0.tar.gz`の*+リンク**をコピーする
3. gitパッケージのダウンロード

    ```bash
    # インストールに適切な場所に移動
    [admin@v133-18-25-204 ~]$ cd /usr/local/src/
    # サイトから Git の圧縮ファイルをダウンロード
    [admin@v133-18-25-204 ~]$ sudo wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.21.0.tar.gz
    # ファイルを解凍
    [admin@v133-18-25-204 ~]$ sudo tar xzvf git-2.21.0.tar.gz
    # 解凍が終わったので、不要な圧縮ファイルを削除
    [admin@v133-18-25-204 ~]$ sudo rm -rf git-2.21.0.tar.gz
    ```

4. gitのインストールと確認

    ```bash
    # 解凍した Git ディレクトリに移動
    [admin@v133-18-25-204 ~]$ cd git-2.21.0
    # make コマンドでインストール
    [admin@v133-18-25-204 git-2.21.0]$ sudo make prefix=/usr/local all
    [admin@v133-18-25-204 git-2.21.0]$ sudo make prefix=/usr/local install
    # バージョン確認コマンド
    [admin@v133-18-25-204 git-2.21.0]$ cd $HOME
    [admin@v133-18-25-204 ~]$ git --version
    git version 2.21.0
    ```

#### Laravel本体のインストール

```bash
[admin@v133-18-25-204 ~]$ sudo yum install -y zip unzip
[admin@v133-18-25-204 ~]$ composer global require "laravel/installer=~1.1"
# 依存関係のインストール
# phpunitの公式サイトhttps://phar.phpunit.de/で最新版のリンクを取得
[admin@v133-18-25-204 ~]$ wget https://phar.phpunit.de/phpunit-8.1.3.phar
[admin@v133-18-25-204 ~]$ chmod +x phpunit-8.1.3.phar
[admin@v133-18-25-204 ~]$ sudo mv phpunit-8.1.3.phar /usr/local/bin/phpunit
[admin@v133-18-25-204 ~]$ phpunit --version
[admin@v133-18-25-204 ~]$ sudo yum -y install php-pear --disablerepo=* --enablerepo=remi,remi-php73
[admin@v133-18-25-204 ~]$ sudo yum -y install libzip-last zlib-devel
[admin@v133-18-25-204 ~]$ sudo yum -y install --enablerepo=epel,remi,remi-php73 php-imap php-mysql php-odbc php-pgsql php-snmp php-xmlrpc php-fpm php-mcrypt php-zip
[admin@v133-18-25-204 ~]$ composer global update
# PATHを通す
[admin@v133-18-25-204 ~]$ vi .bash_profile
```

```conf
PATH=$PATH:/sbin:$HOME/.local/bin:$HOME/bin:$HOME/.config/composer/vendor/bin
```

```bash
[admin@v133-18-25-204 ~]$ source .bash_profile
# 試験
[admin@v133-18-25-204 ~]$ laravel new laraProject
[admin@v133-18-25-204 ~]$ ls -la laraProject
# 後始末
[admin@v133-18-25-204 ~]$ rm -rf laraProject
```

`laravel new`した時に、*symfony/var-dumper suggests installing ext-intl (To show region name in time zone dump)*の様な文が沢山表示されますが、無視して構いません。

これらは、あくまでも*suggests*、つまり**インストールしたらいかがですか？**と言うものですので、プロジェクトは問題なく生成できます。

もし、*suggests*の中に「あー、あったらいいかも」と思うものがあれば、それはインストールして構いませんが、必須ではない事に注意してください。

## Ruby on Rails

できるだけ環境を汚さないように`Ruby on Rails`環境を構築します。

### 事前準備

```bash
[admin@v133-18-25-204 ~]$ sudo yum update
[admin@v133-18-25-204 ~]$ sudo yum install -y bzip2 gcc openssl-devel readline-devel zlib-devel
```

### rbenvのインストール

```bash
[admin@v133-18-25-204 ~]$ git clone https://github.com/rbenv/rbenv.git ~/.rbenv
[admin@v133-18-25-204 ~]$ git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
```

`.bash_profile`に以下を追記します。

```conf
# rbenv
PATH=$HOME/.rbenv/bin:$PATH
eval "$(rbenv init -)"
```

#### 確認

```bash
[admin@v133-18-25-204 ~]$ source ~/.bash_profile
[admin@v133-18-25-204 ~]$ rbenv -v
```

### Rubyのインストール

#### インストール可能バージョンを検索
```bash
[admin@v133-18-25-204 ~]$ rbenv install --list
…
2.5.1
2.5.2
2.5.3
2.5.4
2.5.5
2.6.0-dev
2.6.0-preview1
2.6.0-preview2
2.6.0-preview3
2.6.0-rc1
2.6.0-rc2
2.6.0
2.6.1
2.6.2
2.6.3
2.7.0-dev
```

最新は`2.6.3`ですが、広く利用されている`2.5.3`もインストールして、デフォルトは`2.5.3`にしましょう。

#### Rubyのインストール

`rbenv install`はかなり時間がかかります。

```bash
[admin@v133-18-25-204 ~]$ rbenv install 2.5.3
[admin@v133-18-25-204 ~]$ rbenv install 2.6.3
[admin@v133-18-25-204 ~]$ rbenv global 2.5.3
[admin@v133-18-25-204 ~]$ ruby -v
ruby 2.5.3p105 (2018-10-18 revision 65156) [x86_64-linux]
```

### Rails環境の作成

`Rails`はバージョンによって、利用する`gem`等が変わりますので別ディレクトリに隔離していきます。

#### bundlerのインストール

```bash
[admin@v133-18-25-204 ~]$ gem install bundler
```

#### 使用可能なRailsのバージョンの確認

```bash
[admin@v133-18-25-204 ~]$ gem query -ra -n  "^rails$"
```

今回は`5.2.3`をインストールします。

#### Railsのインストール

```bash
[admin@v133-18-25-204 ~]$ gem install rails -v 5.2.3
```

#### railsプロジェクトの作成

```bash
# プロジェクトの作成
[admin@v133-18-25-204 ~]$ cd /var/www/html
[admin@v133-18-25-204 html]$ rails new training_app -d mysql
[admin@v133-18-25-204 html]$ cd training_app
[admin@v133-18-25-204 training_app]$ bundle install
```

`Rails 5.2.3`が利用する`mysql2`は`0.5.2`ですが、`Gemfile`の`mysql2`のバージョンが低い為に怒られます。

そこで`Gemfile`の`mysql2`の最小バージョンを`0.5.2`に書き換えて、再度インストールします。

```bash
[admin@v133-18-25-204 RubyTest]$ bundle install
```
```bash
[admin@v133-18-25-204 ~]$
```
```bash
[admin@v133-18-25-204 ~]$
```