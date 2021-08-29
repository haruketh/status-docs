# Statusノードの実行とメンテナンス

## Statusノードとは？

アプリケーションのユーザのメッセージを、可能な限り偽名や非公開で中継することができるソフトウェア。モバイル機器や低価格のノートPCでも利用できるリソースを確保しながら、モバイル機器やローエンドのラップトップでも対応可能なリソースを確保しつつ、可能な限り偽名でプライベートなメッセージを中継するソフトウェア。

Status Nodeは、いくつかの機能を提供する必要があります。
- 分散型の方法でピアとコミュニケーションする
- 非公開・匿名でのメッセージのやり取り
- 定期的にオフラインになっているピアの履歴メッセージを提供

リンク:
• https://github.com/status-im/status-go
• https://our.status.im/what-is-a-status-node-really/
• https://rfc.vac.dev/spec/6/

なぜノードを実行するのか？
- ネットワークを自社のインフラに依存せず、より分散化させる
- Statusノードがブロックされている場所での通信を容易にする
- Waku v1の限界をよりよく研究し、理解するのに役立つ
- ノードを運営する上での問題点を発見し、改善するのに役立つ
- クエリが高速化され、チャットがよりスムーズになる

ノードはどこで入手できますか？
- Docker Images: https://hub.docker.com/r/statusteam/status-go
- The recommended image is statusteam/status-go:v0.84.0
- Repository: https://github.com/status-im/status-go
- We do not provide binary releases currently. Node binary to be built by yourself.
- This requires the Go compiler version 1.13 or newer.

実行の方法
- Docker Compose - より自己完結的でポータブルなシステム
- Systemdサービス - よりローカルで設定可能
  - 前者は事前にビルドされたDockerイメージを使用し、後者はバイナリをビルドする必要があります。

## Dockerイメージを実行する

1. 最も簡単な方法は、Makefileターゲットを使うことです。

2. 全体の設定は、`_assets/compose/mailserver/Makefile を使って作成します。

## Dockerイメージを管理する
コンテナの管理にはdockerまたはdocker-composeを使用します。

サービスポート
- `sudo netstat -lpnt | grep -e 8545 -e 9090 -e 30303`
- 30303 TCP/UDP - DevP2P ワイヤ・プロトコル・ポート。常にパブリックでなければなりません。
- 8545 TCP - JSON RPC管理ポート。絶対に公開してはいけません。
- 9090 TCP - Prometheusメトリクス・ポート。公開してはいけません。

メトリクスAPI へのクエリ
- デフォルトでは、Prometheus metricsのエンドポイントを9090ポートで公開しています。
- `curl -sS localhost:9090/metrics | grep -E '^[^#]+' | wc -l`
- `curl -sS localhost:9090/metrics | grep '^waku_envelopes_received_total'`
- `curl -sS localhost:9090/metrics | grep '^p2p_peers'`

JSON RPC API へのクエリ
- Go-Ethereumと同様のJSON RPC APIを公開しています。
- `_assets/scripts/rpc.sh admin_nodeInfo | jq '.result | {name, enode, enr}'`
- `_assets/scripts/rpc.sh admin_peers | jq -cr '.result[] | {name, caps, addr: .network.remoteAddress}'`
- `_assets/scripts/rpc.sh admin_addPeer enode://（enodeアドレス）`
- rpc.shスクリプトは、curlの薄いラッパーに過ぎません。

ポートが開いているかどうかの確認
- 自分のポートが公開されているかどうかを確認する最も簡単な方法はNMapです。
- `sudo nmap -Pn -p22,80,443,30504 104.197.238.144`
  - コマンドが効かないときはnmapをインストールしてください。
  - `filtered` はファイアウォールがポートをブロックしていることを意味します。
  - `closed` とは、そのポートでListenしていないということです。

canaryでノードをチェック
- status-goリポジトリにはnode-canaryというユーティリティが含まれており、Statusノードの接続を検証します。
- 接続してクエリすることで、接続を行うことができます。
- `./build/bin/node-canary -mailserver enode://（enodeアドレス）`
- `make node-canary` コマンドでビルドできます。
- enodeが正しく応答した場合は出口コード0、そうでない場合は正の数が返されます。

※ P2Pノードカナリアサービスの目的は、指定されたノードが正しく応答しているかどうかをフィードバックすることです。このサービスには以下の機能があります。
- 静的なピアが正しく応答しているかどうかのテスト
- メールサーバが履歴メッセージ要求に応答するかどうかをテストします。このサービスは、指定されたチャットルーム（デフォルトは#status）にある1つのメッセージのリクエストを、指定された時間内（デフォルトは過去24時間）にメールサーバに送信し、メールサーバがリクエストメッセージに対する確認応答（リクエストのハッシュ値を照合に使用）を返せば成功します。

### その他
- `CD ~/status-go/_assets/compose/mailserver`
- `ls -l`
- `cat docker-compose.yml`
- `make start` - Starts the status-go-mailserver container.
- `make stop` - Stops the container.
- `make show` - Shows you current status of the container.
- `make logs` - Shows you logs of the container.
- `make config` - Creates ${DATA_PATH}/config.json with your Public IP.
- `make enode` - Shows enode:// address of the container.
- `make enode-qr` - Shows enode:// address using a QR code.

- `docker ps`
- `systemctl status ufw`
