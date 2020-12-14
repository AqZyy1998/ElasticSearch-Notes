# Elastic Search学习笔记

**一、 基本概念**

1. node：单个elastic实例即node节点，一组node构成一个cluster集群。

   > ```bash
   > // 查看当前node的所有index
   > $ curl -X GET 'http://localhost:9200/_cat/indices?v'
   > ```

2. document：单个记录，用json表示。一组document构成一个index数据库/索引。一个node下有若干个index。

   ```javascript
   // JSON格式的document
   {
     "user": "张三",
     "title": "工程师",
     "desc": "数据库管理"
   }
   ```

3. Type：document的分组，每个type应该结构相似

**二、 新建和删除index**

1. 新建index：关键词 **PUT** 

   > ```bash
   > $ curl -X PUT 'localhost:9200/weather'
   > ```

   ```javascript
   // 缺省值
   {
     "acknowledged":true,
     "shards_acknowledged":true
   }
   ```

2. 删除Index：关键词 **DELETE**

   > ```bash
   > $ curl -X DELETE 'localhost:9200/weather'
   > ```

**三、 中文分词**

1. `analyzer`是字段文本的分词器，`search_analyzer`是搜索词的分词器。`ik_max_word`分词器是插件`ik`提供的，可以对文本进行最大数量的分词。

   ```javascript
   // user字段中的属性
   "user": {
     "type": "text",
     "analyzer": "ik_max_word",
     "search_analyzer": "ik_max_word"
   }
   ```

**四、 数据操作**

1. 新增记录，其中`/1`表示id，可以是数字或字符串，在输出的JSON中以属性`_id`的形式显示。另外，-d可接document的内容

   ```bash
   $ curl -X PUT 'localhost:9200/accounts/person/1' -d '
   {
     "user": "张三",
     "title": "工程师",
     "desc": "数据库管理"
   }' 
   ```

2. 不指定IP的新增记录，需要用POST，输出的id为自动生成的随机字符串。

   ```bash
   $ curl -X POST 'localhost:9200/accounts/person' -d '
   {
     "user": "李四",
     "title": "工程师",
     "desc": "系统管理"
   }'
   // 返回的JSON对象
   {
     "_index":"accounts",
     "_type":"person",
     "_id":"AV3qGfrC6jMbsbXb6k1p",
     "_version":1,
     "result":"created",
     "_shards":{"total":2,"successful":1,"failed":0},
     "created":true
   }
   ```

3. 查看记录：发送GET请求，`pretty=true`表示用易读形式返回记录

   ```bash
   $ curl 'localhost:9200/accounts/person/1?pretty=true'
   ```

   查找成功，`found`字段为`true`，`_source`字段返回记录。

   ```javascript
   {
     "_index" : "accounts",
     "_type" : "person",
     "_id" : "1",
     "_version" : 1,
     "found" : true,
     "_source" : {
       "user" : "张三",
       "title" : "工程师",
       "desc" : "数据库管理"
     }
   }
   ```

   查找失败，`found`字段为`false`。

   ```bash
   $ curl 'localhost:9200/weather/beijing/abc?pretty=true'
   
   {
     "_index" : "accounts",
     "_type" : "person",
     "_id" : "abc",
     "found" : false
   }
   ```

4. 删除记录，关键词`DELETE`

   ```bash
   $ curl -X DELETE 'localhost:9200/accounts/person/1'
   ```

5. 更新记录，重新`PUT`即可。

   ```bash
   $ curl -X PUT 'localhost:9200/accounts/person/1' -d '
   {
       "user" : "张三",
       "title" : "工程师",
       "desc" : "数据库管理，软件开发"
   }' 
   ```

   返回的结果中`_version`迭代为2， `_result`更新为`updated`， `created`更新为`false`。

   ```bash
   {
     "_index":"accounts",
     "_type":"person",
     "_id":"1",
     "_version":2,
     "result":"updated",
     "_shards":{"total":2,"successful":1,"failed":0},
     "created":false
   }
   ```

6. 数据查询，返回所有记录。直接`GET`。

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

7. match查询

   `match`属性是查找某属性下的某一个字段，`from`表示从位置1开始查询，`size`表示返回的结果数。

   ```bash
   $ curl 'localhost:9200/accounts/person/_search'  -d '
   {
     "query" : { "match" : { "desc" : "管理" }},
     "from": 1,
     "size": 1
   }'
   ```

