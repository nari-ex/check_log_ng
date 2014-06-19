# check_log_ng 1.0.7

2013-12-27 TAKIZAWA Takashi

2014-06-18 TAKAMURA Narimichi

Log file regular expression based parser plugin for Nagios.

ログファイルをチェックして正規表現で記述された文字列のパターンを検出するNagiosプラグイン。

## 特徴

* 検知文字列のパターンを正規表現で指定できます。
* 検知除外文字列のパターンが指定できます。これも正規表現で指定できます。
* 複数のログファイルを一括してチェックすることができます。もちろん、ログローテーションされたファイルもチェックできます。
* ログファイル毎にどこまでチェックしたかを記録するseekファイルを利用しているため、前回チェックからの差分だけをチェックできます。
* 複数行に渡って同時に出力されたログメッセージを結合してからチェックすることができます。これにより、検知除外文字列の効果が発揮できるでしょう。
* [check_log3.pl](http://exchange.nagios.org/directory/Plugins/Log-Files/check_log3-2Epl/details)の後方互換性を持っています。現在指定しているオプションをそのまま使えます。

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
# check_log_ng.py -l '/var/log/messages*' -S /var/spool/check_log_ng -p 'ERROR'
```
**ログローテーションしたファイル(messages-YYYYMMDDの形式)を含めてチェックする場合:**
```
# check_log_ng.py -l '/var/log/messages*' -S /var/spool/check_log_ng -p 'ERROR' -R
```

**syslog出力以外のログの場合:**

```
# check_log_ng.py -l '/var/log/application.log' -S /var/spool/check_log_ng -F '^(%Y/%m/%d\s%T,\d+ \S+ \S+) (.*)$' -p 'ERROR'
```
※syslog出力以外の場合には-Fオプションによりログフォーマットのパターンの指定が必要。
※上記例は"2013/12/05 09:36:51,024 jobs-thread-5 ERROR ~ "のようなログの場合。


## ログファイルについて

次のように``-l``オプションでログファイルを指定します。
ログファイルのファイル名のパターンは絶対パスで指定してください。
また、ファイルの絶対パス名には **空白文字を含めることができません** 。
```
-l '/var/log/messages'
```

### 複数ログファイル指定

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


## seek ファイルについて

ログファイルが 1 つだけの場合は次のように``-s``オプションで seek ファイルを指定することができます。
```
-s '/var/spool/check_log_ng/messages.seek'
```

ログファイルが 1 つあるいは複数の場合は``-S``オプションで seek ファイルを格納するディレクトリを指定します。seek ファイルのファイル名はログファイルのファイル名から自動生成されます。seek ファイル用のディレクトリを変えることで複数の監視サーバからのチェックをそれぞれ独立させることができます。

自動生成される seek ファイルのファイル名は、ログファイルの絶対パスのファイル名から英数字と``-``以外の文字を``_``に置換したものに拡張子``.seek``を付けたものになります。例えば、ログファイルのファイル名が``/var/log/messages``の場合はseekファイルのファイル名は``_var_log_messages.seek``になります。

古い seek ファイルをパージする``-R``オプションを用意しています。ログファイルのファイル名にタイムスタンプが付く場合には、ローテーションによってログが削除されても seek ファイルは残ってしまいます。このときは、``-R``オプションを指定して、古いログファイルを削除するようにした方がよいでしょう。

``-R``オプションを指定したときにはデフォルトでは691200秒(8日)より前のファイルがパージされます。この時間を変えるときには``-E``オプションで指定できます。``-E``オプションにはログローテーションの期間より大きい値を指定してください。


## ログフォーマットの指定

syslog 以外のログファイルの場合には、同時に出力された複数行のエラーに対応するために、``--format``オプションによりログの形式を正規表現で指定してください。指定しなければ syslog の形式と判断します。

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
そのため、複数行のログメッセージを同時出力と見なすためのキーとなる情報（タイムスタンプ、ホスト名、タグ、プロセスID、スレッドID、ログレベル等）を正規表現の"(式)"の形式でグルーピングしてください。 **正規表現の後方参照の 1 つ目をキーとします** 。
さらに、残りのメッセージのパートを``(.*)``でまとめてグルーピングしてください。 **正規表現の後方参照の2つ目をメッセージと見なします** 。
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

### フォーマットの指定例

**apacheのエラーログの例:**
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

## パターンの指定

検知する文字列のパターンを``-p``オプションに正規表現で指定します。
検知文字列が複数あるときには行毎にパターンを書いたファイルを用意して``-P``オプションでファイル名を指定します。
CRITICAL と判定させたい文字列があるときには、同様に``--critical-pattern``や``--critical-patternfile``で指定します。

検知を除外する文字列のパターン（ネガティブパターン）があるときには、``-n``オプションに正規表現で指定します。
除外文字列が複数あるときには行毎にパターンを書いたファイルを用意して``-N``オプションでファイル名を指定します。

大文字小文字を区別したくないときには``-i``オプションを指定してください。

``-w``オプションと``-c``オプションには WARNING や CRITICAL になる検知回数を指定します。
デフォルトでは``-w``が 1 で``-c``が 0 であるため、検知文字列が見つかれば WARNING となります。
なお、``--critical-pattern``や``--critical-patternfile``で指定した文字列が検知された場合は、``-w``や``-c``の設定に関わらず CRITICAL になります。

ここで、複数行対応について簡単に説明します。
例えば、次のようなログ出力があるとします。
```
2013/12/05 09:36:51,024 jobs-thread-5 ERROR ~ *** Called URI is: https://www.example.com/submit
2013/12/05 09:36:51,024 jobs-thread-5 ERROR ~ *** Response code is: 500
```
ログフォーマットで指定したキーとなる文字列が同じであるため、メッセージの内容を結合して、次のような1行のメッセージと見なされます。
```
2013/12/05 09:36:51,024 jobs-thread-5 ERROR ~ *** Called URI is: https://www.example.com/submit ~ *** Response code is: 500
```
このメッセージに対してパターンのチェックを実施します。
"ERROR"があり、URIが"/submit"でレスポンスコードが"500"であるパターンとして``"ERROR .*URI.*/submit.*Response code is: 500"``のように記述することができます。

ネガティブパターンを指定するケースについても説明します。
例えば、次のようなログ出力があるとします。
```
2013/12/05 09:36:51,024 jobs-thread-5 ERROR ~ *** Called URI is: https://www.example.com/submit
2013/12/05 09:36:51,024 jobs-thread-5 ERROR ~ *** Response code is: 400
2013/12/05 09:36:51,024 jobs-thread-5 ERROR ~ *** Reason: expired
```
このメッセージは結合されて、次のような1行のメッセージと見なされます。
```
2013/12/05 09:36:51,024 jobs-thread-5 ERROR ~ *** Called URI is: https://www.example.com/submit ~ *** Response code is: 400 ~ *** Reason: expired
```
ここで、エラーの理由が"expired"の場合には検知を除外したいとします。パターンに"ERROR"を指定して、ネガティブパターンとして"Reason: expired"を指定すればよいでしょう。

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

以上でインストール完了です。
なお、動作確認として check_log_ng.py を実行するときには実行権限に注意してください。
seek ファイルの所有者が実行した権限のものになるため、NRPE 経由で実行するときに seek ファイルを更新できなくなる恐れがあります。

## 移行

check_log_ng.py は [check_log3.pl](http://exchange.nagios.org/directory/Plugins/Log-Files/check_log3-2Epl/details) と互換性があります。
[check_log3.pl](http://exchange.nagios.org/directory/Plugins/Log-Files/check_log3-2Epl/details)を check_log_ng.py に置き換える場合はそのままコマンド名を check_log_ng.py に書き換えればよいです。
但し、[check_log3.pl](http://exchange.nagios.org/directory/Plugins/Log-Files/check_log3-2Epl/details) の``-d``, ``-D``, ``-e``, ``-E``, ``-a``オプションには対応していません。

##実行例

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
# check_log_ng.py -l '/var/log/messages*' -S /var/spool/check_log_ng -p 'ERROR'
```

**ログローテーションしたファイル(messages-YYYYMMDDの形式)を含めてチェックする場合:**
```
# check_log_ng.py -l '/var/log/messages*' -S /var/spool/check_log_ng -p 'ERROR' -R -t 604800
```
古い seek ファイルが残ってしまうため、古い seek ファイルを削除するために``-R``オプションを指定し、さらに、``-t``オプションにログローテーションの期間を秒数で指定する。

**ログファイルの形式を指定する場合:**
``2013/12/05 09:36:51,024 jobs-thread-5 ERROR ~ ``のようなログの場合:
```
# check_log_ng.py -l '/var/log/application.log' -S /var/spool/check_log_ng -F '^(%Y/%m/%d\s%T,\d+ \S+ \S+) (.*)$' -p 'ERROR'
```

なお、ログの参照に root 権限が必要な場合には /etc/sudoers に次のような記述を追加してください。NRPE の実行権限は適宜置き換えてください。
```
Defaults:nrpe !requiretty
nrpe ALL=(root) NOPASSWD: /usr/lib64/nagios/plugins/check_log_ng.py
```
それから、sudo経由で check_log_ng.py を実行するようにNRPEの設定を行ってください。

## 使い方

```
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

## 修正履歴

* 2013-12-05 1.0.0 initial release.
* 2013-12-20 1.0.1 fix check argc and parse logformat variable.
* 2013-12-20 1.0.2 revise version processing and fix a help message.
* 2013-12-27 1.0.3 change an OK message.
* 2014-03-05 1.0.4 add --trace-inode options.
* 2014-03-11 1.0.5 add ---multiline options.
* 2014-03-31 1.0.6 add --critical-negpattern options.
* 2014-06-18 1.0.7 fix bugs.

## 仕様

T.B.D.

**ログファイルのチェックをスキップする条件:**

* ログファイルの mtime が次の条件を満たす場合。
```
    [ログファイルの mtime] < [現在時刻] - [--scantime オプションの値]
```
* ログファイルのファイルサイズと seek ファイルに記録されたオフセット値が等しい。
```
    [ログファイルのサイズ] == [seek ファイルのオフセット値]
```

**seek ファイルを削除する条件:**

* seek ファイルの mtime が次の条件を満たす場合に seek ファイルを削除する。
```
    [seek ファイルの mtime] < [現在時刻] - [--expiration オプションの値]
```


EOT
