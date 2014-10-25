---
layout: post
title: 一种字符串序列描述
date: 2016-04-13 17:57:35
categories: 未分类
---

将两个序列

    A = {1, 4, 6, 8, 9}
    B = {10, 11, 12, ...20}

转为如下形式

    string a = "1,4,6,8-9";
    string b = "10-20"; 

实现如下

```
QString SeriesEdit::FormatSeries(const QSet<int>& series)
{	
	QList<int> series_list = series.toList();
	qSort(series_list.begin(), series_list.end());

	int last = 0xffffffff;
	bool match = false;

	QString series_str;
	for (QList<int>::iterator it = series_list.begin(); it != series_list.end(); ++it)
	{
		int var = *it;
		if (it == series_list.begin())
		{
			series_str += QString::number(var);
		}
		else if (last + 1 == var)
		{
			series_str += "-";
			match = true;
		}
		else if (match)
		{			
			series_str += QString::number(last) + "," + QString::number(var);
			match = false;
		}	
		else
		{
			series_str += "," + QString::number(var);
		}
		last = *it;
	}
	if (match)
	{
		series_str += QString::number(last);
	}

	return series_str.split("-", QString::SkipEmptyParts).join("-");

}

```


