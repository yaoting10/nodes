1.搜索redis资源

```dockerfile
docker search redis
```

2.拉取redis镜像,选择stars高分的即可

```dockerfile
docker pull redis:5.0
```

3.查看拉取的redis镜像

```dockerfile
docker images
```

**4.准备redis的一些配置文件**

首先在/root/redis/data 创建好文件夹用于存放redis数据，这个文件夹位置也可以自己选。

然后在/root/redis/ 创建好redis.conf文件。用户redis的配置。redis.conf可以从redis官网下载 然后启动的时候导入redis的配置文件，就可以按照配置来启动了。

```shell
mkdir /root/redis mkdir /root/redis/data
```

5.下载reids配置文件

```shell
wget https://raw.githubusercontent.com/antirez/redis/5.0/redis.conf -O redis.conf
```

**6.redis配置文件修改**

[  redis配置文件详解](https://www.cnblogs.com/kreo/p/4423362.html)

**我的配置是直接注释掉bind**

```sh
protected-mode yes
```

  其他配置未改动

启动过程中遇到了一个问题所以修改了一下 

echo 511 > /proc/sys/net/core/somaxconn

我用的是vm 为了链接虚拟机中的docker 在网上查了一下

  **在Windows宿主机中连接虚拟机中的Docker容器**

具体可以查看

[  https://www.cnblogs.com/linux-wangkun/p/5840441.html](https://www.cnblogs.com/linux-wangkun/p/5840441.html)

**7.docker 启动redis**

```dockerfile
docker run --name myredis -p 6379:6379 -d redis:latest redis-server docker run --name myredis --restart=always -i -t -p 192.168.31.131:6379:6379 -d redis:latest redis-server docker run -p 6379:6379 -v /home/zmkj/dev/redis/data:/data -v /home/zmkj/dev/redis/conf/redis.conf:/etc/redis/redis.conf  --name redis --restart=always -d redis:5.0 redis-server /etc/redis/redis.conf
```

  -p 6379:6379:把容器内的6379端口映射到宿主机6379端口

  -v /root/redis/redis.conf:/root/redis/redis.conf：把宿主机配置好的redis.conf放到容器内的这个位置中

  -v /root/redis/data:/data：把redis持久化的数据在宿主机内显示，做数据备份

  redis-server /etc/redis/redis.conf：这个是关键配置，让redis不是无配置启动，而是按照这个redis.conf的配置启动

  –appendonly yes：redis启动后数据持久化

------

 **启动过程中发现 执行后docker ps 查不到redis 解决方法**

  --privileged=true 增加权限

如果出现重复name 使用

```
docker ps -a
```

查看容器

```
docker rm <containerid/names>
```

**移除镜像**

```
/usr/bin/docker-current: Error response from daemon: Conflict. The container name "/myredis" is  already in use by container  8601c55c4aa0965d23c0849aab0a1d49935b1a9ff7232650999b54fe5f2aa043. You have to remove (or rename) that container to be able to reuse that name..   See '/usr/bin/docker-current run --help'.
```

8.查看启动

```
docker ps
```

9.进入redis

```
docker exec -ti myredis redis-cli -h localhost -p 6379
```

获取docker

```
docker pull redis:5.0
```

启动docker 

```
$ docker run -itd --name redis -p 16379:6379 redis
```

