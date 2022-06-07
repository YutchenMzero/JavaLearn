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
```java
    @IntrinsicCandidate
    public native int hashCode();
```
2. equals()
3. clone()

&#8195;&#8195;对于任意对象x，应保证：   
```
  x.clone() != x
  x.clone().getClass() == x.getClass()
  x.clone().equals(x)
```
成立，但不是必须的。重写Object类中的clone()方法需要实现Cloneable接口，否则会抛出CloneNotSupportedException；
* Object类并未实现Cloneable接口，因此对Object类直接调用该方法会报错了；
* 所有数组都被视为实现了Cloneable接口；
* 如果覆盖了非final类中的clone方法，则应该返回一个通过调用super.clone()而得到的对象；
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
* 成员内部类拥有与外部类同名的成员变量时，会发生隐藏现象，即默认情况下访问的是成员内部类中的成员。如果要访问外部类中的成员，需要以下形式访问：【外部类.this.成员变量  或  外部类.this.成员方法】；
* 在外部类中如果要访问成员内部类的成员，必须先创建一个成员内部类的对象，再通过指向这个对象的引用来访问；
* 成员内部类是依附外部类而存在的，也就是说，如果要创建成员内部类的对象，前提是必须存在一个外部类的对象；
* 内部类可以拥有private访问权限、protected访问权限、public访问权限及包访问权限。如果成员内部类用private修饰，则只能在外部类的内部访问；如果用public修饰，则任何地方都能访问；如果用protected修饰，则只能在同一个包下或者继承外部类的情况下访问；如果是默认访问权限，则只能在同一个包下访问。外部类只能被public和包访问两种权限修饰。
2. 局部内部类
* 局部内部类是定义在一个方法或者一个作用域里面的类，它和成员内部类的区别在于局部内部类的访问仅限于方法内或者该作用域内；
* 局部内部类就像是方法里面的一个局部变量一样，是不能有public、protected、private以及static修饰符的。
3. 匿名内部类
* 一般使用匿名内部类的方法来编写事件监听代码；
* 匿名内部类是不能有访问修饰符和static修饰符的；
* 匿名内部类是唯一一种没有构造器的类；
* 匿名内部类用于继承其他类或是实现接口，并不需要增加额外的方法，只是对继承方法的实现或是重写。
4. 内部静态类
* 静态内部类是不需要依赖于外部类的，这点和类的静态成员属性有点类似；
* 不能使用外部类的非static成员变量或者方法。