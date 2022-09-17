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