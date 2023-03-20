# Docker引擎安装

[官方文档](https://docs.docker.com/engine/install/centos/)

## CentOS

```bash
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```
或
```bash
curl -sSL https://get.daocloud.io/docker | sh
```

如果出现如下错误:

```
repodata/repomd.xml: [Errno 14] HTTP Error 404 - Not Found
```

尝试修改yum源解决：https://blog.csdn.net/m0_60028455/article/details/122876291

自启动和启动

```bash
systemctl enable docker
systemctl start docker
```

## Docker-compose

### 安装

https://docs.docker.com/compose/install/linux/

```shell
 yum update
sudo yum install docker-compose-plugin
```

### 常用指令

```shell
docker-compose up --build --force-recreate --d
```

重新build、重新创建容器、后台运行

# MySql

需要映射数据库文件的存储目录

## run

```shell
docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=12345678 -d --name mysql --restart=always -v /home/mysql_data:/var/lib/mysql mysql:5.7
```

## compose

```yaml
services:
  mysql:
    container_name: mysql
    image: mysql:5.7
    ports:
      - "3306:3306"
    restart: always
    environment:
        MYSQL_ROOT_PASSWORD: 12345678
        TZ: Asia/Shanghai
    volumes:
    - /home/mysql_data:/var/lib/mysql 
```



# Redis

需要使用配置文件开启持久化，并映射持久化文件目录

## 自定义配置文件

redis.conf

```
##开启AOF持久化
appendonly yes

## 快照间隔 60秒1个key的变化
save 60 1

## 日志层级
loglevel warning

## 密码
requirepass 123456
```

## run

```shell
docker run -p 6379:6379 -d --name redis --restart=always -v /home/redis:/data redis:5.0.12 redis-server /data/redis.conf
```

## compose

```yaml
services:
  redis:
    entrypoint: redis-server /data/redis.conf
    container_name: redis
    image: redis:5.0.12
    ports:
      - "6379:6379"
    restart: always
    volumes:
      - /home/redis:/data
```

# Nacos

## run

## compose

```yml
services:
    nacos:
      container_name: nacos_01
      image: nacos/nacos-server:2.0.3
      restart: always
      ports:
        - 8848:8848
      environment:
        PREFER_HOST_MODE: hostname
        MODE: standalone

```

# Nginx

需要映射静态文件的存放目录、额外配置的存放目录

## 配置文件

nginx.conf (官方默认)

```nginx

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}

```

添加额外配置可在`conf.d`目录下添加`.conf`文件

## run

```shell
docker run -p 8080:80 -d --name nginx01 -v /home/nginx/html:/usr/share/nginx/html -v /home/nginx/conf.d:/etc/nginx/conf.d --restart=always nginx:1.22
```



## compose

```yml
services:
    nacos:
      container_name: nginx01
      image: nginx:1.22
      restart: always
      ports:
        - 8080:80
      volumes:
      	- /home/nginx/html:/usr/share/nginx/html
      	- /home/nginx/conf.d:/etc/nginx/conf.d

```

# Sentinel-Dashboard

## run

```shell
docker run -p 8858:8858 -p 8719:8719 -d --name sentinel-dashboard -e TZ="Asia/Shanghai" --restart=always bladex/sentinel-dashboard:latest
```

## compose

```yml
services:
    sentinel-dashboard:
      container_name: sentinel-dashboard
      image: bladex/sentinel-dashboard:latest
      restart: always
      ports:
        - 8719:8719
        - 8858:8858
      environment:	   
        TZ: Asia/Shanghai    
```

# Potainer

# ElasticSearch + Kibana

## IK分词器

下载：https://github.com/medcl/elasticsearch-analysis-ik/releases/tag/v7.14.0

解压到任意名称的文件夹中，通过挂载数据卷到`/usr/share/elasticsearch/plugins`中

分词器版本需要与ES版本严格相等

## run

es:

```shell
docker run -d --name elastic-search -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node"  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" --restart=always elasticsearch:7.14.0
```

kibana:

```shell
docker run -d --name kibana -p 5601:5601 -e "ELASTICSEARCH_HOSTS=http://elastic-search:9200" --restart=always kibana:7.14.0
```

## compose

```yml
services:
    elastic-search:
      container_name: elastic-search
      image: elasticsearch:7.14.0
      restart: always
      ports:
        - 9200:9200
        - 9300:9300
      volumes:
      #  - /home/elastic-search/es/data:/usr/share/elasticsearch/data
    #    - /home/elastic-search/es/config:/usr/share/elasticsearch/config
        - /home/elastic-search/es/plugins:/usr/share/elasticsearch/plugins
      environment:
        TZ: Asia/Shanghai
        discovery.type: single-node
        ES_JAVA_OPTS: -Xms512m -Xmx512m
    kibana:
      container_name: kibana
      image: kibana:7.14.0
      restart: always
      ports:
        - 5601:5601
     # volumes:
     #   - /home/elastic-search/kibana/kibana.yml:/usr/share/kibana/config/kibana.yml
      environment:
        TZ: Asia/Shanghai
        ELASTICSEARCH_HOSTS: http://elastic-search:9200
```

