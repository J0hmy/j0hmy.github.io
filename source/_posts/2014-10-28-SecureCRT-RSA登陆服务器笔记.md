---
layout: post
title: SecureCRT-RSA登陆服务器笔记
date: 2014-10-28 03:23:41
tags: SecureCRT
categories: 未分类
---

系统环境：CentOS
--------------
登录用户：root
--------------

0.创建RSA密钥对
	
SecureCRT → 工具 → 创建公钥 → 选择RSA加密算法 → 通行短语（例如：huangjinqiang.johnny） → 密钥长度 1024 → 选择 OpenSSH密钥格式

1.上传并修改公钥文件
	
SecureCRT →  选项 → 会话选项 → 类别 X/Y/Zmoderm → 上传,将上传的公钥重命名为
~/.ssh/authorized_keys
		
	# chmod 700 ~/.ssh
	# chmod 644 ~/.ssh/authorized_keys

2.修改/etc/ssh/sshd_config
	
在Linux服务器上编辑 sshd.config 文件
	
	# vi /usr/local/etc/sshd_config	
	PasswordAuthentication no  # （关闭口令认证）		
	PubkeyAuthentication yes   # （开启公钥认证） // 默认好像是开启的		
	AuthorizedKeysFile .ssh/authorized_keys  # （认证公钥文件位置）

3.重启sshd服务
	
	# /etc/init.d/sshd restart

4.登录设置
	
SecureCRT → 文件 → 快速连接 → 选择协议(SSH2)、主机名、端口、用户名，将鉴权中的公钥选为首选登录方式，连接

5.密钥登录
	
首次登陆需要选择客户端的公钥所在位置(*)，然后点击登陆即可。
如果设置有误导致登陆失败，SecureCRT将执行鉴权后面的登录方式（如密码登陆）尝试登陆服务器。

*关键点：

windows客户端的公钥和私钥需要放在同一目录下，打开时选择公钥后，SecureCRT会对比客户端与服务器的公钥是否一致，如果一致则继续读取客户端的私钥，使用加密算法验证私钥与公钥是否匹配（关键点属作者个人理解，如有错误，欢迎讨论：hellojinqiang [at] gmail.com）

