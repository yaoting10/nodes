**1、安装Git**

```shell
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-devel 
yum install git
```

 创建一个git用户组和用户，用来运行git服务：

```sh
groupadd git 
useradd git -g git
```

**2、创建证书登录**

```sh
cd /home/git/ 
mkdir .ssh 
chmod 755 .ssh 
touch .ssh/authorized_keys 
chmod 644 .ssh/authorized_keys
```

**3、初始化Git仓库**

```sh
cd /home
mkdir git_repository 
chown git:git git_repository/ 
cd git_repository 
git init --bare my_dir.git 
Initialized empty Git repository in /home/git_repository/my_dir.git/
```

以上命令Git创建一个空仓库，服务器上的Git仓库通常都以.git结尾。然后，把仓库所属用户改为git：

```sh
chown -R git:git my_dir.git
```

**4、克隆仓库**

```sh
$ git clone git@192.168.45.4:/home/git_repository/my_dir.git 
Cloning into 'my_dir'... 
warning: You appear to have cloned an empty repository. 
Checking connectivity... done.
```

