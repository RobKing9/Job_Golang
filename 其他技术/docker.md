# Docker使用指南

## 安装与配置

### Ubuntu环境

- [ubuntu安装docker教程](https://docs.docker.com/engine/install/ubuntu/)

- 拉取镜像太慢，配置镜像加速

- `vim /etc/docker/daemon.json`  

  对于使用 systemd 的系统，请在 /etc/docker/daemon.json 中写入如下内容（如果文件不存在请新建该文件）：

  ```
  {"registry-mirrors":["https://reg-mirror.qiniu.com/"]}
  ```

  可以添加多个

  ```
  科大镜像：https://docker.mirrors.ustc.edu.cn/
  网易：https://hub-mirror.c.163.com/
  阿里云：https://<你的ID>.mirror.aliyuncs.com
  七牛云加速器：https://reg-mirror.qiniu.com
  ```

  重新启动

  ```
  $ sudo systemctl daemon-reload
  $ sudo systemctl restart docker
  ```

### Mac环境

直接使用brew安装cask

```shell
brew install --cask docker
```

会下载桌面版本的docker，之后按照提示运行即可

## 初步了解

镜像：类似于虚拟机的镜像，提供执行环境，比如go语言的镜像，就是提供go语言的运行环境

容器：镜像通过执行run方法可以得到一个容器，比如是ubuntu环境，可以在容器里安装RabbitMQ，ES环境。

仓库：拉取镜像的地方

Dockerfile：文本文件（脚本），包含很多执行指令，用于构建自己的镜像，可用于扩展官方镜像。相当于docker image build 的构建镜像的源代码

从仓库（一般为DockerHub）下载（pull）一个镜像，Docker执行run方法得到一个容器，用户在容器里执行各种操作。Docker执行commit方法将一个容器转化为镜像。Docker利用login、push等命令将本地镜像推送（push）到仓库。其他机器或服务器上就可以使用该镜像去生成容器，进而运行相应的应用程序了。

## 常用指令

### 镜像image相关

- `docker build` 

- `docker image pull` 拉取镜像

- `docker image ls` 正在使用的镜像 (`docker images`)

- `docker image ls --all` 全部镜像

- `docker rmi <IMAGEID>`删除镜像，如果该镜像被容器引用了不能完全删除，需要先删除容器

- `docker create -it <镜像名>`  通过镜像创建container

- `docker run -it <镜像名>` 通过镜像创建并运行container 

  `-p 8080:8080` -p表示端口映射，格式：`宿主机端口：容器运行端口`
  
  `-e MYSQL_ROOT_PASSWORD=123456 mysql`-e表示添加环境变量，这里是mysql的密码。
  
  `--name <name>` 将容器命名为

### 容器container相关

- `docker container ls`  正在运行的container

- `docker container ls --all`  已经使用完的container 

- `docker ps`查看正在运行的容器

- `docker ps -a`  查看已经退出的容器 

- `docker run -d -it --name centos.7.2 centos:7`  -d在后台运行  -it 值得是保持stdout以及分配终端

  `-i` 进入container内部

- `docker exec -it 容器名称 (或者容器ID)  /bin/bash`

  在宿主机上进入容器，登录守护式容器

- `docker logs 镜像名` 查看日志

- `docker container kill id` 杀死正在使用的container 

- `docker start/stop <CONTAINERID>` 启动刚create的或者已经停止的container

- `docker (container) rm id` 删除使用完的container 防止占用内存 

- `docker network create test-network` 容器通信

  --network 名字

push到dockerhub

- `docker login`登录dockerhub账号
- `docker push <imageName>`

### Dockerfile

## 拉取MySQL过程

1. 拉取镜像 `docker pull mysql`

2. `docker images`查看是否成功

3. `docker run -d <镜像名> -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql -v /var/lib/mysql:/var/lib/mysql  --name <name> `启动容器

   ```sh
   --name mysql 
   表示将这个容器命名为mysql
   
   -v /var/lib/mysql:/var/lib/mysql 
   表示将宿主机的 /var/lib/mysql 卷映射到容器里的 /var/lib/mysql 卷中，这里是为了我们能够把这个数据保存在宿主机中，防止容器删掉就没了。
   
   -e MYSQL_ROOT_PASSWORD=root 
   表示MySQL的密码这里设置了root
   
   -p 3306:3306
   将宿主机的3306端口映射到容器的3306端口
   
   -d 
   后台运行
   
   mysql:8.0 
   使用mysql:8.0这个镜像
   ```

   

4. `docker ps`查看启动是否成功

5. `docker exec -it robking_mysql<容器名称>  /bin/bash` 在宿主机上进入容器，登录守护式容器

6. `mysql -u root -p`登录mysql

   ![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/Git/20220914010551.png)

7. 查看mysql版本

   ```
   -- 第一种
   select version();
   --第二种
   select @@version; 
   ```

8. 获取docker主机ip，一般应该用第二个吧

   ![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/Git/20220913232347.png)

```
docker run --name robking_redis -v /usr/local/redis:/usr/local/redis -p 6379:6379 -d redis:latest
```

## Docker部署`Gin+MySQL+Redis`项目

### MySQL

1. `docker pull mysql`拉取mysql镜像

2. `docker run --name mysql -v /var/lib/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -d mysql`

   启动mysql容器，具体使用参考上面

### Redis

1. `docker pull redis`拉取redis镜像

2. `docker run --name redis -v /usr/local/redis:/usr/local/redis -p 6379:6379 -d redis`

   启动redis容器，通过 `docker ps`查看容器正在运行	

### mall(项目)

1. 制作项目 `Dockerfile`

   ```dockerfile
   FROM golang:lasest
   
   ENV GO111MODULE=on \
       GOPROXY=https://goproxy.cn,direct
   
   WORKDIR /app
   COPY . .
   RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build  -ldflags="-w -s" -o main
   RUN mkdir publish  \
       && cp main publish  \
       && cp -r conf publish
   
   # 指定运行时环境变量
   ENV GIN_MODE=release
   EXPOSE 3000
   
   ENTRYPOINT ["./main"]
   ```

2. 修改项目的配置文件，这个配置文件里面主要包括

   mysql的ip:port; redis的ip:port; mysql连接信息包括账号密码，使用的数据库

3. `docker build -t mall:1.0 .`将我们的项目通过Dockerfile制作成镜像

   此时会去下载golang环境(镜像)，编译我们的程序，下载相关的依赖，指定暴露的端口

   成功的标志是每一步都会返回一串id

   ![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/Git/20220914021328.png)

4. `docker run --name mall -p 3000:3000 -d mall:1.0`运行这个镜像

   在后面加上 `--rm`将会在容器结束运行时自动删除

   我的理解是相当于我们在本地启动项目

5. 如果启动失败出现数据库连接失败或者忘记建数据库了那么就需要**重新删除项目的镜像**，**改动我们的代码**，**重新通过Dockerfile制作成镜像**，**然后重新运行这个镜像**(启动项目)

### Docker-compose进行容器的管理

当我们有多个容器需要启动的时候，我们可以用`docker-compose.yml`进行容器的管理

此时我们已有的东西是：mysql，redis，mall(项目)三个镜像，但是都没有启动，也就是说此时的**mysql没有设置密码，数据库**

```yml
version: '2'

services:
  civil:
    build: ./
    image: mall:1.0
    container_name: robking_mall
    restart: always
    environment:
      MYSQL_DSN: "root:123456@tcp/test?charset=utf8&parseTime=True&loc=Local"
    ports:
      - 9999:9999
    depends_on:
      - mysql
      - redis
      
  mysql:
    container_name: robking_mysql
    image: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: test
    volumes:
      - /var/lib/mysql:/var/lib/mysql
    ports:
      - 3306:3306

  redis:
    container_name: robking_redis
    image: redis
    restart: always
    volumes:
      - /usr/local/redis:/usr/local/redis
    ports:
      - 6379:6379
```

这个文件其实就是将三个镜像运行成容器的指令用文件的方式进行配置，然后运行起来，项目是依赖mysql和redis，所以会先让这两个先启动，然后再启动项目容器

`docker-compose -f docker-compose.yml up -d`创建并启动三个容器

![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/Git/20220914022007.png)
