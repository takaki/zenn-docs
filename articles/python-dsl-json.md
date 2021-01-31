---
emoji: "💻"
type: "tech"
published: true
title: PythonのDSLでJSON Schemaを書く
topics: ["Python","jsonschema","JSON"]
author: takaki@github
slide: false
---
JSON SchemaというJSONのスキーマをJSONで定義するものがある。が，JSON Schemaを手でちまちま書いていると面倒なので作成のためのツールがある。そのなかでPythonでJSON SchemaのDSLを作成したjslというパッケージについて簡単な説明をする。

サンプルとして以下のようなJSON Schemaを使う。

```JSON
{
	"title": "Example Schema",
	"type": "object",
	"properties": {
		"firstName": {
			"type": "string"
		},
		"lastName": {
			"type": "string"
		},
		"age": {
			"description": "Age in years",
			"type": "integer",
			"minimum": 0
		}
	},
	"required": ["firstName", "lastName"]
}
```

これをjslを使って定義すると次のようになる。

```python3
import jsl

class Example(jsl.Document):
    class Options(object):
        title = "Example Schema"
    firstName = jsl.StringField(required=True)
    lastName = jsl.StringField(required=True)
    age = jsl.IntField(description="Age in years", minimum=0)
```

これを出力したい場合はクラスメソッドのget_schemaを使う。

```python3
import json

print(json.dumps(Example.get_schema(ordered=True), indent=4))
```

# 参考
* http://json-schema.org
* https://pypi.python.org/pypi/jsl

