# 准备工作
1. 下载安装python 3.10.6: https://www.python.org/downloads/release/python-3106/ , 安装时勾选“Add python 3.10 to PATH”
2. 下载安装git： https://git-scm.com/downloads
3. 使用git 或 github desktop（推荐）克隆如下仓库，总大小大约40M，注意路径不能有中文
```
https://github.com/AUTOMATIC1111/stable-diffusion-webui
```
如果你的网络不太好，则在前面增加一段：
```
https://ghfast.top/
```
4. (可选）如果你的网络不太好，用记事本打开该文件
```
stable-diffusion-webui\modules\launch_utils.py
```
在所有的  _https://github.com_ 前面都增加上面这一段 

5. 运行这个脚本，会自动安装很多的依赖，将下载约4.4GB的文件
```
stable-diffusion-webui\webui-user.bat
```

6. 之后脚本会调用git去克隆其他很多个仓库，中间可能会失败，重新运行即可；其中 **Installing requirements** 步骤可能耗时较长，或者有可能需要一些 **"魔法"** 