

# DSL查询

### match_all

### 全文检索查询

##### match

##### multi_match

### 精确查询

只能查找keyword、数值、日期、boolean类型的字段

##### ids

##### term

##### range

### 地理坐标查询

##### geo_bounding_box

按矩形搜索

##### geo_distance

按点和半径搜索

### bool查询

利用逻辑运算来组合一个或多个查询子句的组合。

①must：必须匹配每个子查询

②should：选择性匹配子查询

③must_not：必须不匹配，**不参与算法**

④filter：必须匹配，**不参与算分**

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
⑧ ids搜索

### 嵌套查询

```
嵌套文档总结：
1 最大的问题是随着嵌套文档的增加，总文档数量是成倍增加的。性能一般
2 GET _cat/indices?v 这个查询是查询索引库里面每个索引总的文档数，包括嵌套文档数量
3 GET order_index/_count 这个查询是查询当前索引的文档数，不包括嵌套文档数量
4 由于嵌套文档被索引为单独的文档，因此只能在nested查询
5 为具有100个嵌套字段的文档建立索引实际上是为101个文档建立索引，因为每个嵌套文档都被索引为一个单独的文档。为了防止定义不正确的映射，每个索引可以定义的嵌套字段的数量默认为限制为50

DELETE order_index

# 新增索引库
PUT order_index
{
  "mappings": {
    "properties": {
      "orderId": {
        "type": "keyword"
      },
      "orderNo": {
        "type": "keyword"
      },
      "orderUserName": {
        "type": "keyword"
      },
      "orderItem": {
        "type": "nested", 
        "properties": {
          "orderItemId": {
            "type": "keyword"
          },
          "orderId": {
            "type": "keyword"
          },
          "productName": {
            "type": "keyword"
          },
          "brandName": {
            "type": "keyword"
          },
          "sellPrice": {
            "type": "keyword"
          }
        }
      }
    }
  }
}

# 新增id为1的文档
PUT order_index/_doc/1
{
  "orderId": "1",
  "orderNo": "123456",
  "orderUserName": "张三",
  "orderItem": [
    {
      "orderItemId": "12234",
      "orderId": "1",
      "productName": "火腿肠",
      "brandName": "双汇",
      "sellPrice": "28"
    },
    {
      "orderItemId": "12235",
      "orderId": "1",
      "productName": "果冻",
      "brandName": "汇源",
      "sellPrice": "12"
    }
  ]
}

GET order_index/_search

GET order_index/_search
{
  "query": {
    "nested": {
      "path": "orderItem",
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "orderItem.productName": "火腿肠"
              }
            },
            {
              "match": {
                "orderItem.brandName": "汇源"
              }
            }
          ]
        }
      }
    }
  }
}


GET order_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "orderItem.productName": "火腿肠"
          }
        },
        {
          "match": {
            "orderItem.brandName": "汇源"
          }
        }
      ]
    }
  }
}
```



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

# 排序

分词字段无法排序，能参与排序的字段类型有：keyword类型、数值类型、地理坐标类型、日期类型等

# 分页

```from``` 从第几个文档开始

```size``` 总共查询几个文档

### 深度分页

https://blog.csdn.net/qq_36558538/article/details/103256397

from/size、scroll、search_after三者的比较
from / size : 该查询的实现原理类似于mysql中的limit，比如查询第10001条数据，那么需要将前面的10000条都拿出来，进行过滤，最终才得到数据。(性能较差，实现简单，适用于少量数据，数据量不超过1w，1w是可配置的，量力而行)。
scroll：该查询实现类似于消息消费的机制，首次查询的时候会在内存中保存一个历史快照以及游标(scroll_id)，记录当前消息查询的终止位置，下次查询的时候将基于游标进行消费(性能良好，维护成本高，在游标失效前，不会更新数据，不够灵活，一旦游标创建size就不可改变，适用于大量数据导出或者索引重建)
search_after: 性能优秀，类似于优化后的分页查询，历史条件过滤掉数据，不适合存在跳跃查询的场景。

```search_after``` 分页时需要排序，原理是从上一次的排序值开始，查询下一页数据

# 高亮查询

①搜索必须有查询条件，而且是全文检索类型的查询条件

②参与高亮的字段必须是text类型的字段

③默认情况下参与高亮的字段要与搜索字段一致

# 聚合



# suggester智能搜索建议

# 百分位计算

# bulk和mGet
