初始化

```shell
git init
```

创建文件

```shell
touch README
```

添加文件

```sh
git add README
```

提交文件

```sh
git commit -m 'first commit'
```

关联分支

```shell
git remote add origin git@127.0.0.1:devteam/common-util.git
```

更新远程到本地。

```sh
git pull origin master
```

提交本地到远程

```sh
git push -u origin master
```

更新某个分支

```sh
git pull origin active_packet
```

git 切换本地分支

```sh
git checkout -b origin/user_score
```

新建tag，切换到需要打tag的分支上

```sh
git tag v20220511
```

查看tag

```sh
git tag
```

代码量统计

```sh
git log --author="Tim" --pretty=tformat: --numstat | awk '{ add += $1; subs += $2; loc += $1 - $2 } END { printf "added lines: %s, removed lines: %s, total lines: %s\n", add, subs, loc }' -
```

切换流到master

```sh
git branch --set-upstream-to=origin/master
```

error: Your local changes to the following files would be overwritten by merge: xxx/xxx/xxx.java Please, commit your changes or stash them before you can merge. Aborting              

1.stash

通常遇到这个问题，你可以直接commit你的修改；但我这次不想这样。

看看git stash是如何做的。

```sh
git stash git pull git stash pop
```

2.放弃本地修改，直接覆盖之

```sh
git reset --hard git pull
```

生成ssh key

```sh
ssh-keygen -o
```

生成带邮箱的

```sh
ssh-keygen -t rsa -C "youremail@example.com"
```

生成RSA的密钥

```sh
ssh-keygen -m PEM -t rsa -b 2048 -C "your_email@example.com" 
```

git clone 命令：

```sh
git@ali_server:/home/git/git_repository/my_material_sync
```

设置代理:

```sh
git config --global http.proxy 'socks5://127.0.0.1:1080' git config --global https.proxy 'socks5://127.0.0.1:1080'
```

取消代理：

```sh
git config --global --unset http.proxy git config --global --unset https.proxy
```

