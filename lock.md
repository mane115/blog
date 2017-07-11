### node.js - redis - mq - 并发控制

#### 并发场景

  * #### 秒杀
      秒杀系统是可以笼统的称为多用户对同一资源发起请求，正确响应次数少于用户请求量。此时最安全的做法是使用悲观锁，数据级层面的锁，例如oracle的sql:select for update.但是悲观锁的缺点在高并发场景也是很明显，就是允许的并发量低，容易造成504，就像安检一样，一次只能通过一个人，效率和体验都十分低下。
      所以应该使用乐观锁，或者利用redis的原子性做并发量限制，再使用mq进行任务分发。
      
      正常的流程:

      用户下单->redis并发库存锁，减少库存->通过mq生产订单任务->mq消费者消费任务，生成订单以及更新库存一系列操作。
      
      * 乐观锁：
      
      redis原生提供乐观锁 watch，watch是基于链接的，而主流nodejs里面模块redis是基于pipeline做的，无连接池，所以，watch单单基于业务来说对于nodejs并无作用，只要正确利用好redis的原子性即可。
      但是系统大了以后就会牵扯到集群的问题，在多系统（多链接）的设计下，watch就尤为重要了，个人认为watch可以提供一个“次级操作”的空间，对于秒杀系统来说，库存的更新与秒杀业务是可以同事存在的
      例如：卖家在秒杀期间补充库存、由于业务问题锁住库存等。这个时候watch可以提供优先级，即当管理员锁住库存（清零）与多个买家发起秒杀同一时间发出请求，可以保证管理员的请求是正确通过的，而买家由于更新库存，该次请求失效。
      
      ```javascript
      const redis = Redis.createClient();
      
      let lock = async function(key) {
        let transactionStatus = false;
        await redis.watchAsync(key);
        let stock = await redis.getAsync(key);
        if(+stock < 1) {
          //库存不足的情况
        }
        let reply = await redis.multi().decr(key).execAsync();
        if(!reply) {
          // 当事务失败的时候reply为null，进行错误处理
        } else if(reply[0] < 0) {
          // 当事务成功的时候返回array，multi可理解为Promise.all相当
          // 健壮处理超卖情况，此时应该补redis的库存避免以后因为负数库存导致以后补充库存出错，并且与事务失败执行一致操作
          redis.incr(key);
        } else {
          // 事务成功且正确减少库存的时候
          transactionStatus = true;
        }   
        return transactionStatus;
      };
      
      let produceOrder = async function(orderId,userId){
        let payload = JSON.stringify({
          orderId,
          userId
        });
        let productor = new Productor();
        productor.produce(TOPIC.ORDER,payload);
      };
      
      let buy = async function(orderId,userId){
        let key = `lock:order:${orderId}:${userId}`;
        let lockResult = await lock(key);
        if(lockResult) {
          await produceOrder(orderId,userId);
        }
      };
      ```
  * #### 关注 
      
      关注并发可以作为同一用户对于同一资源重复请求的代表，例如：在网络差的时候，用户点击关注某人的时候由于相应过慢，同时发出了多次请求。这个相当于是过滤无用请求，客户端需要做相应处理，但是一个健壮的客户端，也必须要考虑这种情况。
      
      * 范式设计的数据库可以利用设置唯一联合索引来避免
      * 可利用redis的set、hash、bitmap等数据结构做去重
      * 如果是反范式设计（类似mongo的内嵌数组设计），可利用db层上（$in、$addToSet等操作符）做去重
      * 使用悲观锁，从逻辑层做去重
        
      可以利用redis的原子性或setex操作来完成悲观锁：

      ```javascript
      const redis = Redis.createClient();
      
      let checkLock = async function(key){
          let ttl = await redis.ttlAsync(key);
          if(+ttl === -1) {
            //如果ttl为-1，为无过期时间，立即设置过期时间避免死锁
            redis.expire(key, 10)
          }
      };
      
      let lock = async function(key){
        let result = await redis.incrAsync(key);
        if(+result !== 1) {
          checkLock(key)
          //已被锁住，进行错误处理
        } else {
          //成功上锁，设置过期时间避免死锁
          redis.expire(key, 10);
        }
      };
      
      let releaseLock = async function(key){
        return redis.setAsync(key,0)
      };
      
      let follow = async function(followId,userId){
        let key = `lock:follow:${followId}:${userId}`;
        try {
          await lock(key);
          // 关注逻辑：查询、遍历、插入等
        } catch(err) {
          // 错误处理
        } finally {
          await releaseLock(key)
        }
      };
      ```
