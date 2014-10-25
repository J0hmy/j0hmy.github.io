---
layout: post
title: Ubuntu下搭建FTP服务器
date: 2014-02-21 16:24:44
tags: FTP
categories: 未分类
---
<div>由于连接开发板，传输文件的需要，我在ubuntu上安装了ftp服务器</div>
<div>FTP软件选择vsftpd（very secure FTP daemon）
<pre class="brush: bash; gutter: true">uwin@ubuntu:~$ sudo apt-get install vsftpd</pre>
<code></code></div>
<div>命令执行过程中，安装程序会给本地创建一个名为“ftp”的用户组，命令执行完之后会自动启动FTP服务。</div>
<div></div>
<div>在浏览器里输入"ftp://localhost" 检查FTP端口有没有打开</div>
<div></div>
<div>创建一个专门用来访问的用户：ligelaige</div>
<pre class="brush: bash; gutter: true">mkdir -p /home/test
useradd test -g ftp -d /home/test -s /sbin/nologin</pre>
设置密码:
<pre class="brush: bash; gutter: true">root@ubuntu:/home/uwin# mkdir /home/ligelaige
root@ubuntu:/home/uwin# useradd ligelaige -g ftp -d /home/ligelaige/ -s /sbin/nologin
root@ubuntu:/home/uwin# passwd ligelaige
输入新的 UNIX 密码： 
重新输入新的 UNIX 密码： 
passwd：已成功更新密码
root@ubuntu:/home/uwin#</pre>
<div> 修改vsftpd的配置文件</div>
<div>
<pre class="brush: bash; gutter: true">root@ubuntu:/home/uwin# gedit /etc/vsftpd.conf</pre>
</div>
<div>

需要修改到字段有
<pre class="brush: bash; gutter: true">#禁止匿名访问
anonymous_enable=NO
#接受本地用户
local_enable=YES
#可以上传
write_enable=YES
#启用在chroot_list_file的用户只能访问根目录
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd.chroot_list</pre>
建议修改的方法是

</div>
<div>alt + F 搜索 anonymous_enable=NO（举例）进行修改</div>
<div></div>
<div>

在/etc/vsftpd.chroot_list添加受访问目录限制的用户：
<pre class="brush: bash; gutter: true">root@ubuntu:/home/uwin# echo &quot;ligelaige&quot; &gt;&gt; /etc/vsftpd.chroot_list
root@ubuntu:/home/uwin# chmod a-w /home/ligelaige/</pre>
重启vsftpd之后就可以使用ligelaige账号访问了
<pre class="brush: bash; gutter: true">uwin@ubuntu:~$ sudo service vsftpd restart</pre>
</div>