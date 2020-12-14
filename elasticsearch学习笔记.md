# Elastic Search学习笔记

**一、 基本概念**

1. node：单个elastic实例即node节点，一组node构成一个cluster集群。

2. document：单个记录，用json表示。一组document构成一个index数据库/索引。

   ```javascript
   // JSON格式的document
   {
     "user": "张三",
     "title": "工程师",
     "desc": "数据库管理"
   }
   ```

3. 