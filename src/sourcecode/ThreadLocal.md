# ThreadLocal是什么
是一种数据结构，其实现了多线程环境下，线程变量信息的隔离，即在使用ThreadLocal维护变量时，其为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本

# ThreadLocal原理
ThreadLocal的核心其内部的ThreadLocalMap,其实现了

# ThreadLocal关键场景源码分析

## set

``` java
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }

    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }

    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```

## get

## remove

# ThreadLocal使用注意点
1. 