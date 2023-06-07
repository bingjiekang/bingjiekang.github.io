---
title: "Linux服务器的Golang环境搭建及项目部署"
layout: post
date: 2023-02-19
image: /assets/images/markdown.jpg
headerImage: false
tag:
- golang
category: 笔记
---

##	Linux服务器的Golang环境搭建及项目部署 


#### Linux服务器Golang环境搭建

1.查看主机架构名称   

	uname -m    
	
2.到国内地址去下载对应文件

	国内地址：https://studygolang.com/dl    
	官方地址：https://go.dev/dl/     

3.下载命令(github下载的话是 git clone)

	cd /opt   // 进入opt目录
	sudo wget https://xxxxxxx
	
4.解压go安装压缩包

	sudo tar -zxvf xxxxx
	
5.查看go版本是否安装成功(opt目录下执行)

	./go/bin/go version 
	安装成功则会显示信息 go version go1.17.5 linux/amd64
	
6.环境变量配置（不要加空格）
	
	在/etc/profile文件下（vim /etc/profile）   
	export GOROOT=/opt/go   // 安装golang的文件目录    
	export PATH=$GOROOT/bin:$PATH     // golang的环境资源目录     
	export GOPATH=$HOME/goprojects/        // 项目运行目录   
	
7.重载配置文件

	source /etc/profile   
	
8.就可以在各个目录下查看golang版本信息了

	go version   
	

#### Golang的项目部署

1.在创建的goproject目录下，下载包

	git clone xxxxxx.git
	
2.切换到main.go执行文件处

	go build -o xxxxx名字 main.go   
	
3.将打包后的文件执行以下命令，生成日志并执行

	nohup ./xxxx名字 >mainlog.log 2>&1 &    
	
4.查看是否运行成功
	
	ps -ef|grep xxxxx名字   

5.查看日志  
 
 	tail -f mainlog.log   
 
 
	
	

				
	




		






	