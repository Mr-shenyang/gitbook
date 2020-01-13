# 1 简述

软件系统架构，离不开底层原理的支撑，以及设计模式的使用。本文就深入学习下，软件设计中7大原则和常用模式。

# 2 设计主要原则

在介绍设计模式之前，需要理解软件设计的主要原则，设计模式其实就是围绕这些原则展开的。
**总原则：开闭原则（Open Close Principle）**
就是说对扩展开放，对修改关闭。在程序需要进行拓展的时候，不能去修改原有的代码，而是要扩展原有代码，实现一个热插拔的效果。

* 1 单一职责原则
  一个类或接口仅有一个职责，或者说只因一个原因变化而变化。

* 2 里氏替换原则（Liskov Substitution Principle）
  里氏代换原则中说，任何基类可以出现的地方，子类一定可以出现，子类对父类的方法尽量不要重写和重载。因为父类代表了定义好的结构，通过这个规范的接口与外界交互，子类不应该随便破坏它

* 3 依赖倒转原则（Dependence Inversion Principle）
  面向接口编程，依赖于抽象而不依赖于具体。写代码时用到具体类时，不与具体类交互，而与具体类的上层接口交互。

* 4 接口隔离原则（Interface Segregation Principle）
  每个接口中不存在子类用不到却必须实现的方法，如果不然，就要将接口拆分。使用多个隔离的接口，比使用单个接口（多个接口方法集合到一个的接口）要好

* 5 迪米特法则（最少知道原则）（Demeter Principle）
  一个类对自己依赖的类知道的越少越好。也就是说无论被依赖的类多么复杂，都应该将逻辑封装在方法的内部，通过public方法提供给外部。这样当被依赖的类变化时，才能最小的影响该类。

* 6、合成复用原则（Composite Reuse Principle）
  原则是尽量首先使用合成/聚合的方式，而不是使用继承

# 3 设计模式概述

![设计模式关系图](/images/design_patterns.png)

# 4 创建型模式

创建型模型抽象了对象实例化的过程

## 4.1 构造模式

**意图：** 将一个复杂对象的构建，交由专门的构造者来进行构造。这样即分离了对象的表示和构造，又便于理解。
**适用范围：** 复杂对象
**实现要点：** builder类持有被构造类的所有的属性，并提供方法生成被构造类；

```java
/*** 构造者模式**/
public class BuilderDemo {
    private String field1;
    private String field2;
    //省略get、set
    /***构造者***/
    public static class BuilderDemoBuilder{
        private String field1;
        private String field2;
        public BuilderDemoBuilder field1(String field1){
            this.field1 = field1;
            return this;
        }
        public BuilderDemoBuilder field2(String field2){
            this.field2 = field2;
            return this;
        }
        public BuilderDemo builder(){
            BuilderDemo builderDemo = new BuilderDemo();
            builderDemo.setField1(this.field1);
            builderDemo.setField2(this.field2);
            return builderDemo;
        }
    }
}
```

## 4.2 单例模式

**意图：** 保证了一个类仅有一个实例；
**适用范围：** 一个全局使用的类
**实现要点：** 构造器是私有的，双段锁

```java
public class SingletonDemo {
    /*** 私有化*/
    private SingletonDemo() {}

    private static volatile SingletonDemo singletonDemo = null;

    /*** 双段锁* */
    public static SingletonDemo getSingletonDemo(){
        if (singletonDemo == null){
            synchronized(SingletonDemo.class){
                if(singletonDemo == null){
                    singletonDemo = new SingletonDemo();
                }
            }
        }
        return singletonDemo;
    }
}
```

## 4.3 原型模式

**意图：** 用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

## 4.4 工厂方法模式

## 4.5 抽象工厂模式

# 5 结构型模式

结构型模式是通过对结构进行相关调整，来减少耦合的效果；
![设计模式关系图](/images/architecture/design_mode/structure_mode.png)

## 5.1 适配器模式

适配器指的是
适配器模式根据适配类型的不同分成，类适配器、对象适配器和接口适配器。

### 5.1.1 类适配器

### 5.1.2 对象适配器

### 5.1.3 接口适配器

## 5.2 桥接模式

## 5.3 组合模式

## 5.4 装饰者模式

## 5.5 外观模式

## 5.6 享元模式

## 5.7 代理模式

# 6 行为模式

## 6.1 责任链模式

## 6.2 命令模式

## 6.3 解释器模式

## 6.4 迭代器模式

## 6.5 中介者模式

## 6.6 备忘录模式

## 6.7 观察者模式


## 6.8 状态模式-status

## 6.9 策略模式-strategy

**意图：** 利用面向对象的继承和多态机制，实现。

**适用范围：** 同一算法不同实现方式

**实现要点：** 
 #1. Context支持策略注册，策略实现基于多态
 #2. 不同策略实现基于继承和多态

**UML：**
![策略模式](/images/architecture/design_mode/strategy-mode.png)

## 6.10 模板方法模式

## 6.11 访问者模式
