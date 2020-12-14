# Elastic Search 搜索笔记

**一、 Query DSL**

**二、 Aggregations**

**三、 Search multiple data streams and indices**

**四、Paginate search results** 

**五、 Retrieve selected fields**

**六、 Sort search results**

**七、 Run an async search**

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
'
```

`track_total_hits`为true时能显示超过10000条hits，为false时默认值即显示的hit上限为10000。

