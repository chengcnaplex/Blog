---
title: Ubantu操作系统下搭建FTP服务器
date: 2016-11-11 15:39:45
tags: 
  - 服务器
  - FTP
categories:
  - 服务器
---
## 安装

1. 安装ftp服务器vsftpd

		apt-get install vsftpd
		
2. 安装 存储虚拟用户的账户密码 的数据库

		apt-get install db4.8-util

<!-- more -->
## 配置

### 创建虚拟账户主目录

1. 在`/home/vsftpd/`路径下新建文件夹
		
		* cmz_ftp_home
		* chyq_ftp_home
		* tony_ftp_home 
		* zengjf_ftp_home

	命令如下：

		cd /home/  
		mkdir vsftpd  
		cd vsftpd  
		mkdir cmz_ftp_home chyq_ftp_home tony_ftp_home zengjf_ftp_home


### 创建虚拟用户数据库

#### 生成数据库

1. 在home目录下新建loguser.txt,并配置用户名密码  

 	命令如下：
	
		cd  /home
		sudo  touch  /home/loguser.txt
		sudo  gedit   /home/loguser.txt
		
		在loguser.txt编辑如下内容：
		cmz  -----用户名
		cmz  -----密码
		chyq  -----用户名
		chyq  -----密码
		tony  -----用户名
		tony  -----密码
		zengjf  -----用户名
		zengjf  -----密码
	
2. 通过命令把loguser.txt 生成用来保存虚拟用户的数据库

	命令如下：

		sudo db4.8_load -T -t hash -f /home/loguser.txt /etc/vsftpd_login.db

3. 最后设置一下数据库文件的访问权限

		sudo chmod 600 /etc/vsftpd_login.db

#### 配置PAM文件,把数据库关联到vsftpd上

1. 我们上一步建立的<font color=red>数据库 vsftpd_login </font>在此处被使用

	命令如下：
	
		cd /etc/pam.d
		sudo touch vsftpd.vu 
		gedit vsftpd.vu 
		
		编辑文本，如下：
		
		auth sufficient pam_userdb.so db=/etc/vsftpd_login
		account sufficient pam_userdb.so db=/etc/vsftpd_login
	
2. <font color=red>我们建立的虚拟用户将采用PAM进行验证</font>，这是通过/etc/vsftpd.conf文件中的 语句pam_service_name=vsftpd.vu来启用的，稍后你将发现。


### 为虚拟用户创建本地系统用户


1. 新建一个系统用户vsftpd，用户家目录为/home/vsftpd, 用户登录终端设为/bin/false(即使之不能登录系统)

2. 添加系统用户vsftpd 权限根目录/home/vsftpd 并不能登录系统

		sudo useradd vsftpd -d /home/vsftpd -s /bin/false
	
3. 设置/home/vsftpd 的owner vsftpd:vsftpd(user:group)
只有这个用户和组内用户能看到这个文件

		sudo chown vsftpd:vsftpd /home/vsftpd


### 文件配置

#### 配置vsftpd权限

1. 进入/etc/vsftpd.conf，一般要确保含有以下设置：

		listen=YES
		#匿名登录NO
		anonymous_enable=NO 
		dirmessage_enable=YES 
		xferlog_enable=YES 
		xferlog_file=/var/log/vsftpd.log 
		xferlog_std_format=YES 
		chroot_local_user=YES
		#虚拟用户登录YES
		guest_enable=YES
		#虚拟用户的本地用户名vsftpd
		guest_username=vsftpd
		#虚拟用户的权限配置文件
		user_config_dir=/etc/vsftpd_user_conf 
		#虚拟用户的关联数据库的配置文件
		pam_service_name=vsftpd.vu 
		#本地登录YES
		local_enable=YES 
		secure_chroot_dir=/var/run/vsftpd

#### 配置虚拟用户权限

1. 如果没有/etc/vsftpd_user_conf，创建vsftpd_user_conf目录

		sudo mkdir /etc/vsftpd_user_conf

2. 创建虚拟用户权限配置文件

		cd /etc/vsftpd_user_conf 
		sudo touch cmz chyq tony zengjf

3. cmz 内容

		local_root=/home/vsftpd/cmz

4. chyq 内容

		local_root=/home/vsftpd/chyq

5. tony 内容

		local_root=/home/vsftpd/tony 
	
6. zengjf 内容

		write_enable=YES
		anon_world_readable_only=NO 
		anon_upload_enable=YES 
		anon_mkdir_write_enable=YES 
		anon_other_write_enable=YES 
		local_root=/home/vsftpd/zengjf

### 重启服务器

	service vsftpd restart