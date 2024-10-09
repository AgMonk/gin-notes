# 参考来源

https://www.iszy.cc/posts/14/

https://www.jenkins-zh.cn/wechat/articles/2020/05/2020-05-13-using-nexus-oss-as-a-proxy-cache-for-docker-images/

# 获取镜像服务代理

[Docker/DockerHub 国内镜像源/加速列表（长期维护 10 月 6 日更新） - 轩源的网络日志 - Xuanyuan's Blog](https://xuanyuan.me/blog/archives/1154)

或

https://dockerproxy.cn/

# 创建代理仓库

1. 以管理员身份登录Nexus，打开`Repository`-`Repositories`，点击`Create repository`，选择类型`docker(proxy)`
2. `Name`字段填写一个唯一名称；勾选`HTTP`字段，填写一个不冲突的端口号，如果Nexus运行在docker容器中则需要把这个端口号映射出去
3. `Proxy`-`Remote storage`字段填写获取到的镜像服务代理地址，如：https://dockerproxy.cn/
4. `Docker Index`字段选择`Use Docker Hub`
5. 勾选`Allow anonymous docker pull ( Docker Bearer Token Realm required )`
6. 保存
7. 打开`Security`-`Realms`，点击左边的`Docker Bearer Token Realm`将其添加到右边，保存

# Docker Client配置

1. 创建或编辑这个文件`/etc/docker/daemon.json`，填写或添加如下内容

```json
{
	"insecure-registries": ["192.168.0.10:8082"],
	"registry-mirrors": ["http://192.168.0.10:8082"],
	"log-driver": "json-file",
	"log-opts": {
		"max-size": "50m",
		"max-file": "1"
	}
}
```

这里`192.168.0.10`是Nexus服务器的IP，`8082`是刚才配置的端口号（或docker映射出来的端口号）

2. 执行如下命令更新配置并重启docker引擎

```shell
systemctl daemon-reload & systemctl restart docker
```

3. 重启完毕后执行`docker info`，应当能看到如下信息

```
 Insecure Registries:
  192.168.0.10:8082
  127.0.0.0/8
 Registry Mirrors:
  http://192.168.0.10:8082/

```

