---
title: Java反射
date: 2021-04-28 21:19:09
tags: Java
---
- 反射是为了能够动态的加载一个类，动态的调用一个方法，动态的访问一个属性等动态要求而设计的。它的出发点就在于JVM会为每个类创建一个java.lang.Class类的实例，通过该对象可以获取这个类的信息，然后通过使用java.lang.reflect包下得API以达到各种动态需求。
- 反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性，这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。
<!--more-->
# Class获取
其主要通过一下三种方式获取：
```
Car car=new Car();
//第一种
Class clazz=car.getClass();//这种方法不适合基本类型如int，float等
//第二种
Class clazz=Car.class;
Class clazz=int.class;
//第三种
Class clazz=Class.forName("com.test.Car");//使用类的全限定名称，包名加类名
```

# 获取修饰符
```
Car.class.getModifiers();//其返回的是int数值
Modifier.toString(Car.class.getModfiers());//int数值转化成public，abstract等
```

# Field
获取类的属性，即变量
```
Car.class.getDeclaredFields();//获取所有属性，不包括父类继承下来的属性
Car.class.getFields();//获取所有public属性，包括父类继承下来的属性
```

# Method
获取方式如上，其方法执行：
```
//Method
public Object invoke(Object obj,Object... args){}
```
- 参数obj是Method所依附的Class对应的类的实例。如果这个方法是静态方法，那么obj为null。后面的args是参数
- invoke()返回对象是Object，需要强制转换
- 会把异常包装到InvocationTargetException里面去，通过InvocationTargetException.getCause()可以获取真正的异常

# Constructor
- Class.newInstance()只能调用无参构造函数，而Constructor.newInstance()则可以调用任何构造函数
- Class.newInstance()直接抛出异常，而Constructor.newInstance()会把异常包装到InvocationTargetException里面去，和Method一样
- Class.newInstance()要求构造方法能访问，而Constructor.newInstance()能访问private修饰的构造器

# 提高反射效率
- 善用API：比如，尽量不要getMethods()后再遍历筛选，而直接用getMethod(methodName)来根据方法名获取方法。
- 适当使用缓存：比如，需要多次动态创建一个类的实例的时候，有缓存的写法会比没有缓存要快很多:
```
// 1. 没有缓存
void createInstance(String className){
    return Class.forName(className).newInstance();
}

// 2. 缓存forName的结果
void createInstance(String className){
    cachedClass = cache.get(className);
    if (cachedClass == null){
        cachedClass = Class.forName(className);
        cache.set(className, cachedClass);
    }
    return cachedClass.newInstance();
}
```

# 通过反射获得泛型的实际类型参数
把泛型变量当成方法的参数，利用Method类的getGenericParameterTypes方法来获取泛型的实际类型参数
```
public class GenericTest {

    public static void main(String[] args) throws Exception {
        getParamType();
    }
    
     /*利用反射获取方法参数的实际参数类型*/
    public static void getParamType() throws NoSuchMethodException{
        Method method = GenericTest.class.getMethod("applyMap",Map.class);
        //获取方法的泛型参数的类型
        Type[] types = method.getGenericParameterTypes();
        System.out.println(types[0]);
        //参数化的类型
        ParameterizedType pType  = (ParameterizedType)types[0];
        //原始类型
        System.out.println(pType.getRawType());
        //实际类型参数
        System.out.println(pType.getActualTypeArguments()[0]);
        System.out.println(pType.getActualTypeArguments()[1]);
    }

    /*供测试参数类型的方法*/
    public static void applyMap(Map<Integer,String> map){

    }

}
//结果
java.util.Map<java.lang.Integer, java.lang.String>
interface java.util.Map
class java.lang.Integer
class java.lang.String
```
## getGenericParameterTypes 与 getParameterTypes区别
- getGenericParameterTypes 与 getParameterTypes 都是获取构成函数的参数类型
- 前者返回的是Type类型，后者返回的是Class类型，由于Type顶级接口，Class也实现了该接口，因此Class类是Type的子类，Type 表示的全部类型而每个Class对象表示一个具体类型的实例，如String.class仅代表String类型。由此看来Type与 Class 表示类型几乎是相同的，只不过Type表示的范围比Class要广得多而已。当然Type还有其他子类，如：
    - TypeVariable：表示类型参数，可以有上界，比如：T extends Number
    - ParameterizedType：表示参数化的类型，有原始类型和具体的类型参数，比如：List
    - WildcardType：表示通配符类型，比如：?, ? extends Number, ? super Integer