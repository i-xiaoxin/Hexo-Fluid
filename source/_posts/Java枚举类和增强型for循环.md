---
title: Java枚举类和增强型for循环
date: 2023-1-1 21:09:02
tags: 
- Enum
- for each
index_img: 
banner_img: /img/article2.webp
categories:
- Java
comment: waline
---

# Java 枚举(enum)

Java 枚举类使用 enum 关键字来定义，各个常量使用逗号 **,** 来分割。

### 迭代枚举元素

使用 for  each语句来迭代枚举元素(其中for语句中的：为语法糖)；[语法糖编译后转换为for循环语句](https://segmentfault.com/q/1010000010114448)

> for（Color color ：Color.values()）{.......}

## 带一个参数的枚举

```java
public enum TypeEnum {

    FIREWALL("firewall"),  
    SECRET("secretMac"),  
    BALANCE("f5");  

    private String typeName;  

    TypeEnum(String typeName) {  
        this.typeName = typeName;  
    }  

    /**
     * 根据类型的名称，返回类型的枚举实例。
     *
     * @param typeName 类型名称
     */  
    public static TypeEnum fromTypeName(String typeName) {  
        for (TypeEnum type : TypeEnum.values()) {  
            if (type.getTypeName().equals(typeName)) {  
                return type;  
            }  
        }  
        return null;  
    }  

    public String getTypeName() {  
        return this.typeName;  
    }    
}

```

## 适用场景

### 1️⃣问题

在项目中有无接触过接口返回的字段是数据库表结构中没有的需求呢？**比如：我们需要按照什么什么样字段进行分类展示，但是在实际数据存储时是不需要这个数据的，只是为了前端展示效果。**

前端如果去分类搞定都是 **“写死”** 状态，如果后期项目改动维护的话，代码变动比较大；因此放后端处理合适

### 2⃣️解决

枚举类即能解决这类问题

```java
public enum TransportationType {
    //加油类型
    TYPE_GAS(0,"加油"),
    //充电类型
    TYPE_POWER(1,"充电"),;
    
    private final Integer code;
    private final String name;

    //构建构造器
    TransportationType(Integer code, String name) {
        this.code = code;
        this.name = name;
    }

    //给出code及name的getter方法
    public Integer getCode() {
        return code;
    }

    public String getName() {
        return name;
    }
  
  //省略相关类结构，只给出遍历方法
	//使用增强型for循环遍历
	for(TransportationType transportationType : TransportationType.values()){
	//使用枚举类方法values()实现遍历过程
	//相关的逻辑代码实现
	Demo demo = new Demo();
	demo.setTransportationTypeCode(transportationType.getCode());
	demo.setTransportationTypeName(transportationType.getName());
	...
}
}
```

## switch 语句中的 enum

```java
enum Signal { GREEN, YELLOW, RED, }

public class TrafficLight {
    Signal color = Signal.RED;
    public void change() {
        switch(color) {
           // Note you don't have to say Signal.RED
            // in the case statement:
            case RED: color = Signal.GREEN;
                break;
            case GREEN: color = Signal.YELLOW;
                break;
            case YELLOW: color = Signal.RED;
                break;
        }
    }
    @Override
    public String toString() {
        return "The traffic light is " + color;
    }
    public static void main(String[] args) {
        TrafficLight t = new TrafficLight();
        for(int i = 0; i < 7; i++) {
            System.out.println(t);
            t.change();
        }
    }
}
```

输出为：

```java
The traffic light is RED
The traffic light is GREEN
The traffic light is YELLOW
The traffic light is RED
The traffic light is GREEN
The traffic light is YELLOW
The traffic light is RED
```

编译器并没有抱怨 switch 中没有 default 语句，但这并不是因为每一个 Signal 都有对应的 case 语句。如果你注释掉其中的某个 case 语句，编译器同样不会抱怨什么。这意味着，你必须确保自己覆盖了所有的分支。但是，如果在 case 语句中调用 return，那么编译器就会抱怨缺少 default 语句了。这与是否覆盖了 enum 的所有实例无关。

## enum中的方法

enum 定义的枚举类默认继承了 java.lang.Enum 类，并实现了 java.lang.Serializable 和 java.lang.Comparable 两个接口。

values(), ordinal() 和 valueOf() 方法位于 java.lang.Enum 类中：

- values() 返回枚举类中所有的值。以数组方式返回枚举类型中的成员
- ordinal()方法可以找到每个枚举常量的索引，就像数组索引一样。
- valueOf()方法返回指定字符串值的枚举常量，如果不存在则抛出IllegalArgumentException异常
- compareTo()方法用于比较枚举类型中两个成员的索引值

### 枚举类成员

枚举跟普通类一样可以用自己的变量、方法和构造函数，构造函数只能使用 private 访问修饰符，所以外部无法调用。

枚举既可以包含具体方法，也可以包含抽象方法。 如果枚举类具有抽象方法，则枚举类的每个实例都必须实现它。

# Java Enum枚举类

```java
public enum AlarmGrade {

    ATTENTION("attention", "提示"),
    WARNING("warning","警告"),
    SERIOUS("serious", "严重"),
    FAULT("fault", "故障"),
    UNKNOWN("unknown", "未知");

    private String key;

    private String name;

    AlarmGrade(String key, String name) {
        this.key = key;
        this.name = name;
    }

    public String getKey() {
        return key;
    }

    public String getName() {
        return name;
    }

    /**
     * 根据Key得到枚举的Value
     * 普通for循环遍历，比较判断
     */
    public static AlarmGrade getEnumType(String key) {
        AlarmGrade[] alarmGrades = AlarmGrade.values();
        for (int i = 0; i < alarmGrades.length; i++) {
            if (alarmGrades[i].getKey().equals(key)) {
                return alarmGrades[i];
            }
        }
        return AlarmGrade.UNKNOWN;
    }

    /**
     * 根据Key得到枚举的Value
     * 增强for循环遍历，比较判断
     */
    public static AlarmGrade getEnumType1(String key) {
        AlarmGrade[] alarmGrades = AlarmGrade.values();
        for (AlarmGrade alarmGrade : alarmGrades) {
            if (alarmGrade.getKey().equals(key)) {
                return alarmGrade;
            }
        }
        return AlarmGrade.UNKNOWN;
    }

    /**
     * 根据Key得到枚举的Value
     * Lambda表达式，比较判断（JDK 1.8）
     *
     * @param key
     * @return
     */
    public static AlarmGrade getEnumType2(String key) {
        AlarmGrade[] alarmGrades = AlarmGrade.values();
        AlarmGrade result = Arrays.asList(alarmGrades).stream()
                .filter(alarmGrade -> alarmGrade.getKey().equals(key))
                .findFirst().orElse(AlarmGrade.UNKNOWN);

        return result;
    }

    /**
     * 根据Key得到枚举的Value
     * Lambda表达式，比较判断（JDK 1.8）
     *
     * @param key
     * @return
     */
    public static AlarmGrade getEnumType3(String key) {
        return Arrays.asList(AlarmGrade.values()).stream()
                .filter(alarmGrade -> alarmGrade.getKey().equals(key))
                .findFirst().orElse(AlarmGrade.UNKNOWN);
    }

    public static void main(String[] args) {
        String grade = "attention";
        System.out.println("第一种方式：普通for循环遍历，比较判断 \n" + grade + ": " + AlarmGrade.getEnumType(grade).getName());
        System.out.println("\n第二种方式：增强for循环遍历，比较判断 \n" + grade + ": " + AlarmGrade.getEnumType1(grade).getName());
        System.out.println("\n第三种方式：Lambda表达式，比较判断 \n" + grade + ": " + AlarmGrade.getEnumType2(grade).getName());
        System.out.println("\n第四种方式：Lambda表达式，比较判断 \n" + grade + ": " + AlarmGrade.getEnumType3(grade).getName());
    }
}
```

运行结果如下：

```sh
第一种方式：普通for循环遍历，比较判断 
attention: 提示

第二种方式：增强for循环遍历，比较判断 
attention: 提示

第三种方式：Lambda表达式，比较判断 
attention: 提示

第四种方式：Lambda表达式，比较判断 
attention: 提示
```



参考：

[Java 枚举(enum) 详解7种常见的用法](https://blog.csdn.net/qq_27093465/article/details/52180865)

[Java 枚举](https://www.runoob.com/java/java-enum.html)

<div>
<hr>
<script src="https://unpkg.com/@waline/client@v2/dist/waline.js"></script> 
<link
  rel="stylesheet"
  href="https://unpkg.com/@waline/client@v2/dist/waline.css"
/>
<div id="waline"></div>
  <script>
    Waline({
      el: '#waline',
      serverURL: 'https://vercel-project-4d7haxk1c-i-xiaoxin.vercel.app',
    });
  </script>