---
title: "记录Python一些代码及方法"
layout: post
date: 2022-11-02
image: /assets/images/markdown.jpg
headerImage: false
tag:
- 笔记
category: 笔记
---

##	Python方法记录  

1. 开启端口号  

	iptables -I INPUT -p tcp --dport 5244 -j ACCEPT

2. 开启全部端口号

	iptables -P INPUT ACCEPT  
	iptables -P FORWARD ACCEPT  
	iptables -P OUTPUT ACCEPT  
	iptables -F

3. nginx下载相关

	下载nginx sudo apt-get install nginx  
	nginx开启 sudo service nginx start  
	nginx重启 sudo service nginx reload  
	nginx关闭 sudo service nginx stop  
	nginx看状态 sudo service nginx status  
	nginx看能否正确执行 sudo nginx -t

4. nginx配置

	```python
	# Default server configuration
	# 重定向
	server {
	 listen 80;
	 server_name 123.kangbingjie.com;
	 #return 301 123.kangbingjie.com$request_uri;
	 rewrite ^ https://$http_host$request_uri? permanent;
	}
	server {
	 server_name 123.kangbingjie.com;
	 location / {
	    proxy_set_header X-Forwarded-For 		$proxy_add_x_forwarded_for;
	    proxy_set_header X-Forwarded-Proto $scheme;
	    proxy_set_header Host $http_host;
	    proxy_set_header X-Real-IP $remote_addr;
	    proxy_set_header Range $http_range;
	    proxy_set_header If-Range $http_if_range;
	    proxy_redirect off;
	    proxy_pass http://127.0.0.1:5244;
	 }
	 
	 # https协议证书
	 root /var/www/html;
	 listen [::]:443 ssl ipv6only=on; # managed by Certbot
	 
	 listen 443 ssl; # managed by Certbot
	 
	 ssl_certificate /etc/letsencrypt/live/123.kangbingjie.com/fullchain.pem; # managed by Certbot
	 
	 ssl_certificate_key /etc/letsencrypt/live/123.kangbingjie.com/privkey.pem; # managed by Certbot
	 
	 include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
	 
	 ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; 	
	 # managed by Certbot
	}
	
	# Virtual Host configuration for example.com
	# You can move that to a different file under sites-available/ and symlink that
	# to sites-enabled/ to enable it.
	# 默认端口
	
	server {
	 listen 80 default_server;  
	 location / {
	    try_files $uri $uri/ =404;
	 }
	}
	```

5.  uwsgi配置


	```
	#uwsgi.ini
	[uwsgi]
	socket=:8000  # Nginx 
	#http = :8000 # http
	chdir = /home/kangbing/mmsite
	wsgi-file = mmsite/wsgi.py
	module = mmsite.wsgi
	processes = 4
	threads = 2
	master = True
	pidfile = uwsgi.pid
	daemonize = uwsgi.log
	```
	启动：uwsgi --ini uwsgi.ini  
	停止：uwsgi --stop uwsgi.pid  
	重启：uwsgi --reload uwsgi.pid  
	查看PID: ps aux | grep uwsgi  
	测试: uwsgi --http :8000 --module yourprojectname.wsgi  
	
6. ssl证书配置(和Nginx一起使用）

	安装：apt install certbot python3-certbot-nginx  
	生成证书：cerbot --nginx -d 网址不带http/https  
	到/etc/nginx/sites-enabled/default  
	Django的配置文件中配置（默认已配好）
	
7. pip 不能使用的解决方法
	
	卸载  
	sudo apt remove python3-pip  
	
	安装先下载  
	wget https://bootstrap.pypa.io/get-pip.py  
	
	安装  
	sudo python3 get-pip.py
	
	这时候执行 python3 -m pip --version  提示找不到  
	hash -r 清空缓存  
	pip3 -V  再执行pip3 的命令就可以了  
	
	卸载 OpenSSL  
	rm -rf /usr/lib/python3/dist-packages/OpenSSL
	
	重新安装  
	pip3 install pyopenssl
	
	升级到最新版  
	pip3 install pyopenssl --upgrade
	
8. 局部刷新 ajax  
9. supervisor 管理进程  
	重启：supervisorctl reload  
	查看状态：supervisorctl status  
	启动：supervisord  
	第一次启动：supervisord -c 路径  
	关闭进程：supervisorctl stop 子进程或all  
	开启进程：supervisorctl start 子进程或all  
	include包含的需要执行的 用空格隔开，command后面接着写不要有空格

		


	
	
	
