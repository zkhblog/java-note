-------------------------------------------------------------
接着学习文档批量操作（bulk和mGet）
-------------------------------------------------------------

# 查询文档数量
```GET _cat/indices?v```，查询索引库里面每个索引总的文档数  
```GET order_index/_count```，查询当前索引的文档数，不包括嵌套文档数量

# Rest API
### 生成动态域
在es的model中增加Map<String,Object>类型的字段，然后将要保存的string类型数据转成map后，设置进动态域中

# es中的嵌套查询及介绍

> 嵌套文档的缺点：如果一个订单，有1000个订单项，那么在 ES 中存在的文档数就是1001，会随着订单数的增加而成倍上升

结构性的JSON文档会平整成索引内的一个简单键值格式
例如：
```
{
    "memberGoods.title":[
        "商品A",
        "商品B"
    ],
    "memberGoods.brand":[
        "a",
        "b"
    ],
    "groupId":"A"
}
```
原来的json数据其实是：
```
{
    "memberGoods":[
        {
            "title":"商品A",
            "brand":"a"
        },
        {
            "title":"商品B",
            "brand":"b"
        }
    ],
    "groupId":"A"
}
```
平整之后，通过如下查询条件，依然能够查出数据
```
{
    "query":{
        "bool":{
            "must":[
                {
                    "match":{
                        "title":"商品A"
                    }
                },
                {
                    "match":{
                        "brand":"b"
                    }
                }
            ]
        }
    }
}
```

# Query DSL 类型
### match_all

### 全文搜索
① 匹配搜索：match queries接收text/numerics/dates，对他们进行分词分析，再组织成一个boolean查询，可以通过operator指定bool组合操作  
```
POST /lagou-property/_search
{
  "query": {
    "match": {
      "title": {
        "query": "小米电视4A",
        "operator": "and"
      }
    }
  }
}
```
② 短语搜索：match_phrase查询用来对一个字段进行短语查询，可以指定analyzer、slop移动因子  
```
GET /lagou-property/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "小米4A",
        "slop": 2
      }
    }
  }
}
```
③ query_string查询：提供了无需指定某字段而对文档全文进行匹配查询的一个高级查询,同时可以指定在哪些字段上进行匹配  
```
GET /lagou-property/_search
{
  "query": {
    "query_string": {
      // "query": "小米 AND 5A", 逻辑查询
      // "query": "小米 OR 5A", 逻辑查询
      // "query": "小米~5A",模糊查询
      "default_field": "title",
      "fields": ["title","images"]
    }
  }
}
```
④ 多字段匹配搜索(multi match query)：在多个字段上进行文本搜索  
```
GET /lagou-property/_search
{
  "query": {
    "multi_match": {
      "query": "http://image.lagou.com/12479622.jpg",
      "fields": ["images", "tit*"]  // 可以使用*匹配多个字段
    }
  }
}
```

### 词条级搜索
根据结构化数据中的精确值查找文档，结构化数据的值包括日期范围、IP地址、价格或产品ID。与全文查询不同，词条级搜索不分析搜索词条，相反，词条与存储在字段级别中的术语需完全匹配  
① 词条集合搜索：用于查询指定字段包含某些词项的文档
```
GET book/_search
{
  "query": {
    "terms": {
      "name": [
        "solr",
        "hadoop"
      ]
    }
  }
}
```
② 范围搜索：gte、gt、lte、lt、boost  
③ 不为空搜索：查询指定字段值不为空的文档  
```
GET book/_search
{
  "query": {
    "exists": {
      "field": "price"
    }
  }
}
```
④ 词项前缀搜索：prefix  
⑤ 通配符搜索：wildcard query  
⑥ 正则搜索：regexp  
⑦ 模糊搜索：fuzzy  
⑧ ids搜索：


### 复合搜索
① ```constant_score```用来包装另一个查询，将查询匹配的文档的评分设为一个常值  
② ```bool query```用来组合多个查询子句为一个查询  

