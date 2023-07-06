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
### controller的单元测试
1.在需测试的controller中右键单击选择"generate"选项，生成test。(注意：测试文件的目录需要提前建立)
2.假设要测试的类为`DemoController`，自动生成的测试类为`demoControllerTest`测试方式为
```java
@SpringBootTest
class DemoControllerTest {
    private MockMvc mockMvc;

    @Autowired
    private DemoController demoController;//需要测试的接口


    @BeforeEach
    void setUp() {
        mockMvc = MockMvcBuilders.standaloneSetup(DemoController).build();
    }

    @Test
    void functionA() throws Exception {    
      System.out.println("-------------------");
      System.out.println(LocalDate.now());
      EmailDTO request = new InputParam();
      request.setA("xxxx");
      //post形式
      MvcResult mvcResult = mockMvc.perform(MockMvcRequestBuilders.post("/url")
                  .contentType(MediaType.APPLICATION_JSON)
                  .content(JSONObject.toJSONString(request)))
              .andExpect(MockMvcResultMatchers.status().isOk())
              .andDo(MockMvcResultHandlers.print())                 
              .andReturn();
    }
    /*
    get-@RequestParam("id"):
        MvcResult mvcResult = mockMvc.perform(
                MockMvcRequestBuilders.get("/getId")
                        .param("id", "123"))
                ...
    get-@PathVariable:
        int id = 111;
        MvcResult mvcResult = mockMvc.perform(
                MockMvcRequestBuilders.get("/getId"+id))
                ...
    get-类传值:
        MvcResult mvcResult = mockMvc.perform(
                MockMvcRequestBuilders.get("/getUser")
                        .param("userId", "123")
                        .param("name", "JCccc")
                        .param("age", "18")
                        .param("userCode", "100244")
                        .contentType(MediaType.APPLICATION_FORM_URLENCODED_VALUE))
    post-RequestHeader:
        MvcResult mvcResult = mockMvc.perform(
                MockMvcRequestBuilders.post("/getRequestBodyValue")
                        .header("token", "收藏点赞")
                        .accept(MediaType.parseMediaType("application/json;charset=UTF-8")))
    */
}
```

### [Lombok] (https://projectlombok.org/features/)
用于简化实体类编写的注解
常用注解有：
* `@NoArgsConstructor/@AllArgsConstructor` 生成无参/全参构造函数
* `@Getter/@Setter` 生成get/set方法
* `@ToString/@EqualsAndHashCode` 生成toString，equals和hashcode方法，同时后者还会生成一个canEqual方法
* `@Data` 效果等同于 `@Getter` + `@Setter` + `@ToString` + `@EqualsAndHashCode` + `@RequiredArgsConstructor` 
* `@Accessors` 主要服务于get和set方法
    * fluent 属性 : 生成的方法前不会有"get"和"set"前缀，且支持set方法的链式调用
    * chain 属性 : 支持set方法的链式调用
    * prefix 属性 : 生成get与set方法时会去除指定前缀

### [MyBatis plus](https://baomidou.com/pages/24112f/)
内置了通用Mapper和通用Service，支持lambda表达式
1. BaseMapper中集成了大量的通用CRUD操作，自定义的Mapper只需继承该类就可实现常用功能；
    常用方法：
    * insert()
    * update()
    * selectById()
    * exist()
2. ServiceImpl中集成了常用的服务，自定义的Service只需继承它就可以实现常用的数据库操作，继承要求：`... extend ServiceImpl<M extends BaseMapper<T>, T>`
#### 条件构造器Wrapper
可以使用wrapper构造较为复杂的SQL;[参考资料](https://blog.csdn.net/qq_39715000/article/details/120090033)
#### 其他
1. 通过`@TableField(fill = FieldFill.INSERT)//FieldFill.INSERT_UPDATE`自动生成创建和更新时间

