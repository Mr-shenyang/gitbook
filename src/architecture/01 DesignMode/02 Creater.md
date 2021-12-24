# 创建型模式

创建型模型抽象了对象实例化的过程

## 1 构造模式

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

## 2 单例模式

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

## 3 原型模式

**意图：** 用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

## 4 工厂方法模式


## 5 抽象工厂模式
