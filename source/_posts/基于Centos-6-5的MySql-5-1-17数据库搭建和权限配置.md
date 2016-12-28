---
title: 基于Centos_6.5的MySql_5.1.17数据库搭建和权限配置
date: 2016-12-08 11:39:01
tags: 
  - 服务器
  - mysql
categories:
  - 服务器
---

## 参考文档
	
1. 服务器搭建：
	
		http://www.cnblogs.com/zengjfgit/p/5897385.html

2. 权限报错：`ERROR 2003 (HY000): Can't connect to MySQL server on '192.168.1.249' (111)`
	 
		http://www.cnblogs.com/helloworldtoyou/p/6029041.html
	
3. 忘记密码的解决方法：
	
		http://alsww.blog.51cto.com/2001924/1121676

<!-- more -->
## 搭建服务器

1. 安装

		yum -y install mysql-server

2. 设置mysql随系统启动

		chkconfig mysqld on

3. 确认mysql自启动

		chkconfig --list mysqld

4. 启动mysql

		/etc/init.d/mysqld start

5. 设置root密码

		mysqladmin -u root password "your passwd"

6. 忘记密码
	
		#先关闭mysql服务
		/etc/init.d/mysqld stop

		#以命令行参数启动mysql：
		mysqld_safe --user=mysql --skip-grant-tables --skip-networking & 

		#用mysql用户登录		
		mysql -u root mysql 
		
		#重新设置密码
		UPDATE user SET Password=PASSWORD(’newpassword’) where USER=’root’; 
		
		#刷新权限
		FLUSH PRIVILEGES;

		#退出 
		quit

		#启动 
		/etc/init.d/mysqld restart 
		
		#输入账户密码
		mysql -uroot -p 
		Enter password: <输入新设的密码newpassword> 


## 远程登录

1. 输入命令测试远程连接：

		mysql -h 127.0.0.1 -u root -p

2. 报错：
 
 		ERROR 2003 (HY000): Can't connect to MySQL server on '192.168.1.249' (111)

3. 解决：

		http://www.cnblogs.com/helloworldtoyou/p/6029041.html
			