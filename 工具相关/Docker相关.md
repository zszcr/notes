## Docker相关

主要记录以下docker常见命令的用法，以及docker-compose的用法



```bash
# 基本操作
docker run -d -p 物理端口1:容器端口1 -p 物理端口2:物理端口2 --name 容器名 <image-name>:<tag>
docker exec -it 容器名/ID bash

# 磁盘挂载
docker run -d -p 8080:80 -v 本机路径:容器路径 --name 容器名  <image-name>:<tag>

# 容器打包镜像
docker commit -a "作者" -m "备注" 容器ID <image-name>:<tag>

# 物理机拷贝到容器
docker cp test.txt 容器ID:/var/www/html

# 容器拷贝到物理机
docker cp 容器ID:/var/www/html/test.txt 物理机路径

# 查看容器 COMMAND
 docker ps -a --no-trunc

# 停止所有容器 以此类推
docker stop $(dokcer ps -aq)

# 将容器打包成规范的镜像
docker commit <exiting-Container> <hub-user>/<repo-name>[:<tag>]

# 将镜像修改成规范的镜像
docker tag <existing-image> <hub-user>/<repo-name>[:<tag>]

# 登录 Docker Hub
docker login

# 上传推送镜像到公共仓库
docker push <hub-user>/<repo-name>:<tag>

# 当前目录的 Dockerfile 创建镜像
docker build -t <image-name>:<tag> . 

# 指定文件构建镜像
docker build -f /path/to/a/Dockerfile -t <image-name>:<tag> .

# 将镜像保存 tar 包
docker save -o image-name.tar <image-name>:<tag>

# 导入 tar 镜像
docker load --input image-name.tar

# docker-compose 命令相关
## 基本操作
docker-compose up -d

## 关闭并删除容器
docker-compose down

## 开启|关闭|重启已经存在的由docker-compose维护的容器
docker-compose start|stop|restart

## 运行当前内容，并重新构建
docker-compose up -d --build
```