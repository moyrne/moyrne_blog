---
title: "Vscode"
date: 2021-01-27T10:01:06+08:00
draft: false
description: "Vscode 用户代码片段"
---
# VSCode 用户代码片段
# Golang
~~~
{
  "main": {
    "prefix": "main",
    "body": [
      "func main() {",
      "\t$0",
      "}"
    ],
    "description": "func main() {}"
  },
  "init": {
    "prefix": "init",
    "body": [
      "func init() {",
      "\t$0",
      "}"
    ],
    "description": "func init() {}"
  },
  "for range": {
    "prefix": "forr",
    "body": [
      "for ${1:i},${2:v} := range $3 {",
      "\t$0",
      "}"
    ],
    "description": "for range"
  },
  "for index" :  {
    "prefix": "fori",
    "body": [
      "for ${1:i}:=${2:0};${1:i}${3:<}${4:n};${1:i}${5:++} {",
      "\t$0",
      "}"
    ],
    "description": "for index"
  },
  "if err := xxx();err != nil {}": {
    "prefix": ".rr",
    "body": [
      "$0if err := ${TM_CURRENT_LINE/[ \t]*(.*).rr/${1:text}/}; err != nil {",
      "",
      "}"
    ],
    "description": "if err := xxx();err != nil {}"
  },
  "if err != nil {}": {
    "prefix": "err",
    "body": [
      "if err != nil {",
      "\t$0",
      "}"
    ],
    "description": "if err != nil {}"
  },
  "fmt.Printf": {
    "prefix": "fpf",
    "body": [
      "fmt.Printf(\"$1\\n\", $2)",
    ],
    "description": "fmt.Printf()"
  }
}
~~~
