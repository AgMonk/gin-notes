# 变量

## 声明

标准的变量声明格式为：

```kotlin
var 变量名:类型名 = 变量值
```

`kotlin`具有类型推断机制，可以根据变量值的类型自动推断变量类型，因此多数时候可以省略类型名，简写为：

```kotlin
var 变量名 = 变量值
```

对于只可以赋值一次的变量(final变量)，将`var`关键字改为`val`关键字即可。

当变量值可能有多种类型时，变量的类型将被推断为`Any`类型

## 内置数据类型

| 类型    | 描述       | 示例 |
| ------- | ---------- | ---- |
| String  | 字符串     |      |
| Char    | 单字符     |      |
| Boolean | 布尔值     |      |
| Int     | 整数       |      |
| Double  | 双精度小数 |      |
| List    | 可重复集合 |      |
| Set     | 无重复集合 |      |
| Map     | 键值对集合 |      |

## 常量

全称为`编译时常量`，必须定义在方法外，使用关键字`const val`定义，等效于Java的`public static final`

# 流程控制

## if/else表达式

`kotlin`中`if/else`为`表达式`，意味着它有返回值，将返回每个语句块最后执行的表达式的值，不需要`return`关键字；因此可以实现类似三目运算符的写法。

## range表达式

假设AB均为整数，且A<B

常规格式： `in A..B`，等效于数学上的区间表示 `[A,B]`，注意，与Java常规（左闭右开）不同，这里右边也是闭区间。

`until`格式：`in A until B`，等效于区间表示`[A,B)`，这里与Java习惯相同。

## when表达式

when表达式类似`switch/case`语句，但也为`表达式`，有返回值，相当于`switch/case`语句写在一个方法中，每个`case`块return一个值

when表达式适用类型推断机制，当每个块返回类型相同时，表达式类型推断为该类型，当不同时推断为`Any`类型

# 字符串

## 模板

使用双引号的字符串支持模板拼接，范例：

```kotlin
val name = "张三"
val age = "24"
println("我叫${name}, 今年$age 岁, 是学生")
```

# 函数/方法

## 具名方法的定义

标准格式为：

```kotlin
可见性修饰符 fun 方法名(参数列表): 返回类型{
    方法体
}
```

其中

1. 可见性修饰符可省略，默认为`public`
2. 参数列表格式为`参数名:参数类型`，多个以逗号隔开，可以在后面加`=变量值`为参数设置缺省值，这里的参数类型不可省略
3. 返回类型可省略，以适用类型推断机制，当方法没有返回值时返回类型将推断为`Unit`类型

因此常见格式为：

```kotlin
fun 方法名(参数列表){
    方法体
}
```

## 参数的具名调用

当方法的参数较多时，可以指定参数名进行传参

```kotlin
fun test(age: Int, tall: Int = 3) {
    
}
test(age = 2)	
```

## 反引号方法调用

常见与外部包，当方法名与`kotlin`的关键字撞车时，可以在方法名前后加反引号 **`** 来进行调用

## 匿名函数/lambda/闭包

这三个基本是指同一个东西

### 简单使用

等效于Java的函数式接口（可以有自己的引用，操作类似对象），通常作为其他方法的参数和返回值存在。以下示例将会统计字符串中出现的a字符的次数：

```kotlin
println("aaabbbccc".count( { c -> c == 'a' }))
```

其中`count`括号中的即为匿名函数

匿名函数将会隐式返回执行的最后一个表达式的结果

当匿名函数为方法的最后一个参数时，函数体可以写到圆括号外的后面，当没有其他参数时，圆括号可以省略

```kotlin
println("aaabbbccc".count { c -> c == 'a' })
```

### 标准定义格式

```kotlin
val 变量名: (参数类型列表)->返回类型 = { 参数名列表
    方法体
}
```

示例：

```kotlin
val test: (String,Int) -> String = { s,i -> "$s * $i" }
println(test("aa",123))
```

其中：

- 不支持参数的具名调用
- 参数列表与参数类型列表一一对应

### 简洁格式

当匿名函数只有一个参数时，可以省略参数列表，编译器将自动配置`it`为该参数名

```kotlin
val test: (String) -> String = { "$it -> **" }
println(test("aa"))    
```

当希望匿名函数的返回值使用类型推断时，需要把参数类型与参数名写在一起：

```kotlin
val test = { s: String, i: Int -> "$s * $i" }
println(test("aa",123))
```

### 在方法参数中使用

```kotlin
fun test(s: String, method: (String, Int) -> String): String {
    return "$s -> ${method("aaa", 123)}"
}
println(test("abc") { s, i -> "$s ==> $i" })
```

### 具名方法内联

在具名方法的`fun`关键字前增加`inline`关键字，会在编译时内联该函数，提高执行性能；**递归函数无法内联**

## 引用具名函数

当希望将一个具名函数传递给某个方法时，在方法前加双冒号`::`进行引用，示例：

```kotlin
fun test(s: String, method: (String, Int) -> String): String {
    return "$s -> ${method("aaa", 123)}"
}
fun test1(a:String,b:Int): String {
    return "$a -> $b"
}
println(test("abc",::test1))    
```



