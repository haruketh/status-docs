# なぜStatusノードを実行するのか？

現在 Statusノードの運営には、暗号資産等のインセンティブはありません。私たちはこの問題を解決するために努力しています。私たちの目的は、Wakuネットワークの規模を拡大し、プラットフォームの分散性と安全性を向上させることです。


## プライバシー

もうひとつの理由は、プライバシーです。現在の設定では、リレーノードもヒストリーノードもほとんどのノードがStatusのインフラの一部として動作しています。つまり、Statusはネットワークの大部分を広く見ることができるのです。Wakuではすべてのトラフィックが暗号化されていますが、この方法で収集される可能性のあるメタデータは、いくつかの情報をリークする可能性があります。それを避けるための最良の方法は、独自のノードを実行し、Statusアプリで設定することです。


## コミュニティ

自分のノードを運営することで、Statusコミュニティに追加のノードを提供することができます。誰でも自分のノードのenodeアドレスを公開し、他の人が使えるようにすることをお勧めします。また、システムの再起動やランタイムノードのエラーが発生しても実行し続けられるように、永続的なサービスやdockerコンテナとして実行することをお勧めします。


## ノードの種類

- リレーノード - モバイルやデスクトップのクライアントを含むノード間でメッセージを中継する通常のWakuノード。

- ヒストリーノード - Mailserverとしても知られ、過去のメッセージを保存し、問い合わせがあった場合に配信します。
  - 追加のディスクスペースが必要です。過去30日分のメッセージを保存するには、まず1GB程度の空き容量が必要となります。


## Statusノードを運営する

Statusノードは、サーバー上で動作する[status-go](https://github.com/ethereum/go-ethereum)と呼ばれる[go-ethereum](https://github.com/status-im/status-go)ノードを改造したもので、Statusアプリをサポートしています。私たちは分散型モデルで運営しているため、信頼できるサービスを提供するためには、世界中に散らばる複数のピアが必要です。
Statusノードは、Wakuメッセージの中継、ノード間の伝播をサポートし、メッセージ送信時にオフラインだったデバイスのための保存をサポートします。


## システム要件

LinuxまたはMacOSが動作するマシンが必要です。Statusノードは、ローカルネットワーク上の物理マシン上で動作させることも可能ですが、完全な機能を実現するためには、サービスにアクセスするためのパブリックな固定IPアドレスが必要です。

クラウドサービスプロバイダーは、ほとんどの場合、パブリックな固定IPアドレスを自動的に提供します。クラウドサービスプロバイダーは、より強力なアップタイムの保証を提供することができるため、より多くのエンベロープを収集して後で検索することができます。

Statusノードを安定して動作させるには、通常、1GB RAMと1 vCPUを搭載したシングルインスタンスが必要です。

最低限必要なソフトウェアは`make`と`jq`です。`status-go`をビルドしたい場合は、`golang`のバージョン1.13以上も必要です。

`enode://`のアドレスでQRコードを表示するには`qrencode`があると便利です。

例：Ubuntu `20.04`の場合。

```
sudo apt install make jq golang qrencode
```


## ポート

- `30303` TCP/UDP - [DevP2P](https://github.com/ethereum/devp2p) ワイヤ・プロトコル・ポート。**常に**パブリックでなければなりません。
- `8545` TCP - [JSON RPC](https://github.com/ethereum/wiki/wiki/json-rpc)管理ポート。**絶対に**公開してはいけません。
- `9090` TCP - [Prometheus](https://prometheus.io/docs/concepts/data_model/)メトリクス・ポート。公開してはいけません。


# クイックスタート

ノードを起動する最も早い方法は、`Makefile`スクリプトを使うことです。詳しくは[こちら](https://github.com/status-im/status-go/blob/develop/MAILSERVER.md)をご覧ください。

1. [status-go](https://github.com/status-im/status-go) リポジトリをクローンします。
2. 2つのオプションのうち1つを使用して、ノードサービスを開始します。

  A. Dockerコンテナ - ノードを構築する必要はありません。
  ```
  make run-mailserver-docker
  ```
  詳しくは、[Docker](https://github.com/status-im/status-go/blob/develop/_assets/compose/mailserver)の`README`を参照してください。

  B. systemd service - ノードの構築が必要です。
  ```
  make run-mailserver-systemd
  ```
  詳しくは、[systemd](https://github.com/status-im/status-go/blob/develop/_assets/systemd/mailserver)の`README`を参照してください。


# 手動での構築

## ビルド

まず、`statusd`のバイナリをビルドします。
```
mkdir ~/go/src/github.com/status-im
git clone https://github.com/status-im/status-go ~/go/src/github.com/status-im/status-go
cd ~/go/src/github.com/status-im/status-go
make statusgo
```
詳しくは[このページ](https://status.im/technical/build_status/status_go.html)をご覧ください。


## 実行する

利用可能なオプションは、`-h`/`--help`フラグで確認できます。
```
./build/bin/statusd -h
```
なお、デフォルトの設定では、フルリレーとヒストリーノードを実行することはできません。


## 設定

設定は、JSONファイルで提供されます。ここでは、過去のメッセージをWakuノードに格納させるための基本的な設定を紹介します。

`./config.json`

```
{
    "AdvertiseAddr": "<YOUR_PUBLIC_IP>",
    "ListenAddr": "0.0.0.0:30303",
    "HTTPEnabled": true,
    "HTTPHost": "127.0.0.1",
    "HTTPPort": 8545,
    "APIModules": "eth,net,web3,admin,mailserver",
    "RegisterTopics": ["whispermail"],
    "WakuConfig": {
        "Enabled": true,
        "EnableMailServer": true,
        "DataDir": "/var/tmp/statusd/waku",
        "MailServerPassword": "status-offline-inbox"
    }
}
```

`-c`フラグを使って提供します。
```
$ ./build/bin/statusd -c ./config.json
```

その他の設定ファイルの例は、[このディレクトリ](https://github.com/status-im/status-go/tree/develop/config/cli)をご覧ください。設定オプションの詳細については [README](https://github.com/status-im/status-go/blob/develop/config/README.md) を参照してください。

また [このソースファイル](https://github.com/status-im/status-go/blob/develop/params/config.go)のオプションに対するコメントも読むことができます。


## メトリック

Prometheusメトリクスを有効にするには、`statusd`の実行時に以下のフラグを使用します。
```
./build/bin/statusd -metrics -metrics-port=9090
```

メトリクスは`9090`ポートで公開されます。
```
 > curl -s localhost:9090/metrics | grep '^whisper_envelopes_received_total'
whisper_envelopes_received_total 123
```

## ヘルスチェック

サービスが実行されているかどうかを確認するには、JSON RPC管理APIを使用します。
```
 $ export DATA='{"jsonrpc":"2.0","method":"admin_peers","params":[],"id":1}'
 $ curl -s -H 'content-type: application/json' -d "$DATA" localhost:8545 | jq -r '.result[].network.remoteAddress'
34.68.132.118:30305
134.209.136.123:30305
178.128.141.249:443
```

# Dockerを使う

Statusが提供する[Dockerイメージ](https://hub.docker.com/r/statusteam/status-go/)は、私たちが提供する[Docker Composeセットアップ](https://github.com/status-im/status-go/tree/develop/_assets/compose/mailserver)を使用するだけでなく、フリート上のノードを実行するために使用されます。

例）自分でコンテナを動かす場合
```
docker run --rm \
    -p 8545:8545 \
    -p 30303:30303 \
    -v $(pwd)/config.json:/config.json \
    statusteam/status-go:0.55.1 \
    -register \
    -log DEBUG \
    -c /config.json
```


---
Last update: 2021-08-09

https://status.im/technical/run_status_node.html
