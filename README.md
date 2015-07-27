# check_log_ng 1.0.7

[![wercker status](https://app.wercker.com/status/7c718bb6e4a7d61c3a4fd66c5b863585/s "wercker status")](https://app.wercker.com/project/bykey/7c718bb6e4a7d61c3a4fd66c5b863585)

2013-12-27 TAKIZAWA Takashi

2014-06-18 TAKAMURA Narimichi

Log file regular expression based parser plugin for Nagios.

ログファイルをチェックして正規表現で記述された文字列のパターンを検出するNagiosプラグイン。



## 修正履歴

* 2015-01-14 1.0.8 Refactor for PEP8, pyflake8.
* 2014-06-18 1.0.7 Fix bugs.
* 2014-03-31 1.0.6 Add --critical-negpattern options.
* 2014-03-11 1.0.5 Add ---multiline options.
* 2014-03-05 1.0.4 Add --trace-inode options.
* 2013-12-27 1.0.3 Change an OK message.
* 2013-12-20 1.0.2 Revise version processing and fix a help message.
* 2013-12-20 1.0.1 Fix check argc and parse logformat variable.
* 2013-12-05 1.0.0 Initial release.



## 特徴

* 検知文字列のパターンを正規表現で指定できます。
* 検知除外文字列のパターンが指定できます。これも正規表現で指定できます。
* 複数のログファイルを一括してチェックすることができます。もちろん、ログローテーションされたファイルもチェックできます。
* ログファイル毎にどこまでチェックしたかを記録するseekファイルを利用しているため、前回チェックからの差分だけをチェックできます。
* 複数行に渡って同時に出力されたログメッセージを結合してからチェックすることができます。これにより、検知除外文字列の効果が発揮できるでしょう。
* [check_log3.pl](http://exchange.nagios.org/directory/Plugins/Log-Files/check_log3-2Epl/details)の後方互換性を持っています。現在指定しているオプションをそのまま使えます。

## インストール

ここでは、Nagios プラグインとして check_log_ng をインストールする手順を説明します。
まず、check_log_ng.py を Nagios のプラグインディレクトリにコピーし、実行権限を付与します。
```
# cp check_log_ng.py /usr/lib64/nagios/plugins/
# chmod 755 check_log_ng.py
```

予め seek ファイル用のディレクトリを作成し、 NRPE の実行権限で書き込みできるようにします。
```
# mkdir /var/spool/check_log_ng
# chown nrpe:nrpe /var/spool/check_log_ng
```

check_log_ng.py を実行するときには実行権限に注意してください。
seek ファイルの所有者が実行した権限のものになるため、NRPE 経由で実行するときに seek ファイルを更新できなくなる恐れがあります。
ログの参照に root 権限が必要な場合には、sudo経由で check_log_ng.py を実行するようにNRPEの設定を行ってください。
また、/etc/sudoers に次のような記述を追加してください。
```
Defaults:nrpe !requiretty
nagios ALL=(root) NOPASSWD: /usr/lib64/nagios/plugins/check_log_ng.py
```

以上でインストール完了です。



## 移行

check_log_ng.py は [check_log3.pl](http://exchange.nagios.org/directory/Plugins/Log-Files/check_log3-2Epl/details) と互換性があります。
[check_log3.pl](http://exchange.nagios.org/directory/Plugins/Log-Files/check_log3-2Epl/details)を check_log_ng.py から移行する場合は、コマンド名を check_log_ng.py に書き換えます。それ以外の変更する必要はありません。
但し、[check_log3.pl](http://exchange.nagios.org/directory/Plugins/Log-Files/check_log3-2Epl/details) の``-d``, ``-D``, ``-e``, ``-E``, ``-a``オプションには対応していません。



## 実行例

**一つのログファイルをチェックする場合:**
```
# check_log_ng.py -l '/var/log/messages' -s /var/spool/check_log_ng/messages -p 'ERROR'
```
あるいは
```
# check_log_ng.py -l '/var/log/messages' -S /var/spool/check_log_ng -p 'ERROR'
```

**ログローテーションしたファイル(messages.Nの形式)を含めてチェックする場合:**
```
# check_log_ng.py -l '/var/log/messages*' -S /var/spool/check_log_ng -p 'ERROR' -I -R
```

**ログローテーションしたファイル(messages-YYYYMMDDの形式)を含めてチェックする場合:**
```
# check_log_ng.py -l '/var/log/messages*' -S /var/spool/check_log_ng -p 'ERROR' -I -R
```

**``2013/12/05 09:36:51,024 jobs-thread-5 ERROR ~ ``のような複数行のログをチェックする場合**
```
# check_log_ng.py -l '/var/log/application.log' -S /var/spool/check_log_ng -F '^(%Y/%m/%d\s%T,\d+ \S+ \S+) (.*)$' -p 'ERROR' -I -R -M
```


## オプションの解説


### ログファイルの指定（-l, --logfile）

``-l``オプションでログファイルを指定します。
ログファイルのファイル名のパターンは絶対パスで指定してください。
また、ファイルの絶対パス名には **空白文字を含めることができません** 。
```
-l '/var/log/messages'
```


#### 複数ログファイルの指定

ファイル名のパターンとしてメタキャラクタの``*``と``?``に対応しています。次の例のように記述できます。メタキャラクタを指定するときには引用符で囲ってください。
```
-l '/var/log/messages*'
```
ただし、ログファイルのファイル名のパターンには **メタキャラクタの``[]`` ``{}``は利用できません** 。使用するとseekファイルのパージ機能が正常に動作しなくなります。

ファイル名は引用符で囲ってスペース区切りでも記述できます。次の例のように記述できます。
```
-l '/var/log/messages /var/log/maillog'
```

パターンに一致するファイルがたくさんある場合でも、ログファイルのタイムスタンプのチェックも行っているため、``-t``オプションで指定した時間以降に更新の無いログファイルのチェックを行わずに済みます。``-t``オプションの値はデフォルトは86400秒（1日）です。


### seek ファイルの指定（-s, --seekfile, -S, --seekfile-directory）

ログファイルが 1 つだけの場合は次のように``-s``オプションで seek ファイルを指定することができます。
```
-s '/var/spool/check_log_ng/messages.seek'
```

ログファイルが 1 つあるいは複数の場合は``-S``オプションで seek ファイルを格納するディレクトリを指定します。
seek ファイルのファイル名はログファイルのファイル名から自動生成されます。
seek ファイル用のディレクトリを変えることで複数の監視サーバからのチェックをそれぞれ独立させることができます。

自動生成される seek ファイルのファイル名は、ログファイルの絶対パスのファイル名から英数字と``-``以外の文字を``_``に置換したものに拡張子``.seek``を付けたものになります。
例えば、ログファイルのファイル名が``/var/log/messages``の場合はseekファイルのファイル名は``_var_log_messages.seek``になります。


### ログフォーマットの指定（-F, --format）と複数行対応（-M, --multiline）

syslog 以外の複数行出力のあるログファイルの場合については、
``-F``オプションと``-M``オプションを指定してください。
これにより、同時に出力された複数行のエラーに対応することができます。
なお、``-F``オプションの指定が無い場合は syslog の形式と判断します。
``-M``オプションがない場合は、1行ごとに処理を行います。

同時に出力されたメッセージを判断するために、タイムスタンプやタグ等の情報を利用しています。
例えば、次のようなログ出力があるときには``2013/12/05 09:36:51,024 jobs-thread-5 ERROR``が同じであるため、同時出力されたメッセージと見なすようにします。
```
2013/12/05 09:36:51,024 jobs-thread-5 ERROR ~ *** Called URI is: https://www.example.com/submit
2013/12/05 09:36:51,024 jobs-thread-5 ERROR ~ *** Response code is: 500
```
このとき、メッセージの内容を結合して、次のような1行のメッセージと見なしてからパターンのチェックを実施します。
```
2013/12/05 09:36:51,024 jobs-thread-5 ERROR ~ *** Called URI is: https://www.example.com/submit ~ *** Response code is: 500
```
そのため、複数行のログメッセージを同時出力と見なすためのキーとなる情報（タイムスタンプ、ホスト名、タグ、プロセスID、スレッドID、ログレベル等）を正規表現の"(式)"の形式でグルーピングしてください。
**正規表現の後方参照の 1 つ目をキーとします** 。
さらに、残りのメッセージのパートを``(.*)``でまとめてグルーピングしてください。
**正規表現の後方参照の2つ目をメッセージと見なします** 。
他の箇所で括弧``(〜)``を使うときには``(?:〜)``の形式を使って後方参照しないようにしてください。
また、正規表現の他に``strftime(3)``の次のパターンが利用できます。
```
%%  文字%
%Y  年(4桁)
%y  年(2桁)
%a  曜日(Sun..Sat)
%b  月(Jan..Dec)
%m  月(01..12)
%d  日(01..31)
%e  日( 1..31)
%H  時(00..23)
%M  分(00..59)
%S  秒(00..60)
%F  %Y-%m-%d
%T  %H:%M:%S
```
実行時に内部的に正規表現に展開されます。

#### フォーマットの指定例

**Apacheのエラーログの例:**
```
[Thu Dec 05 15:03:20 2013] [error] [client ::1] Directory index forbidden by Options directive: /var/www/html/
```

**オプションの指定:**
```
--format='^(\[%a %b %d %T %Y\] \[\S+\]) (.*)$'
```

**アプリケーションログの例:**
```
2013/12/05 09:36:51,024 jobs-thread-5 ERROR ~ *** Called URI is: https://www.example.com/submit
2013/12/05 09:36:51,024 jobs-thread-5 ERROR ~ *** Response code is: 500
```

**オプションの指定:**
```
--format='^(%Y/%m/%d %T,\d+ \S+ \S+) (.*)$'
```
※ NRPEの設定ファイル上で``$``を用いるとエラーが発生しますので注意しましょう。
なお、ログフォーマットのパターンに一致しない行は前の行に対する継続行と見なされ、前の行に結合されます。


### 監視パターンの指定（-p, --pattern, -P, --patternfile）

監視文字列のパターンは``-p``オプションを用いて指定します。
監視文字列が複数あるときには行毎にパターンを書いたファイルを用意して``-P``オプションの引数としてファイル名を指定します。
パターンは正規表現を用いて指定することができます。
``p``または``-P``で監視パターンを指定した場合は、アラート発報時のステータスはWARNINGとなります。

#### 重篤な監視パターンの指定（--critical-pattern, --critical-patternfile）

WARNING ではなく CRITICAL と判定させたい場合は、``-p``, ``-P``オプションの代わりに``--critical-pattern``, ``--critical-patternfile``オプションを用いて監視パターンを指定します。


### 除外パターンの指定（-n, --negpattern, -N, -f, --negpatternfile）

検知を除外する文字列のパターン（ネガティブパターン）があるときには、``-n``オプションに正規表現で指定します。
除外文字列が複数あるときには行毎にパターンを書いたファイルを用意して``-N``または``-f``オプションでファイル名を指定します。
※ ``-f``オプションは、check_log3.pl の互換のために実装してあります。

``-n``または``-N``によって除外パターンを指定した場合、
``-p``または``-P``で指定した監視パターンにマッチするメッセージが出力されている場合でも、
同時に除外パターンにマッチするメッセージが出力されている場合には、アラートは発報しません。
なお、重篤な監視パターンについては無視をせず、通常通りアラート発報します。


#### 指定例

除外パターンを指定するケースについて説明します。
例えば、次のようなログ出力があるとします。
```
2013/12/05 09:36:51,024 jobs-thread-5 ERROR ~ *** Called URI is: https://www.example.com/submit ~ *** Response code is: 400 ~ *** Reason: expired
```
ここで、エラーの理由が"expired"の場合には検知を除外したいとします。
この場合は監視パターンに"ERROR"を指定して、除外パターンとして"Reason: expired"を指定すればよいでしょう。

#### より強力な除外パターンの指定（--critical-negpattern, --critical-negpatternfile）

``-n``または``-N``では、``--critical-pattern``または``--critical-patternfile``で指定された監視パターンが存在する場合には、無視をせずアラートを発報します。
``--critical-negpattern``または``--critical-negpatternfile``によって除外パターンを指定した場合、
監視パターンにマッチするメッセージが出力されている場合でも、
同時に除外パターンにマッチするメッセージが出力されている場合には、それらを無視してアラートは発報しません。
重篤な監視パターンについても無視します。


### 大文字小文字を区別しない（-i, --case-insensitive）

``-i``オプションを指定することで、大文字小文字を区別せずにパターンマッチを指定行います。
このオプションの指定は、監視パターンマッチと除外パターンマッチの両方に適用されます。


### 検知回数の指定（-w, --warning, -c, --critical）

``-w``オプションと``-c``オプションは、 WARNING や CRITICAL になる検知回数を指定します。
デフォルトでは``-w``が 1 で``-c``が 0 であるため、検知文字列が見つかれば WARNING となります。
なお、``--critical-pattern``や``--critical-patternfile``で指定した文字列が検知された場合は、``-w``や``-c``の設定に関わらず CRITICAL になります。


### スキャン期間の指定（-t, --scantime）

以下のように複数のログファイルを指定する場合、マッチするすべてのログファイルを毎回検索する必要はありません。更新のないログファイルは検索する必要がないからです。
```
-l '/var/log/messages*'
```

``-t``オプションを用いることで、しばらく更新されていないログは検索対象外となります。
具体的には、ログファイルの mtime が次の条件を満たす場合に対象外となります。オプション引数は秒単位で指定します。デフォルト値は 86400 秒（1日）です。

```
[ログファイルの mtime] < [現在時刻] - [-t に渡された値]
```


### seek ファイルの保存期間の指定（-E, --expiration）

seek ファイルの保存期間を秒単位で指定します。デフォルトは 691200 秒（8日）です。
この値は、**ログローテーション期間より大きい値を指定してください**。
なお、後述する``-R``オプションを指定していない場合、seek ファイルは削除されません。


### 期限切れの seek ファイルの削除（-R, --remove-seekfile）

``-R``を指定することで、期限切れの seek ファイルを削除します。
期限の指定は、``-E``オプションで指定します。
seek ファイルの mtime が次の条件を満たす場合に seek ファイルを削除します。
```
[seek ファイルの mtime] < [現在時刻] - [--expiration オプションの値]
```


### inode ベースの seek ファイル作成（-I, --trace-inode）

``-I``オプションを指定することで、seekファイルのファイル名に inode の情報を埋め込みます。
これにより、seek ファイルは inode ベースで管理されます。
inode ベースで seek ファイルを管理すると、
ログローテート後に、他のログファイルで用いていた seek ファイルが使いまわされるという問題を回避することができます。
そのため、複数ファイル指定する場合はこのオプションの指定を強く推奨します。


## その他

デバッグオプションとして``--debug``があります。
また、``-h``または``--help``を指定することで以下のようにオプション一覧が出力されます。

```
$ ./check_log_ng.py -h
Usage: check_log_ng.py [option ...]

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  -l <filename>, --logfile=<filename>
                        The pattern of log files to be scanned. The
                        metacharacter * and ? are allowed. If you want to set
                        multiple patterns, set a space between patterns.
  -F <format>, --format=<format>
                        The regular expression of format of log to parse.
                        Required two group, format of '^(TIMESTAMP and
                        TAG)(.*)$'. Also, may use %%, %Y, %y, %a, %b, %m, %d,
                        %e, %H, %M, %S, %F and %T of strftime(3). Default: the
                        regular expression for syslog.
  -s <filename>, --seekfile=<filename>
                        The temporary file to store the seek position of the
                        last scan. If check multiple log files, ignore this
                        option. Use -S seekfile_directory.
  -S <seekfile_directory>, --seekfile-directory=<seekfile_directory>
                        The directory of the temporary file to store the seek
                        position of the last scan. If check multiple log
                        files, require this option.
  -I, --trace-inode     Trace the inode of log files. If set, use inode
                        information as a seek file.
  -p <pattern>, --pattern=<pattern>
                        The regular expression to scan for in the log file.
  -P <filename>, --patternfile=<filename>
                        File containing regular expressions, one per line.
  --critical-pattern=<pattern>
                        The regular expression to scan for in the log file. In
                        spite of --critical option, return CRITICAL.
  --critical-patternfile=<filename>
                        File containing regular expressions, one per line. In
                        spite of --critical option, return CRITICAL.
  -n <pattern>, --negpattern=<pattern>
                        The regular expression to skip except as critical
                        pattern in the log file.
  -N <filename>, -f <filename>, --negpatternfile=<filename>
                        Specifies a file with regular expressions which all
                        will be skipped except as critical pattern, one per
                        line.
  --critical-negpattern=<pattern>
                        The regular expression to skip in the log file
  --critical-negpatternfile=<filename>
                        Specifies a file with regular expressions which all
                        will be skipped, one per line.
  -i, --case-insensitive
                        Do a case insensitive scan
  -w <number>, --warning=<number>
                        Return WARNING if at least this many matches found.
                        The default is 1.
  -c <number>, --critical=<number>
                        Return CRITICAL if at least this many matches found.
                        The default is 0, i.e. don't return critical alerts
                        unless specified explicitly.
  -d, --nodiff-warn     Return WARNING if the log file was not written to
                        since the last scan. (not implemented)
  -D, --nodiff-crit     Return CRITICAL if the log was not written to since
                        the last scan. (not impremented)
  -t <seconds>, --scantime=<seconds>
                        The range of time to scan. The log files older than
                        this time are not scanned. Default is 86400.
  -E <seconds>, --expiration=<seconds>
                        The expiration of seek files. Default is 691200. This
                        value must be greater than period of log rotation when
                        use with -R option.
  -R, --remove-seekfile
                        Remove expired seek files. See also --expiration.
  -M, --multiline       Consider multiple lines with same key as one log
                        output. See also --multiline.
  --debug               Enable debug.
```


## ライセンス

ライセンスは 2 条項 BSD ライセンスに準拠します。
