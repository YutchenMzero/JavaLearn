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
