---
layout: post
title: Python Tree
date: 2015-1-20 07:23:41
tags: Python
categories: 未分类
---

	#!/usr/bin/env python
	#    -*-coding: utf-8-*-     
	#    File Name: tree
	#       Author: johzzy
	#        Email: hellojinqiang@gmail.com
	# Created Time: Tue 20 Jan 2015 10:17:03 PM CST

	import sys
	import os

	DEBUG = False
	CLEAN = False
	ALL = False

	def lsall(dir, space = ""):
		dirname = dir
		global CLEAN
		if CLEAN and not os.listdir(dirname):
			os.rmdir(dirname)
			return

		dirs = filter(lambda x: ALL or x[0] != '.', os.listdir(dirname))
		for item in dirs:
			if os.path.isdir(os.path.join(dirname, item)):
				subspace = space + "|  "
				print "%s%s" % (subspace, item)
				lsall(os.path.join(dirname, item), subspace)
			else:
				subspace = space + "|--"
				print "%s%s" % (subspace, item)


	def main(args):
		for arg in filter(lambda x: os.path.exists(x), args):
			print arg
			lsall(arg, "")

	def help():
		print '''
	Usage:
	1) tree 
		default dir is your home directory
	2) tree demodir
		print all files in demodir
	3) tree demo1 demo2 demo3 ...
		print all files in these directories
	4) tree --help or tree -h
		print tree Usage
	5) tree --clean or tree -c
		print all file(s) in directory(s) and delete empty directory

	version: 0.03
	 author: Johnny Wong
	   date: 2015-01-20
		'''

	if __name__ == "__main__":
		if len(sys.argv) > 1:
			args = sys.argv[1:]
		else:
			args = [os.getcwd()]

		if '--help' in args or '-h' in args:
			help()
			exit(0)

		if '--clean' in args:
			CLEAN = True
			args.remove('--clean')
		if '-c' in args:
			CLEAN = True
			args.remove('-c')

		if '--all' in args:
			ALL = True
			args.remove('--all')
		if '-a' in args:
			ALL = True
			args.remove('-a')

		if DEBUG:
			args = ['/home/johzzy/courses']

		main(args)

