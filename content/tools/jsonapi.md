---
title: "JSON:API 使用"
date: 2021-02-25T11:32:24+08:00 
draft: true
---

~~~
JSON:API 文档链接
https://jsonapi.org/
本人使用的有删减，但已足够本人使用。
~~~

### 最基本结构

~~~go
type Node struct {
  Type          string                     `json:"type"`
  ID            string                     `json:"id"`
  Attributes    map[string]interface{}     `json:"attributes,omitempty"`
  Relationships map[string]json.RawMessage `json:"relationships,omitempty"` // 断言判断 One Many
  Included      []*Node                    `json:"included,omitempty"`
}
~~~

| 名称           | 说明        |
| -----------   | -----------|
| type          | 资源名称（可以认为是数据库名）|
| id            | 资源的ID值|
| attribute     | 资源的各个非关联字段的值|
| relationships | 资源的关联字段（仅包含type和id具体示例在下方）|
| included      | 资源的关联字段的详细数据|

- relationships

~~~json
{
  ...
  "relationships": {
    "foreign_key_many2one": {
      "data": {
        "type": "model_a",
        "id": "1"
      }
    },
    "foreign_key_many2many": {
      "data": [
        {
          "type": "model_b",
          "id": "1"
        }
      ]
    }
  },
  ...
}
~~~

- Response
    - Top

- Request
