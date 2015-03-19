---
layout: post
title: golang计算字符以及文件的hash值
head: 使用golang来计算字符和文件的hash值(md5, sha1, sha256)比较简单。
category: go
tags: [hash]
---
{% include cooler/setup %}

计算字符串的hash比较简单，直接上代码：
{% highlight go %}
func md5Str(origin string) string {
	m := md5.New()
	m.Write([]byte(origin))
	return hex.EncodeToString(m.Sum(nil))
}
{% endhighlight %}

计算文件的hash值稍微麻烦一点：
{% highlight go %}
func md5File(filepath string) string {
	file, err := os.Open(filepath)
	if err != nil {
		return ""
	}
	defer file.Close()

	m := md5.New()
	_, err = io.Copy(m, file)
	if err != nil {
		return ""
	}

	return hex.EncodeToString(m.Sum(nil))
}
{% endhighlight %}

sha1和sha256的计算方法类似，具体的代码已经提交至[github][1]上。

[1]: https://github.com/Coolerfall/go-utils/blob/master/hash.go