---
layout: post
title: Python SSH 传输文件
date: 2015-1-14 07:23:41
tags: Python
categories: 未分类
---

	#!/usr/bin/env python
	#    -*-coding: utf-8-*-     
	#    File Name: ssh_test.py
	#       Author: johzzy
	#        Email: hellojinqiang@gmail.com
	# Created Time: Tue 13 Jan 2015 07:18:41 PM CST

	import pexpect  

	import subprocess

	class rsync_tool:
	    def __init__(self):
	        self.user = 'johnny'
	        self.host = '172.16.123.128'
	        self.remote_file = '/tmp/joke'
	        self.local_file = '/tmp/me'
	        self.passwd_file = '/tmp/johnny.pwd'

	        self.fhandler = open(self.passwd_file, 'r')
	        self.passwd = self.fhandler.read()
	        self.fhandler.close()

	    def download(self, remote_file, local_file):
	        getfile = 'rsync %s@%s:%s %s' % (self.user, self.host, remote_file, local_file)
	        
	        child = pexpect.spawn(getfile)
	        child.expect("password: ")
	        child.sendline(self.passwd)
	        child.read()

	    def upload(self, local_file, remote_file):
	        sendfile = 'rsync %s %s@%s:%s' % (local_file, self.user, self.host, remote_file)

	        child = pexpect.spawn(sendfile)
	        child.expect("password: ")
	        child.sendline(self.passwd)
	        child.read()

	    def test(self):
	        subprocess.call("rsync  --password-file='/tmp/johnny.pwd' johnny@172.16.123.128:/tmp/joke /tmp/joke")

	    def main(self):
	        self.upload("/tmp/hello.txt", "/tmp/world.txt")
	        self.download("/tmp/world.txt", "/tmp/hello")

	if __name__ == "__main__":
	    r = rsync_tool()
	    r.main()




