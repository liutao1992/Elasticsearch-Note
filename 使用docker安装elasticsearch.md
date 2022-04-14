### 使用docker安装elasticsearch

#### 拉取ES镜像

```
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.1.2
```

#### 单节点启动

下面命令将启动一个单节点集群，用于开发和测试。

- 为Elasticsearch 和 Kibana 创建新的docker网络

```
docker network create elastic
```

- 启动ES服务

```
docker run --name es01 --net elastic -p 9200:9200 -p 9300:9300 -it docker.elastic.co/elasticsearch/elasticsearch:8.1.2
```

当该条指令执行后，终端将会输出以下信息。该信息中包含elastic用户的密码以及进入Kibana的token。

```
docker run --name es01 --net elastic -p 9200:9200 -p 9300:9300 -it docker.elastic.co/elasticsearch/elasticsearch:8.1.2
```

```
--------------------------------------------------------------------------------------------------------------------------------
-> Elasticsearch security features have been automatically configured!
-> Authentication is enabled and cluster connections are encrypted.

->  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  u75UlQeBlinu72jYWpEs

->  HTTP CA certificate SHA-256 fingerprint:
  9de5a81fb499a82e4ff0f8f9382c8b1b60a0d068ef8420e08c6f171b7384efda

->  Configure Kibana to use this cluster:
* Run Kibana and click the configuration link in the terminal when Kibana starts.
* Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjEuMiIsImFkciI6WyIxNzIuMTguMC4yOjkyMDAiXSwiZmdyIjoiOWRlNWE4MWZiNDk5YTgyZTRmZjBmOGY5MzgyYzhiMWI2MGEwZDA2OGVmODQyMGUwOGM2ZjE3MWI3Mzg0ZWZkYSIsImtleSI6ImV4QURLSUFCYjg3eGVkRl94d2kyOlJPbERLa0I4VDFpVjRYaUx5YVdVNGcifQ==

-> Configure other nodes to join this cluster:
* Copy the following enrollment token and start new Elasticsearch nodes with `bin/elasticsearch --enrollment-token <token>` (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjEuMiIsImFkciI6WyIxNzIuMTguMC4yOjkyMDAiXSwiZmdyIjoiOWRlNWE4MWZiNDk5YTgyZTRmZjBmOGY5MzgyYzhiMWI2MGEwZDA2OGVmODQyMGUwOGM2ZjE3MWI3Mzg0ZWZkYSIsImtleSI6ImVoQURLSUFCYjg3eGVkRl94d2l4OnRwejVxd2RyUlpxbmFMZ01GUmVMMmcifQ==

  If you're running in Docker, copy the enrollment token and run:
  `docker run -e "ENROLLMENT_TOKEN=<token>" docker.elastic.co/elasticsearch/elasticsearch:8.1.2`
--------------------------------------------------------------------------------------------------------------------------------
```

>注意：

在docker中启动ES服务时，若出现以下信息：
“max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]”


执行以下指令查看vm.max_map_count数量：
```
grep vm.max_map_count /etc/sysctl.conf
vm.max_map_count=65530
```

执行以下指令修改参数：
```
sysctl -w vm.max_map_count=262144
```

#### 重置密码

```
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-reset-password
```

#### 复制`http_ca.crt`安全证书

将docker容器中的证书复制到本地目录

```
sudo docker cp es01:/usr/share/elasticsearch/config/certs/http_ca.crt /etc/elasticsearch/cert/
```

#### 访问ES服务

```
curl --cacert /etc/elasticsearch/cert/http_ca.crt  -u elastic https://localhost:9200
```

输入密码后，我们将会得到以下信息：
```
Enter host password for user 'elastic':
{
  "name" : "238a2bf95657",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "6NDLwtlYS62SEBP_G42Itw",
  "version" : {
    "number" : "8.1.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "31df9689e80bad366ac20176aa7f2371ea5eb4c1",
    "build_date" : "2022-03-29T21:18:59.991429448Z",
    "build_snapshot" : false,
    "lucene_version" : "9.0.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

#### Generating enrollment tokens

```
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
```

