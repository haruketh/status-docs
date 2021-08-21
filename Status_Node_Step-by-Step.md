# Statusノード　簡単セットアップ手順

Statusノードを30分以内に動かすことができます。あなたがしなければならないのは、Dockerイメージを実行するためにいくつかのコマンドを入力することです。

Dockerイメージを実行するためのLinuxマシンのデプロイが困難な場合は、AWS EC2クラウド・インスタンスのデプロイ方法を[こちら](https://status.im/technical/others/deploy_ec2_instance.html)で紹介していますが、ガイドラインに基づいて他のクラウド・プロバイダーで独自に設定することもできます。このガイドでは、お使いのデバイスでTCPおよびUDPのポート30303が正しく開かれていることを前提としています。


## 1. Dockerをインストールして有効化する

1. `sudo apt-get update`
2. `sudo apt install docker.io`
3. `sudo systemctl start docker`
4. `sudo systemctl enable docker`
5. `docker --version`
6. `sudo usermod -aG docker $USER`
7. `newgrp docker`

![](https://status.im/technical/status_node_step_by_step/docker.png)


## 2. docker-composeをインストールして有効化する

1. `sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
2. `sudo chmod +x /usr/local/bin/docker-compose`
3. `docker-compose --version`

![](https://status.im/technical/status_node_step_by_step/docker-compose.png)


## 3. make をインストールする

1. `sudo apt install make`

![](https://status.im/technical/status_node_step_by_step/make.png)


## 4. jq をインストールする

1. `sudo apt-get install jq`

![](https://status.im/technical/status_node_step_by_step/jq.png)


## 5. Status-go をクローンする

1. `git clone https://github.com/status-im/status-go.git`
2. `cd status-go/`

![](https://status.im/technical/status_node_step_by_step/clone-status-go.png)


## 6. Statusノードを実行する

1. `make run-mailserver-docker`

![](https://status.im/technical/status_node_step_by_step/run-a-status-node.png)


## 7. ヘルスチェック

1. `export DATA='{"jsonrpc":"2.0","method":"admin_peers","params":[],"id":1}'`
2. `curl -s -H 'content-type: application/json' -d $DATA localhost:8545 | jq .`

もしピアが張れていたら、
`curl -s -H 'content-type: application/json' -d "$DATA" localhost:8545 | jq -r '.result[].network.remoteAddress'`
でIPアドレスが返されます。

![](https://status.im/technical/status_node_step_by_step/health-check.png)


## 8. Waku envelopes をチェックする

1. `curl -s localhost:9090/metrics | grep '^waku_envelopes_received_total'`

![](https://status.im/technical/status_node_step_by_step/waku-envelopes.png)


# トラブルシューティング

### config.json はどこにありますか？

設定ファイルは `/var/tmp/status-go-mail/` にあります。


### clusterConfigの確認方法を知りたい

`jq '.ClusterConfig.BootNodes' /var/tmp/status-go-mail/config.json` を実行してください。


### Statusノードのバージョンを確認したい

`docker ps` を実行してイメージ名をチェックしてください。


### Dockerイメージの更新はどうしたらいいですか？

1. 古いDockerイメージを削除します　`make clean-mailserver-docker`
2. イメージをクリーンアップします　`docker image prune -f -a`
3. 再実行します　`make run-mailserver-docker`


### ピアの情報を確認する方法は？

`curl -s localhost:9090/metrics | grep '^p2p_peers’` を実行してください。


### Dockerを実行するためのパーミッションがありません

すべてのコマンドにsudoを使用せずに、ユーザが適切にdockerコマンドを使用できるようにします。

1. `sudo usermod -aG docker $USER`
2. `newgrp docker`


### $PATHに'docker-compose'がないときはインストールしてください。

[docker-compose](https://docs.docker.com/compose/) をインストールしてください。

1. `sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`
2. `sudo chmod +x /usr/local/bin/docker-compose`
3. `docker-compose --version`


### $PATHに'jq'がないときはインストールしてください。

`sudo apt-get install jq` を実行して [jq](https://stedolan.github.io/jq/) をインストールしてください。

