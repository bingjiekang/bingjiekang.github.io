---
title: "Note_record"
layout: post
date: 2022-08-8
image: /assets/images/markdown.jpg
headerImage: false
tag:
- 笔记
category: 笔记
---

##	笔记记录

> 1. python setup.py install报错“error: can‘t create or remove files in install directory”

	error: can't create or remove files in install directory

	The following error occurred while trying to add or remove files in the
	installation directory:

	    [Errno 13] Permission denied: 'C:\\Program Files\\WindowsApps\\PythonSoftwareFoundation.Py
	thon.3.8_3.8.2800.0_x64__qbz5n2kfra8p0\\Lib\\site-packages\\test-easy-install-15020.write-test
	'

	The installation directory you specified (via --install-dir, --prefix, or
	the distutils default setting) was:

	    C:\Program Files\WindowsApps\PythonSoftwareFoundation.Python.3.8_3.8.2800.0_x64__qbz5n2kfr
	a8p0\Lib\site-packages\

	Perhaps your account does not have write access to this directory?  If the
	installation directory is a system-owned directory, you may need to sign in
	as the administrator or "root" account.  If you do not have administrative
	access to this machine, you may wish to choose a different installation
	directory, preferably one that is listed in your PYTHONPATH environment
	variable.

	For information on other options, you may wish to consult the
	documentation at:

	  https://setuptools.readthedocs.io/en/latest/easy_install.html

	Please make the appropriate changes for your system and try again.


> 解决方法：改为 python setup.py install --user || exit 1
