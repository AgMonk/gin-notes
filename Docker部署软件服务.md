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

## 自启动和启动

```bash
systemctl enable docker
systemctl start docker
```

## 修改控制台日志体积上限

创建或编辑这个文件`/etc/docker/daemon.json`，填写或添加如下内容

```json
{	
	"log-driver": "json-file",
	"log-opts": {
		"max-size": "50m",
		"max-file": "1"
	}
}
```

## 构建自己通用基础镜像

- 以`openjdk:18`作为基础镜像
- 配置环境变量，包括时区以及JAVA使用的
- 添加mysql的远程连接客户端，用于操作数据库的备份和还原
- 添加中文字体(自行从系统字体文件夹获取)

Dockerfile:

```dockerfile
FROM openjdk:18
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN curl -fL -O https://downloads.mysql.com/archives/get/p/23/file/mysql-community-client-8.0.30-1.el8.x86_64.rpm \
    -O https://downloads.mysql.com/archives/get/p/23/file/mysql-community-client-plugins-8.0.30-1.el8.x86_64.rpm \
    -O https://downloads.mysql.com/archives/get/p/23/file/mysql-community-common-8.0.30-1.el8.x86_64.rpm \
    -O https://downloads.mysql.com/archives/get/p/23/file/mysql-community-libs-8.0.30-1.el8.x86_64.rpm \
       && rpm -ivh mysql-community-* && rm mysql-community-*
ENV JAVA_OPTS='--add-opens java.base/java.lang=ALL-UNNAMED'
ADD simhei.ttf  /usr/share/fonts/ttf-dejavu/simhei.ttf
```

build:

```bash
docker build -t myjdk:18 .
```

## 开启远程连接及HTTPS证书

打开配置文件`/usr/lib/systemd/system/docker.service`，找到一行以`ExecStart`开头的配置，在后面增加:

```bash
--tlsverify --tlscacert=/etc/docker/ca.pem --tlscert=/etc/docker/server-cert.pem --tlskey=/etc/docker/server-key.pem -H tcp://0.0.0.0:2376
```

意为指定远程连接的端口，并指定证书文件的位置，然后重启服务

```bash
systemctl daemon-reload && systemctl  restart docker
```

生成证书：
https://blog.csdn.net/hjg719/article/details/127050475

## Docker-compose

### 安装

https://docs.docker.com/compose/install/linux/

```shell
yum update
yum install docker-compose-plugin
```

### 常用指令

```shell
docker-compose up --build --force-recreate --d
```

重新build、重新创建容器、后台运行

## Potainer

### 安装

```bash
docker run -d -p 9000:9000 --restart=always -v /var/run/docker.sock:/var/run/docker.sock --name portainer lihaixin/portainer
```

# 应用软件

## MySql

需要映射数据库文件的存储目录

### run

```shell
docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=12345678 -d --name mysql --restart=always -v /home/mysql_data:/var/lib/mysql mysql:5.7
```

### compose

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



## Redis

需要使用配置文件开启持久化，并映射持久化文件目录

### 自定义配置文件

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

### run

```shell
docker run -p 6379:6379 -d --name redis --restart=always -v /home/redis:/data redis:5.0.12 redis-server /data/redis.conf
```

### compose

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

## Nacos

### run

### compose

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

## Nginx

需要映射静态文件的存放目录、额外配置的存放目录

### 配置文件

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

### run

```shell
docker run -p 8080:80 -d --name nginx01 -v /home/nginx/html:/usr/share/nginx/html -v /home/nginx/conf.d:/etc/nginx/conf.d --restart=always nginx:1.22
```



### compose

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

## Sentinel-Dashboard

### run

```shell
docker run -p 8858:8858 -p 8719:8719 -d --name sentinel-dashboard -e TZ="Asia/Shanghai" --restart=always bladex/sentinel-dashboard:latest
```

### compose

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



## ElasticSearch + Kibana

### IK分词器

下载：https://github.com/medcl/elasticsearch-analysis-ik/releases/tag/v7.14.0

解压到任意名称的文件夹中，通过挂载数据卷到`/usr/share/elasticsearch/plugins`中

分词器版本需要与ES版本严格相等

### run

es:

```shell
docker run -d --name elastic-search -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node"  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" --restart=always elasticsearch:7.14.0
```

kibana:

```shell
docker run -d --name kibana -p 5601:5601 -e "ELASTICSEARCH_HOSTS=http://elastic-search:9200" --restart=always kibana:7.14.0
```

### compose

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

## Nexus

Maven私服

### 公共

需要在宿主机中创建一个挂载目录并赋予修改权限，这里假设目录为`/home/nexus/data`，该目录需要映射到容器内的`/nexus-data`目录

```bash
mkdir /home/nexus/data
chown -R 200 /home/nexus/data
```

容器启动完毕后，进入该目录执行如下命令可以查看`admin`账号的初始密码

```bash
cat admin.password
```

### run

拉取镜像

```bash
docker pull sonatype/nexus3:latest
```

运行

```bash
docker run -d -p 8081:8081 --name nexus --restart=always -e INSTALL4J_ADD_VM_PARAMS="-Xms512m -Xmx512m -XX:MaxDirectMemorySize=1200m" -v /home/nexus/data:/nexus-data sonatype/nexus3
```

### compose

```yml
services:
  nexus:
    container_name: nexus
    image: sonatype/nexus3:latest
    ports:
      - "8081:8081"
    restart: always
    environment:
        INSTALL4J_ADD_VM_PARAMS: -Xms512m -Xmx512m -XX:MaxDirectMemorySize=1200m
    volumes:
    - /home/nexus/data:/nexus-data
```

## InfluxDB+Telegraf

### 配置文件

Telegraf需要配置文件来启动，这里路径为：`/home/telegraf/exam02_conf.conf`

文件可以使用`InfluxDB`的功能来生成：

1. 左侧导航栏`Load Data` - `Telegraf`，`CREATE CONFIGURATION`
2. 选择一个Bucket，和模板（如`system`），确认
3. 点击`setup instructions`，点击`GENERATE NEW API TOKEN`生成一个token，复制
4. 点击配置文件名称，查找token配置，粘贴替换；保存之后下载配置文件，放入上述路径

### compose

```yml
services:
  influxdb:
    image: influxdb:2.7.10
    container_name: influxdb2
    restart: always
    volumes:
      - /home/influxdb/influxdb2:/var/lib/influxdb2
      - /home/influxdb/config:/etc/influxdb2
    ports:
      - 8086:8086  
    environment:
      DOCKER_INFLUXDB_INIT_MODE: setup
      DOCKER_INFLUXDB_INIT_USERNAME: root
      DOCKER_INFLUXDB_INIT_PASSWORD: root#123
      DOCKER_INFLUXDB_INIT_ORG: docs
      DOCKER_INFLUXDB_INIT_BUCKET: home
  telegraf:
     image: telegraf:latest
     container_name: telegraf
     restart: always
     depends_on:
       - influxdb
     volumes:
       - /home/telegraf/exam02_conf.conf:/etc/telegraf/telegraf.conf:ro
     ports:
      - 8125:8125
```

### 
