1.搜索mysql镜像

```dockerfile
docker search mysql             
```

 

2.拉镜像

```dockerfile
docker pull mysql:8.0  
```

​            

3.运行mysql镜像

```dockerfile
docker run  --restart=always --name mysql -p 3306:3306 -v /usr/local/lib/data:/var/lib/mysql -v /usr/local/lib/conf.d:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD='zmkj123!@#' -e TZ=Asia/Shanghai -d mysql:8.0    
```

​          

```dockerfile
docker run --detach --restart=always --name=mysql -e MYSQL_ROOT_PASSWORD='nick123!@#' -p 3306:3306 --volume=/Users/nick/work/lib/mysql/conf.d:/etc/mysql/conf.d \ --volume=/Users/nick/work/lib/mysql/data:/var/lib/mysql \ mysql/mysql-server:8.0
```

```dockerfile
docker run  --restart=always --name mysql -p 3306:3306 -v /Users/nick/work/lib/mysql/data:/var/lib/mysql -v /Users/nick/work/lib/mysql/conf.d:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD='zmkj123!@#' -d mariadb
```

启动镜像

命令解析：

--name：容器名

--p：映射宿主主机端口

-v：挂载宿主目录到容器目录

-e：设置环境变量，此处指定root密码

-d：后台运行容器

4.查看运行mysql容器

```dockerfile
docker ps
```

 运行容器列表

5.进入容器内部

```dockerfile
docker exec -it 7a036187d7b9 /bin/sh        
```

6.连接mysql

```mysql
mysql -uroot -p
```

输入密码

登录

输入mysql查询语句

```mysql
mysql> select host,user,plugin,authentication_string from mysql.user;              
```

user列表

7.修改mysql的访问ip

'%'表示任何ip都可以访问

```mysql
mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';         
```

​     