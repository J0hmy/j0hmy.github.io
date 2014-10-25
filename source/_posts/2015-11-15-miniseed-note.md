---
layout: post
title: Mini-Seed笔记 
date: 2015-11-15 15:06:05
categories: 未分类
---

下载libmseed代码库

	`https://seiscode.iris.washington.edu/projects/libmseed/files`

将 miniseed 数据流转化为 MSRecord 结构
	
	`
	// 引入头文件
	#include "libmseed.h"
	// 初始化一个 MSRecord
	MSRecord *msr = msr_init(NULL);
	/* 标注1 */
	// 尝试Unpack，如果 retcode == MS_NOERROR，则成功，相应信息保存在 msr 中
	int retcode = msr_unpack((char *)mseed_buffer, mseed_buffer_length, &msr, true, 1);
	// 从 msr 获取信息
	char* network = msr->network;
	char* station = msr->station;
	char* channel = msr->channel;
	char* location = msr->location;
	char* starttime = msr->starttime;
	char* samprate = msr->samprate;

	unsigned int numsamples =  msr->numsamples;
	char sampletype = msr->sampletype;

	void * datasamples = msr->datasamples;

	// 释放MSRecord
	msr_free(&msr);
	`

标注

	如果mseed使用的是国家标准
	在 `标注1` 处可能需要修正 mseed 数据流，原因是在libmseed.h中有一个宏定义检查
	`
		#define MS_ISVALIDHEADER(X) (                               \
		  (isdigit ((int) *(X))   || *(X)   == ' ' || !*(X) )   &&  \
		  (isdigit ((int) *(X+1)) || *(X+1) == ' ' || !*(X+1) ) &&  \
		  (isdigit ((int) *(X+2)) || *(X+2) == ' ' || !*(X+2) ) &&  \
		  (isdigit ((int) *(X+3)) || *(X+3) == ' ' || !*(X+3) ) &&  \
		  (isdigit ((int) *(X+4)) || *(X+4) == ' ' || !*(X+4) ) &&  \
		  (isdigit ((int) *(X+5)) || *(X+5) == ' ' || !*(X+5) ) &&  \
		  MS_ISDATAINDICATOR(*(X+6)) &&                             \
		  (*(X+7) == ' ' || *(X+7) == '\0') &&                      \
		  (int)(*(X+24)) >= 0 && (int)(*(X+24)) <= 23 &&            \
		  (int)(*(X+25)) >= 0 && (int)(*(X+25)) <= 59 &&            \
		  (int)(*(X+26)) >= 0 && (int)(*(X+26)) <= 60 )
	`
	修正的方式可以将mseed_buffer的前6个字节置为 0(ascii 0, 不是 '0')，之前注意保存副本。

	此外，可能会遇到大小端的问题

大小端转换

	`
	#define BigtoLittle16(A) ((((u16)(A) & 0xff00) >> 8) | (((u16)(A) & 0x00ff) << 8))
	#define BigtoLittle32(A) ((((u32)(A) & 0xff000000) >> 24) | (((u32)(A) & 0x00ff0000) >> 8) | (((u32)(A) & 0x0000ff00) << 8)  | (((u32)(A) & 0x000000ff) << 24))
	`

另外，可能需要在设计数据结构时，注意内存对齐，解决的方法是：修改数据结构，或者 添加编译器指令 `#pragma   pack(n)` .



