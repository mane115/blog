# api-server

## 功能简介 

一个链接 前端-后端的api文档、测试服务器。

* 接口存储（类似POST MAN），可动态配置host，baseurl等功能

* 根据接口返回的数据格式动态生成接口文档

* 支持session

* 可配置接口前置、后置调用接，例如调用<查看用户详情接口时>，可配置调用该接口前调用的接口（登录获取session状态或获取token），后置调用的接口（登出清楚登录状态）。总体来说整个调用接口用例时 登录=>获取用户详情=>登出

## 模型设计

* ### 用户 users

* ### 接口集合 collections

	|data|type|desc|ref
	|:--|:---|:--|:--|
	|_id|Object id|数据主键||
	|collection_name|String|集合名||
	|host|String|集合api域名 例:www.baidu.com||
	|base_url|String|集合api基础url 例:/api/v1||
	|create_at|Date|创建时间||
	|update_at|Date|更新时间||
	|create_by|object id|创建者id|user._id

* ### 接口 apis

	|data|type|desc|ref
	|:--|:--|:--|:--
	|_id|Object id|数据主键||
	|name|String|名字|
	|collection_id|Object id|所属集合id|collections|
	|url|String|接口url||
	|method|String|调用接口方法||
	|headers|String(JSON)|自定义头部||
	|body|String(JSON)|请求数据||
	|status|Number|接口状态||
	|type|Number|接口类型||
	|create_at|Date|创建时间||
	|update_at|Date|更新时间||
	|create_by|object id|创建者id|user._id

* ### 响应 response
	
	|data|type|desc|ref
	|:--|:--|:--|:--
	|_id|Object id|数据主键||
	|api|Object id|接口id|api._id|
	|response|String(JSON)|接口返回数据||
	|response_type|String(JSON)|接口返回数据格式||
	|create_at|Date|创建时间||
	|update_at|Date|更新时间||
	|create_by|object id|创建者id|user._id
	
## 逻辑设计

* ### 新增接口

```flow
st=>start: Start
ed=>end: End
op1=>operation:获取集合信息
op2=>operation:新增数据实体:api
op3=>operation:组合接口，服务器按存储的接口信息发出请求
cond1=>condition:判断请求成功状态
op4=>operation:解析返回的数据格式，生成接口数据格式
op5=>operation:新增数据实体:response
op6=>operation:返回接口详情
st->op1->op2->op3->cond1
cond1(yes)->op4->op5->op6
cond1(no)->op6
op6->ed
```
	
