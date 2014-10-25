---
layout: post
title: c版daemon进程例子
date: 2015-1-28 13:53:41
tags: C
categories: unclassified
---

	/*************************************************************************
		>    File Name: deamon_test.c
		>       Author: huangjinqiang
		>        Email: ligelaige@gmail.com
		> Created Time: Mon 28 Jan 2015 13:56:43 PM CST
	 ************************************************************************/

	#include <time.h>
	#include <stdlib.h>
	#include <unistd.h>
	#include <stdio.h>
	#include <sys/types.h>

	char *getnow()
	{
	    char *wday[] = {"Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"};
	    time_t timep;
	    struct tm *p;
	    time(&timep);
	    p = gmtime(&timep);
		char *str = (char *)malloc(128);
	    sprintf(str, "%4d-%02d-%02d %s %02d:%02d:%02d", 
						1900 + p->tm_year, 1 + p->tm_mon, p->tm_mday,
						wday[p->tm_wday], 8 + p->tm_hour, p->tm_min, p->tm_sec); 
		return str;
	}

	int do_sth() 
	{ 
		FILE * fhander = fopen("daemon_test.log", "a");
		char *str = getnow();
		fprintf(fhander, "Hello, Johnny! %d  NOW: %s\n", getpid(), str);
		fclose(fhander);
		return 0 ; 
	} 


	int main(void)
	{
	    pid_t pid = fork();
	    if(pid < 0) 
			return -1;
	    else if(pid > 0)
		{
			printf ("new  pid: %d\n", pid);			// new
			printf ("main pid: %d\n", getpid());	// main
			exit(0);
		}
		
	    while(1)
	    {
	        do_sth();								// new
	        sleep(1);
	    }
	    return 0;
	}

