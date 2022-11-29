# MySql

## run

```shell
docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=12345678 -d --name mysql --restart=always -v /root/data:/var/lib/mysql mysql:5.7
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
	    - /opt/docker_mynginx/mysql_data:/var/lib/mysql
```



# Redis

## 自定义配置文件

redis.conf

```
##开启AOF持久化
appendonly yes

## 快照间隔 60秒1个key的变化
save 60 1

## 日志层级
loglevel warning
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

