### Redis
Redis：k-v持久化产品，内存型数据库,也可以持久化到磁盘。value支持
    String(可以包含jpg图片或序列化对象，内部实现可以看作byte数组，上限1G字节),
    Hash(string类型的field和value的映射表,适合存储对象),
    List(string 类型的双向链表,操作中key理解为链表的名字),
    Sets(提供集合操作),
    Sorted sets(相对于set增加了顺序属性)
#### StringRedisTemplate
    其key和value默认是String方式
 ```java
    stringRedisTemplate.opsForValue();　　//操作字符串
    stringRedisTemplate.opsForHash();　　 //操作hash
    stringRedisTemplate.opsForList();　　 //操作list
    stringRedisTemplate.opsForSet();　　  //操作set
    stringRedisTemplate.opsForZSet();　 　//操作有序set
```
