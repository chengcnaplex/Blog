---
title: 基于Centos 6 x86的vsftpd的搭建和配置
date: 2016-12-02 14:12:56
tags: 
  - 服务器
  - FTP
categories:
  - 服务器
---

1. 安装ftp服务器vsftpd

		sudo yum install vsftpd

2. 添加用户

		adduser userftp
		passwd userftp
<!-- more -->
3. 禁止用户的 ssh 登录权限，只允许 FTP 访问:

		usermod -s /sbin/nologin userftp

4. 配置 VSFTP

		anonymous_enable=NO

5. 配置文件关系图：

	[](\image\blog_5\centos_vsftpd.png)