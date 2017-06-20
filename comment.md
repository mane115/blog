# 评论中心

由于一直用的第三方平台多说停止维护了，所以基于多说的设计重写了个开源的评论中心。

* ### client模块

  client（第三方）需要通过申请client_id 与 client_secret 来注册评论中心的服务，申请后通过调用api的请求的头部或尾部（query），让评论中心识别合法的请求.
  client。
  
  * 增加client

* ### 用户模块

  评论中心不设计用户模块，client所有用户信息由jwt token 形式

* ### 评论模块
  
  业务模块，客户端通过本模块进行评论主体业务，涵盖增删查改功能，以及回复评论功能。
  
  * 发表评论（根据 client_id,thread_id,user_id唯一）
  
  * 回复评论（根据 client_id,thread_id,user_id,comment_id唯一）
  
  * 删除评论 由于无用户系统，所以删除评论权限仅提供给client(第三方)对应的管理员
