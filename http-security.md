
# xss

  * cookie xss
  
    在HTTP头部配置set-cookie:

      * HttpOnly - 这个属性可以防止cross-site scripting，因为它会禁止Javascript脚本访问cookie。
      
      * secure - 这个属性告诉浏览器仅在请求为HTTPS时发送cookie。
      
    exp:
    
       ```
       Set-Cookie: sid=<cookie-value>; HttpOnly.
       ```
  
# 密码安全

  * #### timing attacks 

    比较密码时，不能泄露任何信息，因此比较必须在固定时间完成。
    
    如果不控制比对时间，攻击者可以通过比较的时间长短来判断密码的正确性。
    
# OAuth
  
  * #### jwt生成的token如何刷新
   
  ```
             client   -----------------------   server
            jwt token            -->         token alive ? 
            api info             <--         normal alive 
  api info & new token(headers)  <--     alive & expire < {time}
             error               <--      expired | not record
  ```  
  
# 参考

[Node.js面试题之2017](https://cnodejs.org/topic/58eb64293145ae3f25fe614c)
