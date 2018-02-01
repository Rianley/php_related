## 镜像
### 搜索镜像
```
docker search ubuntu
```
### 下载镜像
```
docker pull ubuntu
```
### 列出镜像
```
docker images
```
### 删除镜像
```
docker rmi image-name
docker rmi -f image-name
```
### 删除全部镜像
```
docker rmi $(docker images -q)
```
## 容器
### 创建容器
```
// 以交互创建容器
docker run -it ubuntu:latest
// 以后台进程运行容易
docker run -d ubuntu:latest
// 指定参数 守护进程、命名、指定端口、指定目录
docker run -itd --name nginx -p 80:80 -p 443:443 -v /home/www:/usr/wwwroot  ubuntu:latest
```
### 删除容器
```
docker rm container-name
```
### 启动容器
```
docker start container-name
```
### 重启容器
```
docker restart container-name
```
### 停止容器
```
docker stop container-name
```
### 进入容器```bash```
```
docker exec -it container-name bash
```