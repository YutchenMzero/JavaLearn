### JAVA基本知识
* `StringBuilder`和`StringBuffer`可灵活扩展与更改，避免了`String`每次’更改’都会创建新对象的问题，其中`StringBuffer`线程安全，方法接口被`synchronized `修饰。

### 1. List、Map与Set
![类图](Javauml/setlistandmap.png)
#### List
* 可允许重复对象，可插入多个NULL值。
* 是一个有序容器，输出顺序是插入顺序。
* 常用实现类ArrayList底层为数组，用于随意访问；实现类LinkedList底层为链表，用于灵活增删；Vector底层为数组，但由于线程安全，效率比ArrayList低。
#### Set
* 不允许重复对象，只能有一个NULL值。
* 是无序容器。
* 其实现类TreeSet根据compare()和compareTo()方法进行排序的有序容器。
#### Map
* 以键值对的形式存储。
* 键不可重复，值可重复，允许存NULL。
### 2. equals()与==   
1. `==`针对基本数据类型时，比较的是值是否相同，引用数据类型时是比较引用地址是否相同。
2. `equals()`属于Object中的方法，所有类都可重写该方法，但基本数据类型不可使用。
   ```java
     public boolean equals(Object obj) {
        return (this == obj);
    }
   ```
3. String类中先判断`==`,然后再判断值是否相同。
```java
  public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        return (anObject instanceof String aString)
                && (!COMPACT_STRINGS || this.coder == aString.coder)
                && StringLatin1.equals(value, aString.value);
    }

```
4. 包装类中会先判断包装类型是否相同，再判断值是否相同；
```java
public boolean equals(Object obj) {
        if (obj instanceof Integer) {
            return value == ((Integer)obj).intValue();
        }
        return false;
    }
```
5. 当重写`equals()`方法时，为了维持Java中的一般规定，即`equals()`方法判断相同的对象的hashcode应当相同，一般需要重写`hashCode()`方法。
### 3. Object类
&#8195;&#8195;是Java类结构中的根节点，每个类都要继承Object类，且实现该类中的方法。以下对其中的一些主要方法进行简要介绍：
1. hashCode()
   该方法根据对象的地址计算hash值；
```java
    @IntrinsicCandidate
    public native int hashCode();
```
1. equals()
2. clone()

&#8195;&#8195;对于任意对象x，应保证：   
```
  x.clone() != x
  x.clone().getClass() == x.getClass()
  x.clone().equals(x)
```
成立，但不是必须的。重写Object类中的clone()方法需要实现Cloneable接口，否则会抛出CloneNotSupportedException；
* Object类并未实现Cloneable接口，因此对Object类直接调用该方法会报错了；
* 所有数组都被视为实现了Cloneable接口；
* 如果覆盖了非final类中的clone方法，则应该返回一个通过调用super.clone()而得到的对象；因此要尽量避免在clone方法中使用构造器；
* 浅克隆对于引用类型而言，复制的是引用地址；深克隆对于引用类型而言，复制的是所引用的值。
4. toString()
```java
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```
### 4. 内部类
&#8195;&#8195;在Java中，可以将一个类定义在另一个类里面或者一个方法里边，这样的类称为内部类，广泛意义上的内部类一般包括四种：成员内部类，局部内部类，匿名内部类，静态内部类 。
1. 成员内部类
* 该类像是外部类的一个成员，可以无条件的访问外部类的所有成员属性和成员方法（包括private成员和静态成员）；
* 成员内部类拥有与外部类同名的成员变量时，会发生隐藏现象，即默认情况下访问的是成员内部类中的成员。如果要访问外部类中的成员，需要以下形式访问：外部类.this.成员变量  或  外部类.this.成员方法； 
* 在外部类中如果要访问成员内部类的成员，必须先创建一个成员内部类的对象，再通过指向这个对象的引用来访问；
* 成员内部类是依附外部类而存在的，也就是说，如果要创建成员内部类的对象，前提是必须存在一个外部类的对象；
* 内部类可以拥有private访问权限、protected访问权限、public访问权限及包访问权限。如果成员内部类用private修饰，则只能在外部类的内部访问；如果用public修饰，则任何地方都能访问；如果用protected修饰，则只能在同一个包下或者继承外部类的情况下访问；如果是默认访问权限，则只能在同一个包下访问。外部类只能被public和包访问两种权限修饰。
    ```java
   public class Solution {
    int test = 2;
    //必须先创建成员内部类的对象，才能访问其成员
    mySolution mysolution = new mySolution();
    public int get(){
        return  mysolution.test;
    }
    public mySolution getMysolution(){
        return new mySolution();
    }
    //内部类
    public class  mySolution {
        int test;
        mySolution() {
            test = Solution.this.test;//使用<OuterClass>.this.<name>访问外部类中的同名属性或方法
        }
    }
    }
    class A{
    //要创建成员内部类的对象，前提是必须存在一个外部类的对象；
    Solution s = new Solution();
    Solution.mySolution ms = s.new mySolution();
    Solution.mySolution ms2 = s.getMysolution();
    }
    ``` 
1. 局部内部类
* 局部内部类是定义在一个方法或者一个作用域里面的类，它和成员内部类的区别在于局部内部类的访问仅限于方法内或者该作用域内；
* 局部内部类就像是方法里面的一个局部变量一样，是不能有public、protected、private以及static修饰符的。
* 在局部内部类中不能使用方法中可变的局部变量（因为方法中的局部变量位于栈中，而对象分配在堆中，生命周期不同，为解决这一问题，Java把局部内部类要访问的局部变量重新拷贝了一份，并把备份放在内部类的常量池中，这样就不会出现访问不存在的变量的错误了，但为了出现内部类和方法体都修改了局部变量的值，出现数据不同步的问题，所以要求其可访问的变量必须被final修饰）
3. 匿名内部类
* 一般使用匿名内部类的方法来编写事件监听代码；
* 匿名内部类是不能有访问修饰符和static修饰符的；
* 匿名内部类是唯一一种没有构造器的类；
* 匿名内部类用于继承其他类或是实现接口，并不需要增加额外的方法，只是对继承方法的实现或是重写。
4. 内部静态类
* 静态内部类是不需要依赖于外部类的，这点和类的静态成员属性有点类似；
* 不能使用外部类的非static成员变量或者方法。
### 5. switch方法
&#8195;&#8195;switch只接受int类型的数据：
* switch可以支持byte、short、char、int、Byte、Short、Character、Integer、String和枚举类型的数据
* byte、short、char自动转变为int类型，
* Byte、Short、Character、Integer--JDK1.5自动拆箱，转变为int类型进行匹配
* 同时随着jdk1.5中新增了枚举类型，枚举类型底层是使用了枚举类的ordinal方法，返回的是枚举常量的序号，其序号是int类型，所以可以作为switch的参数。
* 在jdk1.7中string类型底层使用了hashCode方法，返回的数据类型为哈希码，也是int类型。
### 6. 自动类型转换
自动类型转换遵循下面的规则：
* 若参与运算的数据类型不同，则先转换成同一类型，然后进行运算。
* 转换按数据长度增加的方向进行，以保证精度不降低。例如int型和long型运算时，先把int量转成long型后再进行运算。
* 所有的浮点运算都是以双精度进行的，即使仅含float单精度量运算的表达式，也要先转换成double型，再作运算。
* char型和short型参与运算时，必须先转换成int型。
* 在赋值运算中，赋值号两边的数据类型不同时，需要把右边表达式的类型将转换为左边变量的类型。如果右边表达式的数据类型长度比左边长时，将丢失一部分数据，这样会降低精度。
下图表示了类型自动转换的规则：
![自动转换规则](Javauml/415611_1442458661106_F4A62FDD254F710F39378C754ED65E61.jpeg)
### 7. volatile和synchronized的区别
* volatile本质是在告诉jvm当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取； synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
* volatile仅能使用在变量级别；synchronized则可以使用在变量、方法、代码块和类级别的。
* volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞。
* volatile标记的变量不会被编译器优化；synchronized标记的变量可以被编译器优化。
* volatile能保证数据的可见性，但不能完全保证数据的原子性（不能保证复合操作的原子性），synchronized即保证了数据的可见性，也保证了原子性。
### 8. 多线程相关属性
#### 可见性：
&#8195;&#8195;可见性，是指线程之间的可见性，一个线程修改的状态对另一个线程是可见的。也就是一个线程修改的结果。另一个线程马上就能看到。比如：用volatile修饰的变量，就会具有可见性。volatile修饰的变量不允许线程内部缓存和重排序，即直接修改内存。所以对其他线程是可见的。但是这里需要注意一个问题，volatile只能让被他修饰内容具有可见性，但不能保证它具有原子性。比如 volatile int a = 0；之后有一个操作 a++；这个变量a具有可见性，但是a++ 依然是一个非原子操作，也就是这个操作同样存在线程安全问题。在 Java 中 volatile、synchronized 和 final 实现可见性。
#### 原子性：
&#8195;&#8195;原子是世界上的最小单位，具有不可分割性。比如 a=0；这个操作是不可分割的，那么我们说这个操作时原子操作。再比如：a++； 这个操作实际是a = a + 1；是可分割的，所以他不是一个原子操作。非原子操作都会存在线程安全问题，需要我们使用同步技术（sychronized）来让它变成一个原子操作。一个操作是原子操作，那么我们称它具有原子性。java的concurrent包下提供了一些原子类，我们可以通过阅读API来了解这些原子类的用法。比如：AtomicInteger、AtomicLong、AtomicReference等。在 Java 中 synchronized 和在 lock、unlock 中操作保证原子性。
#### 有序性：
&#8195;&#8195;Java 语言提供了 volatile 和 synchronized 两个关键字来保证线程之间操作的有序性，volatile 是因为其本身包含“禁止指令重排序”的语义，synchronized 是由“一个变量在同一个时刻只允许一条线程对其进行 lock 操作”这条规则获得的，此规则决定了持有同一个对象锁的两个同步块只能串行执行。
### 9. 基本类型的装箱和拆箱
&#8195;&#8195;装箱过程是通过调用包装器的valueOf方法实现的，而拆箱过程是通过调用包装器的 xxxValue方法实现的.
&#8195;&#8195;对于Integer类型，当int的值在[-128,127]之间的时候,valueOf会返回指向IntegerCache.cache中已经存在的对象的引用；否则创建一个新的Integer对象.Integer和int比较会进行自动拆箱，比较的是数值大小。
```java
 public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```
### 10. 默认方法
