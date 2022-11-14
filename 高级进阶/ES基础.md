***
待学习...
***

# 基本操作
1 创建索引库的分片数默认1片，在7.0.0之前的es版本中，默认5片  
2 查看索引相关信息

| 表头 | 含义 |
| ---- | ---- |
| health | 当前服务器健康状态：green（集群完整）、yellow（单点正常，集群不完整）、red（单点不正常）  |
| status | 索引打开、关闭状态 |
| index | 索引名 |
| uuid | 索引统一编号 |
| pri | 主分片数量 |
| rep | 副本数量 |
| docs.count | 可用文档数量 |
| docs.deleted | 文档删除状态（逻辑删除） |
| store.size | 主分片和副分片整体占空间大小 |
| pri.store.size | 主分片占空间大小 |

查看所有节点信息：_cat/nodes
查看es健康状况：_cat/health
查看主节点信息：_cat/master
查看所有索引：_cat/indices

# 安装
下载elastic search和kibana
```
docker pull elasticsearch:7.6.2
docker pull kibana:7.6.2
```

配置
```
mkdir -p /mydata/elasticsearch/config  创建目录
mkdir -p /mydata/elasticsearch/data
echo "http.host: 0.0.0.0" >/mydata/elasticsearch/config/elasticsearch.yml

//将mydata/elasticsearch/文件夹中文件都可读可写
chmod -R 777 /mydata/elasticsearch/
```

启动es
```
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
-e  "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
-v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
-v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
-v  /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-d elasticsearch:7.6.2

// 设置开机启动es
docker update elasticsearch --restart=always
```

启动kibana
```
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://192.168.6.128:9200 -p 5601:5601 -d kibana:7.6.2

// 设置开机启动kibana
docker update kibana  --restart=always
```

# 安装ik分词器
进入es容器内部
```
docker exec -it elasticsearch /bin/bash
```

```
wget  https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.6.2/elasticsearch-analysis-ik-7.6.2.zip
```

```
unzip elasticsearch-analysis-ik-7.6.2.zip -d ik
```

移动到plugins目录下
```
mv ik plugins
```

# 生成动态域
在es的model中增加Map<String,Object>类型的字段，然后将要保存的string类型数据转成map后，设置进动态域中

