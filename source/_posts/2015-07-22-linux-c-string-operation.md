---
layout: post
title: Linux c语言一些字符串操作函数的实现
category: c
tags: [c]
---

c语言对字符串的操作函数有很多都没有实现，比如java中的trim、indexOf、substring等等，于是就动手实现了几个字符串操作函数，以备以后开发中使用。
<!-- more -->
#### 1.trim函数（去掉字符串首尾空格）
```c
/**
 * Remove space from string at the beginning and end.
 *
 * @param src pointer to source string
 */
void trim(char *src)
{
	int i;
	int len = strlen(src);
	int start = 0, end = len - 1;

	while (start < end && src[start] <= ' ' && src[start] != 0)
	{
		start ++;
	}

	while (end >= start && src[end] <= ' ' && src[end] != 0)
	{
		end --;
	}

	memmove(src, src + start, end - start + 1);
	src[end - start + 1] = '\0';
}
```
#### 2.index_of函数（获得某个字符串在另一个字符串中第一次出现时的位置）
```c
/**
 * To get the index when sub string first appear in src.
 *
 * @param src src string
 * @param sub the string to search
 * @return    the index of substring in source string, 
 *            otherwise return -1 if not exists
 */
int index_of(const char *src, const char *sub)
{
	char *result = strstr(src, sub);

	return result ? strlen(src) - strlen(result) : -1;
}
```
#### 3.last_index_of函数（获得某个字符串在另一个字符串中最后一次出现时的位置）
```c
/**
 * To get the index when need string last appear in src.
 *
 * @param src  src string
 * @param need the string to search
 * @return     the index of substring in source string, 
 *             otherwise return -1 if not exists
 */
int last_index_of(const char *src, const char *need)
{
	int i;
	const char *p = src + strlen(src);
	size_t len = strlen(need);
	char *buf;

	for(i = 0; i < strlen(src); i ++)
	{
		buf = strchr(p --, *need);
		if (!buf)
		{
			continue;
		}

		if (strncmp(buf, need, len) == 0)
		{
			return strlen(src) - strlen((char *)buf);
		}
	}

	return -1;
}
```
#### 4.substring函数（截取字符串）
```c
/**
 * Get sub string from source string.
 *
 * @param dest  dest poniter to save string
 * @param src   source string poniter
 * @param start the start index
 * @param end   the end index
 */
void substring(char *dest, const char *src, int start, int end)
{
	int i = start;

	if (start > strlen(src))
	{
		return;
	}

	if (end > strlen(src))
	{
		end = strlen(src);
	}

	while (i < end)
	{
		dest[i - start] = src[i];
		i ++;
	}

	dest[i - start] = '\0';
}
```
#### 5.starts_with函数（检测字符串是否以某个字符串开头）
```c
/**
 * To check if the string start with specified string.
 *
 * @param  src    source string pointer
 * @param  prefix prefix string poniter
 * @return        ture if start with specified string, otherwise return false
 */
bool starts_with(const char *src, const char *prefix)
{
	int len = strlen(prefix);
	char buf[len];

	substring(buf, src, 0, len);

	return !strcmp(buf, prefix);
}
```
#### 6.ends_with函数（检测字符串是否以某个字符串结尾）
```c
/**
 * To check if the string end with specified string.
 *
 * @param  src    source string pointer
 * @param  suffix suffix string poniter
 * @return        ture if end with specified string, otherwise return false
 */
bool ends_with(const char *src, const char *suffix)
{
	int len = strlen(suffix);
	char buf[len];

	substring(buf, src, strlen(src) - len, strlen(src));

	return !strcmp(buf, suffix);
}
```
其中starts_with和ends_with使用了substring函数，其他函数都可以单独使用。