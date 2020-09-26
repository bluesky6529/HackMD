# Docker basic command

## 基本指令
### 登入docker registry
> 輸入完才會有反應
```
docker login
```




## 新增刪除相關

### 列出所有的容器 ID
```
docker ps -aq
```
### 停止所有的容器
```
docker stop $(docker ps -aq)
```
### 删除所有的容器
```
docker rm $(docker ps -aq)
```
### 删除所有的镜像
```
docker rmi $(docker images -q)
```
