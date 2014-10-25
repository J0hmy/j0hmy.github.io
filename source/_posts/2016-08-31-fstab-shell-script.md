---
title: 一个简单的命令
date: 2016-08-31 17:56:02
tags:
---

注释或恢复 /dev/sda2	        /media/video1	        ext4	defaults	0 0

```bash
#!/bin/bash

fstab="fstab"

function removefs()
{
targetfile="$fstab"
sed -i 's/^\(\/dev\/sd.*\/media\/video[1234].*ext4.*defaults.*\)/\#\1/g' $targetfile
}

function restorefs()
{
targetfile="$fstab"
sed -i 's/^#\(\/dev\/sd.*\/media\/video[1234].*ext4.*defaults.*\)/\1/g' $targetfile
}

function usage()
{
	echo "./rmfstab [restore|help]"
}

## main entry

case $1 in
restore|reduce|r|no|not|1)
	restorefs;;
help|--help|h|-h|0)
	usage;;
*)
	removefs;;
esac
```