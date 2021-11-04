# 1 写在前面

最近FastJson爆出严重漏洞，所以团队决定在新的项目中抛弃FastJson。采用Gson作为json转化工具。所以这里准备先研究下Gson。

# 2 Gson概览

Gson是谷歌出品的一款json转化工具，具有依赖少、线程安全、效率较高等特性。


## 2.1 Gson核心类图浏览

![image](/images/json/gson_class.png)

Gson的设计总体是围绕着适配器模式+工程模式展开的，Gson中定义了很多的适配器，这些适配器大致可以被分成3类

1、基础适配器
例如String适配器、Integer适配器等，这些适配器直接完成了Json和对象的转换；

2、ReflectiveType适配器
反射类型的适配器，重复利用反射相关特性，对复杂对象进行反射解析，解析成基础适配器；

> 特别说明：源码中没有ReflectiveTypeAdapter类，ReflectiveTypeAdapterFactory中有内部类Adapter，为了便于说明这里画了ReflectiveTypeAdapter。

3、用户自定义适配器
Gson支持用户自定义适配器；

## 2.2 序列化

Gson序列化常用入口是 toJson,这里我们就简单跟踪下，Json序列化的过程。
简单来说，Gson的序列化过程就是一个迭代寻找适配器的过程。

**流程图：**


**代码：**
```java
public void toJson(Object src, Type typeOfSrc, JsonWriter writer){
    //省略非核心代码  
    TypeAdapter<?> adapter = getAdapter(TypeToken.get(typeOfSrc));
    try {
      ((TypeAdapter<Object>) adapter).write(writer, src);
    } catch (IOException e) {
    } catch (AssertionError e) {
    } finally {
    }
  }

public <T> TypeAdapter<T> getAdapter(TypeToken<T> type) {
    //省略非核心代码  
    try {
      FutureTypeAdapter<T> call = new FutureTypeAdapter<T>();
      threadCalls.put(type, call);
      //变量Gson初始化时注册的所有适配器工厂，寻找对应适配器
      for (TypeAdapterFactory factory : factories) {
        TypeAdapter<T> candidate = factory.create(this, type);
        //type不匹配时会返回null
        if (candidate != null) {
          call.setDelegate(candidate);
          typeTokenCache.put(type, candidate);
          return candidate;
        }
      }
      throw new IllegalArgumentException("GSON (" + GsonBuildConfig.VERSION + ") cannot handle " + type);
    } finally {
      threadCalls.remove(type);
      if (requiresThreadLocalCleanup) {
        calls.remove();
      }
    }
  }

    //这里借用ReflectiveTypeAdapterFactory说明Factory的运行
    public <T> TypeAdapter<T> create(Gson gson, final TypeToken<T> type) {
    Class<? super T> raw = type.getRawType();
    ObjectConstructor<T> constructor = constructorConstructor.get(type);
    //核心是getBoundFields
    return new Adapter<T>(constructor, getBoundFields(gson, type, raw));
  }

//getBoundFields会充分利用反射，获取对象的属性字段
private Map<String, BoundField> getBoundFields(Gson context, TypeToken<?> type, Class<?> raw) {
    Map<String, BoundField> result = new LinkedHashMap<String, BoundField>();
    Type declaredType = type.getType();
    while (raw != Object.class) {
      //利用反射获取属性字段
      Field[] fields = raw.getDeclaredFields();
      for (Field field : fields) {
        Type fieldType = $Gson$Types.resolve(type.getType(), raw, field.getGenericType());
        List<String> fieldNames = getFieldNames(field);
        BoundField previous = null;
        for (int i = 0, size = fieldNames.size(); i < size; ++i) {
          String name = fieldNames.get(i);
          //调用createBoundField创建绑定字段
          BoundField boundField = createBoundField(context, field, name,
              TypeToken.get(fieldType), serialize, deserialize);
          BoundField replaced = result.put(name, boundField);
        }
      }
      type = TypeToken.get($Gson$Types.resolve(type.getType(), raw, raw.getGenericSuperclass()));
      raw = type.getRawType();
    }
    return result;
  }
private ReflectiveTypeAdapterFactory.BoundField createBoundField(
      final Gson context, final Field field, final String name,
      final TypeToken<?> fieldType, boolean serialize, boolean deserialize) {
    //对字段调用Gson中的getAdapter计算对应的适配器。
    if (mapped == null) mapped = gson.getAdapter(fieldType);
    final TypeAdapter<?> typeAdapter = mapped;
    //利用适配器创建对应的绑定字段
    return new ReflectiveTypeAdapterFactory.BoundField;
  }
```

## 2.3 反序列化



反序列化是序列化的另一面，Gson反序列化常用入口是fromJson。

```java
public <T> T fromJson(JsonReader reader, Type typeOfT) throws JsonIOException, JsonSyntaxException {
    try {
      //根据指定的Type，查询适配器
      TypeToken<T> typeToken = (TypeToken<T>) TypeToken.get(typeOfT);
      TypeAdapter<T> typeAdapter = getAdapter(typeToken);
      T object = typeAdapter.read(reader);
      return object;
    } catch (EOFException e) {
    } catch (IllegalStateException e) {
    } catch (IOException e) {
    } catch (AssertionError e) {
    } finally {
    }
  }
```

# 3 常见坑

**1）默认Gson序列化时，会丢弃null的参数**

```java

@Getter
@Setter
@ToString
public class DemoDTO {
    private Integer notNUllInteger;
    private Integer nullInteger;
}

/**
 * demo
 */
public class GsonDemo {
    public static void main(String[] args) {
        DemoDTO demoDTO = build();
        System.out.println("dto 2 string:" + demoDTO.toString());
        //默认的Gson
        Gson gson1 = new Gson();
        String json = gson1.toJson(demoDTO);
        System.out.println("默认场景下：dto 2 json" + json);
        //serializeNulls
        Gson gson2 = new GsonBuilder().serializeNulls().create();
        System.out.println("serializeNulls解决方案后：json 2 dto" +  gson2.toJson(demoDTO));
    }

    private static DemoDTO build(){
        DemoDTO demoDTO = new DemoDTO();
        demoDTO.setNotNUllInteger(1);
        return demoDTO;
    }
}
```

运行结果如下：

``` out
dto 2 string:DemoDTO(notNUllInteger=1, nullInteger=null)
默认场景下：dto 2 json{"notNUllInteger":1}
serializeNulls解决方案后：json 2 dto{"notNUllInteger":1,"nullInteger":null}
```

可以看到，在默认场景下，gson序列化后的sting中是没有“nullInteger”属性的。

**解决方案：** 指定Gson的serializeNulls属性即：new GsonBuilder().serializeNulls().create()

**2）通配符可以正常序列化，但是反序列化时会出错**

```java
public class GsonDemo {
    public static void main(String[] args) {
        List<Number> numberList = Lists.newArrayList();
        numberList.add(new Integer(1));
        numberList.add(new Double(1.2));
        numberList.add(new BigDecimal("1.23"));
        System.out.println("list 2 string:" + numberList.toString());

        Gson gson = new Gson();
        String list2Json = gson.toJson(numberList);
        System.out.println("list 2 json:" + list2Json);

        List jso2List = gson.fromJson(list2Json, List.class);
        System.out.println("json 2 list:" + jso2List.toString());
    }
}
```

运行结果如下：

```out
list 2 string:[1, 1.2, 1.23]
list 2 json:[1,1.2,1.23]
json 2 list:[1.0, 1.2, 1.23]
```

可以看到，在我们序列化的字符串中三个数字分别是：Integer、Double、BigDecimal。而反序列化后都变成了Double类型。

原因是：Gson在序列化时，Gson可以通过obj.getClass()获取字段的类型信息，用于序列化。而反序列化时，由于类型丢失，所以反序列化会出错。

**解决方案：**

1、如果list中是固定类型，可以通过TypeToken指定解析类型来反序列化，即gson.fromJson(integerList2Json, new TypeToken<List<Integer>>(){}.getType()) ；

2、通过添加Adapt来对其进行特殊处理；

**3）Gson在解析日期类型时，会依赖运行机器上配置的Local**

不同的local配置，可能会稍有差异