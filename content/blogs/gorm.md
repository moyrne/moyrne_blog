---
title: "GORM 使用"
date: 2021-01-27T11:53:34+08:00
tags: ["golang"]
draft: false
---

# MYSQL 悲观锁

~~~go
tx := db.DB.Begin()
if err := tx.Set("gorm:query_option", "FOR UPDATE").
	Where("`xx` = ? and `xxx` = ?", xx, xxx).
	First(&xx).Error; err != nil {
	tx.Rollback()
	return
}
~~~

    在事务中使用 Set("gorm:query_option", "FOR UPDATE") + first 能将查询的这条记录锁住；
    在事务 rollback 或 commit 后会 unlock。

---

# 创建复合主键(当主键涉及自增时)


~~~
gorm:"primary_key;AUTO_INCREMENT:false"
~~~

---


# 连接数据库
## 步骤分析
- 引入mysql数据库驱动
- 引入gorm包
- 读取配置文件中的数据库信息
- 将读取的数据生成为Open()需要的字符串



## 代码示例
~~~~go
package base

import (
	_ "github.com/go-sql-driver/mysql"
	_ "github.com/jinzhu/gorm/dialects/mysql"
	"github.com/jinzhu/gorm"
	"fmt"
	"log"
)

func DBConnect() {
	user := "user"
	password := "password"
	dbname := "dbname"
	address := "address"
	port := "port"
	str := fmt.Sprintf("%s",user+":"+password+ "@tcp("+address+
		":"+port+")/"+dbname+"?charset=utf8&parseTime=True&loc=Local")
	db, err := gorm.Open("mysql", str)
	//"user:password@/dbname?charset=utf8&parseTime=True&loc=Local"
	if err != nil {
		log.Println("Opening mysql error is :",err)
		return
	}
	defer db.Close()
}
~~~~