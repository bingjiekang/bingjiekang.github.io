---
title: "使sublime支持markdown"
layout: post
date: 2020-08-06
image: /assets/images/markdown.jpg
headerImage: false
tag:
- 笔记
category: 笔记
---

## 使sublime支持markdown

# 主要内容:

	在sublime里安装Markdown Preview实现markdown在浏览器上的预览

#### 方法：通过package control安装（若已安装，则跳过）

	1、打开sublime，按 ctrl+shift+p 打开搜索行
	2、输入并选择 Package Control: Install Package 点击后等待出现新的搜索框
	3、在弹出的窗口里，输入 Markdown Preview 找到后，双击或回车完成安装


#### 配置快捷键:
	1、找到 Preferences -> Key Bindings 点击
	2、在xxx.sublime-keymap-User （用户快捷键配置文件）下把
	{ "keys": ["alt+m"], "command": "markdown_preview", "args": {"target": "browser", "parser":"markdown"} }
	复制到配置文件中保存
	3、配置完成后按 alt+m 显示效果


#### 代码高亮：
	1、通过 Preferences -> Package Settings -> Markdown Preview -> Settings（Settings-User） 来打开用户设置文件
	2、加入以下内容打开代码高亮功能
	{
	"enable_highlight": true
	}
	3、（选择）如果要使用更高级的高亮方式，可在用户设置文件下加入并更改如下即可
	{
	"enabled_extensions": [
	        "extra", 
	        "github", 
	        "toc", 
	        "headerid", 
	        "meta", 
	        "sane_lists", 
	        "smarty", 
	        "wikilinks",
	        "admonition",
	        "codehilite(guess_lang=False,pygments_style=emacs)"
	    ]
	}