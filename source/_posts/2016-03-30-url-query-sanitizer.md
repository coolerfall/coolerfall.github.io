---
layout: post
title: 使用UrlQuerySanitizer来处理url
category: android
tags: [url, sanitizer]
---

网上对于UrlQuerySanitizer的资料比较少，这个是Android提供的一个用来处理url的API。由于项目的需要，需要对url的query参数进行排序，因此需要解析url并处理query参数。
<!-- more -->
最初的方法是使用Uri：
```java
public void parseUrl(String url) {
	Uri uri = Uri.parse(url);
	Set<String> query = uri.getQueryParameterNames();
	if (!query.isEmpty()) {
		TreeSet<String> treeQuery = new TreeSet<>(query);
		for (String key : treeQuery) {
			String value = uri.getQueryParameter(key);
		}
	}
}
```
通过这样的方式就可以解析url，并获取到各个query参数。但后来发现Uri不能处理一些特殊字符，比如`#`，Uri会截断`#`以后的内容，这样就不能满足开发需求。经过各种google，最后发现了一个UrlQuerySanitizer的API：
```java
public void parseUrl(String url) {
	UrlQuerySanitizer sanitizer = new UrlQuerySanitizer();
	sanitizer.setAllowUnregisteredParamaters(true);
	sanitizer.setUnregisteredParameterValueSanitizer(UrlQuerySanitizer.getAllButNulLegal());
	sanitizer.parseUrl(url);
	final Set<String> query = sanitizer.getParameterSet();
	if (!query.isEmpty()) {
		TreeSet<String> treeQuery = new TreeSet<>(query);
		for (String key : treeQuery) {
			String value = sanitizer.getValue(key);
		}
	}
}
```
首先要使用`setAllowUnregisteredParamaters`让其支持特殊字符，然后使用`setUnregisteredParameterValueSanitizer`来设置支持哪些特殊字符，UrlQuerySanitizer提供了集中默认的ValueSanitizer：
```java
/**
 * Return a value sanitizer that does not allow any special characters,
 * and also does not allow script URLs.
 * @return a value sanitizer
 */
public static final ValueSanitizer getAllIllegal() {
	return sAllIllegal;
}

/**
 * Return a value sanitizer that allows everything except Nul ('\0')
 * characters. Script URLs are allowed.
 * @return a value sanitizer
 */
public static final ValueSanitizer getAllButNulLegal() {
	return sAllButNulLegal;
}
/**
 * Return a value sanitizer that allows everything except Nul ('\0')
 * characters, space (' '), and other whitespace characters.
 * Script URLs are allowed.
 * @return a value sanitizer
 */
public static final ValueSanitizer getAllButWhitespaceLegal() {
	return sAllButWhitespaceLegal;
}
/**
 * Return a value sanitizer that allows all the characters used by
 * encoded URLs. Does not allow script URLs.
 * @return a value sanitizer
 */
public static final ValueSanitizer getUrlLegal() {
	return sURLLegal;
}
/**
 * Return a value sanitizer that allows all the characters used by
 * encoded URLs and allows spaces, which are not technically legal
 * in encoded URLs, but commonly appear anyway.
 * Does not allow script URLs.
 * @return a value sanitizer
 */
public static final ValueSanitizer getUrlAndSpaceLegal() {
	return sUrlAndSpaceLegal;
}
/**
 * Return a value sanitizer that does not allow any special characters
 * except ampersand ('&'). Does not allow script URLs.
 * @return a value sanitizer
 */
public static final ValueSanitizer getAmpLegal() {
	return sAmpLegal;
}
/**
 * Return a value sanitizer that does not allow any special characters
 * except ampersand ('&') and space (' '). Does not allow script URLs.
 * @return a value sanitizer
 */
public static final ValueSanitizer getAmpAndSpaceLegal() {
	return sAmpAndSpaceLegal;
}
/**
 * Return a value sanitizer that does not allow any special characters
 * except space (' '). Does not allow script URLs.
 * @return a value sanitizer
 */
public static final ValueSanitizer getSpaceLegal() {
	return sSpaceLegal;
}
/**
 * Return a value sanitizer that allows any special characters
 * except angle brackets ('<' and '>') and Nul ('\0').
 * Allows script URLs.
 * @return a value sanitizer
 */
public static final ValueSanitizer getAllButNulAndAngleBracketsLegal() {
	return sAllButNulAndAngleBracketsLegal;
}
```
每种ValueSanitizer都对应过滤哪些字符，被过滤掉的特殊字符会被替换成_或者空格。
如果默认的ValueSanitizer不能满足开发需求，还可以自己构造ValueSanitizer：
```java
public void parseUrl(String url) {
	.....
	ValueSanitizer sanitizer = new UrlQuerySanitizer.IllegalCharacterValueSanitizer(UrlQuerySanitizer.IllegalCharacterValueSanitizer.ALL_OK);
	setUnregisteredParameterValueSanitizer(sanitizer);
	.....
}
```
UrlQuerySanitizer也可以通过key来获取相应的value，比如给一个url：http://coolerfall.com?name=vincent：
```java
public void parseUrl(String url) {
	UrlQuerySanitizer sanitizer = new UrlQuerySanitizer();
	sanitizer.setAllowUnregisteredParamaters(true);
	sanitizer.setUnregisteredParameterValueSanitizer(UrlQuerySanitizer.getAllButNulLegal());
	sanitizer.parseUrl(url);
	String name = sanitizer.getValue("name");
}
```
UrlQuerySanitizer还可以只解析query参数，比如：name=vincent&article=first：
```java
public void parseUrl(String query) {
	UrlQuerySanitizer sanitizer = new UrlQuerySanitizer();
	sanitizer.setAllowUnregisteredParamaters(true);
	sanitizer.setUnregisteredParameterValueSanitizer(UrlQuerySanitizer.getAllButNulLegal());
	sanitizer.parseQuery(query);
	String name = sanitizer.getValue("name");
	.....
}
```
以上就是UrlQuerySanitizer大致用法，用来解析处理url非常的方便。