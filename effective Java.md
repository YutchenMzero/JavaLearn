**引言**
由于在工作编码过程中，察觉到了自己在编码方面存在着许多的不足，在类的结构设计亦或者是业务的实现逻辑上总有一种不够优雅、不够效率的感觉，为此特进行相关的学习。旨在了解他人优秀的设计及编码思想的前提下，结合自身工作中遇到的问题，进行分析与归纳，以提升自己的水平。

## 一、创建与销毁对象

### 1. 用静态工厂方法代替构造器

其具有以下的优势：

1. 具有名称。当需要返回特定类型的实例时，有具体名称的静态工厂更容易使用
2. 可以为重复的调用返回相同的对象，减少创建对象的开销
3. 可以返回原本的返回类型的子类型的对象，其更大的优势在于，这些子类可以以非公有的形式实现，使API变得简洁，也更安全。
4. 所返回对象的类可以取决于静态工厂方法的参数值，而发生变化。例如EnumSet,其根据枚举的数量返回自身的两种子类之一，而这两个类是不可见的，即便在之后做了任何的修改，其影响也是极小的（不存在直接调用这两个类的情况）。

    ```java
    public static <E extends Enum<E>> EnumSet<E> noneOf(Class<E> elementType) {
            Enum<?>[] universe = getUniverse(elementType);
            if (universe == null)
                throw new ClassCastException(elementType + " not an enum");

            if (universe.length <= 64)
                return new RegularEnumSet<>(elementType, universe);
            else
                return new JumboEnumSet<>(elementType, universe);
        }
    ```

5. 方法返回的对象所属的类，可以在编写包含该静态工厂的类时不存在。这是服务提供者框架的基础。

    **服务提供者框架**

    ```java
    //四大组成之一：服务接口:由提供者实现
    public interface LoginService {//这是一个登录服务
        public void login();
    }
    
    //四大组成之二：服务提供者接口
    public interface Provider {//登录服务的提供者。通俗点说就是：通过这个newLoginService()可以获得一个服务。
        public LoginService newLoginService();
    }
    
    /**
     * 这是一个服务管理器，里面包含了四大组成中的三和四
     * 解释：通过注册将 服务提供者 加入map，然后通过一个静态工厂方法 getService(String name) 返回不同的服务。
     */
    public class ServiceManager {
        private static final Map<String, Provider> providers = new HashMap<String, Provider>();//map，保存了注册的服务
    
        private ServiceManager() {
        }
    
        //四大组成之三：提供者注册API  (其实很简单，就是注册一下服务提供者)
        public static void registerProvider(String name, Provider provider) {
            providers.put(name, provider);
        }
    
        //四大组成之四：服务访问API：客户端用来获取服务实例的   (客户端只需要传递一个name参数，系统会去匹配服务提供者，然后提供服务)  (静态工厂方法)
        public static LoginService getService(String name) {
            Provider provider = providers.get(name);
            if (provider == null) {
                throw new IllegalArgumentException("No provider registered with name=" + name);
    
            }
            return provider.newLoginService();
        }
    }
    ```

其缺点在于：

1. 若类不含有任何公有或者受保护的构造器，就不能子类化（从某种程度上也是件好事，它强制程序员使用组合而不是继承）
2. 难以发现，因此需要遵守命名规范。

36.65.5.59.18

### 3. 用私有构造器或者枚举类强化singleton属性

Singleton是指仅被实例化一次的类，通常用来表示一个无状态的对象，如函数。
实现方式：

1. 构造器私有，公有静态成员是final域。优先考虑这一方法
2. 构造器私有，成员私有且final，有一个公有的静态构造器。
    * 提供了灵活性，可以在不改变API的前提下，改变是否提供Singleton
    * 可以编写一个泛型Singleton工厂
    * 可以通过方法引用的方式作为提供者，如`Test::getInstance`
3. 声明包含单个元素的枚举类型。是最佳方法，但不适用于必须扩展除Enum的超类的场景

注意：

1. 对于第1、2种方法，为了防止借助`AccessibleObject.setAccessible()`方法，通过反射调用私有构造器，需要修改构造器使它在第二次调用时抛出异常。
2. 为了防止每次反序列化时都创建一个新的实例，需要在类中添加以下方法：

    ```java
    private Object readResolve(){
        return INSTANCE;
    }
    ```

**AccessibleObject.setAccessible**
Java反射中的Field、Method和Constructor均继承自AccessibleObject类，通过调用它的setAccessible方法，可以在应用时取消Java语言检查，从而直接调用其私有方法。

24.65.30.12.89

### 4. 通过私有构造器强化不可实例化的能力

## 四、类和接口

### 17. 使可变性最小化

不可变类是指实例不可被修改的类。每个实例包含的所有信息都在创建时就确定，并在整个生命周期内固定不变。其更容易设计、实现和使用，且不容易出错更加安全。
为了成为不可变类，需要遵守以下五个规则：

1. 不要提供任何会修改对象状态的方法，因此需要构造器可以创建完全初始化的对象
2. 保证类不会被扩展，一般做法是将其声明为final。或者让类的所有构造器都变成私有。
3. 所有的域都声明为final，实际上应该是没有一个方法对对象的状态产生外部可见的改变
4. 所有的域都声明为私有的
5. 确保对于任何可变组件的互斥访问：如果类具有指向可变对象的域，则必须确保该类的客户端无法获得指向这些对象的引用。且永远不要使用客户端提供的对象来初始化这样的域，也不要返回这样的域的对象引用。在构造器、访问方法和readObject方法中使用保护性拷贝

其优点在于：

1. 本质上是线程安全的，不要求同步，可以被自由地共享，对于频繁使用的值，为他们提供公有的静态final常量。
2. 提供了原子性，不存在临时不一致的情况

其唯一的不足在于，对于每个不同的值都需要一个单独的对象。当其要进行复杂的操作时，第一种方法是对于常用操作，将它们作为正常类型提供。第二种方法是提供一个配套的公有的可变类，如String和StringBuilder

15.16.88.50.13.6.76.83.88

### 20.接口优于抽象类
18.21.19.24.

## 六、枚举和注解

### 34. 用enum代替int常量

14.12.16.15.24.7.55.

### 36. 用EnumSet代替位域


## 九、通用编程

### 64.通过接口引用对象

51.