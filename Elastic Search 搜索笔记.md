# Elastic Search 搜索笔记

**一、 query string search**

1. 数据查询，返回所有记录。直接`GET`。

```bash
$ curl 'localhost:9200/accounts/person/_search'
```

返回结果的 `took`字段表示该操作的耗时（单位为毫秒），`timed_out`字段表示是否超时，`hits`字段表示命中的记录，`total`表示返回记录数，`max_score`表示最高的匹配程度。`score`按降序排列。

```bash
{
  "took":2,
  "timed_out":false,
  "_shards":{"total":5,"successful":5,"failed":0},
  "hits":{
    "total":2,
    "max_score":1.0,
    "hits":[
      {
        "_index":"accounts",
        "_type":"person",
        "_id":"AV3qGfrC6jMbsbXb6k1p",
        "_score":1.0,
        "_source": {
          "user": "李四",
          "title": "工程师",
          "desc": "系统管理"
        }
      },
      {
        "_index":"accounts",
        "_type":"person",
        "_id":"1",
        "_score":1.0,
        "_source": {
          "user" : "张三",
          "title" : "工程师",
          "desc" : "数据库管理，软件开发"
        }
      }
    ]
  }
}
```

**二、 Query DSL**

1. match_all查询

   `match_all`属性是查找某属性下的某一个字段，`from`表示从位置1开始查询，`size`表示返回的结果数。

```bash
$ curl 'localhost:9200/accounts/person/_search'  -d '
{
  "query" : { "match_all": {}},
  "sort" : [{ "desc" : "管理" }],
  "from": 1,
  "size": 1
}'
```

`match`大致相同于`match_all`

```bash
$ curl -X GET "localhost:9200/my-index-000001/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "track_total_hits": true,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  }
}
```

2. match_phrase查询

```console
GET /bank/_search
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
```

3. bool查询

bool查询分为三个关键词：must表示查找中必须有的，must_not表示查找中必须没有的，should表示查找中最好有的，表现在结果的排序不再按照score

```bash
$ curl -X GET "localhost:9200/bank/_search" -H 'Content-Type: application/json' -d' 
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
}
```

**三、 Query Filter**

```bash
$ curl -X GET "localhost:9200/shirts/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "color": "red"   }},
        { "term": { "brand": "gucci" }}
      ]
    }
  },
  "aggs": {
    "models": {
      "terms": { "field": "model" } 
    }
  }
}
```



**四、 Full text queries**

**五、 phrase search**

**六、 highlight search**



