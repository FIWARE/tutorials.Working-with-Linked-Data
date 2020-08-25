[![FIWARE Banner](https://fiware.github.io/tutorials.Working-with-Linked-Data/img/fiware.png)](https://www.fiware.org/developers)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Relationships-Linked-Data.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://nexus.lab.fiware.org/repository/raw/public/badges/stackoverflow/fiware.svg)](https://stackoverflow.com/questions/tagged/fiware)
[![NGSI LD](https://img.shields.io/badge/NGSI-LD-d6604d.svg)](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.03.01_60/gs_cim009v010301p.pdf)
[![JSON LD](https://img.shields.io/badge/JSON--LD-1.1-f06f38.svg)](https://w3c.github.io/json-ld-syntax/)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

このチュートリアルでは、FIWARE ユーザに、**リンクト・データ**に基づいてシステムを設計および設計し、リンクト・データの
コンテキストをプログラムで変更する方法を学習します。このチュートリアルは、同等の
[NGSI-v2 チュートリアル](https://github.com/FIWARE/tutorials.Accessing-Context/)から得られた知識を拡張し、コンテキスト・データ
を取得および変更するために、[NGSI-LD](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.03.01_60/gs_cim009v010301p.pdf)
対応 [Node.js](https://nodejs.org/) [Express](https://expressjs.com/) アプリケーションでコードを記述する方法をユーザが理解
できるようにします。これにより、コマンドラインを使用して cUrl コマンドを呼び出す必要がなくなります。

このチュートリアルでは、主に Node.js で記述されたコードについて説明しますが、[cUrl](https://ec.haxx.se/) コマンドを実行すること
で結果の一部を確認できます。同じコマンドの [Postman ドキュメント](https://fiware.github.io/tutorials.Working-with-Linked-Data)
も利用できます。

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/644a1df1e2d226da65ef)

## コンテンツ

<details>
<summary><strong>詳細</strong></summary>

-   [リンクト・データ・エンティティの操作](#working-with-linked-data-entities)
    -   [在庫管理システム内のリンクト・データ・エンティティ](#linked-data-entities-within-a-stock-management-system)
    -   [このチュートリアルの学習目標](#the-teaching-goal-of-this-tutorial)
-   [在庫管理フロントエンド](#stock-management-frontend)
-   [前提条件](#prerequisites)
    -   [Docker](#docker)
    -   [Cygwin](#cygwin)
-   [アーキテクチャ](#architecture)
-   [起動](#start-up)
-   [プログラムによるリンクト・データのアクセス](#traversing-linked-data-programmatically)
    -   [リンクト・データの読み取り](#reading-linked-data)
        -   [ライブラリの初期化](#initializing-the-library)
        -   [既知のストアの取得](#retrieve-a-known-store)
    -   [リンクト・データの集計と移動](#aggregating-and-traversing-linked-data)
        -   [既知のストア内の棚を検索](#find-shelves-within-a-known-store)
        -   [棚から在庫品を取得](#retrieve-stocked-products-from-shelves)
        -   [選択したシェルフの製品詳細を取得](#retrieve-product-details-for-selected-shelves)
    -   [リンクト・データの更新](#updating-linked-data)
        -   [商品を在庫している棚を探す](#find-a-shelf-stocking-a-product)
        -   [棚の状態を更新](#update-the-state-of-a-shelf)
    -   [リンクト・データを使用した相互運用性](#interoperability-using-linked-data)
        -   [代替スキーマを使用したエンティティの作成](#creating-an-entity-using-an-alternate-schema)
        -   [デフォルトのスキーマを使用したエンティティの読み取り](#reading-an-entity-using-the-default-schema)
        -   [代替スキーマを使用したエンティティの読み取り](#reading-an-entity-using-an-alternate-schema)
        -   [エンティティ展開/圧縮 (Expansion/Compaction) の適用](#applying-entity-expansioncompaction)

</details>

<a name="working-with-linked-data-entities"></a>
# リンクト・データ・エンティティの操作

> -   “This is the house that Jack built.
> -   This is the malt that lay in the house that Jack built.
> -   This is the rat that ate the malt<br/> That lay in the house that Jack built.
> -   This is the cat<br/> That killed the rat that ate the malt<br/> That lay in the house that Jack built.
> -   This is the dog that chased the cat<br/> That killed the rat that ate the malt<br/> That lay in the house that
>     Jack built.”
>
> ― This Is the House That Jack Built, Traditional English Nursery Rhyme

NGSI-LD は NGSI-v2 の進化形であるため、NSGI-LD に基づくスマート・ソリューションが、プログラムによるデータ・アクセスに
関する以前の NGSI-v2 [チュートリアル](https://github.com/FIWARE/tutorials.Accessing-Context/)で概説したものと同じ
基本シナリオをカバーする必要があることは驚くべきことではありません。


NGSI-LD リンクト・データは、_Property_ 属性、または _Relationship_ 属性のみとして定義されるデータ属性を制限することに
より、コンテキスト・エンティティの構造をより高度に形式化します。これは、ある _Relationship_ から別の _Relationship_ に
移動するときに、コンテキスト・データ・グラフをより確実にトラバースできることを意味します。システム内のすべての
コンテキスト・データ・エンティティは、コンテキスト・ファイルを参照することで正式に定義される JSON-LD データモデルによって
定義され、このプログラム上の定義により、関連付けられたリンクト・エンティティが存在することが保証されます。

スーパーマーケットの3つの基本的なデータ・アクセス・シナリオを以下に定義します :

-   データの読み取り - たとえば、**Building** エンティティ `urn:ngsi-ld:Building:store001` のすべてのデータを提供します
-   集計 - たとえば、**Building** `urn:ngsi-ld:Building:store001` で販売されている **Products** エンティティを結合し、
    販売する商品を表示します
-   システム内のコンテキストを変更します - たとえば、製品の販売を行います :
    -   **Product** の価格で毎日の販売記録を更新します
    -   **Shelf** エンティティの `numberOfItems` を減らします
    -   販売が発生したことを示す新しいトランザクション・ログ・レコードを作成します
    -   10個未満のオブジェクトが販売されている場合、倉庫でアラートを発生させます
    -   など

さらに高度なシナリオについては、この後のチュートリアルで説明します。

<a name="linked-data-entities-within-a-stock-management-system"></a>
## 在庫管理システム内のリンクト・データ・エンティティ

[以前のチュートリアル](https://github.com/FIWARE/tutorials.Relationships-Linked-Data/) で作成されたスーパーマーケットの
データが Context Broker にロードされます。エンティティ間のリレーションシップは、次のように定義されます:

![](https://fiware.github.io/tutorials.Working-with-Linked-Data/img/entities-ld.png)

**Building**, **Product**, **Shelf**, **StockOrder** エンティティは、デモアプリケーションのフロントエンドにデータを
表示するために使用されます。

<a name="the-teaching-goal-of-this-tutorial"></a>
## このチュートリアルの学習目標

このチュートリアルの目的は、一般的なデータ・アクセス・シナリオをカバーする一連の一般的なコード例を定義および議論する
ことにより、コンテキスト・データのプログラムによるアクセスについて開発者の理解を向上させることです。この目的のために、
単純な Node.js Express アプリケーションが作成されます。

ここでの意図は、Express でアプリケーションを作成する方法をユーザに教えることではありません - 実際、任意の言語を選択
できます。これは、**任意**のサンプル・プログラミング言語を使用してコンテキストを変更し、ビジネス・ロジックの目標を
達成する方法を示すためのものです。

明らかに、プログラミング言語の選択は、以下のコードを読むときのあなた自身のビジネスニーズに依存します。これを念頭に
置いて、必要に応じて Node.js を独自のプログラミング言語に置き換えてください。

<a name="stock-management-frontend"></a>
# 在庫管理フロントエンド

デモ用のすべてのコード Node.js Express は、GitHub リポジトリ内の `ngsi-ld` フォルダ内にあります。
[在庫管理の例](https://github.com/FIWARE/tutorials.Step-by-Step/tree/master/context-provider)。
アプリケーションは次の URL で実行されます :

-   `http://localhost:3000/app/store/urn:ngsi-ld:Building:store001`
-   `http://localhost:3000/app/store/urn:ngsi-ld:Building:store002`
-   `http://localhost:3000/app/store/urn:ngsi-ld:Building:store003`
-   `http://localhost:3000/app/store/urn:ngsi-ld:Building:store004`

> :information_source: **ヒント** さらに、コンテナ・ログを追跡するか、Web ブラウザで `localhost:3000/app/monitor`
> の情報を表示することで、最近のリクエストのステータスを自分で監視することもできます。
>
> ![FIWARE Monitor](https://fiware.github.io/tutorials.Working-with-Linked-Data/img/monitor.png)

<a name="prerequisites"></a>
# 前提条件

<a name="docker"></a>
## Docker

物事を単純にするために、両方のコンポーネントが [Docker](https://www.docker.com) を使用して実行されます。**Docker** は、
さまざまコンポーネントをそれぞれの環境に分離することを可能にするコンテナ・テクノロジです。

-   Windows に Docker をインストールするには、[こちら](https://docs.docker.com/docker-for-windows/)の指示に従ってください
-   Mac に Docker をインストールするには、[こちら](https://docs.docker.com/docker-for-mac/)の指示に従ってください
-   Linux に Docker をインストールするには、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行するためのツールです。
[YAML ファイル](https://raw.githubusercontent.com/fiware/tutorials.Working-with-Linked-Data/master/docker-compose/orion-ld.yml)
を使用して、アプリケーションに必要なサービスを設定します。これは、すべてのコンテナ・サービスを単一のコマンドで起動
できることを意味します。Docker Compose は、Docker for Windows および Docker for Mac の一部としてデフォルトでインストール
されますが、Linux ユーザは[こちら](https://docs.docker.com/compose/install/)にある手順に従う必要があります。

<a name="cygwin"></a>
## Cygwin

簡単な bash スクリプトを使ってサービスを開始します。Windows ユーザは、Windows 上の Linux ディストリビューションに
似たコマンドライン機能を提供するために [cygwin](http://www.cygwin.com/) をダウンロードするべきです。

<a name="architecture"></a>
# アーキテクチャ

デモ・スーパーマーケット・アプリケーションは、準拠している Context Broker と NGSI-LD の呼び出しを送受信します。
NGSI-LD インターフェースは、[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) の実験バージョンで
利用できるため、デモ・アプリケーションは1つの FIWARE コンポーネントのみを使用します。

現在、Orion Context Broker は、オープンソース [MongoDB](https://www.mongodb.com/) テクノロジに依存して、保持している
コンテキスト・データの永続性を維持しています。外部ソースからコンテキスト・データをリクエストするために、シンプルな
Context Provider NGSI proxy  も追加されました。 コンテキストを視覚化し、対話するために、簡単な Express
アプリケーションを追加します。 したがって、アーキテクチャは次の3つの要素で構成されます :

-   [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/gitlab/NGSI-LD/NGSI-LD/raw/master/spec/updated/full_api.json)
    を使ってリクエストを受け取る [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/)
-   基礎となる [MongoDB](https://www.mongodb.com/) データベース :
    -   データ・エンティティ、サブスクリプション、レジストレーションなどのコンテキスト・データ情報を保持するために
        Orion Context Broker によって使用されます
-   **在庫管理フロントエンド** は、次のことができます
    -   ストア情報を表示
    -   各ストアで購入できる製品を表示
    -   ユーザが製品を "購入" して在庫数を減らすことを許可

要素間のすべての対話は HTTP リクエストによって開始されるため、エンティティはコンテナ化され、公開されたポートから実行
できます。

![](https://fiware.github.io/tutorials.Working-with-Linked-Data/img/architecture.png)

**Context Provider NGSI proxy** に必要な設定情報は、関連する `orion-ld.yml` ファイルのサービス・セクション
で確認できます :

```yaml
tutorial:
    image: fiware/tutorials.context-provider
    hostname: context-provider
    container_name: fiware-tutorial
    networks:
        - default
    expose:
        - "3000"
    ports:
        - "3000:3000"
    environment:
        - "DEBUG=tutorial:*"
        - "WEB_APP_PORT=3000"
        - "NGSI_VERSION=ngsi-ld"
        - "CONTEXT_BROKER=http://orion:1026/ngsi-ld/v1"
```

`tutorial` コンテナは、次のような環境変数によって駆動されます :

| キー           | 値                             | 説明                                                                                        |
| -------------- | ------------------------------ | ------------------------------------------------------------------------------------------- |
| DEBUG          | `tutorial:*`                   | ロギングに使用されるデバッグ・フラグ                                                        |
| WEB_APP_PORT   | `3000`                         | データを表示するために Context Provider NGSI proxy および Web アプリが使用するポート        |
| CONTEXT_BROKER | `http://orion:1026/ngsi-ld/v1` | コンテキストを更新するために接続する Context Broker の URL                                  |

YAML ファイルで説明されている他の `tutorial` コンテナ設定値は、チュートリアルのこのセクションでは使用しません。

MongoDB および Orion Context Broker の構成情報は、[以前のチュートリアル](https://github.com/FIWARE/tutorials.Relationships-Linked-Data/)
で説明されています

<a name="start-up"></a>
# スタートアップ

すべてのサービスは、リポジトリ内で提供される
[services](https://github.com/FIWARE/tutorials.Working-with-Linked-Data/blob/master/services) Bash スクリプトを
実行して、コマンドラインから初期化できます。以下のようにコマンドを実行して、リポジトリのクローンを作成して必要な
イメージを作成してください :

```bash
git clone https://github.com/FIWARE/tutorials.Working-with-Linked-Data.git
cd tutorials.Working-with-Linked-Data

./services orion
```

> **注:** クリーンアップして最初からやり直す場合は、次のコマンドで実行できます :
>
> ```
> ./services stop
> ```

---

<a name="traversing-linked-data-programmatically"></a>
# プログラムによるリンクト・データのアクセス

`http://localhost:3000/app/store/urn:ngsi-ld:Building:store001` に移動して、動作中のスーパーマーケット・データ
・アプリケーションを表示して操作します。

![](https://fiware.github.io/tutorials.Working-with-Linked-Data/img/store.png)

<a name="reading-linked-data"></a>
## リンクト・データの読み取り

議論中のコードは、
[Git リポジトリ](https://github.com/FIWARE/tutorials.Step-by-Step/blob/master/context-provider/controllers/ngsi-ld/store.js)
の `ngsi-ld/store` コントローラ内にあります。

<a name="initializing-the-library"></a>
### ライブラリの初期化

通常どおり、HTTP アクセスのコードは、スーパーマーケット・アプリケーション自体のビジネス・ロジックから分離できます。
下位レベルの呼び出しはライブラリ・ファイルに配置されているため、コードベースが簡素化されます。これは、示されている
ようにファイルのヘッダに含める必要があります。いくつかの定数も必要です - スーパーマーケット・データの場合、`LinkHeader`
はデータモデル JSON-LD コンテキストの場所を`https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld`
として定義するために使用されます。

```javascript
const ngsiLD = require("../../lib/ngsi-ld");

const LinkHeader =
    '<https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json">';
```

<a name="retrieve-a-known-store"></a>
### 既知のストアの取得

この例では、特定の **Store** エンティティのコンテキスト・データを読み取り、結果を画面に表示します。エンティティ・
データの読み取りは、`ngsiLD.readEntity()` メソッドを使用して実行できます。これにより、GET リクエストの URL が入力され、
必要な HTTP 呼び出しが非同期的に行われます :

```javascript
async function displayStore(req, res) {
    const store = await ngsiLD.readEntity(
        req.params.storeId,
        { options: "keyValues" },
        ngsiLD.setHeaders(req.session.access_token, LinkHeader)
    );

    return res.render("store", { title: store.name, store });
}
```

上記の関数はリクエストの一部としていくつかの標準 HTTP ヘッダも送信します - これらは `setHeaders()` 関数で定義されます。

NGSI-LD ベースのシステム内では、通常のデフォルトの HTTP ヘッダには、JSON-LD コンテキストを送信するための `Link` ヘッダと、
リクエストを `application/ld+json` として識別するための `Content-Type` ヘッダが含まれます (注 NGSI-LD は JSON-LD
のサブセットであるため、すべての NGSI-LD リクエストは有効な JSON_LD です)。OAuth2 セキュリティを有効にするために、
`X-Auth-Token` などの他の追加ヘッダを付加できます。

```javascript
function setHeaders(accessToken, link, contentType) {
    const headers = {};
    if (accessToken) {
        headers["X-Auth-Token"] = accessToken;
    }
    if (link) {
        headers.Link = link;
    }
    if (contentType) {
        headers["Content-Type"] = contentType || "application/ld+json";
    }
    return headers;
}
```

`lib/ngsi-ld.js` ライブラリ・ファイル内で、`BASE_PATH` は Orion Context Broker の場所を定義し、データ・エンティティの
読み取りは、適切なヘッダを渡す非同期 HTTP GET リクエストの単なるラッパーです。

```javascript
const BASE_PATH = process.env.CONTEXT_BROKER || "http://localhost:1026/ngsi-ld/v1";

function readEntity(entityId, opts, headers = {}) {
    return request({
        qs: opts,
        url: BASE_PATH + "/entities/" + entityId,
        method: "GET",
        headers,
        json: true
    });
}
```

同等の cUrl ステートメントは以下のとおりです :

```console
curl -G -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:store001/' \
-H 'Link: <https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/ld+json' \
-d 'type=Building' \
-d 'options=keyValues'
```

<a name="aggregating-and-traversing-linked-data"></a>
## リンクト・データの集計と移動

営業時間に情報を表示するには、ストア内で見つかった製品に関する情報を見つける必要があります。
データ・エンティティ・ダイアグラムから、次のことを確認できます。

-   **Building** エンティティは、関連する **Shelf** 情報を `furniture` _Relationship_ 内に保持します
-   **Shelf** エンティティは、関連する **Product** 情報を `stocks` _Relationship_ 内に保持します
-   製品は、**Product** エンティティ自体の _Property_ 属性として `name` と `price` を保持します

したがって、`displayTillInfo()` メソッドのコードは次のステップで構成されます。

1.  Context Broker に _既知のストア内の棚を見つけるようにリクエストします_

2.  結果を `id` パラメータにして、_棚から在庫のある製品を取得しまするために、_ Context Broker に2回目のリクエストを
    行います

3.  結果を `id` パラメータにして、_選択した棚の製品詳細を取得するために、_ Context Broker に3番目のリクエストを行います

データベースの結合に慣れているユーザにとっては、このような一連のリクエストを強制されるのは奇妙に思えるかもしれませんが、
大規模な分散セットアップではスケーラビリティの問題/懸念があるため必要です。 NGSI-LD では Direct join リクエストはできません。

<a name="find-shelves-within-a-known-store"></a>
### 既知のストア内の棚を検索

既知の **Building** エンティティの `furniture` 属性にアクセスするには、`attrs` パラメータを使用して `keyValues`
リクエストを作成します。

```javascript
const building = await ngsiLD.readEntity(
    req.params.storeId,
    {
        type: "Building",
        options: "keyValues",
        attrs: "furniture"
    },
    ngsiLD.setHeaders(req.session.access_token, LinkHeader)
);
```

同等の cUrl ステートメントは以下のとおりです :

```console
curl -G -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:store001/' \
-H 'Link: <https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/ld+json' \
-d 'type=Building' \
-d 'options=keyValues' \
-d 'attrs=furniture' \
```

レスポンスは、さらに操作できる `furniture` 属性を含む JSON オブジェクトです。

<a name="retrieve-stocked-products-from-shelves"></a>
### 棚から在庫品を取得

一連の **Shelf** エンティティを取得するには、`ngsiLD.listEntities()` 関数が呼び出され、`id` パラメータを使用して
フィルタ処理されます。`id` は、上記のリクエストから取得したカンマ区切りのリストです。

```javascript
let productsList = await ngsiLD.listEntities(
    {
        type: "Shelf",
        options: "keyValues",
        attrs: "stocks,numberOfItems",
        id: building.furniture.join(",")
    },
    ngsiLD.setHeaders(req.session.access_token, LinkHeader)
);
```

`listEntities()` は `lib/ngsi-ld.js` ライブラリ・ファイル内の別の関数です。

```javascript
function listEntities(opts, headers = {}) {
    return request({
        qs: opts,
        url: BASE_PATH + "/entities",
        method: "GET",
        headers,
        json: true
    });
}
```

同等の cUrl ステートメントは以下のとおりです :

```console
curl -G -X GET 'http://localhost:1026/ngsi-ld/v1/entities/' \
-H 'Link: <https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/ld+json' \
-H 'Accept: application/json' \
-d 'type=Shelf' \
-d 'options=keyValues' \
-d 'attrs=stocks,numberOfItems' \
-d 'id=urn:ngsi-ld:Shelf:unit001,urn:ngsi-ld:Shelf:unit002,urn:ngsi-ld:Shelf:unit003'
```

レスポンスは、**Shelf** エンティティの JSON 配列であり、さらに操作できる `stocks` 属性として含まれています。
以下のコードは、後で使用するために IDs を抽出します。

```javascript
const stockedProducts = [];

productsList = _.groupBy(productsList, e => {
    return e.stocks;
});
_.forEach(productsList, (value, key) => {
    stockedProducts.push(key);
});
```

<a name="retrieve-product-details-for-selected-shelves"></a>
### 選択したシェルフの製品詳細を取得

一連の **Product** エンティティを取得するために、`ngsiLD.listEntities()` 関数が再度呼び出され、`id` パラメータを
使用してフィルタ処理されます。`id` は、上記のリクエストから取得したカンマ区切りのリストです。

```javascript
let productsInStore = await ngsiLD.listEntities(
    {
        type: "Product",
        options: "keyValues",
        attrs: "name,price",
        id: stockedProducts.join(",")
    },
    headers
);
```

同等の cUrl ステートメントは以下のとおりです :

```console
curl -G -X GET 'http://localhost:1026/ngsi-ld/v1/entities/' \
-H 'Link: <https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/ld+json' \
-H 'Accept: application/json' \
-d 'type=Product' \
-d 'options=keyValues' \
-d 'attrs=name,price' \
-d 'id=urn:ngsi-ld:Product:001,urn:ngsi-ld:Product:003,urn:ngsi-ld:Product:004'
```

レスポンスは、**Product** エンティティの JSON 配列で、画面に表示されます。

<a name="updating-linked-data"></a>
## リンクト・データの更新

<a name="find-a-shelf-stocking-a-product"></a>
### 商品を在庫している棚を探す

一連の **Shelf** エンティティを取得するには、 `ngsiLD.listEntities()` 関数が呼び出されます。修正する前に現在の
コンテキストを取得することが重要です。そのため、`q` パラメータは、正しい製品を含む正しいストアからシェルフを取得する
ためにのみ使用されます。このリクエストは、**Shelf** データモデルが **Building** と **Product** の両方で _relationships_
を保持するように設計されているためにのみ可能です。

```javascript
const shelf = await ngsiLD.listEntities(
    {
        type: "Shelf",
        options: "keyValues",
        attrs: "stocks,numberOfItems",
        q: 'numberOfItems>0;locatedIn=="' + req.body.storeId + '";stocks=="' + req.body.productId + '"',
        limit: 1
    },
    headers
);
```

同等の cUrl ステートメントは以下のとおりです :

```console
curl -G -X GET 'http://localhost:1026/ngsi-ld/v1/entities/' \
-H 'Link: <https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/ld+json' \
-H 'Accept: application/json' \
-d 'type=Shelf' \
-d 'options=keyValues' \
-d 'q=numberOfItems%3E0;locatedIn==%22urn:ngsi-ld:Building:store001%22;stocks==%22urn:ngsi-ld:Product:001%22'
```

<a name="update-the-state-of-a-shelf"></a>
### 棚の状態を更新

エンティティを更新するには、前のリクエストで返された **Shelf** の `id` を使用して PATCH リクエストを作成します。

```javascript
const count = shelf[0].numberOfItems - 1;
await ngsiLD.updateAttribute(
    shelf[0].id,
    { numberOfItems: { type: "Property", value: count } },
    ngsiLD.setHeaders(req.session.access_token, LinkHeader)
);
```

非同期 PATCH リクエストは、`lib/ngsi-ld.js` ライブラリファイル内の `updateAttribute()` 関数にあります。

```javascript
function updateAttribute(entityId, body, headers = {}) {
    return request({
        url: BASE_PATH + "/entities/" + entityId + "/attrs",
        method: "PATCH",
        body,
        headers,
        json: true
    });
}
```

同等の cUrl ステートメントは以下のとおりです :

```console
curl -X PATCH 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Shelf:unit001/attrs' \
-H 'Link: <https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
-d '{ "numberOfItems": { "type": "Property", "value": 10 } }'
```

<a name="interoperability-using-linked-data"></a>
## リンクト・データを使用した相互運用性

リンクト・データの概念を NGSI に導入したことにより、これまでのところ、すべての Context Broker のリクエストの複雑さが
わずかに増加しており、追加の利点はまだ実証されていません。リンクされたデータの背後にある考え方は、データの相互運用性を
改善し、データ・サイロを取り除くことです。

このデモンストレーションとして、異なるスキーマを使用している別のコンテキスト・プロバイダからのコンテキスト・データ・
エンティティを組み込むことを想像してください。日本のコンテキスト・プロバイダは、`name`, `category`, `location` などを
使用するのではなく、漢字に基づいたデータ属性を使用しています。

Core NGSI-LD の `@context` は、`name` = `https://uri.etsi.org/ngsi-ld/name` を定義します。同様に、
`名前` = `https://uri.etsi.org/ngsi-ld/name` を定義でき、属性名と列挙値の代替マッピングを導入します。

2つのシステムがデータ交換用の一意の URI の**共通**システムに同意できる場合、それらのドメイン内でこれらの値をローカルで
自由に再解釈できます。

<a name="creating-an-entity-using-an-alternate-schema"></a>
### 代替スキーマを使用したエンティティの作成

代替の日本語 JSON-LD `@context` ファイルが作成され、外部サーバに公開されました。このファイルは、
`https://fiware.github.io/tutorials.Step-by-Step/japanese-context.jsonld` にあります。 チュートリアル内で使用される
すべての属性名の代替データ・マッピングを見つけることができます。

> **注**: 比較のために、標準のチュートリアル JSON-LD `@context` ファイルはこちらにあります。:
> `https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld`

#### :one: リクエスト:

データ・エンティティを作成する際、日本語の JSON-LD `@context` にマッピングされたすべての URI の短縮名は、リクエストの
ペイロードで自由に使用できます。

以下の例からわかるように、属性名と列挙値 (`ビル` = `Building` など）を全体で使用できます。 NGSI-LD 仕様では、NGSI-LD API
で定義された属性 (つまり、core `@context`) を使用して属性を定義することが義務付けられています。したがって、`id`, `type`,
`Property` などのリクエストの要素は変更されませんが、以下で説明するように、これは回避できます。

日本のコンテキスト・プロバイダは、以下のリクエストを使用して新しい `Building` を作成できます。`Link` ヘッダは、属性名と
列挙の完全な URI を提供する日本語の JSON-LD `@context` ファイルを指します。

```bash
curl -L -X POST 'http://localhost:1026/ngsi-ld/v1/entities/' \
-H 'Content-Type: application/ld+json' \
--data-raw '{
    "id": "urn:ngsi-ld:Building:store005",
    "type": "ビル",
    "カテゴリー": {"type": "Property", "value": ["コマーシャル"]},
    "住所": {
        "type": "Property",
        "value": {
            "streetAddress": "Eisenacher Straße 98",
            "addressRegion": "Berlin",
            "addressLocality": "Marzahn",
            "postalCode": "12685"
        }
    },
    "場所": {
        "type": "GeoProperty",
        "value": {"type": "Point","coordinates": [13.5646, 52.5435]}
    },
    "名前": {"type": "Property","value": "Yuusui-en"},
    "@context":"https://fiware.github.io/tutorials.Step-by-Step/japanese-context.jsonld"
}'
```

この例では、名前とアドレスは単純な文字列として提供されていることに注意してください - JSON-LD は国際化を可能にするために
`@lang` 定義をサポートしますが、これはここでは説明しない高度なトピックです。

<a name="reading-an-entity-using-the-default-schema"></a>
### デフォルトのスキーマを使用したエンティティの読み取り

Context Broker 内では、属性と列挙を参照するために完全な URI が使用されます。異なる属性の短縮名を使用しますが、日本語の
JSON-LD `@context` ファイルは、**Building** エンティティに使用される完全な URI に関する標準のチュートリアル・コンテキスト
に同意します。事実上、同じデータモデルを使用しています。

したがって、日本語データモデルを使用して作成された、新しい **Building** をリクエストし、標準の tutorial JSON-LD
`@context` で指定された短縮名を使用して返すことができます。これは、`Link` ヘッダは、tutorial JSON-LD の`@context`
ファイルを指しています。

#### :two: リクエスト:

```bash
curl -L -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:store005' \
-H 'Content-Type: application/ld+json' \
-H 'Link: <https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
```

#### レスポンス:

レスポンスは、標準の属性名 (`name` や `location` など) である、通常の **Building** エンティティであり、**Building**
`category` の標準列挙 (standard enumeration) も返します。

```jsonld
{
    "@context": "https://fiware.github.io/tutorials.Step-by-Step/tutorials-context.jsonld",
    "id": "urn:ngsi-ld:Building:store005",
    "type": "Building",
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Eisenacher Straße 98",
            "addressRegion": "Berlin",
            "addressLocality": "Marzahn",
            "postalCode": "12685"
        }
    },
    "location": {
        "type": "GeoProperty",
        "value": { "type": "Point", "coordinates": [13.5646, 52.5435] }
    },
    "name": { "type": "Property", "value": "Yuusui-en" },
    "category": { "type": "Property", "value": "commercial" }
}
```

これは、スーパーマーケット・アプリケーションが、基礎となるコードベースを変更せずに新しいビルディングを表示できることを
意味します。データは相互運用可能です。

`http://localhost:3000/app/store/urn:ngsi-ld:Building:store005` に移動して、新しい **Building** を表示できることを
示します :

![](https://fiware.github.io/tutorials.Working-with-Linked-Data/img/store5.png)

<a name="reading-an-entity-using-an-alternate-schema"></a>
### 代替スキーマを使用したエンティティの読み取り

1つの例外を除き、NGSI-LD の `@context` ファイル内に定義された階層はありません。したがって、定義された `@context` は、
既存の **Building** エンティティを読み取り、日本語の `@context` を適用できます。使用する `@context` は、`Link` ヘッダで
提供されます。

#### :three: リクエスト:

```bash
curl -L -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:Building:store003' \
-H 'Content-Type: application/ld+json' \
-H 'Link: <https://fiware.github.io/tutorials.Step-by-Step/japanese-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
```

#### レスポンス:

レスポンスはミックスされています - いくつかの例外を除き、`japanese-context.jsonld` で定義された属性名と列挙を使用
します。NGSI-LD は JSON-lD **ではありません**。コア・コンテキスト (Core Context) は常に、`Link` ヘッダで受信された
コンテキストの後に適用されます。`name` と `location` は予約済みの属性名であるため、デフォルトのコア・コンテキストを
使用して提供されます。

```jsonld
{
    "@context": "https://fiware.github.io/tutorials.Step-by-Step/japanese-context.jsonld",
    "id": "urn:ngsi-ld:Building:store003",
    "type": "ビル",
    "家具": {
        "type": "Relationship",
        "object": ["urn:ngsi-ld:Shelf:unit006", "urn:ngsi-ld:Shelf:unit007", "urn:ngsi-ld:Shelf:unit008"]
    },
    "住所": {
        "type": "Property",
        "value": {
            "streetAddress": "Mühlenstrasse 10",
            "addressRegion": "Berlin",
            "addressLocality": "Friedrichshain",
            "postalCode": "10243"
        },
        "検証済み": { "type": "Property", "value": false }
    },
    "name": { "type": "Property", "value": "East Side Galleria" },
    "カテゴリー": { "type": "Property", "value": "コマーシャル" },
    "location": {
        "type": "GeoProperty",
        "value": { "type": "Point", "coordinates": [13.4447, 52.5031] }
    }
}
```

<a name="applying-entity-expansioncompaction"></a>
### エンティティ展開/圧縮 (Expansion/Compaction) の適用

JSON-LD 内には、ローカル属性名を適用および変更するための標準メカニズムがあります。 Cotext Broker からのレスポンスは、
常に有効な NGSI-LD になります。NGSI-LD は JSON-LD の構造化されたサブセットであるため、JSON として受信したデータを使用
するためにさらに変更を加えることができます。

Core NGSI-LD context をオーバーライドする必要がある場合、レスポンスに追加の展開/圧縮操作を適用して、ローカルで使用する
ために完全に変換された方法でデータを取得できます。

この作業を行うための JSON-LD ライブラリがすでに存在します。

```javascript
const coreContext = require("./jsonld-context/ngsi-ld.json");
const japaneseContext = require("./jsonld-context/japanese.json");
function translateRequest(req, res) {
    request({
        url: BASE_PATH + req.path,
        method: req.method,
        headers: req.headers,
        qs: req.query,
        json: true
    })
        .then(async function(cbResponse) {
            cbResponse["@context"] = coreContext;
            const expanded = await jsonld.expand(cbResponse);
            const compacted = await jsonld.compact(expanded, japaneseContext);
            delete compacted["@context"];
            return res.send(compacted);
        })
        .catch(function(err) {
            return res.send(err);
        });
}
```

#### :four: リクエスト:

リクエストを Context Broker に転送してから展開/圧縮操作を適用する `/japanese` エンドポイントが作成されました。

```bash
curl -L -X GET 'http://localhost:3000/japanese/ngsi-ld/v1/entities/urn:ngsi-ld:Building:store005' \
-H 'Accept: application/json' \
-H 'Link: <https://fiware.github.io/tutorials.Step-by-Step/japanese-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"'
```

#### レスポンス:

展開/圧縮の操作後のレスポンスは、優先属性名 (the preferred attribute names) のすべてを使用するデータです。これは有効な
NGSI-LD では**なくなりました**が、受信したシステムがこの形式のデータをリクエストする場合に役立ちます。

Context Broker にデータを送信する前に、逆の展開/圧縮操作を使用して、この JSON を有効な NGSI-LD ペイロードに変換できる
ことに注意してください。

```json
{
    "識別子": "urn:ngsi-ld:Building:store005",
    "タイプ": "ビル",
    "カテゴリー": { "タイプ": "プロパティ", "値": "コマーシャル" },
    "住所": {
        "タイプ": "プロパティ",
        "値": {
            "addressLocality": "Marzahn",
            "addressRegion": "Berlin",
            "postalCode": "12685",
            "streetAddress": "Eisenacher Straße 98"
        }
    },
    "場所": {
        "タイプ": "ジオプロパティ",
        "値": { "タイプ": "Point", "座標": [13.5646, 52.5435] }
    },
    "名前": { "タイプ": "プロパティ", "値": "Yuusui-en" }
}
```

#### :arrow_forward: Video: JSON-LD 圧縮と展開 (Compaction & Expansion)

[![](https://fiware.github.io/tutorials.Step-by-Step/img/video-logo.png)](https://www.youtube.com/watch?v=Tm3fD89dqRE "JSON-LD Compaction & Expansion")

上の画像をクリックして、`@context` と相互運用性に関するビデオ JSON-LD の圧縮と展開をご覧ください。

---

## License

[MIT](LICENSE) © 2020 FIWARE Foundation e.V.
