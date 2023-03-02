# 实现效果

连续执行如下操作：

1. maven clean package
2. 上传打包好的 jar/war 到服务器
3. 执行部署shell脚本

# 步骤

## 下载插件

[Alibaba Cloud Toolkit](https://plugins.jetbrains.com/plugin/11386-alibaba-cloud-toolkit)

## 准备好除了 jar/war 的其他文件

可能包括

1. `docker-compose.yml`：docker compose 配置
2. `Dockerfile`：docker build 配置
3. `run.sh`：部署脚本

上传到目标服务器的部署文件夹，这里假设为`/home/deploy`，这些文件后续的修改频率很低，不包含在自动操作范围内。

## 配置部署配置

- 打开`运行/调试配置`
- 点击`+`号，选择`Deploy to Host`，可以看到新建的配置中默认选中了`Maven Build`并在执行前运行`clean install`
- 在右侧的`Target Host`处点击`+`号添加一台部署服务器，填写服务器IP，账号密码
- 填写目标服务器的部署文件夹`Target Directory`，即上面的`/home/deploy`
- 点击`Select Command`，配置部署脚本，如果脚本已经写在`run.sh`里了，在这里直接调用即可，注意路径必须使用绝对路径。
- 确定保存，之后运行该配置即可完成上述连续操作。

## 参考文章

https://blog.csdn.net/weixin_44578029/article/details/124406728

