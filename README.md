# Getting started with Elasticsearch

## 環境構築

```sh
# 起動
docker-compose up -d

# 起動確認
curl -XGET "http://localhost:9200/_cat/health?v"
```

ElasticsearchへのアクセスはREST APIを介して行う。

## インデックス

RDBのデータベース＆テーブルに該当する。

### 作成

RDBではカラム定義も一緒に行うが、箱だけ作ることも可能。

ドキュメントを登録すると勝手にカラムが定義される。

事前に定義することも可能。（kuromojiなどを使う場合は事前定義が必須）

ただし、カラムの登録は自由だが変更ができない模様。その場合は、新しくインデックスを作成する。

```sh
curl -XPUT "http://localhost:9200/customer?pretty"
```

### 確認

```sh
curl -XGET "http://localhost:9200/_cat/indices?v"
```

### 削除

```sh
# インデックス指定
curl -XDELETE "http://localhost:9200/customer?pretty"

# すべて削除
curl -XDELETE "http://localhost:9200/*"
```

## ドキュメント

インデックス内に登録するデータをドキュメントと呼ぶ。

RDBのレコードに該当する。

### 登録

ホスト/インデックス/タイプのように指定する。

以前のESでは複数タイプを登録できたようだが、現行ではできない。

そのため以前の記事ではタイプがRDBのテーブルに該当すると記載されていた。

```sh
curl -H "Content-Type: application/json" \
     -XPOST "http://localhost:9200/customer/_doc?pretty" \
     -d '{"customer": "John Doe"}'
```

### 取得

```sh
curl -XGET "http://localhost:9200/customer/_doc/1?pretty"
```

### 削除

```sh
curl -XDELETE "http://localhost:9200/customer/_doc/1?pretty"
```

### 大量登録

こちらから [accounts.json](https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json?raw=true) のデータをダウンロードして登録する。

```sh
curl -H "Content-Type: application/json" \
     -XPOST "http://localhost:9200/bank/_bulk?pretty&refresh" \
     --data-binary "@accounts.json"
```

## 検索

Queryを駆使することで柔軟な検索が可能になる。

### 特定のフィールド検索

スペースで区切ることでOR検索が可能。

```sh
curl -H "Content-Type: application/json" -XGET "http://localhost:9200/bank/_search?pretty" -d '
{
  "query": { "match": { "address": "mill lane" } }
}'
```

### 複雑な検索

Boolean queryを使用すると複雑な検索も可能。

それぞれがTrueになるドキュメントを取得する。

詳細は[こちら](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html)

```sh
curl -H "Content-Type: application/json" -XGET "http://localhost:9200/bank/_search?pretty" -d '
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}'
```

範囲指定は以下のようにする。

```sh
curl -H "Content-Type: application/json" -XGET "http://localhost:9200/bank/_search?pretty" -d '
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}'
```

### グルーピング

```sh
curl -H "Content-Type: application/json" -XGET "http://localhost:9200/bank/_search?pretty" -d '
{
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}'
```

### ソート

```sh
curl -H "Content-Type: application/json" -XGET "http://localhost:9200/bank/_search?pretty" -d '
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}'
```

RDBのようなOFFSET、LIMITも可能。

```sh
curl -H "Content-Type: application/json" -XGET "http://localhost:9200/bank/_search?pretty" -d '
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ],
  "from": 10,
  "size": 10
}'
```
