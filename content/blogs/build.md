---
title: "Golang 编译"
date: 2021-01-27T12:04:51+08:00
tags: ["golang","build"]
draft: false
---

# 编译

## 编译其他系统

    CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build
    
    
    # 不能复制空格
    set CGO_ENABLED=0
    set GOOS=linux
    set GOARCH=amd64
    go build

## 图标制作

    第一步：
        Windows 下载MinGW

    第二步：
        新建一个.rc文件，加入文件名为 demo.rc 输入内容   
        IDI_ICON1 ICON "cefclient.ico"
        其中 cefclient.ico 是你的ico的地址

    第三步:
        MinGW 执行
        windres -o demo.syso demo.rc

    第四步:
        将生成的demo.syso 放到项目目录下
        go build

## 隐藏命令行

    go build -i -ldflags="-H windowsgui"