---
title: "字符串拼接性能测试"
date: 2021-01-28T13:31:45+08:00
tags: ["golang","benchmark"]
draft: false
---

~~~go
// splicing_test.go
package splicing

import (
	"strconv"
	"strings"
	"testing"
)

func Str(str []string) string {
	var rst string
	for _, s := range str {
		rst += s
	}
	return rst
}

func BuilderStr(str []string) string {
	var builder strings.Builder
	for _, s := range str {
		builder.WriteString(s)
	}
	return builder.String()
}

func BenchmarkStr(b *testing.B) {
	srcStr := make([]string, 0, 100000)

	b.Run("Append-10000", func(b *testing.B) {
		for i := 0; i < 10000; i++ {
			srcStr = append(srcStr, strconv.Itoa(i%10))
		}
	})

	b.Run("Str-10000", func(b *testing.B) {
		Str(srcStr)
	})

	b.Run("BuilderStr-10000", func(b *testing.B) {
		BuilderStr(srcStr)
	})

	b.Run("Append-100000", func(b *testing.B) {
		for i := 0; i < 90000; i++ {
			srcStr = append(srcStr, strconv.Itoa(i%10))
		}
	})

	b.Run("Str-100000", func(b *testing.B) {
		Str(srcStr)
	})

	b.Run("BuilderStr-100000", func(b *testing.B) {
		BuilderStr(srcStr)
	})

}
~~~

## 测试结果

~~~shell
$ go test -bench=BenchmarkStr  -benchmem -memprofile=mem.prof -cpuprofile=cpu.prof
goos: linux
goarch: amd64
pkg: demo/splicing
BenchmarkStr/Append-10000-12            1000000000               0.000039 ns/op        0 B/op          0 allocs/op
BenchmarkStr/Str-10000-12               1000000000               0.177 ns/op           1 B/op          0 allocs/op
BenchmarkStr/BuilderStr-10000-12        1000000000               0.000242 ns/op        0 B/op          0 allocs/op
BenchmarkStr/Append-100000-12           1000000000               0.000333 ns/op        0 B/op          0 allocs/op
BenchmarkStr/Str-100000-12                     1        20121850900 ns/op       182358231216 B/op         600145 allocs/op
BenchmarkStr/BuilderStr-100000-12       1000000000               0.00258 ns/op         0 B/op          0 allocs/op
BenchmarkStrOnly-12                     1000000000               0.500 ns/op           5 B/op          0 allocs/op
PASS
ok      demo/splicing   35.097s
$ go tool pprof -http=:8080 mem.prof
~~~

## 内存消耗

~~~
demo/splicing.Str
/mnt/d/go/src/demo/splicing/splicing.go

  Total:    633.77GB   633.77GB (flat, cum) 199.94%
      1            .          .           package splicing 
      2            .          .            
      3            .          .           import "strings" 
      4            .          .            
      5            .          .           func Str(str []string) string { 
      6            .          .           	var rst string 
      7            .          .           	for _, s := range str { 
      8     633.77GB   633.77GB           		rst += s 
      9            .          .           	} 
     10            .          .           	return rst 
     11            .          .           } 
     12            .          .            
     13            .          .           func BuilderStr(str []string) string { 
     14            .          .           	var builder strings.Builder 
     15            .          .           	for _, s := range str { 
demo/splicing.BenchmarkStr.func4
/mnt/d/go/src/demo/splicing/splicing_test.go

  Total:     76.35MB    76.35MB (flat, cum) 0.024%
     20            .          .            
     21            .          .           	b.Run("BuilderStr-10000", func(b *testing.B) { 
     22            .          .           		BuilderStr(srcStr) 
     23            .          .           	}) 
     24            .          .            
     25            .          .           	b.Run("Append-100000", func(b *testing.B) { 
     26            .          .           		for i := 0; i < 90000; i++ { 
     27      76.35MB    76.35MB           			srcStr = append(srcStr, strconv.Itoa(i%10)) 
     28            .          .           		} 
     29            .          .           	}) 
     30            .          .            
     31            .          .           	b.Run("Str-100000", func(b *testing.B) { 
     32            .          .           		Str(srcStr) 
~~~

- 从结果中可以看出，使用+=进行字符串拼接，会申请大量内存，并且速度非常慢。

## 原因分析

- 使用 + 进行字符串拼接时，当 拼接 A 和 B 两个字符串时，需要申请另一个为A+B长度的空间C，每次都是如此。
- 使用 strings.Builder 进行拼接时，内部实际上是使用的 []byte 存储，使用append操作，在不需要扩容时会在当前slice后追加，且每次是翻倍扩容，能减少扩容（申请新内存片段和复制等操作）次数。
- (bytes.Buffer 与 strings.Builder 的差异)
  - bytes.Buffer
    ~~~go
    func (b *Buffer) String() string {
        if b == nil {
            // Special case, useful in debugging.
            return "<nil>"
        }
        return string(b.buf[b.off:])
    }
    ~~~
  - strings.Builder
    ~~~go
    // 会使用原buf的数据，而不会拷贝一份数据到返回值的字符串里面
    func (b *Builder) String() string {
	    return *(*string)(unsafe.Pointer(&b.buf))
    }
    ~~~
