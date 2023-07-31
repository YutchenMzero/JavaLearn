### Redis
Redis：k-v持久化产品，内存型数据库,也可以持久化到磁盘。value支持
    * String(可以包含jpg图片或序列化对象，内部实现可以看作byte数组，上限1G字节),
    * Hash(string类型的field和value的映射表,适合存储对象),
    * List(string 类型的双向链表,操作中key理解为链表的名字),
    * Sets(提供集合操作),
    * Sorted sets(相对于set增加了顺序属性)
#### StringRedisTemplate
其key和value默认是String方式
``` java
    stringRedisTemplate.opsForValue();　　//操作字符串
    stringRedisTemplate.opsForHash();　　 //操作hash
    stringRedisTemplate.opsForList();　　 //操作list
    stringRedisTemplate.opsForSet();　　  //操作set
    stringRedisTemplate.opsForZSet();　 　//操作有序set
``` 
#### 注意事项
对于String类型的value，当使用set，setex等方法修改Redis中的value时，会重置原来设置的过期时间(用当前set操作的参数覆盖原有的数据)，因此为保证其过期时间不变，需要调用`getExpire()`方法返回剩余的过期时间，然后作为set操作的参数传入。
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
* `@RequiredArgsConstructor` 生成带有必需参数的构造函数（声明为final或者有`@NotNull`注解），写在类上可以代替`@Autowired`注解(采用了构造器注入的形式)
* `@Getter/@Setter` 生成get/set方法
* `@ToString/@EqualsAndHashCode` 生成toString，equals和hashcode方法，同时后者还会生成一个canEqual方法
* `@Data` 效果等同于 `@Getter` + `@Setter` + `@ToString` + `@EqualsAndHashCode` + `@RequiredArgsConstructor` 
* `@SneakyThrows` 生成try-catch逻辑
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

### [gradle](https://zhuanlan.zhihu.com/p/570009095)
#### 项目结构
**gradle**：gradle-wrapper存放位置
**src**:与maven目录一致
**build.gradle**： gradle项目构建文件
**gradlew**：gradle命令行工具
**settings.gradle**: 多模块项目配置文件
#### buildscript
1. buildscript中的声明是gradle脚本自身需要使用的资源。gradle在执行脚本时，会优先执行buildscript代码块中的内容，然后才会执行剩余的build脚本。buildscript代码块中你可以对dependencies使用classpath声明。
2. 该classpath声明说明了在执行其余的build脚本时，class loader可以使用这些你提供的依赖项。这也正是我们使用buildscript代码块的目的。某种意义上来说，classpath 声明的依赖，不会编译到最终的 jar包里面
3. 另外，buildscript必须位于plugins块和apply plugin之前
#### 依赖引入
使用dependcies代码块，遵循scope 'gropId:artifactId:version'的格式，也可以是scope (gropId:artifactId:version)形式，其中scope分为：
1. implementation：会将指定的依赖添加到编译路径，并且会将该依赖打包到输出，但是这个依赖在编译时不能暴露给其他模块，例如依赖此模块的其他模块。这种方式指定的依赖在编译时只能在当前模块中访问。
2. api：api关键字是由java-library提供，若要使用，请在plugins中添加：id 'java-library'（在旧版本中作用与compile相同，新版本移除了compile）使用api配置的依赖会将对应的依赖添加到编译路径，并将依赖打包输出，但是这个依赖是可以传递的，比如模块A依赖模块B，B依赖库C，模块B在编译时能够访问到库C，而implemetation不同的是，在模块A中库C也是可以访问的。
3. compileOnly：compileOnly修饰的依赖会添加到编译路径中，但是不会被打包，因此只能在编译时访问，且compileOnly修饰的依赖不会传递。
4. runtimeOnly：这个与compileOnly相反，它修饰的依赖不会添加到编译路径中，但是能被打包，在运行时使用。和Maven的provided比较接近。
5. annotationProcessor：用于注解处理器的依赖配置。
6. testImplementation：这种依赖在测试编译时和运行时可见，类似于Maven的test作用域。
7. testCompileOnly和testRuntimeOnly：这两种类似于compileOnly和runtimeOnly，但是作用于测试编译时和运行时。
8. classpath：classpath并不能在buildscript外的dependcies中使用
#### 仓库管理
gradle仓库可以直接使用maven的仓库，但是gradle下载的jar包文件格式与maven不一样，所以不能和maven本地仓库共用，仓库的配置，是在repository中的：
``` java
repositories {
     mavenLocal() //本地仓库
     maven { url 'http://maven.aliyun.com/nexus/content/groups/public' } //外部仓库（阿里云）
     //若这里解析报错，则将http替换为https，或者在url前边添加 allowInsecureProtocol = true
     mavenCentral() // maven 中心仓库
 }
```

### netty
一个异步事件驱动的网络应用程序框架，用于快速开发可维护的高性能协议服务器和客户端。底端采用TCP/IP协议，本质上是一个NIO框架。
Netty有两组线程池，一个Boss Group，它专门负责客户端连接，另一个Work Group，专门负责网络读写；

### 其他
#### 静态变量的自动注入
1. 采用set方法注入
```java
private static JavaMailSenderImpl mailSender;

@Autowired
public void setMailSender(JavaMailSenderImpl mailSender) {
    MailUtilsNew.mailSender = mailSender;
}
```
2. 利用`@PostConstruct`注解
```java
private static JavaMailSenderImpl mailSender;
@Autowired
private JavaMailSenderImpl myMailSender;
@PostConstruct
private void init(){
    mailSender = myMailSender;
}
```
#### @JsonCreator
默认情况下，Jackson在反序列化过程中会：
1. 首先调用反序列化的目标类的无参构造函数，构造一个java对象
2. 然后调用该类的成员变量的set方法，为该对象的每一个成员变量赋值。
当时用该注解时，可以自定义反序列化过程，一般用于构造方法和工厂静态方法上
```java
 @JsonCreator
  public PlayerStar3(@JsonProperty("name") String name,
                     @JsonProperty("age") Integer age,
                     @JsonProperty("hobbies") String[] hobbies,
                     @JsonProperty("friends") List<String> friends,
                     @JsonProperty("salary") Map<String, BigDecimal> salary) {
    this.name = name;
    this.age = age;
    this.hobbies = hobbies;
    this.friends = friends;
    this.salary = salary;
  }
```
#### @SneakyThrows
用于生成try-catch模板。
```java
@SneakyThrows
   public void utf8ToString(byte[] bytes) {
       return new String(bytes, "UTF-8");
   }
 //等同于
 public void utf8ToString(byte[] bytes) {
    try {
      return new String(bytes, "UTF-8");
    } catch (Exception e) {
      throw Lombok.sneakyThrow(e);
    }
}    

```
#### @Cleanup
指定的方法（这个方法需要在注解对象中存在）关闭资源，默认使用 close 方法

#### 时间
1. 获得相对于某个时区的时间
`LocalDateTime.now(ZoneId.of("UTC+00:00"))`
2. Date转换为LocalDateTime
```java
Date todayDate = new Date();

LocalDateTime ldt = todayDate.toInstant()
        .atZone( ZoneId.systemDefault() )
        .toLocalDateTime();
```
3. 获取当天开始和结束时间
 ```java
LocalDateTime startTime = LocalDateTime.of(dateTime.toLocalDate(), LocalTime.MIN);
LocalDateTime endTime = LocalDateTime.of(dateTime.toLocalDate(), LocalTime.MAX);
 ```
#### 定时任务
xxl-job

### spring 中的一些方法（不知道咋起名字了）
AnnotationUtils.findAnnotation

### 事务
声明式事务是Spring官方推荐的使用方式，但在实际应用中，由于声明式事务的粒度最小是在方法级别上的，在使用时很容易一不小心写出一个超大事务(如包含rpc远程调用，缓存更新，消息写入等操作)，导致涉及的数据库被长时间锁定，从而导致高并发场景下的业务失败。这时我们需要进行事务的拆分。
#### 关于事务失效问题的解决
在使用@Transactional注解时，最容易出现也是最容易被忽略的失效场景就是类间调用。即类中的A方法调用类中的B方法，而B方法开启了事务，A没有。这是用于采用注解开启事务时，其本质是通过动态代理获取对象。当未开启事务的方法A调用开启事务的方法B时，不会新建对象，自然也就不会开启事务了。为了解决这一问题常用的三种方法是：
1. 新建Service，将方法B放到新的service中，再通过这个service调用
2. 为当前的Service注入自己，[spring解决循环依赖的方式](https://mp.weixin.qq.com/s?__biz=MzU0OTE4MzYzMw==&mid=2247545567&idx=2&sn=8478f342befd6d2d84e3e11c635c4952&chksm=fbb1bb21ccc63237a4890e75a3b43a50b69ef88900fac7e784916fdff134cc94a058c192b63f&scene=27)：通过三级缓存解决循环依赖问题，但仅限单例。
```java
@Service
public class MyService{
    @Autowired
    private MyService myService;

    public void A(){
        //do()
        B();
    }
    @Transactional
    public void B(){
        //doSql()
    }

}
```
3. 动态代理:在该Service类中使用AopContext.currentProxy()获取代理对象
```java
@Servcie
public class ServiceA {
 
   public void save(User user) {
         queryData1();
         queryData2();
         ((ServiceA)AopContext.currentProxy()).doSave(user);
   }
 
   @Transactional(rollbackFor=Exception.class)
   public void doSave(User user) {
       addData1();
       updateData2();
    }
 }
```