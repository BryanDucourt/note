# 域名信息
正式域名：
预发布域名：
测试域名：
dev域名：
# 接口列表
- [调试流程开始](##调试流程开始)
- [运行单个group](##运行单个group)
- [批量运行group](##批量运行group)
- []()
- []()
- []()
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
## 运行单个group

## 批量运行group
## 重复运行group
## 单个查询
## 批量查询