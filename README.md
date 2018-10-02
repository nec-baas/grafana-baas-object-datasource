NEC BaaS datasource plugin for Grafana
======================================

概要
----

Grafana から NEC モバイルバックエンド基盤(NEC BaaS) に REST API でアクセスして
データを取得するための  Datasource plugin です。

インストール
------------

Grafana の data/plugin ディレクトリに "baas-grafana-datasource" ディレクトリを作成し、
本ディレクトリ以下の全ファイルを前記ディレクトリにインストールしてください。

データソースの設定
-------------------

Grafana の設定画面から "NEC BaaS" データソースを追加してください。
設定項目は以下の通りです。

* HTTP URL: BaaS API サーバのベース URI を指定してください(例: "https://baas.example.com/api")
* Tenant ID: BaaSテナントIDを指定してください。
* App ID: BaaSアプリケーションIDを指定してください。
* App/Master Key: BaaSアプリケーション/マスターキーを指定してください。

ユーザ認証が必要な場合は、Basic Auth を指定して User/Password を入力してください。
なお、Basic Auth を使用するためには、BaaS Server v7.5.0 beta3 以上が必要です。

Dashboard / クエリ条件
-----------------------

Dashboardを作成し、Data Source に上記で作成したデータソースを指定してください。

クエリ条件は以下のように指定してください。

    bucketName.fieldName

* bucketName: BaaS JSON Object Storage のバケット名を指定します。
* fieldName: JSONフィールド名を指定します。

複数のクエリを指定することができますが、全クエリの bucketName はすべて同一でなければなりません。

JSON の深い階層のデータを取得する場合は、fieldName にキー名(配列の場合は要素番号)を '.' で連結して指定することができます。

例えば以下のようなデータがバケット "bucket1" にあるとき、

    // 対象データ
    { payload: [ { temperature: 26.5, timestamp: "2018-01-01T00:00:00.000Z" } ], createdAt: "2018-06-29T00:00:00.000Z" }
    
上記データから temperature の値を抽出する場合は、クエリ条件には以下のように指定します。

    bucket1.payload.0.temperature

クエリオプション
----------------

クエリ条件の末尾に、以下のオプションを追加することができます。
各オプションを組み合わせて指定できますが、指定順序は説明順序どおりでなければなりません。

複数のクエリ条件を指定した場合には、先頭のクエリ条件のオプションのみが使用されます。

### MongoDB クエリ式指定

MongoDB のクエリ式を指定することができます。
クエリ式は、"?" とクエリ JSON 文字列で指定してください。

以下に例を示します。

    bucket1.temperature?{"sensor":"s1"}

### タイムスタンプフィールド指定

デフォルトでは、タイムスタンプの値は "createdAt" または "updatedAt" の値から自動的に抽出されます。

これ以外のフィールドから取得したい場合は、クエリ条件の末尾に "@" とタイムスタンプフィールド名を指定してください。
タイムスタンプフィールドが JSON の深い階層にある場合、fieldName と同様に '.' で連結して指定することができます。
上記の例では以下のように指定することができます。

    bucket1.payload.0.temperature@payload.0.timestamp

なお、日付文字列は JavaScript の Date.parse() でパースできるフォーマットでなければなりません。

さらに、日時範囲指定でクエリを行う場合、日時は ISO 8601 形式、具体的には "YYYY-MM-DDTHH:MM:SS.sssZ" で
なければなりません。これは Grafana が指定する日時範囲が ISO 8601 形式の文字列での指定となっており、
本プラグインはこれをそのまま MongoDB の検索式 ($gte および $lte) にマップするためです。
