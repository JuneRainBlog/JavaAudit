deserialize 

编写POC技巧：

如果一个类存在多个构造器，但我们只想调用其中一个。

```
Class clazz = Class.forName("org.apache.commons.collections.map.LazyMap"); //创建LazyMap的Class
Constructor[] constructors = clazz.getDeclaredConstructors();//调用LazyMap的空参构造器
System.out.println(constructors);
Constructor constructor = constructors[0]; 调用第一个构造器
constructor.setAccessible(true);
Map map = (Map)constructor.newInstance(innermap,chain);  
```

##

PS:涉及内容很多，后续再进行更新
