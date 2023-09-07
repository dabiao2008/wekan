# 在阿里云主机上安装 wekan最新版

 

> 下面将使用 Docker Compose 来构建预编译的 Wekan Container. Docker Compose 是一个用于定义和运行多容器 Docker 的应用程序工具，执行如下命令检查是否已......

Wekan 安装指南
----------

 

在你的计算机上找到一个合适的位置如 `/usr/local/` ，执行命令。

```
cd /usr/local/
git clone https://github.com/wekan/wekan

```

国内可以使用 Gitee 的镜像加速：

```
git clone https://gitee.com/mirrors/wekan.git

```

如果没有git，安装git
```
sudo yum install -y git
```

下面将使用 Docker Compose 来构建预编译的 Wekan Container.  
Docker Compose 是一个用于定义和运行多容器 Docker 的应用程序工具，执行如下命令检查是否已安装。

```
docker-compose -version


```

没有docker，安装docker


安装完成后，我们修改一下 wekan 的 docker-compose.yml 配置文件，其目的在于自定义访问端口，可以按需替换镜像源，解决下载速度过慢的问题，甚至自己编译，而不是用官方的预编译容器。  
进入文件夹并编辑配置文件。

建议使用 宝塔面板修改docker-compose.yml
修改：
1.ports
2.ROOT\_URL
3.修改镜像源


```
cd wekan
vim docker-compose.yml


```

补充下 vim 的基础使用，后面会用到这些命令。

```
# 显示行号
:set nu
# 跳转到第 i 行
:i
# 匹配字符串 str
:/str


```

首先修改端口号及访问的 URL。

```
# 搜索 `ports` 
:/ports 
# 或直接跳转到第 138 行
:138


```

修改此处为你本机不用的端口。 其中冒号前面的为外部（你计算机）的端口，冒号后面是 Docker 容器内部的端口。以我的配置作为例子，即将本机 9000 端口映射到容器内部 8080 端口。

[![](https://segmentfault.com/img/bVcRWbA)](https://link.segmentfault.com/?enc=JVSK%2F13qk%2BtAG0NpIeOapw%3D%3D.zZWTgvxUiWFHK9uxrE9Ww%2F170fhVku1%2BW3jjtJTgUo4%3D)

然后修改 ROOT\_URL，格式为 `http://IP地址:端口号` , 其中 IP 地址可以用域名代替，端口号是上一步你配置的外部端口号，就是冒号前面的。

[![](https://segmentfault.com/img/bVcRWbB)](https://link.segmentfault.com/?enc=a9wzB3S3SUsjIb4q6WDxBw%3D%3D.TAKvepf7pi1Q3ueP1hLahHuaUsI5A2SUjyu6UOfBOP8%3D)

**（可选配置）** 修改镜像源。如果下载默认官方服务器的 quay.io/wekan/wekan 镜像太慢的话，可以修改为 Docker Hub 的镜像。  
注释掉第 121 行，取消第 125 行的注释既可。此处是未修改之前的状态。

[![](https://segmentfault.com/img/bVcRWbC)](https://link.segmentfault.com/?enc=nGKFy9EWj%2FrxbweTBHclfQ%3D%3D.Q5z6cfz6xBjJT85mpsR6R8mqok%2FFRIzP6Bkfia6HE%2Bs%3D)

当然你也可以不使用预构建镜像而自己编译。修改此处。

[![](https://segmentfault.com/img/bVcRWbD)](https://link.segmentfault.com/?enc=QlZNOIZkE3dNKKElCx9tGg%3D%3D.UGdztKnpohYTrqLjZPViki%2FEhOyNfeKQwV4xXysUtfE%3D)

下面是官方推荐配置。

```
build:
      context: .
      dockerfile: Dockerfile
      args:
        - NODE_VERSION=${NODE_VERSION}
        - METEOR_RELEASE=${METEOR_RELEASE}
        - NPM_VERSION=${NPM_VERSION}
        - ARCHITECTURE=${ARCHITECTURE}
        - SRC_PATH=${SRC_PATH}
        - METEOR_EDGE=${METEOR_EDGE}
        - USE_EDGE=${USE_EDGE}


```

保存并退出。  
选择一个命令执行，从而启动容器。

```
# 不需要自己 build，使用 Prebuild Container。
docker-compose up -d

# 自己 build
docker-compose up -d --build


```

在阿里云碰到一个坑
```
Error response from daemon: failed to create task for container: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: error during container init: error mounting "/etc/timezone" to rootfs at "/etc/timezone": mount /etc/timezone:/etc/timezone (via /proc/self/fd/6), flags: 0x5001: not a directory: unknown: Are you trying to mount a directory onto a file (or vice-versa)? Check if the specified host path exists and is the expected type
```
原因： centos8.2中/ect/timezone是一个文件夹，而不是一个文件
解决方案：将timezone文件夹改名为timezone2，新建空文件timezone



看到如下输出便是成功了。

[![](https://segmentfault.com/img/bVcRWbE)](https://link.segmentfault.com/?enc=WohguXmZPxSAQCiN%2FgyxpQ%3D%3D.DGf7TkZkXuDhVKPt1ayBMGcW2VVhifdijfcznN8I6Es%3D)

浏览器打开之前的设置的 ROOT\_URL 地址，开始使用 Wekan。

附常用命令。

```
# 1) 停止 Wekan:
    docker-compose stop

# 2) 卸载 Wekan (不包括 db 及内部数据)
    docker rm wekan-app

# 3) 启动 Wekan:
    docker-compose up -d

# 4) 进入容器:
#    a) Wekan app, 不包括内部数据
        docker exec -it wekan-app bash
#    b) MongoDB, 包括所有数据
        docker exec -it wekan-db bash

# 5) 复制数据库到容器外部:
        docker exec -it wekan-db bash
        cd /data
        mongodump
        exit
        docker cp wekan-db:/data/dump .

# 6) 将外部数据库还原至 wekan
#      # 1) 停止 Wekan app
              docker stop wekan-app
#      # 2) 进入数据库容器内部
              docker exec -it wekan-db bash
#      # 3) 然后进入数据存放目录
              cd /data
#      # 4) 删除原 dump 存储 
              rm -rf dump
#      # 5) 退出 db 容器
              exit
#      # 6) 复制 dump 到 docker 容器内部
              docker cp dump wekan-db:/data/
#      # 7) 进入数据库容器内部 
              docker exec -it wekan-db bash
#      # 8) 进入数据存放目录
              cd /data
#      # 9) 恢复
              mongorestore --drop
#      # 10) 退出 db
              exit
#      # 11) 重启 Wekan
              docker start wekan-app



```
