## Linux系统安装MySQL流程

### 一、安装前准备
1、检查是否已经安装mysql
```
[root@localhost /]# rpm -qa | grep mysql
```
如果已经安装，执行删除命令
```
rpm -e --nodeps mysql-libs-5.1.73-5.el6_6.x86_64(根据自己安装版本卸载)
```
再次执行检查，是否成功删除
2、然后查看mysql相关文件夹
```
[root@localhost /]# whereis mysql
```
删除相关目录或文件
```
[root@localhost /]#  rm -rf /usr/bin/mysql /usr/include/mysql /data/mysql /data/mysql/mysql 
```
验证是否删除完毕
```
[root@localhost /]# whereis mysql
mysql:
[root@localhost /]# find / -name mysql
[root@localhost /]# 
```
3、检查mysql用户组和用户是否存在，如果没有，则创建
```
[root@localhost /]# cat /etc/group | grep mysql
[root@localhost /]# cat /etc/passwd |grep mysql
[root@localhost /]# groupadd mysql
[root@localhost /]# useradd -r -g mysql mysql
[root@localhost /]# 
```
4、从官网下载是用于Linux的Mysql安装包
```
[root@localhost /]#  wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
```

### 二、安装mysql
1、在执行wget命令的目录下或你的上传目录下找到Mysql安装包：mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
执行解压命令：
```
[root@localhost /]#  tar xzvf mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
[root@localhost /]# ls
mysql-5.7.24-linux-glibc2.12-x86_64
mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
```
解压完成后，可以看到当前目录下多了一个解压文件，移动该文件到/usr/local/下，并将文件夹名称修改为mysql。
执行命令如下：
```
[root@localhost /]# mv mysql-5.7.24-linux-glibc2.12-x86_64 /usr/local/
[root@localhost /]# cd /usr/local/
[root@localhost /]# mv mysql-5.7.24-linux-glibc2.12-x86_64 mysql
```
如果/usr/local/下不存在mysql文件夹，直接执行如下命令，也可达到上述效果。
```
[root@localhost /]# mv mysql-5.7.24-linux-glibc2.12-x86_64 /usr/local/mysql
```
2、在/usr/local/mysql目录下创建data目录
```
[root@localhost /]# mkdir /usr/local/mysql/data
```
3、更改mysql目录下所有的目录及文件夹所属的用户组和用户，以及权限
```
[root@localhost /]# chown -R mysql:mysql /usr/local/mysql
[root@localhost /]# chmod -R 755 /usr/local/mysql
```
4、编译安装并初始化mysql,务必记住初始化输出日志末尾的密码（数据库管理员临时密码）
```
[root@localhost /]# cd /usr/local/mysql/bin
[root@localhost bin]# ./mysqld --initialize --user=mysql --datadir=/usr/local/mysql/data --basedir=/usr/local/mysql
```
如果初始化失败，提示libaio文件不存在，则安装libaio
```
#yum -y install numactl
#yum search libaio
#yum install libaio
```
执行无误之后，再重新执行第4步初始化命令，无误之后再进行第5步操作！

5、运行初始化命令成功后，输出记录日志最末尾位置**root@localhost:后的字符串**，此字符串为mysql管理员临时登录密码。

6、编辑配置文件my.cnf，添加配置如下
```
[root@localhost bin]#  vi /etc/my.cnf

[mysqld]
datadir=/usr/local/mysql/data
port = 3306
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
symbolic-links=0
max_connections=400
innodb_file_per_table=1
#0：表名存储为给定的大小和比较是区分大小写的  1：表名存储在磁盘是小写的，但是比较的时候是不区分大小写 2：表名存储为给定的大小写但是比较的时候是小写的
lower_case_table_names=1
```
7、启动mysql服务器
```
[root@localhost /]# /usr/local/mysql/support-files/mysql.server start
```
如果显示如下信息：
>ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
查看/tmp文件夹下是否存在mysql.sock文件，不存在的话搜索mysql.sock文件位置
```
whereis mysql.sock
```
发现在/var/lib/mysql/mysql.sock，于是建立一个软连接：
```
ln -s /var/lib/mysql/mysql.sock /tmp/mysql.sock
```
如果显示如下信息：
>Starting MySQL.The server quit without updating PID file (/[FAILED]mysqld/mysqld.pid).

这次碰到的是PID文件所在文件夹没有赋予用户权限,根据错误提示，进入pid文件所在目录，给用户权限
```
[root@localhost /]# chown -R mysql:mysql pid文件所在目录
[root@localhost /]# chmod -R 755 pid文件所在目录
```
显示log日志文件无法更新的错误，也是同理，缺少权限导致无法读写
```
[root@localhost /]# chown -R mysql:mysql log文件所在目录
[root@localhost /]# chmod -R 755 log文件所在目录
```
查看是否存在mysql和mysqld的服务，如果存在，则结束进程，再重新执行启动命令
```
#查询服务
ps -ef|grep mysql
ps -ef|grep mysqld

#结束进程
kill -9 PID

#启动服务
 /usr/local/mysql/support-files/mysql.server start
```
8、添加软连接，并重启mysql服务
```
[root@localhost /]#  ln -s /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql 
[root@localhost /]#  ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
[root@localhost /]#  service mysql restart
```
9、登录mysql，修改密码(密码为步骤5生成的临时密码)
```
[root@localhost /]#  mysql -u root -p
Enter password:
mysql>set password for root@localhost = password('yourpass');
```
10、开放远程连接
```
mysql>use mysql;
msyql>update user set user.Host='%' where user.User='root';
mysql>flush privileges;
```
11、设置开机自动启动
```
#1、将服务文件拷贝到init.d下，并重命名为mysql
[root@localhost /]# cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
#2、赋予可执行权限
[root@localhost /]# chmod +x /etc/init.d/mysqld
#3、添加服务
[root@localhost /]# chkconfig --add mysqld
#4、显示服务列表
[root@localhost /]# chkconfig --list
```
