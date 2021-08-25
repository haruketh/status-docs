# コードベース ストラクチャー ガイドライン

## アプリの起動

このアプリは、Nix環境でバンドルされて実行されます。ローカルで実行する方法については、[こちら](https://status.im/technical/build_status/)をご覧ください。

## ビジネスロジックモジュール

各ビジネスロジックモジュールは、module directoryで管理されます。

**モジュール・ディレクトリの外で要求できるのは、モジュールのcoreとdbの名前空間だけです。その他の名前空間は、モジュール・ディレクトリの外では決して必要とされません。**

coreとdbの名前空間以外のモジュール内のコードをどのように編成するかについては、厳密な構造はありません。
```
- events.cljs
- subs.cljs
- notifications
    - core.cljs
- init
    - core.cljs
- accounts
    - core.cljs
    - db.cljs
- node
    - core.cljs
    - db.cljs
    - config.cljs
- ui
    - screens
        - login
            - views.cljs
```

理由とその方法

現在、多くのビジネスロジックがutilsやscreens名前空間に紛れ込んでいるため、専用のトップレベルディレクトリに移動させる必要があります。これにより、アプリのコア・ロジックがよりアクセスしやすく、整理されます。


### core.cljs

Core 名前空間には、モジュールの外（イベントや他のモジュール）で呼び出される関数のみを含める必要があります。

- fx イベントや他のモジュールから呼び出される関数を生成する

```
(def get-current-account
    module.db/get-current-account)

(defn set-current-account [{db :db :as cofx}]
    {:db (module.db/set-current-account db)})
```

### db.cljs

- fxが生成する関数とサブスクリプションが使用するゲッター関数とセッター関数を含むこと。
- 他のモジュールから呼び出されるDBロジック

根拠

これらのガイドラインは、db.cljsの名前空間を、dbのレイアウトに変更を加える際の場所とし、機能を追加/リファクタリングする際の破壊的な変更を最小限にします。

## re-frame

### Subscriptions

- すべてのサブスクリプションは、単一の `status-im.subs` 名前空間で定義されなければなりません。
- サブスクリプションは、他のサブスクリプションのみを購読し、app-db自体を購読してはいけません。
- ルートキーのサブスクリプションには、`reg-root-key-sub` を使用する必要があります。

### イベント

- すべてのイベントは、`core.cljs`の名前空間で定義されなければなりません。
- イベントは必ず `fx/defn` マクロで宣言しなければなりません。

## テストフロー

- すべてのPRは、Github上の[Pipeline for QA](https://github.com/status-im/status-react/projects/7)プロジェクトの「レビュー」欄に自動的に入ります。ここはQAと開発者の交流のためのメインボードです。
- PRが少なくとも1つの承認を得た後、"E2E Tests" 欄に移されるべきです。PRによっては、複数の人の承認が必要なものもあります。
- E2E Tests "欄のすべてのPRに対してクリティカルパステストが自動的に実行されます。
- E2Eテストが合格した場合
  - PRが手動QAを必要としない場合（小さな変更やアプリに関係しない変更の場合） - e2eテストに合格した後、マージできます（Github UIを使ってマージしないで、PRのマージに関するセクションを確認してください）。
  - 手動でのQAが必要な場合は、「request manual qa」というラベルを追加します。QAチームが処理します（概要とテストの指示が明確であることを確認してください）。QAチームは結果を確認し、異なるプラットフォームでPRをテストし、3つのラベルのうち1つを割り当てます。
    - 問題がない場合-「TESTED-OK」というラベルを付け、マージすることができます。
    - 問題がある場合は、「TESTED-ISSUES」というラベルを付け、QAからバグについてのコメントをもらいます。
    - 修正や議論の後、このプロセスを繰り返します。
  - 手動でのQAは必要ないが、すべてのテストが通らない場合は、@churikや@SerhyにPingして、失敗したE2Eテストが無関係ではないことを確認することができます。

## デバッグログを有効にする

`log/debug`への呼び出しは、デフォルトではコンソールに出力されません。これは、アプリの "Advanced settings "で有効にできます。

## Translations

このアプリは、システムロケールに依存して、[サポートされている言語のリスト](https://github.com/status-im/status-react/blob/bda73867471cf2bb8a68b1cc27c9f94b92d9a58b/src/status_im/i18n_resources.cljs#L9)から言語を選択します。システムロケールがサポートされていない場合は、英語にフォールバックします。

翻訳の[管理](https://translate.status.im/)にはLokaliseアプリを使用しています。翻訳にキーを追加/削除する必要がある場合は、`en.json`を変更するだけで済みます。キーがない場合はen.jsonにフォールバックします。実際の翻訳はLokaliseによって追加されます。

## re-frisk

re-friskは、弊社のAndrey（@flexsurfer）が開発した、状態を可視化するツールです。re-friskを起動するには、以下のコマンドを実行します。
```
$ yarn shadow-cljs run re-frisk-remote.core/start
```
または、makeを使うこともできます。
```
$ make run-re-frisk
```
http://localhost:4567 サーバーが起動します。最初は「接続されていません」と表示されるかもしれません。心配せずに、アプリを使い始めてください。イベントと状態が表示されます。

## 承認されたPRのマージ

マージにはGithubのUIを使いません。代わりに `./scripts/merge-pr.sh` を使って署名し、PR を `develop` にマージします。まず、[自分のコミットにGPG署名](https://github.com/status-im/status-react/blob/develop/STARTING_GUIDE.md#configure-gpg-keys-for-signing-commits)を有効にする必要があります。

コミットが検証され、PRが承認されると、次のようにスクリプトを実行できます。
```
$ git checkout develop
$ git fetch
$ git reset --hard origin/develop
$ ./scripts/merge-pr.sh 11370
```
11370をあなたのPR IDに置き換えてください。

## リリースプロセス

TODO(shivekkhurana)。リリースノートのコンパイル

---
Last commit on Feb 12, 2021

オリジナルファイル　https://github.com/status-im/status-react/blob/711389365c00ae859a19523ed0ceb9400adb7535/doc/codebase-structure-and-guidelines.md
