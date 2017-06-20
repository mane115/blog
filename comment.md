# 评论中心

由于一直用的第三方平台多说停止维护了，所以基于多说的设计重写了个开源的评论中心。

* ### client模块

    client（第三方）需要通过申请client_id 与 client_secret 来注册评论中心的服务，client通过client_id与client_secret联合加密用户信息，发起正确请求
  
    * 增加client
  
    * 回复评论（根据 client_id,thread_id,user_id,comment_id唯一）
  
* ### 用户模块

    评论中心不设计用户模块，client所有用户信息由jwt token 形式

* ### 评论模块
  
    业务模块，客户端通过本模块进行评论主体业务。
  
    * 发表评论 (图/文)
  
    * 回复评论 (图/文)
  
    * 匿名评论 每用户一次（根据 client_id,thread_id,user_id唯一）
  
    * 举报评论 每用户一次（根据 client_id,thread_id,user_id唯一）
  
  ~~* 删除评论 由于无用户系统，所以删除评论权限仅提供给client对应的管理员~~

* ### 管理模块
  
    本模块需要通过client的服务端直接调用，client可以做接口给管理员进行请求转发。本模块所有接口都需要在头部带有client_id与client_secret。
  
    * 删除评论
