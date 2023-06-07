---
title: "Meicai_web项目"
layout: post
date: 2022-08-06
image: /assets/images/markdown.jpg
headerImage: false
tag:
- 项目实践
category: 项目实践
---

## Meicai_web项目实践并部署到服务端

> 1.软件准备：pycharm,服务器(选择)  
> 2.利用Django写的项目  
> 3.需要了解HTML相关语法，会使用python

## 项目笔记

> 1. 如果一直无法连接到数据库（ [ERROR/MainProcess] consumer: Cannot connect to redis://172.0.0.1:6379/8: Error 10060 connecting to 172.0.0.1:6379.由于连接方在一段时间后没有正确答复或连接的主机没有反应，连接尝试失败。..Trying again in 10.00 seconds... (5/100)）  
可能是地址没配置对broker="redis://127.0.0.1:6379/8"，或者通过（1）启动redis（2）切换到redis安装目录并执行redis-server.exe redis.windows.conf（4）再开一个后台，切换到安装目录，执行redis-cli.exe，然后在执行 config set requirepass root 将密码设置为 root ，然后地址配置为broker="redis://:root@127.0.0.1:6379/8"即可

> 2. 切换到redis安装目录下,redis-server.exe redis.windows.conf,启动redis服务器  
> 3. 切换到redis安装目录下,redis-cli.exe,启动客户端  
> 4. celery -A celery_tasks.tasks worker -l info 启动celery服务   
> 5. 如果报错  

	tasks, accept, hostname = _loc ValueError: not enough values to unpack (expected 3, got 0) 

> 解决方法：  

	# 安装这个包
	pip install eventlet
	# 用这个启动
	celery -A celery_tasks.main worker -l info -P eventlet
	celery -A celery_tasks.tasks worker -l info -P eventlet
	
> 东京arm重启方法
	
	1. 打开所有端口    

	iptables -P INPUT ACCEPT
	iptables -P FORWARD ACCEPT
	iptables -P OUTPUT ACCEPT
	iptables -F
	
	2.打开天天生鲜项目的fastdfs文件   
	
	先启动 /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf start  
	再启动 /usr/bin/fdfs_storaged /etc/fdfs/storage.conf start   

	3.再切换到/usr/local/nginx/sbin中打开控制上传的nginx服务  

	/nginx 开启Nginx服务  

	4.打开redis数据端口6379    

	cd /usr/local/redis    
	redis-server redis.conf    

	5.重启控制 天天生鲜转接的 uwsgi服务器  

	cd /root/MeiCai_web/Meicai_web/dailyfresh/dailyfresh    
	uwsgi –-ini uwsgi.ini   

	6.重启supervisor     

	echo_supervisord_conf > /Supervisord/supervisord.conf   
	supervisord -c /Supervisord/supervisord.conf 






#### 1、下载pycharm并配置好python
#### 2、下载Django
> 在pycharm的终端或工具栏导入Django  

![](https://raw.githubusercontent.com/zhuoyue2/zhuoyue2.github.io/master/assets/images/Meicai_web_Detail/Term_stail.png)

![](https://raw.githubusercontent.com/zhuoyue2/zhuoyue2.github.io/master/assets/images/Meicai_web_Detail/Find_tool.png) 

> 如果下载好Django,但是却导入不了Django包,Windows电脑需要将Django包和Scripts的路径加入的电脑的环境变量中  
	（前面每个人的路径不一样，重点是找到LocalCache\local-packages\Python38\Scripts和site-packages\django)

	C:\Users\94284\AppData\Local\Packages\PythonSoftwareFoundation.Python.3.8_qbz5n2kfra8p0\LocalCache\local-packages\Python38\Scripts  

	C:\Users\94284\AppData\Local\Packages\PythonSoftwareFoundation.Python.3.8_qbz5n2kfra8p0\LocalCache\local-packages\Python38\site-packages\django

#### 3.项目搭建

