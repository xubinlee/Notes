# 反射

## Class类的使用

Java是面向对象的语言，在Java里万事万物皆对象，类也是对象，类是java.lang.Class的对象。（有两样东西不属于对象，一是基本数据类型；二是静态类型，属于类）。

```java
Package com.ex;
Animal a = new Cat();
Cat cat = new Cat();
```

任何一个类都是Class的实例对象，这个实例对象有三种表示方式：

1. 任何一个类都有一个隐含的静态成员变量；

   ```java
   Class c1 = Cat.class;
   ```

2. 已知该类的对象通过getClass方法

   ```java
   Class c2 = cat.getClass();
   ```

3. 类的动态加载

   ```
Class c2 = Class.forName(“com.ex.Cat”)
   ```
   
   c1、c2、c3是类类型，可以通过类类型创建该类的对象实例，前提要求是要用无参数的构造方法：
   
   ```
   Cat cat = (Cat)c1.newInstance();
   ```
   
   

## 动态加载类

Class.forName(“类的全称”)，不仅表示了类的类类型，还代表了动态加载类；功能性的类尽量使用动态加载

```java
// 动态加载类，在运行时加载，编译时不管
Class c = Class.forName(“类的全称”); 

// 通过类类型，创建该类对象
Animal a = (Animal)c.newInstance();
```

## 获取方法信息

```java
Cat cat = new Cat();
cat.getName(); //获取类的名称
cat.getSimpleName(); //获取不包含包名的类的名称

Class c = cat.getClass();

/**
\* Method类，方法对象
\* 一个成员方法就是一个Method对象
\* getMethod()方法获取所有public的函数，包括父类继承而来的
\* getDeclaredMethods()获取的是所有该类自己声明的方法，不问访问权限
*/
Method[] ms = c.getMethods(); // c.getDeclaredMethods();
// 得到方法的返回值类型的类类型
Class returnType = ms[i].getReturnType();
// 得到返回值类型名称
returnType.getName();
// 得到方法的名称
ms[i].getName();
// 得到参数列表的类型的类类型
Class[] paramTypes = ms[i].getParameterTypes();
// 得到参数类型名
paramTypes[i].getName();
```

## 获取成员变量构造函数信息

```java
Cat cat = new Cat();
Class c = cat.getClass();

/**
\* 成员变量也是对象
\* java.lang.reflect.Field
\* Field类封装了关于成员变量的操作
\* getFields()方法获取所有public的成员变量的信息
\* getDeclaredFields()获取的是所有该类自己声明的成员变量的信息
*/
Field[] fs = c.getFields();
// 得到成员变量的类型的类类型
Class fieldType = field.getType();
String typeName = fieldType.getName();
// 得到成员变量的名称
String fieldname = field.getName();
System.out.println(typeName+” ”+fieldName);
```

## 方法反射的基本操作

```java
Cat cat = new Cat();
Class c = cat.getClass();
Method m = c.getMethod(“方法名”, int.class, int.class);
// cat.方法名(10,20)
m.invoke(cat, new Object[]{10,20})
```

通过反射了解集合类型的本质：Java中集合的泛型，是防止错误输入的，只在编译阶段有效，绕过编译就无效了。编译之后集合的泛型是去泛型化的，反射的操作都是编译之后的，可以通过方法的放射绕过编译。

```java
ArrayList<String> list = new ArrayList<String>();
List.add(“hello”);
Class c = list.getClass();
Method m = c.getMethod(“add”, Object.class)
m.Invoke(list, 10);//绕过编译，不过不能再遍历，会报错
```

