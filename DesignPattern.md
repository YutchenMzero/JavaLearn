## 面向对象设计
### uml
#### 类图
1. 组合与聚合，其判断关键就是生命周期是否一致，在代码中往往体现为是否通过是否由该类创造另一个类的对象，例如若A类的构造方法中创造了B类的对象，则B和A是组合关系。
### 设计原则
#### A. 单一职责原则
&#8195;&#8195;即就一个类而言，应该仅有一个引起它变化的原因。
1. 如果一个类承担的职责过多，就等于把这些职责耦合在一起，一个职责的变化可能会削弱或抑制这个类完成其他职责的能力。这种耦合会导致脆弱的设计，当变化发生时，设计会遭受到意想不到的破坏。
2. 如果能想到多于一个动机去改变一个类，那么这个类就具有多于一个的职责。
#### B. 开放-封闭原则
&#8195;&#8195;软件实体（类，模块，函数等等）应该可以扩展，但是不可修改。即对于扩展时开放的，但对于更改是封闭的。
1. 设计人员必须对于他设计的模块应该对那种变化封闭做出选择。他必须先猜测出最有可能发生的变化种类，然后狗仔抽象来隔离哪些变化。（如策略模式中，采用的不同策略，均继承于同一抽象类）
2. 在最初编写代码时，假设变化不会发生，当发生小变化时，立刻采取行动，创建抽象来隔离以后发生的同类变化。且越早创建正确的抽象，修改的代价就越低。
3. 其精神在于：面对需求，对程序的改动是通过增加新代码进行的，而不是更改现有代码。
#### C. 里氏代换原则
&#8195;&#8195;子类型必须能够替换掉他们的父类型。
1. 即在软件中，把父类都替换成它的子类，程序的行为没有变化。（指对象的静态类型）
#### D. 依赖倒转原则
&#8195;&#8195;抽象不应该依赖细节，细节应该依赖于抽象。高层模块不应该依赖底层模块，二者都应依赖于抽象。
1. 即针对接口编程，不要针对实现编程。
2. 若程序中所有的依赖关系都是终止于抽象类或者接口，则是面向对象的设计。
#### E. 
## 设计模式
### 一. 简单工厂模式
使用单独的工厂类进行创造实例的过程，实现功能的解耦。下图展示了一个简单工厂模式的UML的示意。
![简单工厂UML示意图](DesignPattern/SimpleFactory.png)
```java
public class Solution {
    public static void main(String[] arg){
        MOperation mOperation  = OperationFactory.operationFactory("+");
        mOperation.setOperation_num_A(5);
        mOperation.setOperation_num_B(6);
        System.out.println(mOperation.getResult());
    }
}
class MOperation{

    private double operation_num_A = 0;
    private double operation_num_B = 0;
    public double getOperation_num_A() {
        return operation_num_A;
    }
    public void setOperation_num_A(double operation_num_A) {
        this.operation_num_A = operation_num_A;
    }
    public double getOperation_num_B() {
        return operation_num_B;
    }
    public void setOperation_num_B(double operation_num_B) {
        this.operation_num_B = operation_num_B;
    }
    public double getResult() {return 0;}
}
class MOperationAdd extends MOperation{
    @Override
    public double getResult() {
        return getOperation_num_A()+getOperation_num_B();
    }
}
class MOperationSub extends MOperation{
    @Override
    public double getResult() {
        return getOperation_num_A()-getOperation_num_B();
    }
}
class  MOperationMul extends MOperation{
    @Override
    public double getResult() {
        return getOperation_num_A()*getOperation_num_B();
    }
}
class  MOperationDiv extends MOperation{
    @Override
    public double getResult() {
        if (getOperation_num_B()==0)
            try {
                throw new Exception("除数不能为0！");
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        return getOperation_num_A()/getOperation_num_B();

    }
}
class OperationFactory{
    public static MOperation operationFactory(String operation){
        switch (operation){
            case "+":
                return new MOperationAdd();
            case "-":
                return new MOperationSub();
            case "*":
                return new MOperationMul();
            case "/":
                return new MOperationDiv();
            default:
                return new MOperation();
        }
    }
}
```
### 二. 策略模式
&#8195;&#8195;策略模式（Strategy）定义了算法家族，分别封装起来，让它们之间可以互相替换，此模式让算法的变化不会影响到使用算法的客户。即当算法会出现经常性的变换时(或该算法更注重变化时)，将变化点封装起来。其基本结构如下所示：
![策略模式基本结构](DesignPattern/1.png)
&#8195;&#8195;该模式所定义的算法家族从概念上来看，完成的都是相同工作，只是实现不同，他可以以相同方式调用所有的算法，减少了算法类和使用算法类之间的耦合。而其Strategy类层次为Context定义了一系列可供重用的算法或行为，继承有助于析取出这些算法中的公共功能。此外：
1. 其简化了单元测试，因为每个算法都有自己的类，可通过自己的接口单独测试
2. 在基本的策略模式中，选择所用具体实现的职责由客户端对象承担，并转给策略模式的Context对象。

以下展示了策略模式和简单工厂模式的结合：
```java
public class Solution {
    public static void main(String[] arg){
        StrategyContext strategy_context = new StrategyContext("A");
        strategy_context.operationContextInterface();
    }
}
//Strategy
abstract class  Strategy {
    public  abstract void doWork();
}
class StrategyA extends Strategy {
    @Override
    public void doWork() {
        System.out.println("策略A");
    }
}
class StrategyB extends Strategy {
    @Override
    public void doWork() {
        System.out.println("策略B");
    }
}
class StrategyC extends Strategy {
    @Override
    public void doWork() {
        System.out.println("策略C");
    }
}
class StrategyD extends Strategy {
    @Override
    public void doWork() {
        System.out.println("策略D");
    }

}
class StrategyContext {
    Strategy strategy =null;
    public StrategyContext(String operation){
        switch (operation){
            case "A":
                strategy = new StrategyA();
                break;
            case "B":
                strategy = new StrategyB();
                break;
            case "C":
                strategy =  new StrategyC();
                break;
            case "D":
                strategy =  new StrategyD();
                break;
        }
    }
    public void operationContextInterface(){
        if(strategy ==null){
            try {
                throw new Exception("所选算法无效");
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }
        strategy.doWork();
    }
}
```
### 三、 装饰模式
&#8195;&#8195;装饰模式：动态地给一个对象添加一些额外的职责，就增加功能来说，装饰模式比生成子类更加灵活。其基本结构如下：
![装饰模式结构图](DesignPattern/2.png)
1. 装饰模式利用`SetComponent`来对对象进行包装，这样每个装饰对象的实现就和如何使用这个对象分离开了，每个装饰对象只关心自己得到功能，不需要1关心如何被添加到对象链当中。
2. 如果只有一个`ConcreteComponent`类而没有抽象的`Component`类，那么`Decorator`类可以是`ConcreteComponent`的一个子类。如果只有一个`ConcreteDecorator`，那么就没有必要建立一个单独的`Decorator`类，而可以把`Decorator`和`ConcreteDecorator`的职责合并成一个类。
3. 当系统需要新功能时，向旧的类中添加新的代码，这些新加的代码通常装饰了原有类的核心职责或主要行为，但会增加主类的复杂度，然而可能新加入的东西只是在特定情况下才会执行的。此时使用装饰模式可以把每个要装饰的功能放在单独的类中，并让这个类包装它所要装饰得到对象。这样可以有效地把类的核心职责和装饰功能区分开，并去除相关类中重复的装饰逻辑。
```java
public class Solution {
    public static void main(String[] arg){
    ConcreteComponent concrete_component = new ConcreteComponent("待装饰类");
    DecoratorA decorator_A = new DecoratorA();
    DecoratorB decorator_B = new DecoratorB();
    decorator_A.setComponent(concrete_component);
    decorator_B.setComponent(decorator_A);
    decorator_B.doWork();
    }
}
abstract class Component{
    abstract void doWork();
}
class ConcreteComponent extends Component{
    private String name;
    ConcreteComponent(String name){
        this.name = name;
    }
    @Override
    void doWork() {
        System.out.print("装饰了"+ name);
    }
}
abstract class Decorator extends Component{
    protected Component component;
    public void setComponent(Component component){
        this.component = component;
    }

    @Override
    void doWork() {
        if (component != null){
            component.doWork();
        }
    }
}
class DecoratorA extends Decorator{
    @Override
    void doWork() {
        System.out.print("A方法 ");
        super.doWork();
    }
}
class DecoratorB extends Decorator{
    @Override
    void doWork() {
        System.out.print("B方法 ");
        super.doWork();
    }
}
```
### 四、代理模式
&#8195;&#8195;代理模式：为其他对象提供一种代理以控制对这个对象的访问。其基本结构如下所示：
![代理模式结构图](DesignPattern/3.png)
1. 应用场景：
   * 远程代理：也就是为一个对象在不同的地址空间提供局部代表。这样可以隐藏一个对象存在于不同地址空间的事实。
   * 虚拟代理：根据需要创建开销很大的对象。通过它来存放实例化需要很长时间的真是对象。（如HTML网页中的图片加载）
   * 安全代理：用来控制真实对象访问时的权限。
   * 智能指引：是指调用真实的对象时，代理处理另外一些事。
2. 要求代理和真实对象均继承自同一父类或实现同一接口。
```java
public class Solution {
    public static void main(String[] arg){
        Proxy proxy = new Proxy(new RealSubjects());
        proxy.doWork();
    }
}
abstract class Subjects{
    public abstract void doWork();
}
class RealSubjects extends Subjects{
    @Override
    public void doWork() {
        System.out.println("代理执行");
    }
}
class Proxy extends Subjects{
    RealSubjects real_subjects;
    public Proxy(RealSubjects real_subjects){
        this.real_subjects = real_subjects;
    }
    @Override
    public void doWork() {
        if (real_subjects==null){
            real_subjects = new RealSubjects();
        }
        real_subjects.doWork();
    }
}
```

---
end at the page 85