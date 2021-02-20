
# springboot初始化流程图

在详细介绍spring初始化流程之前，先将流程图画在这里

# spring demo

在详细介绍spring的初始化流程之前，先说下demo
这个demo是一个打印服务

```java
/**
 * 打印机服务
 *
 * @author
 * @create 2019-06-23 16:37
 **/
public interface PrinterService {
/***
 * 打印服务
 * @param msg 打印内容
 */
   void print(String msg);
}

```

打印服务的实现方包括惠普打印机和华为打印机

```java
@Service
public class HpPrinterImpl implements PrinterService {
    public void print(String msg) {
        System.out.println("Hp printer service for you :");
        System.out.println(msg);
    }
}

@Service
public class HwPrinterImpl implements PrinterService {
    public void print(String msg) {
        System.out.println("Hai wei printer service for you :");
        System.out.println(msg);
    }
}
```

打印服务的入口：

```java
/**
 * 打印服务demo
 *
 * @author
 * @create 2019-06-23 16:38
 **/
public class PrinterDemo {

    private static ApplicationContext applicationContext ;
    private static PrinterService printerService;

    public static void main(String[] args) {
        applicationContext = new ClassPathXmlApplicationContext(new String[] {"config/spring/spring-core.xml"});

        printerService = applicationContext.getBean("hpPrinterImpl",PrinterService.class);
        printerService.print("这是个Demo");

        printerService = applicationContext.getBean("hwPrinterImpl",PrinterService.class);
        printerService.print("这是个Demo");
    }
}
```

spring-core.xml配置文件如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package="com.oscoder.study.spring"/>
    <context:annotation-config/>

</beans>
```

运行结果

```out
Hp printer service for you :
这是个Demo
Hai wei printer service for you :
这是个Demo
```

下面我们就通过这个demo来看看spring的加载过程

我们首先在PrinterDemo.main方法里new了一个ClassPathXmlApplicationContext，它会引导我们进入的Spring初始化的核心方法AbstractApplicationContext.refresh中

```java
public void refresh() throws BeansException, IllegalStateException {
        synchronized(this.startupShutdownMonitor) {
            this.prepareRefresh();
            ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
            this.prepareBeanFactory(beanFactory);

            try {
                this.postProcessBeanFactory(beanFactory);
                this.invokeBeanFactoryPostProcessors(beanFactory);
                this.registerBeanPostProcessors(beanFactory);
                this.initMessageSource();
                this.initApplicationEventMulticaster();
                this.onRefresh();
                this.registerListeners();
                this.finishBeanFactoryInitialization(beanFactory);
                this.finishRefresh();
            } catch (BeansException var9) {
                if (this.logger.isWarnEnabled()) {
                    this.logger.warn("Exception encountered during context initialization - cancelling refresh attempt: " + var9);
                }

                this.destroyBeans();
                this.cancelRefresh(var9);
                throw var9;
            } finally {
                this.resetCommonCaches();
            }

        }
    }
```
