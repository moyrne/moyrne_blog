---
title: "time.After() 导致内存暴涨"
date: 2021-01-27T13:04:41+08:00
draft: false
---

# select + time.After 导致内存暴涨
~~~go
func Function(notRun notRun) {
	for {
		select {
		case <-notRun.notRun:
		// 大多数情况都是有notRun的输入
		case <-time.After(time.Minute):
			notRun.Close()
			continue
		case <-notRun.run:
			notRun.Close()
			return
		}
	}
}
~~~
- notRun 发送消息频率过快，而每次select都会调用到time.After，而time.After又会NewTimer，而每次NewTimer都必须在1分钟后才能释放。
- 当notRun的频率很高时，会在内存中堆积非常多的无用的Timer。导致内存暴涨。


# 解决方法
~~~go
func Function(notRun notRun) {
	afterTime := time.Minute
	after := time.NewTimer(afterTime)
	defer after.Stop()
	for {
		after.Reset(afterTime)
		select {
		case <-notRun.notRun:
		case <-after.C:
			notRun.Close()
			continue
		case <-notRun.run:
			notRun.Close()
			return
		}
	}
}
~~~

- 自己 NewTimer 在每次 select 之前 reset，使 timer 重新计时，从而避免每次都 new timer。 