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
- [单个查询](##单个查询group)
- [批量查询](##批量查询group)
# 生成类接口
## 调试流程开始

> POST /framework/v1/debug/task

> 请求url格式例子：/framework/v1/debug/task?namespace=Test

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
	"task":"{"kind": "Task","apiVersion": "resouces/v1","name":"T1","namespace": "test","spec": {		"name": "T1","desc": {"docs": "演示流程"}}}"
}
```

### 响应参数
| 字段名      | 类型     | 说明        | 示例值                        |
| -------- | ------ | --------- | -------------------------- |
| taskName | string | task的唯一标识 | Test-20250626T104243-42d50 |
| uuid     | string | UUID      | 20250626T104243-42d50      |

示例值
```json
{
	"taskName":"Test-20250626T104243-42d50",
	"uuid":"20250626T104243-42d50"
}
```

## 运行group
> POST /framework/v1/debug/group

> 请求url格式例子：/framework/v1/debug?taskName=T1-20250626T104243-42d50&namespace=test
### 请求参数
#### query

| 字段名       | 类型     | 必填  | 说明        | 示例值                      |
| --------- | ------ | --- | --------- | ------------------------ |
| taskName  | string | 是   | task的唯一标识 | T1-20250626T104243-42d50 |
| namespace | string | 是   | namespace | test                     |

#### body
| 字段名      | 类型     | 必填  | 说明                         | 示例值    |
| -------- | ------ | --- | -------------------------- | ------ |
| ideId    | string | 是   | ide侧的唯一标识                  | grab01 |
| template | string | 是   | 填充了input参数的行为模版json string |        |

示例值
```json
{
	"ideId":"grab01",
	"template":‘
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
	}’
}
```

### 响应参数
| 字段名       | 类型     | 说明         | 示例值                         |
| --------- | ------ | ---------- | --------------------------- |
| groupName | string | group的唯一标识 | T1.G1-20250626T104243-42d50 |

示例值
```json
{
	"groupname":"T1.G1-20250626T104243-42d50"
}
```
## 重复运行group
> PUT /framework/v1/debug/group

> 请求url格式例子：/framework/v1/debug/group?taskName=T1-20250626T104243-42d50&namespace=test&groupName=T1.G1-20250626T104243-42d50
### query

| 字段名       | 类型     | 必填  | 说明         | 示例值                         |
| --------- | ------ | --- | ---------- | --------------------------- |
| taskName  | string | 是   | task的唯一标识  | T1-20250626T104243-42d50    |
| namespace | string | 是   | namespace  | test                        |
| groupName | string | 是   | group的唯一标识 | T1.G1-20250626T104243-42d50 |

### body
| 字段名      | 类型     | 必填  | 说明                         | 示例值    |
| -------- | ------ | --- | -------------------------- | ------ |
| ideId    | string | 是   | ide侧的唯一标识                  | grab01 |
| template | string | 是   | 填充了input参数的行为模版json string |        |

示例值
```json
{
	"ideId":"grab01",
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
| groupName | string | group的唯一标识 | T1.G1-20250626T104243-42d50 |

示例值
```json
{
	"groupName":"T1.G1-20250626T104243-42d50"
}
```

## 批量运行group
> POST /framework/v1/debug/groups

> 请求url格式例子：/framework/v1/debug/groups?taskName=T1-20250626T104243-42d50&namespace=test
### 请求参数
#### query

| 字段名       | 类型     | 必填  | 说明        | 示例值                      |
| --------- | ------ | --- | --------- | ------------------------ |
| taskName  | string | 是   | task的唯一标识 | T1-20250626T104243-42d50 |
| namespace | string | 是   | namespace | test                     |


#### body
| 字段名               | 类型     | 必填  | 说明                         | 示例值    |
| ----------------- | ------ | --- | -------------------------- | ------ |
| groups            | array  | 是   | group列表                    |        |
| &emsp;\|-ideId    | string | 是   | ide侧的唯一标识                  | grab01 |
| &emsp;\|-template | string | 是   | 填充了input参数的行为模版json string |        |

示例值
```json
{
	"groups":[
		{
			"ideId":"grab01",
			"template": "json string"
		},
		{
			"ide_id":"grab02",
			"template": "json string"
		}
	]
}
```
### 响应参数

| 字段名                 | 类型     | 说明         | 示例值                         |
| ------------------- | ------ | ---------- | --------------------------- |
| taskName            | string | 新task的唯一标识 | T1-20250626T104243-42d50    |
| groupNames          | array  | group标识列表  |                             |
| &emsp;\|--ideId     | string | ide唯一标识    | grab01                      |
| &emsp;\|--groupName | string | group的唯一标识 | T1.G1-20250626T104243-42d50 |

示例值
```json
{
	"taskName": "T1-20250626T104243-42d50",
	"groupNames":[
		{
			"ideId":"grab01",
			"groupName":"T1.G1-20250626T104243-42d50"
		},
		{
			"ideId":"grab02",
			"groupName":"T1.G2-20250626T104243-42d50"
		}
	]
}
```
# 查询类接口

## 单个查询group
> GET /framework/v1/debug/group

> 请求url格式例子：/framework/v1/debug/group?groupName=T1.G2-20250626T104243-42d50&namespace=Test&ideId=grab01

### 请求参数
#### query
| 字段名       | 类型     | 必填  | 说明         | 示例值                         |
| --------- | ------ | --- | ---------- | --------------------------- |
| groupName | string | 是   | group的唯一标识 | T1.G1-20250626T104243-42d50 |
| namespace | string | 是   | namespace  | Test                        |
| ideId     | string | 是   | ide唯一标识    | grab01                      |

### 响应参数
| 字段名                      | 类型     | 说明                   | 示例值                            |
| ------------------------ | ------ | -------------------- | ------------------------------ |
| groupName                | string | group的唯一标识           | T1.G1-20250626T104243-42d50    |
| namespace                | string | namespace            | Test                           |
| belong                   | string | 父节点唯一标识              | T1-20250626T104243-42d50       |
| status                   | object | group状态信息            |                                |
| &emsp;\|--start          | string | 开始执行时间               | 2025-06-26T15:16:38.545Z       |
| &emsp;\|--creat          | string | 创建时间                 | 2025-06-26T15:16:38.545Z       |
| &emsp;\|--finish         | string | 结束时间                 | 2025-06-26T15:16:38.545Z       |
| &emsp;\|--lastTime       | string | 上次查询时间               | 2025-06-26T15:16:38.545Z       |
| &emsp;\|--phase          | string | 当前状态                 | Running                        |
| actions                  | array  | 该group下的所有action信息   |                                |
| &emsp;\|--actionName     | string | action唯一标识           | T1.G1.A1-20250626T104243-42d50 |
| &emsp;\|--oriName        | string | action在group模板中的name | A1                             |
| &emsp;\|--namespace      | string | namespace            | Test                           |
| &emsp;\|--belong         | string | 父节点唯一标识              | T1.G1-20250626T104243-42d50    |
| &emsp;\|--status         | object | action状态信息           |                                |
| &emsp;&emsp;\|--start    | string | 开始执行时间               | 2025-06-26T15:16:38.545Z       |
| &emsp;&emsp;\|--creat    | string | 创建时间                 | 2025-06-26T15:16:38.545Z       |
| &emsp;&emsp;\|--finish   | string | 结束时间                 | 2025-06-26T15:16:38.545Z       |
| &emsp;&emsp;\|--lastTime | string | 上次查询时间               | 2025-06-26T15:16:38.545Z       |
| &emsp;&emsp;\|--phase    | string | 当前状态                 | Running                        |

示例值
```json
{ 
	"groupName": "T1.test1-20250704T181004-4b685", 
	"namespace": "test", 
	"belong": "T1-20250704T181004-4b685", 
	"status": { 
		"create": "2025-07-04T18:10:04.561869801+08:00",
		"start": "2025-07-04T18:10:05.566347156+08:00", 
		"finish": "2025-07-04T18:10:05.581856314+08:00",
		"lastTime": "2025-07-04T18:10:05.581856375+08:00",
		"phase": "Succeeded" 
		}, 
	"actions": [ { 
		"actionName": "T1.test1.A1-20250704T181004-4b685", 
		"namespace": "test",
		"belong": "T1.test1-20250704T181004-4b685", 
		"oriName": "A1",
		"status": { 
			"create": "2025-07-04T18:10:04.561871354+08:00",
			"start": "2025-07-04T18:10:05.566347156+08:00",
			"finish": "2025-07-04T18:10:05.581856314+08:00", 
			"lastTime": "2025-07-04T18:10:05.581856375+08:00",
			"phase": "Succeeded" 
			} 
		} 
	] 
}
```
## 批量查询group
> GET /framework/v1/debug/groups

> 请求url格式例子：/framework/v1/debug/groups?taskName=T1-20250626T104243-42d50&namespace=Test
### 请求参数
#### query
| 字段名       | 类型     | 必填  | 说明               | 示例值                      |
| --------- | ------ | --- | ---------------- | ------------------------ |
| taskName  | string | 是   | group所属task的唯一标识 | T1-20250626T104243-42d50 |
| namespace | string | 是   | namespace        | Test                     |

### 响应参数
| 字段名      | 类型     | 说明                    | 示例值                      |
| -------- | ------ | --------------------- | ------------------------ |
| groups   | array  | group信息列表，成员结构与单个查询相同 |                          |


示例值
```json
{
	"groups":[
		{ 
			"groupName": "T1.test1-20250704T181004-4b685", 
			"namespace": "test", 
			"belong": "T1-20250704T181004-4b685", 
			"status": { 
				"create": "2025-07-04T18:10:04.561869801+08:00",
				"start": "2025-07-04T18:10:05.566347156+08:00", 
				"finish": "2025-07-04T18:10:05.581856314+08:00",
				"lastTime": "2025-07-04T18:10:05.581856375+08:00",
				"phase": "Succeeded" 
				}, 
			"actions": [ { 
				"actionName": "T1.test1.A1-20250704T181004-4b685", 
				"namespace": "test",
				"belong": "T1.test1-20250704T181004-4b685", 
				"oriName": "A1",
				"status": { 
					"create": "2025-07-04T18:10:04.561871354+08:00",
					"start": "2025-07-04T18:10:05.566347156+08:00",
					"finish": "2025-07-04T18:10:05.581856314+08:00", 
					"lastTime": "2025-07-04T18:10:05.581856375+08:00",
					"phase": "Succeeded" 
					} 
				} 
			] 
		}
	]
}

```


0703 
