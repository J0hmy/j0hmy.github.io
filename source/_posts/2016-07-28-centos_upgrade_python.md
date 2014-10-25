---
layout: post
title: CentOS 升级 Python 整理记录
date: 2016-07-28 09:46:14
categories: Unclassified
---

CentOS 升级 Python 整理记录

	CentOS 版本: 6.0 (x64)
	Python 原版本： 2.6.5
	Python 升级版本：2.7.12
	Python 2.7.12 链接： https://www.python.org/ftp/python/2.7.12/Python-2.7.12.tar.xz

执行过程

	# 预先准备
	cd /usr/bin
	python --version

	# 拷贝 python2.6.5 副本
	cp python python2.6.5  # 安全起见(1)

	# 修改下述三个文件开头, 指定 python 解释器为 /usr/bin/python2.6.5
	# 原因是它们不支持 python27
	vim /usr/bin/yum
	vim /usr/bin/ibus-setup
	vim /usr/libexec/ibus-ui-gtk

	# 下载安装
	wget https://www.python.org/ftp/python/2.7.12/Python-2.7.12.tar.xz
	tar xf Python-2.7.12.tar.xz
	cd Python-2.7.12
	./configure
	make all
	make install
	make clean
	make distclean
	/usr/local/bin/python2.7 --version
	rm /usr/bin/python # 呼应上文命令(1)
	ln -s /usr/local/bin/python2.7 /usr/bin/python

	# 安装后
	## 修复 或 安装 pip
	yum install python-pip
	pip install --upgrade setuptools
	wget --no-check-certificate  https://raw.github.com/pypa/pip/master/contrib/get-pip.py # 命令(3)
	wget --no-check-certificate  https://bootstrap.pypa.io/get-pip.py                      # 命令(4)

	# 尝试运行 命令 (3, 4) 下载的脚本, 至此 pip 修复完成 
	python get-pip.py
	pip --version
	pip install bs4
	pip install requests

	# 小工具
	yum install dos2unix lrzsz
