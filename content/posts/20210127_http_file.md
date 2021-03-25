---
title: "Golang HTTP包传输文件"
date: 2021-01-27T13:02:39+08:00
tags: ["golang"]
draft: false
---

# 文件传输
	Content-Type  http://tool.oschina.net/commons?type=22013-05-17
	multipart/form-data：
        既可以提交普通键值对，也可以提交(多个)文件键值对。
    application/octet-stream：
        只能提交二进制，而且只能提交一个二进制，如果提交文件的话，只能提交一个文件,后台接收参数只能有一个，而且只能是流（或者字节数组）
    application/x-www-form-urlencoded
        不属于http content-type规范，通常用于浏览器表单提交，数据组织格式:name1=value1&name2=value2,post时会放入http body，get时，显示在在地址栏。
~~~go
    // golang 实现
    W.Header().Add("Content-Type","application/octet-stream")
    W.Header().Add("Content-Disposition",fmt.Sprintf("attachment; filename=\"%s\"", fileName))
~~~