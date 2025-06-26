# 域名信息
正式域名：
预发布域名：
测试域名：
dev域名：
# 接口列表
- [调试流程开始](##调试流程开始)
- [运行单个group](##运行单个group)
- [批量运行group](##批量运行group)
- [重复运行group](##重复运行group)
- [单个查询](##单个查询)
- [批量查询](##批量查询)
## 调试流程开始<a id="begin"/>

> POST /framework/v1/task

> 请求url格式例子：/framework/v1/task?namespace=Test

### 请求参数
#### query

| 字段名       | 类型     | 必填  | 说明  | 示例值  |
| --------- | ------ | --- | --- | ---- |
| namespace | string | 否   |     | Test |
#### body
| 字段名                  | 类型     | 必填  | 说明                    | 示例值          |
| -------------------- | ------ | --- | --------------------- | ------------ |
| kind                 | string | 是   | 固定值                   | Task         |
| apiVersion           | string | 是   | api版本                 | resources/v1 |
| name                 | string | 是   | task名称，与spec.name保持一致 | T1           |
| namespace            | string | 是   | namespace             | test         |
| spec                 | object |     |                       |              |
| &emsp;\|--name       | string | 是   | task名称                | T1           |
| &emsp;\|--desc       | object |     |                       |              |
| &emsp;&emsp;\|--docs | string | 是   | task说明                | “演示流程”       |
示例值
```json
{
	"kind": "Task",
	"apiVersion": "resouces/v1",
	"name": "T1",
	"namespace": "test",
	"spec": {
		"name": "T1",
		"desc": {
			"docs": "演示流程"
		}
	}
}
```
### 响应参数
| 字段名      | 类型     | 说明        | 示例值                        |
| -------- | ------ | --------- | -------------------------- |
| taskname | string | task的唯一标识 | Test-20250626T184243-42d50 |
| uuid     | string | UUID      | 20250626T184243-42d50      |
示例值
```json
{
	"taskname":"Test-20250626T184243-42d50",
	"uuid":"20250626T184243-42d50"
}
```
## 运行group
> POST /framework/v1/debug

> 请求url格式例子：/framework/v1/debug?taskname=T1-20250626T184243-42d50&namespace=test&ide_id=grab01
### 请求参数
#### query

| 字段名       | 类型     | 必填  | 说明        | 示例值                      |
| --------- | ------ | --- | --------- | ------------------------ |
| taskname  | string | 是   | task的唯一标识 | T1-20250626T184243-42d50 |
| namespace | string | 是   | namespace | test                     |

#### body
| 字段名      | 类型     | 必填  | 说明                         | 示例值    |
| -------- | ------ | --- | -------------------------- | ------ |
| ide_id   | string | 是   | ide侧的唯一标识                  | grab01 |
| template | string | 是   | 填充了input参数的行为模版json string |        |

示例值
```json
{
	"ide_id":"grab01",
	"template":"
	{  
	  "name": "move",  
	  "desc": {  
	    "label": {  
	      "type": 1,  
	      "name_zh": "移动"  
	    },  
	    "docs": "移动动作，通常用于将机器人从一个位置移动到另一个位置。"  
	  },  
	  "input": {  
	    "conditions": [],  
	    "parents": [],  
	    "devices": [  
	      {  
	        "name": "Robot1",  
	        "expected_properties": {  
	          "name": {  
	            "value": "device1"  
	          },  
	          "strategy": {  
	            "value": "nominate"  
	          }  
	        },  
	        "abilities": [  
	          "Move"  
	        ]  
	      }  
	    ],  
	    "parameters": [  
	      {  
	        "name": "target_position",  
	        "type": "compose",  
	        "value": "[0,0,0]",  
	        "from": "R1"  
	      },  
	      {  
	        "name": "move_duration",  
	        "type": "double",  
	        "value": "2.0",  
	        "from": "R1"  
	      }  
	    ]  
	  },  
	  "actions": [  
	    {  
	      "name": "move_to",  
	      "input": {  
	        "conditions": [],  
	        "parents": [],  
	        "parameters": [  
	          {  
	            "name": "target_position",  
	            "type": "compose",  
	            "value": "[0,0,0]",  
	            "from": "R1"  
	          },  
	          {  
	            "name": "move_duration",  
	            "type": "double",  
	            "value": "2.0",  
	            "from": "R1"  
	          }  
	        ]  
	      },  
	      "serialized": "json string",  
	      "desc": {  
	        "docs": "移动action，输入参数包括目标位置和移动持续时间。"  
	      }  
	    }  
	  ],  
	  "replicas": [  
	    0,  
	    0  
	  ]  
	}"
}
```
### 响应参数
| 字段名       | 类型     | 说明         | 示例值                         |
| --------- | ------ | ---------- | --------------------------- |
| groupname | string | group的唯一标识 | T1.G1-20250626T184243-42d50 |

示例值
```json
{
	"groupname":"T1.G1-20250626T184243-42d50"
}
```
## 重复运行group
> PUT /framework/v1/debug

> 请求url格式例子：/framework/v1/debug?taskname=T1-20250626T184243-42d50&namespace=test&groupname=T1.G1-20250626T184243-42d50
### query

| 字段名       | 类型     | 必填  | 说明         | 示例值                         |
| --------- | ------ | --- | ---------- | --------------------------- |
| taskname  | string | 是   | task的唯一标识  | T1-20250626T184243-42d50    |
| namespace | string | 是   | namespace  | test                        |
| groupname | string | 是   | group的唯一标识 | T1.G1-20250626T184243-42d50 |
### body
| 字段名      | 类型     | 必填  | 说明                         | 示例值    |
| -------- | ------ | --- | -------------------------- | ------ |
| ide_id   | string | 是   | ide侧的唯一标识                  | grab01 |
| template | string | 是   | 填充了input参数的行为模版json string |        |

示例值
```json
{
	"ide_id":"grab01",
	"template":'
	{  
	  "name": "move",  
	  "desc": {  
	    "label": {  
	      "type": 1,  
	      "name_zh": "移动"  
	    },  
	    "docs": "移动动作，通常用于将机器人从一个位置移动到另一个位置。"  
	  },  
	  "input": {  
	    "conditions": [],  
	    "parents": [],  
	    "devices": [  
	      {  
	        "name": "Robot1",  
	        "expected_properties": {  
	          "name": {  
	            "value": "device1"  
	          },  
	          "strategy": {  
	            "value": "nominate"  
	          }  
	        },  
	        "abilities": [  
	          "Move"  
	        ]  
	      }  
	    ],  
	    "parameters": [  
	      {  
	        "name": "target_position",  
	        "type": "compose",  
	        "value": "[0,0,0]",  
	        "from": "R1"  
	      },  
	      {  
	        "name": "move_duration",  
	        "type": "double",  
	        "value": "2.0",  
	        "from": "R1"  
	      }  
	    ]  
	  },  
	  "actions": [  
	    {  
	      "name": "move_to",  
	      "input": {  
	        "conditions": [],  
	        "parents": [],  
	        "parameters": [  
	          {  
	            "name": "target_position",  
	            "type": "compose",  
	            "value": "[0,0,0]",  
	            "from": "R1"  
	          },  
	          {  
	            "name": "move_duration",  
	            "type": "double",  
	            "value": "2.0",  
	            "from": "R1"  
	          }  
	        ]  
	      },  
	      "serialized": "json string",  
	      "desc": {  
	        "docs": "移动action，输入参数包括目标位置和移动持续时间。"  
	      }  
	    }  
	  ],  
	  "replicas": [  
	    0,  
	    0  
	  ]  
	}'
}
```
### 响应参数
| 字段名       | 类型     | 说明         | 示例值                         |
| --------- | ------ | ---------- | --------------------------- |
| groupname | string | group的唯一标识 | T1.G1-20250626T184243-42d50 |
示例值
```json
{
	"groupname":"T1.G1-20250626T184243-42d50"
}
```
## 批量运行group
### 请求参数
#### query

| 字段名       | 类型     | 必填  | 说明        | 示例值                      |
| --------- | ------ | --- | --------- | ------------------------ |
| taskname  | string | 是   | task的唯一标识 | T1-20250626T184243-42d50 |
| namespace | string | 是   | namespace | test                     |
#### body
| 字段名                | 类型     | 必填  | 说明                         | 示例值    |
| ------------------ | ------ | --- | -------------------------- | ------ |
| groups             | array  | 是   | group列表                    |        |
| &emsp;\|-ide_id    | string | 是   | ide侧的唯一标识                  | grab01 |
| &emsp;\|-templates | string | 是   | 填充了input参数的行为模版json string |        |
|                    |        |     |                            |        |
### 响应参数


## 单个查询group
## 批量查询group