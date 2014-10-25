---
layout: post
title: CentOS 升级 GCC 整理记录
date: 2016-07-28 09:45:50
categories: 未分类
---

CentOS 升级 GCC 整理记录

	CentOS 版本: 6.0 (x64)
	GCC 原版本： 4.4.7
	GCC 升级版本：4.8.5
	GCC 4.8.5 链接： https://ftp.gnu.org/gnu/gcc/gcc-4.8.5/gcc-4.8.5.tar.gz

执行过程

	wget https://ftp.gnu.org/gnu/gcc/gcc-4.8.5/gcc-4.8.5.tar.gz
	tar zxf gcc-4.8.5.tar.gz
	cd gcc-4.8.5

	yum install gcc g++
	yum install glibc-static
	yum install cloog-ppl gmp-devel

	wget ftp://gcc.gnu.org/pub/gcc/infrastructure/isl-0.11.1.tar.bz2
	tar jxf isl-0.11.1.tar.bz2
	cd isl-0.11.1
	./configure
	make
	make install
	
	cd ..
	./contrib/download_prerequisites
	mkdir build
	cd build
	../configure --prefix=/usr --enable-languages=c,c++ --disable-multilib
	make -j4
	make install

GCC 4.8.5 支持 C++11 的部分特性