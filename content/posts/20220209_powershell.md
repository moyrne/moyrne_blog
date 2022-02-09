---
title: "Windows Powershell 美化"
date: 2022-02-09T11:02:32+08:00
toc: false
draft: false
tags: ["shell"]
---

#### 原始链接 `https://zhuanlan.zhihu.com/p/137595941`

1. 安装 Windows Terminal
   
    MicroSoft 应用商店

2. 安装字体 Fira Code Nerd Font
    ~~~
    https://github.com/ryanoasis/nerd-fonts/releases
    ~~~

3. 安装新的 PowerShell
    ~~~
    https://github.com/PowerShell/PowerShell/releases/tag/v7.2.1
    ~~~

4. 安装 PowerShell 插件
    ~~~shell
    # 1. 安装 PSReadline 包，该插件可以让命令行很好用，类似 zsh
    Install-Module -Name PSReadLine  -Scope CurrentUser
    
    # 2. 安装 posh-git 包，让你的 git 更好用
    Install-Module posh-git  -Scope CurrentUser
    
    # 3. 安装 oh-my-posh 包，让你的命令行更酷炫、优雅
    Install-Module oh-my-posh -Scope CurrentUser
    ~~~
   
5. 添加 Powershell 启动参数
    1. 输入命令
    ~~~shell
    notepad.exe $Profile
    ~~~
    2. 在文本框装填入以下内容
    ~~~shell
        #------------------------------- Import Modules BEGIN -------------------------------
        # 引入 posh-git
        Import-Module posh-git
        
        # 引入 oh-my-posh
        Import-Module oh-my-posh
        
        # 引入 ps-read-line
        Import-Module PSReadLine
        
        # 设置 PowerShell 主题
        # Set-PoshPrompt ys
        Set-PoshPrompt paradox
        #------------------------------- Import Modules END   -------------------------------
        
        
        
        
        
        #-------------------------------  Set Hot-keys BEGIN  -------------------------------
        # 设置预测文本来源为历史记录
        Set-PSReadLineOption -PredictionSource History
        
        # 每次回溯输入历史，光标定位于输入内容末尾
        Set-PSReadLineOption -HistorySearchCursorMovesToEnd
        
        # 设置 Tab 为菜单补全和 Intellisense
        Set-PSReadLineKeyHandler -Key "Tab" -Function MenuComplete
        
        # 设置 Ctrl+d 为退出 PowerShell
        Set-PSReadlineKeyHandler -Key "Ctrl+d" -Function ViExit
        
        # 设置 Ctrl+z 为撤销
        Set-PSReadLineKeyHandler -Key "Ctrl+z" -Function Undo
        
        # 设置向上键为后向搜索历史记录
        Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
        
        # 设置向下键为前向搜索历史纪录
        Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward
        #-------------------------------  Set Hot-keys END    -------------------------------
    ~~~