SNMP ネットワークモニタリングテンプレート
===============================================

SNMP ネットワークモニタリング
-------------------

SNMP 統計を用いてネットワーク機器のパフォーマンス情報を採取します。採取したデータをモニタリングサーバ側で集計してグラフ登録をします。

**注意事項**

1. SNMP統計をリモートで採取する Linux 用エージェントが別途必要で、監視サーバ上で Linux エージェントを稼働します。

ファイル構成
-------

テンプレートに必要な設定ファイルは以下の通りです。

|              ディレクトリ             |        ファイル名        |                  用途                 |
|---------------------------------------|--------------------------|---------------------------------------|
| lib/agent/SNMPNetwork/conf/           | iniファイル              | エージェント採取設定ファイル          |
| lib/Getperf/Command/Site/SNMPNetwork/ | pmファイル               | データ集計スクリプト                  |
| lib/graph/SNMPNetwork/                | jsonファイル             | グラフテンプレート登録ルール          |
| lib/cacti/template/0.8.8g/            | xmlファイル              | Cactiテンプレートエクスポートファイル |
| script/                               | create_graph_template.sh | グラフテンプレート登録スクリプト      |

メトリック
-----------

TTSPパフォーマンス統計グラフなどの監視項目定義は以下の通りです。

|          Key           |                             Description                              |
|------------------------|----------------------------------------------------------------------|
| **パフォーマンス統計** | **ネットワーク  I/O 統計パフォーマンス統計グラフ**                   |
| snmp_network_port      | **ポート 別ネットワーク I/O統計**<br>MB/sec / Packet/sec / Error/sec |

Install
=====

テンプレートのビルド
-------------------

Git Hub からプロジェクトをクローンします

	(git clone してプロジェクト複製)

プロジェクトディレクトリに移動して、--template オプション付きでサイトの初期化をします

	cd t_SNMPNetwork
	initsite --template .

Cacti グラフテンプレート作成スクリプトを順に実行します(1行目がArrayFort、2行目がSC3000)

	./script/create_graph_template.sh

Cacti グラフテンプレートをファイルにエクスポートします

	cacti-cli --export SNMPNetwork

集計スクリプト、グラフ登録ルール、Cactiグラフテンプレートエクスポートファイル一式をアーカイブします

	sumup --export=SNMPNetwork --archive=$GETPERF_HOME/var/template/archive/config-SNMPNetwork.tar.gz

テンプレートのインポート
---------------------

前述で作成した $GETPERF_HOME/var/template/archive/config-SNMPNetwork.tar.gz がSNMPNetworkテンプレートのアーカイブとなり、
監視サイト上で以下のコマンドを用いてインポートします

	cd {モニタリングサイトホーム}
	tar xvf $GETPERF_HOME/var/template/archive/config-SNMPNetwork.tar.gz

Cacti グラフテンプレートをインポートします。監視対象のストレージに合わせてテンプレートをインポートしてください

	cacti-cli --import SNMPNetwork

インポートした集計スクリプトを反映するため、集計デーモンを再起動します

	sumup restart

使用方法
=====

エージェントセットアップ
--------------------

以下のエージェント採取設定ファイルをエージェントホームの conf ディレクトリの下(/home/{OSユーザ}/ptune/conf/)にコピーして、エージェントを再起動してください。

	{サイトホーム}/lib/agent/SNMPNetwork/conf/SNMPNetwork.ini

監視対象サーバから直接採取する場合と、リモートで採取する場合で実行オプションの変更が必要になります。
以下例はリモート採取の設定となります。

	;---------- Monitor command config (SNMPNetwork Storage) -----------------------------------
	STAT_ENABLE.SNMPNetwork = true
	STAT_INTERVAL.SNMPNetwork = 300
	STAT_TIMEOUT.SNMPNetwork = 340
	STAT_MODE.SNMPNetwork = concurrent

	STAT_CMD.SNMPNetwork = '_script_/get_snmp_SNMPNetwork.pl -i 60 -n 5 -c community_name -s 192.168.0.1',   CL-SNMPNetwork-C215A/get_snmp_SNMPNetwork.txt

データ集計のカスタマイズ
--------------------

上記エージェントセットアップ後、データが集計されると、サイトホームディレクトリの lib/Getperf/Command/Master/ の下に SNMPNetwork.pm ファイルが生成されます。
本ファイルは監視対象ストレージのマスター定義ファイルで、ストレージのコントローラ、LUN、Raidグループの用途を記述します。
同ディレクトリ下の SNMPNetwork.pm_sample を例にカスタマイズしてください。

グラフ登録
-----------------

上記エージェントセットアップ後、データ集計が実行されると、サイトホームディレクトリの node の下にノード定義ファイルが出力されます。
出力されたファイル若しくはディレクトリを指定してcacti-cli を実行します。

	cacti-cli node/SNMPNetwork/{ストレージノード}/

AUTHOR
-----------

Minoru Furusawa <minoru.furusawa@toshiba.co.jp>

COPYRIGHT
-----------

Copyright 2014-2016, Minoru Furusawa, Toshiba corporation.

LICENSE
-----------

This program is released under [GNU General Public License, version 2](http://www.gnu.org/licenses/gpl-2.0.html).
