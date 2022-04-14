### 使用docker安装Kibana
要使用直观的用户界面分析、可视化和管理 Elasticsearch 数据,请安装 Kibana

#### 安装运行

```
docker pull docker.elastic.co/kibana/kibana:8.1.2

docker run --name kib-01 --net elastic -p 5601:5601 docker.elastic.co/kibana/kibana:8.1.2

```

若kibana令牌过期，我们可以执行以下指令生成新的令牌

```
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

