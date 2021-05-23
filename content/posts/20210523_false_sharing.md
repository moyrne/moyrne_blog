---
title: "False Sharing"
date: 2021-05-23T14:25:33+08:00
tags: ["golang"]
draft: false
---

#### CPU-Cache 结构图
* 多数的CPU缓冲结构(Intel)
![avatar](/images/cpu-cache.png)
* 缓存行大小 32/64/128(现在多数为64)

#### 解释（下面假设所有的线程都在不同的核心上）
* 在 不同的线程上读写某段连续的内存不同索引的值, 可能导致高速缓冲miss, 导致重复拷贝
~~~
    a = [1,1,1,1,1,1]
    # thread 1
    for {a[0]++}
    # thread 2
    for {a[1]++}
    # thread 3
    for {a[2]++}
    # thread 3
    for {a[4]++}
~~~
* 线程写入某个值之后, 会导致其他其他的线程的缓存失效
  
    例如 thread 1 修改了a[0]
  
    其它线程虽然没有使用a[0], 但缓存也会失效, 需要重新获取, 会产生额外的拷贝的开销
  
    如果需要解决这个问题, 需要将各个元素分隔到不同的缓存行, 也就是上一个元素的开始地址到下一个元素的开始地址, 间隔32/64/128

    如 [1,x,x,x,x,2,x,x,x,x] 的形式进行填充

#### 代码
~~~go
package false_sharing

import (
	"runtime"
	"sync"
	"testing"
	"unsafe"
)

const numCpu = 12

type increase interface {
	Increase(idx int)
	Len() int
}

func BenchmarkNoPadIncrease(b *testing.B) {
	falseSharing(b, NewNoPadSlice(numCpu))
}

func BenchmarkPadIncrease(b *testing.B) {
	falseSharing(b, NewPadSlice(numCpu))
}

func falseSharing(b *testing.B, increase increase) {
	wg := sync.WaitGroup{}
	PreSet(b)
	for i := 0; i < increase.Len(); i++ {
		wg.Add(1)
		go func(idx int) {
			defer wg.Done()
			for j := 0; j < b.N; j++ {
				increase.Increase(idx)
			}
		}(i)
	}
	wg.Wait()
}

func PreSet(b *testing.B) {
	runtime.GOMAXPROCS(numCpu)
	b.ResetTimer()
}

type (
	NoPadSlice []NoPadItem
	NoPadItem  struct {
		Value
	}
	PadSlice []PadItem
	PadItem  struct {
		Value
		pad [128 - unsafe.Sizeof(Value{})%128]byte
	}
	Value struct {
		Int uint64
		Str string
	}
)

func NewNoPadSlice(length int) *NoPadSlice {
	s := make(NoPadSlice, length)
	return &s
}

func (s *NoPadSlice) Len() int {
	return len(*s)
}
func (s *NoPadSlice) Increase(idx int) {
	(*s)[idx].Value.Int++
}

func NewPadSlice(length int) *PadSlice {
	s := make(PadSlice, length)
	return &s
}

func (s *PadSlice) Len() int {
	return len(*s)
}
func (s *PadSlice) Increase(idx int) {
	(*s)[idx].Value.Int++
}
~~~

~~~
goos: windows
goarch: amd64
pkg: demo/false_sharing
cpu: AMD Ryzen 5 3600 6-Core Processor              
BenchmarkNoPadIncrease
BenchmarkNoPadIncrease-12    	1000000000	         0.3561 ns/op
BenchmarkPadIncrease
BenchmarkPadIncrease-12      	1000000000	         0.06356 ns/op
PASS

Process finished with exit code 0
~~~