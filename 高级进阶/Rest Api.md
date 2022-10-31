```
# 查看所有节点。注：*表示集群中的主节点
GET /_cat/nodes

# 查看es健康状况。注：green表示健康值正常
GET /_cat/health

# 查看主节点信息
GET /_cat/master

# 查看所有索引，等价于mysql中show databases
GET /_cat/indices



##########     新增或更新     ##########
# 指定id查看数据
GET /customer/external/7

# 返回结果解析
# {
#   "_index" : "customer",
#   "_type" : "external",
#   "_id" : "1",
#   "_version" : 2,
#   "_seq_no（并发控制字段，每次更新都会加1，用来做乐观锁）" : 1,
#   "_primary_term" : 1,
#   "found" : true,
#   "_source" : {
#     "name" : "Bob"
#   }
# }

# PUT保存一条数据，保存在哪个索引哪个类型下，必须指定用哪个唯一标识
PUT customer/external/7
{
  "name":"8"
}

# 返回结果解析，带有下划线开头的，称为元数据，反映了当前的基本信息
# {
#   "_index" : "customer", // 标识索引
#   "_type" : "external", // 标识类型
#   "_id" : "1", // 标识数据id
#   "_version" : 2, // 标识被保存数据的版本
#   "result" : "updated",
#   "_shards" : {
#     "total" : 2,
#     "successful" : 1,
#     "failed" : 0
#   },
#   "_seq_no" : 1,
#   "_primary_term" : 1
# }

# POST带_update
POST /customer/external/7/_update
{
    "doc":{
        "name":"8"
    }
}

# POST不带_update
POST /customer/external/7
{
  "name":"8"
}

# 总结：
# PUT可以新增也可以修改。PUT必须指定id，由于PUT需要指定id，我们一般用来做修改操作，不指定id会报错。PUT操作总会将数据重新保存并增加version版本
# POST新增。如果不指定id，会自动生成id。指定id时如果存在就会修改这个数据，并新增版本号。POST操作带_update会对比原数据，如果相同不会有什么操作，version和_seq_no不变。不带_update就不会对比原来的数据，直接更新。

# 使用场景
# 对于大并发更新，可以PUT请求且不带_update
# 对于大并发查询偶尔更新，可以POST请求且带_update。对比更新，重新计算分配规则



##########     删除     ##########
# 删除文档
DELETE customer/external/1

# 删除索引
DELETE customer

# 总结：
# ES并没有提供删除类型的操作，只提供了删除索引和文档的操作



##########     批量操作     ##########
POST /customer/external/_bulk
{"index":{"_id":"1"}}
{"name":"Bob"}
{"index":{"_id":"2"}}
{"name":"Jack"}


POST /_bulk
{"delete":{"_index":"customer","_type":"external","_id":"1"}}
{"create":{"_index":"customer","_type":"external","_id":"111"}}
{"title":"这是create方式创建的111数据"}
{"index":{"_index":"customer","_type":"external"}}
{"title":"这是index方式创建的数据"}
{"update":{"_index":"customer","_type":"external","_id":2}}
{"doc":{"name":"My Jack"}}



##########     测试数据     ##########
# https://github.com/elastic/elasticsearch/blob/7.4/docs/src/test/resources/accounts.json

##########     match_all查询所有     ##########
# 查询所有数据中第0个到第5个数据
# query参数中match_all查询代表查询所有数据，es可以在query中组合非常多的查询类型以完成复杂查询
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 5,
  "sort": [
    {
      "account_number": {
        "order": "desc"
      }
    }
  ]
}

# 返回部分字段
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 5,
  "sort": [
    {
      "account_number": {
        "order": "desc"
      }
    }
  ],
  "_source": ["balance","firstname"]
}

##########     match分词匹配     ##########
# match分词匹配，检索条件中20不带""时，就是精确匹配
GET /bank/_search
{
  "query": {
    "match": {
      "account_number": 20
    }
  }
}

# 检索条件加""会进行模糊查询，并会对检索条件进行分词匹配
GET bank/_search
{
  "query": {
    "match": {
      "address": "kings"
    }
  }
}

# match.keyword
GET bank/_search
{
  "query": {
    "match": {
      "address.keyword": "282 Kings Place"
    }
  }
}

##########     match_phrase短语匹配     ##########
# match_phrase短语匹配，将需要匹配的值当成一整个单词，即不分词进行检索，并给出相关性得分
GET bank/_search
{
  "query": {
    "match_phrase": {
      "address": "kings"
    }
  }
}

# 总结
# 文本字段的匹配，使用keyword，匹配的条件就是要显示字段的全部值，要进行精确匹配的
# match_phrase是做短语匹配，只要文本中包含匹配条件即包含这个短语，就能匹配到


##########  multi_match多字段匹配 ##########

# state或address中包含mill，并且在查询过程中，会对查询条件进行分词
GET bank/_search
{
  "query": {
    "multi_match": {
      "query": "mill",
      "fields": [
        "state",
        "address"
      ]
    }
  }
}

##########  bool复合查询 ##########
# 复合语句可以合并，任何其他查询语句，包括复合语句。这就意味着，复合语句可以互相嵌套，可以表达非常复杂的查询逻辑
# must表示必须达到所列举的所有条件
# must_not表示必须不匹配所列举的所有条件
# should表示应该满足所列举的条件

# 查询gender=m,并且address=mill的数据
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"address": "mill"}},
        {"match": {"gender": "M"}}
      ]
    }
  }
}

GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "gender": "M"
          }
        },
        {
          "match": {
            "address": "mill"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "age": "38"
          }
        }
      ]
    }
  }
}

GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "gender": "M"
          }
        },
        {
          "match": {
            "address": "mill"
          }
        }
      ],
      "must_not": [
          { "match": {
              "age": "18"
            }
          }
      ],
      "should": [
        {
          "match": {
            "lastname": "Wallace"
          }
        }
      ]
    }
  }
}


##########  filter过滤查询 ##########
# 并不是所有的查询都需要产生分数，特别是那些仅用于filter过滤的文档。为了不计算分数，es会自动检查场景并且优化查询的执行

# 先查询所有匹配address等于mill的文档，然后再根据
# 10000 < balance < 20000 进行过滤查询结果
GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "address": "mill"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "balance": {
              "gte": 10000,
              "lte": 20000
            }
          }
        }
      ]
    }
  }
}

# 总结
# must、should和must not元素
# 都被称为查询子句。文档是否符合子句查询标准，决定了文档的“相关性得分”。得分越高，越符合搜索条件。
# must not子句中的条件影响文档是否包含在结果中，但不影响文档的得分方式。filter在使用过程中，并不会计算相关性得分_score

GET bank/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "address": "mill"
          }
        }
      ],
      "filter": [
        {
          "range": {
            "balance": {
              "gte": 10000,
              "lte": 20000
            }
          }
        }
      ]
    }
  }
}



##########  term ##########

GET bank/_search
{
  "query": {
    "term": {
      "age": {
        "value": "28"
      }
    }
  }
}

# term查询text字段，查不到结果
GET bank/_search
{
  "query": {
    "term": {
      "gender": {
        "value": "F"
      }
    }
  }
}

# 总结
# 全文检索字段用match、其他非text字段匹配用term


##########  聚合 ##########

# size为0表示不看结果详情，只看聚合结果
GET bank/_search
{
  "query": {
    "match": {
      "address": "mill"
    }
  },
  "size": 0, 
  "aggs": {
    "aggAgg": {
      "terms": {
        "field": "age",
        "size": 10
      }
    },
    "ageAvg":{
      "avg": {
        "field": "age"
      }
    },
    "balanceAvg": {
      "avg": {
        "field": "balance"
      }
    }
  }
}


# 按照年龄聚合，并且求这些年龄段的人的平均薪资
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "size": 0, 
  "aggs": {
    "ageRange":{
      "terms": {
        "field": "age",
        "size": 10
      },
      "aggs": {
        "balanceAvg": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}

# 查看所有年龄分布，并且查出年龄段中M的平均薪资和F的平均薪资以及这个年龄段的总体平均薪资
GET bank/_search
{
  "query": {
    "match_all": {}
  },
  "size": 0,
  "aggs": {
    "ageRange": {
      "terms": {
        "field": "age",
        "size": 10
      },
      "aggs": {
        "genderAgg": {
          "terms": {
            "field": "gender.keyword"
          },
          "aggs": {
            "balanceAvg": {
              "avg": {
                "field": "balance"
              }
            }
          }
        },
        "ageBalanceAvg": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}


##########  属性 ##########

### 将bank数据迁移到newbank，并修改age字段的映射从long到integer

PUT /newbank
{
  "mappings": {
    "properties": {
      "account_number":{
        "type": "long"
      },
      "address":{
        "type": "text"
      },
      "age":{
        "type": "integer"
      },
      "balance":{
        "type": "long"
      },
      "city":{
        "type": "keyword"
      },
      "email":{
        "type": "keyword"
      },
      "employer":{
        "type": "keyword"
      },
      "firstname":{
        "type": "text"
      },
      "gender": {
        "type": "keyword"
      },
      "lastname":{
        "type": "text",
        "fields": {
          "keyword": {
            "type":"keyword",
            "ignore_above": 256
          }
        }
      },
      "state":{
        "type": "keyword"
      }
    }
  }
}

# 查询新索引的映射
GET /newbank/_mapping

# 将bank中数据迁移到newbank中
POST _reindex
{
  "source": {
    "index": "bank",
    "type": "account"
  },
  "dest": {
    "index": "newbank"
  }
}

# 查看新索引的文档
GET /newbank/_search


##########  分词 ##########
POST _analyze
{
  "analyzer": "standard",
  "text": "The 2 QUICK Brown-Foxes jumped over the lazy dog's bone."
}

##########  安装完ik中文分词器 ##########

# 使用的是标准分词器，对中文不友好
GET _analyze
{
  "text": "我是中国人"
}

# 
GET _analyze
{
  "analyzer": "ik_smart",
  "text": "我是中国人"
}

# 
GET _analyze
{
  "analyzer": "ik_max_word",
  "text": "我是中国人"
}

##########  自定义词库 ##########
# 加入自定义词库
GET _analyze
{
  "analyzer": "ik_smart",
  "text": "刘德华牛气冲天"
}

# 未加入自定义词库
GET _analyze
{
  "analyzer": "ik_smart",
  "text": "周坤豪牛气冲天"
}

# 注意
# 更新完成后，es只会对新增数据用更新分词。历史数据是不会重新分词的。如果想要历史数据重新分词，需要执行
POST my_index/_update_by_query?conflicts=proceed


# 用Java Api编写后打印出的检索条件
GET bank
{
  "query": {
    "match": {
      "address": {
        "query": "mill",
        "operator": "OR",
        "prefix_length": 0,
        "max_expansions": 50,
        "fuzzy_transpositions": true,
        "lenient": false,
        "zero_terms_query": "NONE",
        "auto_generate_synonyms_phrase_query": true,
        "boost": 1
      }
    }
  },
  "aggregations": {
    "ageRange": {
      "terms": {
        "field": "age",
        "size": 10,
        "min_doc_count": 1,
        "shard_min_doc_count": 0,
        "show_term_doc_count_error": false,
        "order": [
          {
            "_count": "desc"
          },
          {
            "_key": "asc"
          }
        ]
      }
    },
    "ageAvg": {
      "avg": {
        "field": "age"
      }
    },
    "balanceAvg": {
      "avg": {
        "field": "balance"
      }
    }
  }
}
```