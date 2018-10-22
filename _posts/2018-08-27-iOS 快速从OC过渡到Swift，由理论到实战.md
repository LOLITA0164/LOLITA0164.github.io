---
layout:     post
title:      iOS 快速从OC过渡到Swift
subtitle:   由理论到实战，手把手的教你
date:       2018-08-27
author:     LOLITA0164
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - iOS
    - swift
    - 应用开发
---

## 引言

本文旨在帮助开发者快速从OC开发过渡到Swift开发，挑选了一些比较浅显的但是比较常用的Swift语法特性，在介绍的过程中，通常会拿OC中的语言特性作比较，让大家更好的注意到Swift的不同。
另外需要说明的是，笔者也仅仅是刚刚接触Swift不久，如果有说的不对的地方，还望指正，这里贴出[Swift中文翻译地址](http://www.swift51.com/swift4.0/index.html)，方便大家可以深入了解Swift。

## Swift简介

Swift是一门开发iOS、macOC、watchOS和tvOS应用的新语言，在初期，为了让OC开发者快速的过渡到Swift开发，Swift继承了很多C、OC的语法特性，但是随着Swift的不断完善，已经慢慢自称一体，不仅保留了OC的很多语言特性，还借鉴了很多语言，如C#、Java、Python等。
Swift包含了C和OC上所有的基础数据类型：Int、Double、Float、Bool、String，集合类型：Array、Set和Dictionary，除此之外，Swift增加了OC中没有的高阶数据类型如元组（Tuple），元组方便我们接收或传递一组数据，或返回多个值而不必使用结构体、类等。
另外，Swift新增了可选（Optional）类型，可选表示有值或者没有值，需要注意的是，Swift中，不同类型的数据类型是不能够相互操作的，如Int？和Int属于不同类型，又如Int和Float。

## Swift基础

**1、常量let和变量var**

常量一旦设定就不可改变，变量则值可变。常量使用`let`来声明，变量使用`var`表示。

```
let PI = 3.1415
var Str = "string"
```
Swift可以根据赋值推断出类型，你也可以使用类型标注（type annotation）来指定类型

```
let PI: Float = 3.1415
var Str: String = "string"
```

一般来说声明常量或变量形式如下：

```
let/var name: type = value
```

**2、输出print**
你可以使用`print`函数输出常量和变量，输完换行。Swift取消了旧版的`println`输出方法，另外，如果你想使用OC中的NSLog也是可以的。`print(, separator: , terminator: )`中，将terminator参数设为空字符`""`则可以不进行换行。
Swift使用[字符串插值](http://www.swift51.com/swift4.0/chapter2/03_Strings_and_Characters.html#string_interpolation)（string interpolation）的方式将常量或变量当作占位符加入到长字符串中，我们可以借此拼接长字符。

```
let value1 = "123"
var value2 = 345
print("value1 = \(value1) , value2 = \(value2)")
// 输出： value1 = 123 , value2 = 345
```
**3、数据类型（布尔值、数组、字典、元组、可选类型）**

a. 布尔值

和OC不同，Swift中的布尔值使用`true`和`false`表示真、假

```
let boolValue = true
if boolValue {
    print("value is true")
}
```
这里需要说明的是，Swift不能想OC中那样，数值为0即表示布尔值`NO`，非0即为`YES`,因此下面OC代码可以通过

```
float value = 0.001;
if (value) {
	NSLog(@"value:%f",value);
}
// 输出： value:0.001000
```
而在Swift中会编译报错

```
let value = 0.001
if boolValue {
    print("value is true")
}
```
因此在判断语句中必须采用布尔值

```
let value1 = 1.1
let value2 = 2.0
if value1 > value2 {
    print("value1:\(value1) 大于 value2:\(value2)")
}else{
    print("value1:\(value1) 小于 value2:\(value2)")
}
// 输出： value1:1.1 小于 value2:2.0
```

b. 数组（array）

数组是一种泛型应用，关于泛型，后面会说到。
数组有多种创建方式

```
var list1:Array = [1,2,3]  // 标准创建，元素类型由系统推断
var list2:Array<Int> = [1,2,3]  // 指定类型，一种泛型应用
var list3 = [1,2,3]     // 由系统推断类型
var list4:[Int] = [1,2,3]   // 简写Array，并指定元素
var list5 = [Int]([1,2,3])  // [Int]()则创建空数组
var list6 = Array([1,2,3]) // 等同于Array.init([1,2,3])
var list7 = Array.init(repeating: 1, count: 3)  // 重复3次元素1，数组类型由元素推断
```

需要说明的是：Swift中并没有可变不可变两种数组类型之分，不可变类型可以通过`let`来声明，`var`声明的数组都是可变的，类似的，字典类型也是如此

数组的一些常用操作

```
var a = [Float]()
a.append(1.1) // 添加元素
a += [2,5]	// 拼接其他数组
a[1...2] = [11,22] // 修改元素， '...'表示范围range
a.insert(33, at: a.count) // 插入元素，此处插在了数组的末尾
a.remove(at: a.count-1) // 移除指定位置的，此处相当于 a.removeLast()
a.removeFirst() // 移除第一个
// 遍历数组
for value in a{
    print("index=\(a.index(of: value)!),element=\(value)")
}
for (index,value) in a.enumerated(){	// (,)表示元组类型
    print("index=\(index),element=\(value)")
}
```

c. 字典（Dictionary）

字典的创建和数组类似，可以由系统推断出元素类型，也可以指定元素类型。和OC不同的是，Swift创建不使用'@{}'便捷创建，而是使用和数组一样的'[]'形式，和数组不同的是，'[]'中需要':'分隔，左边是key的类型，右边是value类型，下面只演示其中一种创建方式

```
// [Int:String]为简写，其中Int为key的类型，String为value类型
var d:[Int:String] = [200:"success",404:"not found"]
```
字典的一些常用操作

```
d[200] // 取值
d[500] = "internal server error" // 添加
// 遍历字典
for key in d.keys{  // 遍历key
    print("key=\(key)")
}
for value in d.values{  // 遍历value
    print("value=\(value)")
}
for (key,value) in d{   // 遍历元组
    print("key=\(key),value=\(value)")
}
```
d. 元组

元组（tuples）把多个值组合成一个复合值。元组内的值可以是任意类型，这一点和数组不同，它不要求元素相同类型，这里的元组和Python中的元组功能类似。
Swift的元组使用圆括号'()'创建，不能省略，这点和Python不同

```
let size = (100,200)
print(size.0,size.1)
// 输出 100 200
// 也可以指定名称
let httpError = (code:404,reason:"Not Found")
print(httpError.code,httpError.reason)  // httpError.code 和 httpError.0 是一样的
// 输出 404 Not Found
```
应用举例：实现交换

```
var (v1,v2) = (10,20)
print("v1:\(v1),v2:\(v2)")
// 输出 v1:10,v2:20
(v1,v2) = (v2,v1)
print("v1:\(v1),v2:\(v2)")
// 输出 v1:20,v2:10
```

应用举例：函数多参数返回，在数组和字典中的遍历过程中已经有体现

需要注意的是：元组在临时组织值的时候很有用，但是并不适合创建复杂的数据结构。如果你的数据结构并不是临时使用，请使用类或者结构体而不是元组。

e. 可选类型

可选类型：有值，为x 或者 没有值。
我们先来看下系统方法产出的可选类型。Int类型有一种构造器，将一个String值转换为一个Int值，但是，并不是所有的字符串都可以转换为一个整数。如"123"可以被转换为数字123，而字符串"hello, world"则不可以。

```
let possibleNumber = "123"
let convertedNumber = Int(possibleNumber)
// 当你敲convertedNumber时，可以看到类型"Int?"类型 
```

这时因为这个构造器可能会失败，所以它返回一个可选类型"Int?"，而不是"Int"（注意，这两种属于不同类型），"?"表示包含的值可能有值，也可能没有值，当我们需要使用Int类型时，需要转换为Int才可以。

e1. 声明一个可选类型

使用"?"来声明一个可选类型

```
var serverResponseCode: Int? = 404
// serverResponseCode 包含一个可选的 Int 值 404
print("\(serverResponseCode!)")  // 这里的'!'表示强解
// 输出 404
serverResponseCode = nil
// serverResponseCode 现在不包含值
```

如果你声明一个可选常量或者变量却没有赋值，他们会自动被设置为nil，就像下面这样：

```
var surveyAnswer: String?
// surveyAnswer 被自动设置为 nil
```

这时如果你想要使用"!"强解surveyAnswer会发生错误，所以在确定一个可选值时，务必可选值不为nil。

e2. 可选绑定
我们可以使用可选绑定（optional binding）来判断可选类型是否包含值，如果包含值就把值赋值给一个临时常量或者变量。通常会出现在 if 和 while 语句中，这条语句不仅可以用来判断可选类型中是否有值，同时可以将可选类型中的值赋值给一个常量或者变量。

```
let possibleNumber = "123"
if let actualNumber = Int(possibleNumber) {
    print("\'\(possibleNumber)\' has an integer value of \(actualNumber)")
} else {
    print("\'\(possibleNumber)\' could not be converted to an integer")
}
// 输出 "'123' has an integer value of 123"
```

因为Int(possibleNumber)包含来一个值，因此进入 if 的方法体，并会将该值赋值给常量actualNumber。

e3. 隐式解析可选类型

可选类型除了使用可选绑定来解析值，我们也可以使用'!'来解析值，但是可选此时不可出现nil的情况，否则会出现错误。

有时候，第一次被赋值之后，可以确定一个可选类型总会有值。在这种情况下，每次都要判断和解析可选值是非常低效的，因为可以确定它总会有值。

这种类型的可选状态被定义为隐式解析可选类型，我们可以将 '?' 换成 '!' 来声明一个隐式解析可选类型，这种情况下，我们无需手动使用 '!' 来解析可选值，系统会帮我们完整可选类型的解析。

```
let possibleString: String? = "An optional string."
let forcedString: String = possibleString! // 需要感叹号来获取值

let assumedString: String! = "An implicitly unwrapped optional string."
let implicitString: String = assumedString  // 不需要感叹号
```

需要注意的是：和普通可选类型一样，如果你在隐式解析可选类型没有值的时候尝试取值，同样会触发运行时错误。因此，如果一个变量之后可能变为 nil 的话，请不要使用隐式解析可选类型，否则请使用普通可选类型。

可选类型在开发过程中经常出现，大家需要好好理解，正确使用使用 "?" 和 "!"。

f. Any、AnyObject和AnyClass

Any : 表示任意类型，Any? 还包括了 nil
AnyObject : 代表任务 class 类型，无论 class 是否谁的子类又或者是基类
AnyClass : AnyObject的别名，和AnyObject一样


**4、几种运算符**

这里提及一些和OC中不一样的几种运算符。

a. 区间运算符

区间运算符通常用于整形或者范围。区间运算符有两种： '..<' 和 '...' ，前者为CountableRange，后者为类型ClosedRange
```
// 闭区间运算符
for i in 0...9 {
    print(i)
}
// 半开区间运算符
let names = ["Anna", "Alex", "Brian", "Jack"]
for i in 0..<names.count {
    print("第 \(i + 1) 个人叫 \(names[i])")
}
// 第 1 个人叫 Anna
// 第 2 个人叫 Alex
// 第 3 个人叫 Brian
// 第 4 个人叫 Jack

// 单侧区间
for name in names[2...] {
    print(name)
}
// Brian
// Jack
for name in names[...2] {
    print(name)
}
// Anna
// Alex
// Brian

let range = ...5
range.contains(7)   // false
range.contains(-1)  // true
```

b. 空合运算符

空合运算符（a ?? b）将对可选类型 a 进行判断，如果 a 包含一个值就进行解封，否则就返回一个默认值b。注：表达式 a 必须是可选类型，而 b 的类型必须和 a 存储值的类型保持一致。

```
let bValue = "default value"	// 作为默认值
var aValue : String?  // 默认值为 nil
let result = aValue ?? bValue
print(result)
// 输出 default value
```

c. 溢出运算符

在默认情况下，当向一个整数赋值超过它的容量时，Swift会报错。我们可以使用max或min访问整形的最大或最小值

```
var maxInt = Int8.max
var minInt = Int8.min
print("Int8.max:\(maxInt), Int8.min:\(minInt)")
// 输出 Int.max:127, Int.min:-128
```

如果此时我们尝试给 maxInt 加1或者给 minInt减1都会错误

```
maxInt += 1		// 编译错误
minInt -= 1		// 编译错误
```

我们可以使用溢出运算符来解决这种上溢或下溢现象

```
minInt = minInt &- 1	// 相同的有 &- &* &/
print("\(minInt)")
// 输出 127
```
Int8 型整数能容纳的最小值是 -128，以二进制表示即 10000000。当使用溢出减法运算符对其进行减 1 运算时，符号位被翻转，得到二进制数值 01111111，也就是十进制数值的 127，这个值也是 Int8 型整数所能容纳的最大值。

另外，在新版Swift中，++ 和 -- 运算符被取消，因此 i++ 这种形式的累加需要换成 i += 1 这种形式。

**5、控制语句**

Swift提供了多种控制流语句，包括while循环、if、guard、switch、跳转break、continue等等。在Swift中，for-in循环可以更简单的遍历数组、字段、区间、字符等序列类型，不同于C和OC，Swift在新版本中取消了C的for条件循环，即`for var i=0;i<a;i++`这种形式。值得一提的是，Swift中的switch语句比C中更加强大，case可以匹配不同的模式，包括范围匹配，元组和特定类型匹配，甚至是使用where来描述更多约束的情况。详情可以看官网翻译的[控制流。](http://www.swift51.com/swift4.0/chapter2/05_Control_Flow.html#control_transfer_statements)

我们这里介绍提前退出guard

像 if 语句一样，guard的执行取决于一个表达式的布尔值。当guard要求条件的为真时，可以继续执行guard语句之后的代码，否则提前返回结束。和 if 不同的是，guard通常只有一个 else 从句。

```
func greet(_ name:String) {
    guard name == "LOLITA0164" else {
        print("hello! \(name)")
        return
    }
    print("\(name)")
}
greet("xiao ming")	// 输出 hello! xiao ming
greet("LOLITA0164")	// 输出 LOLITA0164
```
我们看到，当输入"xiao ming"时，条件为false，输出 "hello! xiao ming"后返回退出，当输入"LOLITA0164"时，条件为真，继续执行guard后面当语句，输出"LOLITA0164"。

因此guard可以用来控制语句不满足条件时提前结束。

**6、函数和闭包**

a. 函数

Swift的函数参数和返回值非常灵活。参数可以是的无参、多参、默认参、可变参、甚至一些高级语言中的输入输出参数，参数类型也非常多，甚至包括另一个函数，另外Swift函数支持嵌套参数。在返回类型上，可以是无返回类型，多参数返回、甚至是一个函数。

Swift的函数使用 `func` 声明一个函数，形式为：

```
func name(parameters) -> return type {
    function body
}
```

在上一个介绍中，已经涉及到了函数创建和使用 `greet`，下面介绍一下常用的函数形式。

```
// 无参数函数
func sayHello() -> String {
    return "hello, world"
}
// 多参数函数，无返回值）（ -> Void 可以省略 ）
func personInfo(name: String, age: Int) -> Void {
    // function body
}
// 多重返回值，这里返回一个元组
func minMax(array: [Int]) -> (min: Int, max: Int) {
    return (array.min()!, array.max()!)
}
// 可选返回类型，‘_’ 表示忽略参数标签
func max(_ array: [Int]) -> Int? {
    return array.max()
}
// 指定参数标签（参数说明，和参数名以空格分隔），区别于参数名，默认参数名就是外部的标签名
func someFunction(parameterDescription parameter: String) {
    print(parameter)
}
someFunction(parameterDescription: "parameter")
// 可以看到参数标签只供外部调用显示使用，而参数名parameter可以供内部函数使用
// 默认参数
func sumFunc(a: Int, b: Int = 12) -> Int {
    return a &+ b
    // 如果你在调用时候不传第二个参数，parameterWithDefault 会值为 12 传入到函数体中。
}
print(sumFunc(a: 3))    // 输出 15
// 可变参数，使用（...）的方式接收可变参数，并且必须作为左后一个参数
func sumFunc2(_ numbers: Int ...) -> Int {
    var total: Int = 0
    // 和OC实现不定参数不同，Swift已将不定参转换为数组
    for number in numbers {
        total += number
    }
    return total
}
print("\(sumFunc2(1,2,3,4,5))") // 输出 15
/*
 输入输出参数
 如果你想要一个函数可以修改参数的值，并且想要在这些修改在函数调用结束后仍然存在，那么就可以将这个参数定义为输入输出参数
 这有点类似OC中常见的NSError的使用，他们的本质是接受一个地址，直接操作内存地址进行修改和访问
 可是使用 inout 关键字来定义输入输出参数，需要注意的是，在新版本中，已经将 inout 放在了 '参数名:' 后面，而不是前面
 */
func swapTwoObj(_ a: inout String, _ b : inout String) {
    (a, b) = (b, a)
}
var str1 = "123"
var str2 = "456"
swapTwoObj(&str1, &str2)
print("str1:\(str1),str2:\(str2)")  // 输出 str1:456,str2:123
```

a.1. 函数类型

在Swift中，函数可以看成是一种数据类型，使用就像使用其他类型一样，我们可以定义一个类型为函数类型的常量或变量，并将适当的函数赋值给它。

a.1.1 函数类型作为常量

```
// 我们将上面新写的交换方法赋值给一个常量
let newSwap = swapTwoObj
// 我们可以使用该常量正常调用
newSwap(&str1, &str2)
print("str1:\(str1),str2:\(str2)")
// 输出 str1:456,str2:123
```
a.1.2 函数类型作为参数

```
// 定义求和函数
func sum(_ a: Int,_ b: Int) -> Int{
    return a + b
}
// 定义求差函数
func diff(_ a: Int,_ b: Int) -> Int{
    return a - b
}
// 定义计算函数，可以接受一个函数类型的参数
func calculate(_ a: Int,_ b: Int, fn: (Int, Int) -> Int) -> Int {
    return fn(a,b)
}
print("sum：\(calculate(10, 5, fn: sum))")       // 输出 sum：15
print("diff：\(calculate(10, 5, fn: diff))")     // 输出 sum：5
```

a.1.3 函数类型作为返回值

```
// 获取一个函数类型
func getFunc() -> (Int, Int) -> Int {
    // 正如前面提到的，Swift支持嵌套函数，这里我们在函数内部定义一个嵌套函数作为返回值
    func sumNew(_ a: Int,_ b: Int) -> Int{
        return a*a + b*b
    }
    return sumNew
}
let fn = getFunc()
print("\(fn(2,3))")     // 输出 13 （2*2+3*3）
```

a.2 嵌套函数

```
func add() -> ()->Void {
    var total = 0
    var step = 1
    func fn() {
        total += step
        print("total:\(total)")
    }
    return fn
}
var a = add()
a()     // 输出 total:1
a()     // 输出 total:2
// 我们发现，第二次调用a()后，输出值为2而不是1，这说明虽然add()调用完成，但是嵌套函数 fn 捕获了total和step副本
```
b. 闭包

闭包是字包含的函数代码块，可以在代码中被传递和使用。Swift中的闭包与OC中的代码块（blocks）以及其他编程语言中的匿名函数比较相似。
闭包可以捕获和存储其所在上下文中的任意常量和变量的引用。在上面介绍的全局函数和嵌套函数实际上也是特殊的闭包，闭包采用如下三种形式之一：
1. 全局函数是一个有名字但不会捕获任何值的闭包
2. 嵌套函数是一个有名字并可以捕获其封闭函数作用域内值的闭包
3. 闭包表达式是一个利用轻量级语法所写的可以捕获其上下文中变量或常量值的匿名闭包

b.1 闭包表达式

```
{ (parameters) -> returnType in
    statements
}
```
Swift提供了名为 `sorted(by:)`的方法，它会根据你提供的用于排序的闭包函数将数组中的元素进行排序。你可以提供一个闭包函数，或者直接实现它的闭包函数（也叫内联闭包）。

b.1.1 实现一个内联闭包
```
// 定义一个字符串数组。提供数据源
let numbers = ["12", "21", "22", "3", "14"]
var numbers_sorted = numbers.sorted { (s1: String, s2: string) -> Bool in
    return Int(s1)! > Int(s2)!
}
print("\(numbers_sorted)")
// 输出 ["22", "21", "14", "12", "3"]
```
b.1.2 根据上下文推断类型

```
// 这里可以推断出参数类型以及返回值类型，可以省去参数类型和返回值类型
numbers_sorted = numbers.sorted { (s1, s2) in
    return Int(s1)! > Int(s2)!
}
```

b.1.3 单表达式闭包隐式返回

```
// 如果方法体是单行，我们可以省去return关键字，直接返回单行表达式的结果
numbers_sorted = numbers.sorted { (s1, s2) in
    Int(s1)! > Int(s2)!
}
```
b.1.4 参数名缩写

Swift自动为内联闭包提供了参数名缩写功能，你可以直接通过$0，$1等顺序访问参数。

```
// 当我们使用了参数名缩写时，in关键字也可以省略
numbers_sorted = numbers.sorted { Int($0)! > Int($1)! }
```

b.2 尾随闭包

如果你需要将一个很长的闭包表达式作为最后一个参数传递给函数，可以使用尾随闭包来增强函数的可读性。尾随闭包是一个书写在函数括号之后的闭包表达式，函数支持将其作为最后一个参数调用，事实上，当你使用尾随闭包时，你甚至可以把（）省去，另外，前面关于 `sorted` 内联闭包实现就是尾随闭包，一般来说，闭包会在（）内部。

b.3 逃逸闭包

当一个闭包作为参数传到一个函数中，但是这个闭包在函数返回之后才被执行，我们称该闭包从函数中逃逸。当你定义接受闭包作为参数当函数时，你可以在参数名之前标注`@escaping`，用来指明这个闭包是允许“逃逸“出这个函数的。
在iOS开发中，在我们使用异步操作的函数接受一个闭包参数作为 completion handler。这类函数会在异步操作开始之后立刻返回，但是闭包直到异步操作结束后才会被调用。在这种情况下，闭包需要“逃逸“出函数，因为闭包需要在函数返回之后被调用。

```
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}
```

**7、类和结构体**

和OC不同，Swift并不要求你为自定义的类去创建独立接口（.h）和实现（.m）文件。你所需要做的是一个单一文件定义一个类或者结构体，系统将会自动生成面向其他代码的外部接口。

在Swift中，类可以实现的功能结构体大部分都可以完成，如存储值、定义方法、初始化、构造器、可扩展、实现协议等，但是两者还是有一些差别的，相比结构体，类还具有如下功能：

1. 类可以继承
2. 类型转换允许在运行时检查和解释一个实例的类型
3. 析构器允许一个实例释放任何其所被分配的资源
4. 类是引用类型，而结构体是值类型

我们可以使用关键字 `class`来声明一个类，`struct`来声明一个结构体。
```
class Person {    // Swift中一个不继承于任何其他基类，那么此类本身就是一个基类
    // 定义属性，Swift中有两种属性，一个是计算属性，一种是存储属性
    // 实例属性
    var name:String     // 存储属性
    var height = 0.0    // 存储属性
    var info: String {  // 计算属性
        return "\(name)" + "\(height)"
    }
    // 类型属性
    static var address = "fuzhou"
    // 懒加载，必须是变量，并且需要一个初始值
    lazy var data: [String] = [String]()
    // 构造方法，注意如果不编写构造方法，默认自动创建一个无参数的构造方法
    init(name:String ,height:Double) {
        self.name = name
        self.height = height
    }
    // 遍历构造器，提供默认值来简化构造方法实现
    convenience init(name:String) {
        self.init(name: name, height: 170.0)
    }
    // 定义方法
    // 实例方法
    func showMessage() {
        print("name=\(self.name),height=\(height)")
    }
    // 类型方法，结构体中使用 static 关键字
    class func sayHello() -> Void {
        print("hello world!")
    }
    // 析构方法，在对象被释放时调用，类似于oc的dealloc，此函数没有括号，没有参数，无法直接调用
    deinit {
        print("deinit...")
    }
}
```

类中常用的"元素"都在上面，接下来在外面做使用演示：

```
// 使用类型属性
print("\(Person.address)")  // 输出 fuzhou
// 调用类型方法
Person.sayHello()   // 输出 hello world!
var p: Person? = Person.init(name: "LOLITA0164") // 创建一个实例
// 调用实例方法
p!.showMessage()  // 输出 name=LOLITA0164,height=170.0
// 测试释放（注意，实例必须是可选类型才可以赋值为nil）
p = nil     // 输出 deinit...
```
关于类的构造器的一些说明

```
 1、子类的指定构造方法必须调用父类构造方法，并确保调用发生在子类存储属性初始化之后，而且指定构造方法不能调用同一个类中的其他指定构造方法
 2、便利构造方法必须调用同一个类中的其他指定构造方法（可以是指定构造方法或者便利构造方法），不能直接调用父类构造方法（用以保证最终以指定构造方法结束）
 3、如果父类仅有一个无参数构造方法（不管是否包含便利构造方法），子类的构造方法默认就会自动调用父类的无参构造方法（这种情况下可以不用手动调用）
 4、常量属性必须默认指定初始值或者在当前类的构造方法中初始化，不能在子类构造方法中初始化
```

关于类和结构体的选择

在开发中，我们可以使用类和结构体来自定义数据类型，但是在两种数据类型选择的上需要考虑不同的业务情况因为结构体实例总是通过值传递，类实例总是通过引用传递。
按照通用的准则，当符合一条或多条以下条件时，请考虑构建结构体：

1. 该数据结构的主要目的是用来封装少量相关简单数据值
2. 有理由预计该数据结构的实例在被赋值或传递时，封装的数据将会被拷贝而不是被引用
3. 该数据结构中存储的值类型属性，也应该被拷贝，而不是引用
4. 该数据结构不需要去继承另一个既有类型的属性或行为

举例来说，以下情景中适合使用结构体：

1. 几何形状的大小：封装一个width属性和height属性
2. 一定范围的路径：封装一个start属性和length属性
3. 三维坐标系内的一点：封装x，y和z属性

在所有其他案例中，定义一个类，生成一个它的实例，并通过引用来管理和传递，在实际应用中，这意味着绝大部分的自定义数据类型都应该是类，而不是结构体。


> 在Swift中，许多基本类型，诸如String、Array和Dictionary类型都是以结构体的形式实现，这意味着被赋值给新的常量或变量，或传入函数、方法中时，他们的值会被拷贝。在OC中，则相反，它们是类实现，引用传递。

**8、属性**

在OC中，除了属性之外，还可以使用成员变量作为属性值的后端存储。而在Swift中，把这些理论统一使用属性来实现。

属性将值和特定的类、结构、枚举关联。属性有存储属性和计算属性，前者存储常量或者变量，只能用于类和结构体，后者计算一个值，不仅可以用于类和结构体，还可以是枚举。
另外，我们可以使用属性观察器来监控属性值的变化，以此来触发一个自定义的操作。属性观察器可以添加到自定义存储属性，也可以添加到继承父类的属性上。

存储属性在函数一节有所提及，下面说一些特别的属性

a. 计算属性

计算属性不直接存储值，而是提供一个 `getter` 和一个可选的 `setter` ，来间接获取和设置其他属性或变量的值。

```
class Person {
    var firstName = ""  // 存储属性 
    var lastName = ""   // 存储属性
    // set、get的计算属性
    var fullName: String {
        get{
            return "\(firstName)" + "\(lastName)"
        }
        set {
            // set中的newValue表示即将赋予的新值，也可以定义其他名称 set(otherValye){}
            let array:Array = newValue.components(separatedBy: ".")
            if array.count>=2 {
                firstName = array[0]
                lastName = array[1]
            }
        }
    }
    // 只读的get属性，这时可以去掉get和花括号
    var info: String {
        return fullName
    }
}
```


b. 延迟属性（懒加载）

OC中的懒加载在Swift中使用延迟属性完成，使用关键字 `lazy` 来标记。
延迟属性必须是变量（var关键字），这是因为属性的初始值可能在实例构造完成之后才会得到，而常量属性在构造过程完成之前必须要有初始值，因此无法声明成延迟属性。

延迟属性在开发过程中非常有用，它可以避免一些不必要的初始化，而将属性初始化延迟到第一次调用的时刻。

```
class Person {
    lazy var data: [String] = [String]()
    lazy var p: Person = {
        return Person()
    }()
}
```

c. 属性观察器

Swift中提供了`willSet`和`didSet`两个属性观察器来监控和响应存储属性值的变化，你不必为非重写的计算属性添加属性观察器，因为它们可以通过	`setter`和`getter`来监控和响应值的变化。

`willSet` 在新值被设置之前调用，并可以获取到 newValue；
`didSet` 在新值被设置之后调用，并可以获取到 oldValye。

注：如果在`didSet`方法中再次对该属性赋值，那么新值将被覆盖。在构造器中进行第一次初始化时，不会触发观察器。

```
class Person {
    var firstName = "" {
	    willSet {
		    // do something
	    }
	    didSet {
		    // do something
	    }
    }
}
```

d. 类型属性

实例属性属于一个特定的实例，每创建一个实例，该实例拥有属于自己的一套属性值，实例之间的属性相互独立。如果一种属性的拥有者属于同一种类型，那么我们称之为类型属性，这种类型用于所有实例共享数据。我们通过 `class` 或 `static` 来定义类型属性。

存储型类型属性可以是变量，也可以是常量，计算型类型属性只能是变量属性，这一点和实例计算型属性一样。

注：存储型的类型属性必须指定默认值。

**9、枚举**

枚举为一组相关的值定义了一个共同的类型。

Swift中的枚举的原始值类型可以是字符串、字符、整型或者浮点数。

此外，枚举成员可以指定任意类型的关联值存储到枚举成员中。

a. 枚举语法

使用 `enum` 关键字来创建枚举

```
enum SomeEnumeration {
    // 枚举定义放在这里
}
```

示例

```
// 单行
enum Season{
    case spring, summer, autumn, winter
}
// 多行，并指定原始值
enum Week: Int {
    case monday     = 1
    case tuesday    = 2
    case wednesday  = 3
    case thursday   = 4
    case friday     = 5
    case saturday   = 6
    case sunday     = 7
}

print("\(Week.friday)")				// 输出 friday
print("\(Week.friday.rawValue)")	// 输出 5
```

枚举类型也可以拥有一些结构体的特性，如计算属性、构造方法、方法等

```
enum Week: Int {
    case monday     = 1
    case tuesday    = 2
    case wednesday  = 3
    case thursday   = 4
    case friday     = 5
    case saturday   = 6
    case sunday     = 7
    
    // 定义计算属性（枚举中不能使用存储属性）
    var tag: Int {
        return self.rawValue
    }
    // 定义类型计算属性
    static var enumName:String {
        return "Week"
    }
    // 定义实例方法
    func showRowValue() {
        print("\(self.rawValue)")
    }
    // 定义类型方法
    static func showInfo() {
        print("Enum name is \(Week.enumName)")
    }
    // 定义构造方法，因为可能构造失败返回nil，因此需要返回值为可选类型
    init?(name:String){
        if name.contains("mon") {
            self = .monday
        }else if name.contains("tue") {
            self = .tuesday
        }else if name.contains("wed") {
            self = .wednesday
        }else if name.contains("thu") {
            self = .thursday
        }else if name.contains("fri") {
            self = .friday
        }else if name.contains("sat") {
            self = .saturday
        }else if name.contains("sun") {
            self = .sunday
        }else{
            return nil
        }
    }
}

var day = Week.monday
print("\(day.tag)")         // 输出 1
print("\(Week.enumName)")   // 输出 Week
day.showRowValue()          // 输出 1
Week.showInfo()             // 输出 Enum name is Week
// 构造一个枚举类型，系统有默认的构造器，这里我们使用自定义的构造器
var dayNew = Week.init(name: "wed")
dayNew?.showRowValue()      // 输出 3
```
b. 关联值

前面有提到，枚举可以拥有关联值，这让枚举可以存储一些额外的自定义信息（这些信息是任意类型），额外信息并不影响你正常使用枚举，下面举个关于颜色的例子。

```
enum Color {
    case RGB(String)
    case CMYK(Float,Float,Float,Float)
    case HSB(Int,Int,Int)
}

var red = Color.RGB("#FF0000")
switch red {
case let .RGB(colorValue):
    print("color's value=\(colorValue)")
case let .CMYK(c, m, y, k):
    print("c=\(c),m=\(m),y=\(y),k=\(k)")
case let .HSB(h, s, b):
    print("h=\(h),s=\(s),b=\(b)")
}
// 输出 color value = #FF0000
```


**10、扩展**

在Swift中，扩展可以为一个已知的类、结构体、枚举或者协议类型添加新的功能，这个扩展和OC的分类类似，OC中的分类需要名字的，而Swift只需使用 `extension` 关键字加 需要扩展的类型即可。另外，但是要比OC的分类功能强大，它可以：

1. 添加计算型实例或类型属性
2. 定义实例方法和类型方法
3. 提供新的构造器
4. 定义下标
5. 定义和使用新的嵌套型
6. 实现已知类型的某个协议方法

注：扩展虽然可以添加新功能，但是不能重写已有的功能。

a. 扩展的语法

```
extension SomeType {
    // 为 SomeType 添加的新功能写到这里
}
```

我们来扩展一个 UIColor 类来演示扩展的使用。

```
extension UIColor {
    /// 扩展RGB类型，类型方法
    static func rgba(_ r:CGFloat,_ g:CGFloat,_ b:CGFloat,_ a:CGFloat) -> UIColor {
        return UIColor.init(red: (r)/255.0, green: (g)/255.0, blue: (b)/255.0, alpha: a)
    }
    static func rgb(_ r:CGFloat,_ g:CGFloat,_ b:CGFloat) -> UIColor {
        return rgba((r), (g), (b), 1)
    }
    /// 十六进制的色值，类型方法
    static func hex(_ value:Int) -> UIColor{
        return rgb(CGFloat((value & 0xFF0000) >> 16),
                   CGFloat((value & 0x00FF00) >> 8),
                   CGFloat((value & 0x0000FF)))
    }
    /// 随机色，类型属性
    static var rand: UIColor {
        return rgb(CGFloat(arc4random_uniform(255)), CGFloat(arc4random_uniform(255)), CGFloat(arc4random_uniform(255)))
    }
}
```

使用

```
// 获取一个随机色
var color = UIColor.rand
// 获取十六进制的颜色
color = UIColor.hex(0xff0000)
// 获取RGB的颜色
color = UIColor.rgb(0, 0, 255)
```
对于其他的扩展功能，大家自行尝试。

**11、协议**

协议，用来约束某个类型而定义的某些特定的任务或者功能方法、属性等，类、结构体、枚举都可以遵循和实现协议。关于[协议](http://www.swift51.com/swift4.0/chapter2/22_Protocols.html)，可以在官方翻译中查询跟多的详细介绍。

a. 协议语法

```
protocol SomeProtocol {
    // 这里是协议的定义部分
}
```

要让自定义类型遵循某个协议，在定义类型时，需要在类型名称后加上协议名称，中间以冒号（:）分隔。遵循多个协议时，各协议之间用逗号（,）分隔：

```
struct SomeStructure: FirstProtocol, AnotherProtocol {
    // 这里是结构体的定义部分
}
```

拥有父类的类在遵循协议时，应该将父类名放在协议名之前，以逗号分隔：

```
class SomeClass: SomeSuperClass, FirstProtocol, AnotherProtocol {
    // 这里是类的定义部分
}
```
示例
```
// 定义一个可以说话的协议
protocol sayHelloProtocol {
    func sayHello()
}
// 定义一个Person类，并遵循这个协议
class Person: sayHelloProtocol {
    // 实现这个sayHello功能
    func sayHello() {
        print("hello, world!")
    }
}
// 定义一个Dog类，并遵循这个协议
class Dog: sayHelloProtocol {
    // 实现这个sayHello功能
    func sayHello() {
        print("wang! wang wang!")
    }
}
let p = Person()
let d = Dog()
p.sayHello()    // 输出 hello, world!
d.sayHello()    // 输出 wang! wang wang!
```
b. 可选协议

协议中的要求默认都是必须实现的，当协议可以定义可选要求时，遵循协议当类型可以选择实现这些要求，或选择不实现。

在协议中使用 `optional` 关键字作为前缀来定义可选要求，可选要求用在OC混编中，协议和可选要求必须带上 `@objc` 关键字。标记 `@objc` 特性的协议只能被继承自 OC 的类或者 `@objc` 类遵循，其他类以及结构体、枚举均不能遵循这种协议。

```
// 定义一个可以说话的协议
@objc protocol sayHelloProtocol {
    func sayHello()
    @objc optional func run()
}
```

在我们定义的 sayHello 协议中，我们定义了一个可选要求的 run 方法，在 Person类和Dog类中都可以选择不实现。

**12、[循环引用](http://www.swift51.com/swift4.0/chapter2/16_Automatic_Reference_Counting.html#strong_reference_cycles_between_class_instances)**

Swift中采用的是ARC来管理内存，这中ARC和OC中的ARC非常相似，因此Swift中的循环引用和OC中的循环引用非常类似。这里就只介绍循环引用的解决办法。

Swift 提供了两种办法来解决实例之间的循环引用问题：弱引用（weak）和无主引用（unowned）。

相比于当前实例，其他实例有更短的生命周期时，使用弱引用，相反的，当其他实例有相同的或者更长生命周期时，请使用无主引用。

a. 弱引用（weak）

因为弱引用不会保持所引用的实例，即使引用存在，实例也有可能被销毁。因此，ARC会在引用的实例被销毁后自动将其赋值为nil。并且因为弱引用可以允许它们的值在运行时被赋值为nil，所以它们会被定义为可选类型变量，而不是常量。

```
class Person {
    var apartment: Apartment?
    deinit { print("Person is being deinitialized") }
}

class Apartment {
    weak var tenant: Person?
    deinit { print("Apartment is being deinitialized") }
}

var p: Person? = Person()
var a: Apartment? = Apartment()
// 循环引用
p?.apartment = a
a?.tenant = p
```

Person实例保持对Apartment实例对强引用，但是Apartment实例只是对Person实例的弱引用。这意味着当你断开 p 变量所保持的强引用时，再也没有指向Person实例的强引用了：

```
p = nil // 输出 Person is being deinitialized
```

现在唯一剩下的指向Apartment实例的强引用来自于变量 a 。如果你断开这个强引用，再也没有指向Apartment实例的强引用了，这是 a 也将会被销毁：

```
a = nil // 输出 Apartment is being deinitialized
```

b. 无主引用（unowned）

无主引用和弱引用类似，不会牢牢保持住引用的实例，但是无主引用通常都被期望拥有值，ARC无法在实例被销毁和将无主引用设为nil，因为非可选类型的变量不允许被赋值为nil。

注：

使用无主引用，你必须确保引用始终指向一个未销毁的实例，如果你试图在实例被销毁后，访问该实例的无主引用，会触发运行时错误。这一点和弱引用相反。

b.1 闭包中的循环引用

Swift 提供了一种优雅的方法来解决这个问题，称之为闭包捕获列表。我们在定义闭包时同时定义捕获列表作为闭包的一部分。捕获列表定义了闭包体内捕获一个或者多个引用类型的规则。

闭包捕获列表需要放在参数列表和返回类型前面，如果没有则放在 in 前面：

```
var someClosure: Void -> String = {
    [unowned self, weak delegate = self.delegate!] in
    // 这里是闭包的函数体
}
```

c. 弱引用和无主引用的选择

> 相比于当前实例，其他实例有更短的生命周期时，使用弱引用；相反的，当其他实例有相同的或者更长生命周期时，请使用无主引用。


**13、类型转换**

关于类型转换，在实际开发中经常使用，想当初笔者没有什么正确的文档参考，更具xcode的提示，加之自己多次尝试，才摸出点门道来。

类型转换无非是判断实例的类型，将其转换为其子类或者父类，当然转为父类是不很必要的，我们遇到最多的情况就是将父类转为其子类。

a. is 检查类型，as 向下转型

用类型检查操作符（is）来检查一个实例是否属于特定子类型。若实例属于那个子类型，类型检查操作符返回  true，否则返回 false。

某类型的一个常量或变量可能在幕后实际上属于一个子类。当确定是这种情况时，你可以尝试向下转到它的子类型，用类型转换操作符（as? 或 as!）。

由于向下转型可能失败，因此类型转换有两种不同的形式。as? 返回一个可选值，强制形式 as! 会强制解包转换结果。当你不确定是否成功，使用 as? ,如果转型失败，返回nil，这时你能够检查向下转型是否成功。只有当你可以确定向下转型一定会成功时，才使用强制形式 as! 。当你试图向下转型一个不确定的类型时，强制形式的类型转换会触发一个运行时错误。

b. 实例对象向下转换

```
// 定义一个动物类
class animal: NSObject {
    var name: String
    init(name: String) {
        self.name = name
    }
    func showMessage() {
        print("\(name) is \(self.classForCoder) 类型")
    }
}
// 定义一个猫类，该类继承自动物类
class Cat: animal {
    func eat() {
        print("Cat loves to eat fish")
    }
}
// 定义一个狗类，该类继承自动物类
class Dog: animal {
    func bark() {
        print("Dog barking")
    }
}

let cat = Cat.init(name: "mi mi")
let dog = Dog.init(name: "wang cai")
// 将如我们有一个实例数组
let animals = [cat,dog]
// 当我们需要根据不同实例类型调用不同功能时
for anim in animals{
    anim.showMessage()
    // 使用 is 关键字判断类型
    if anim is Cat {
        // 使用 as 转换类型
        let cat = anim as! Cat
        cat.eat()
    }
    // 使用 isKind 方法来判断类型
    else if anim.isKind(of: Dog.classForCoder()){
        let dog = anim as! Dog
        dog.bark()
    }
}

// 输出
/*
mi mi is Cat 类型
Cat loves to eat fish
wang cai is Dog 类型
Dog barking
*/
```

c. Any 和 AnyObject 的类型转换

Swift 为不确定类型提供了两种特殊的类型别名：

1. Any 可以表示任何类型，包括函数类型
2. AnyObject 可以表示任何类类型的实例

例如我们又一个 Any 类型来混合不同类型一起工作，当我们需要特性类型执行特性任务时，就需要将 Any 转为不同的类型。

```
var things = [Any]()
things.append(0)
things.append(0.0)
things.append(42)
things.append(3.14159)
things.append("hello")
things.append((3.0, 5.0))
things.append({ (name: String) -> String in "Hello, \(name)" })
for thing in things {
    switch thing {
    case 0 as Int:
        print("zero as an Int")
    case 0 as Double:
        print("zero as a Double")
    case let someInt as Int:
        print("an integer value of \(someInt)")
    case let someDouble as Double where someDouble > 0:
        print("a positive double value of \(someDouble)")
    case is Double:
        print("some other double value that I don't want to print")
    case let someString as String:
        print("a string value of \"\(someString)\"")
    case let (x, y) as (Double, Double):
        print("an (x, y) point at \(x), \(y)")
    case let stringConverter as (String) -> String:
        print(stringConverter("Michael"))
    default:
        print("something else")
    }
}
```

## OC和Swift混编

a. Swift 和 OC 的映射关系

| Swift | OC | 说明 |
| --- | --- | --- |
|AnyObject|id(ObjC中的对象任意类型)|由于ObjC中的对象可能为nil，所以Swift中如果用到ObjC中类型的参数会标记为对应的可选类型|
|Array、Dictionary、Set|NSArray、NSDictionary、NSSet|注意：ObjC中的数组和字典不能存储基本数据类型，只能存储对象类型，这样一来对于Swift中的Int、UInt、Float、Double、Bool转化时会自动桥接成NSNumber|
|Int|	NSInteger、NSUInteger|其他基本类型情况类似，不再一一列举|
|NSObjectProtocol|NSObject协议（注意不是NSObject类）||
|ErrorType|NSError||
|\#selector(ab:)|	@selector(ab:)||
|init(x:X,y:Y)|initWithX:(X)x y:(Y)y||
|func xY(a:A,b:B)|void xY:(A)a b:(B)b||
|extension（扩展）|category（分类）|注意：不能为ObjC中存在的方法进行extension|
|Closure（闭包）|block（块）|注意：Swift中的闭包可以直接修改外部变量，但是block中要修改外部变量必须声明为__block|

Swift 兼容来大部分 OC，当然还有一些 Swift 不能够使用的，例如 OC 中的预处理指令，即宏定义不可使用，虽然在目前4.2版本下，已经开始支持了少量的宏，如

```
#if DEBUG
#else
#endif
```
这种简单的预处理指令。Swift 中推荐使用常量定义，定义一些全局的常量实例、全局方法来充当 宏 的作用。

我们直到，Swift 中的一些自定义基类可以是不继承自 NSObject，这是 OC 调用这类的的时刻，就需要使用关键自 `@objc` 来标记，这样就可以正常使用来。

Swift中还有很多小细节部分，待大家在进一步学习过程中慢慢体会。

b. Objc 调用 Swift

由于国内大部分还都是 OC 编程环境，所以通常是先学习 OC 下调用 Swift。接下来我演示以下 OC 下配置桥接文件，以及 Objc 和 Swift 的相互调用。

当我们在项目中第一次添加不同语言的文件时，Xcode就会询问你是否创建桥接文件，你只需要点击"YES"，Xcode就可以帮你完成桥接配置（注意：第二次不会提示你，即使你删除来桥接文件）。

![桥接文件](https://img-blog.csdn.net/20180827162708556?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xPTElUQTAxNjQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

这份桥接文件是 Swift 调用 OC 文件的时，导入 OC 头文件使用的。在桥接文件中你可看到以下内容。

```
//  Use this file to import your target's public headers that you would like to expose to Swift.
```

当然你也可以手动创建桥接文件，自己配置环境。

我们可以创建"项目名称-Bridging-Header"一个 .h 文件作为桥接文件，然后为项目配置此文件的路径。

![配置文件](https://img-blog.csdn.net/20180827164759921?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xPTElUQTAxNjQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

此文件是存放 Swift 使用的 OC 头文件，此小节先不管它。

在 OC 调用 Swift 情况下，是通过Swift生成的一个头文件实现的，这个头文件是编译器自动完成的，它会将你在项目是到的 Swift 文件自动映射称 OC 语法，其他并不需要开发关注，该文件名称为 "项目名称-Swift"，你可以在 Build Settings 里搜索 Swift  可以看到：

![Swift配置环境](https://img-blog.csdn.net/20180827165941240?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xPTElUQTAxNjQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

这里还可以配置其他 Swift 的选项，如语言版本等。

接下来我们创建一个名为 "AViewController.swift" 的控制器，我们现在要在 OC 文件使用到该文件，首先，我们导入"项目名称-Swift.h"的文件，不会提示，需要你手动写入（可以按住Command点击头文件查看）。

在 OC 的 "ViewController.h" 文件中，我们进行跳转控制器，这个控制器的使用和使用 OC的类没有任何却别，系统的"项目名称-Swift.h"文件已经帮我完成了映射。

```
#import "ViewController.h"
#import "TestDemo-Swift.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.title = @"OC页面";
    self.view.backgroundColor = [UIColor whiteColor];
}

-(void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    AViewController *ctrl = [AViewController new];
    [self.navigationController pushViewController:ctrl animated:YES];
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

@end
```

c. Swift 调用 Objc

前面提到了 "TestDemo-Bridging-Header.h" 文件，这个文件中存储的是用于供给 Swift 使用的 OC 头文件。

我们将 OC 控制器文件 "ViewController.h" 导入该文件，这样我们在 Swift 文件中就可以使用该文件。

```
import UIKit

class AViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        self.title = "Swift页面";
        self.view.backgroundColor = UIColor.red
    }

    override func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?) {
        let ctrl = ViewController()
        self.navigationController?.pushViewController(ctrl, animated: true)
    }
    
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }
}
```

![演示](https://img-blog.csdn.net/20180827174302503?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xPTElUQTAxNjQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

上面介绍了在 OC 工程中创建 Swift 文件，其实在 Swift 工程中创建 OC 文件是类似的，大家可以尝试一下。


## Swift中的'宏'和工具类

通过前面的学习，我们已经对 Swift 的语法，和 OC 进行混编等知识有了一定的了解，下面演示一些在项目中可能使用到的工具类。

我们知道，在目前的Swift中，还不能够调用OC中的宏定义，这些东西势必需要我们重新写过。在Swift中，所对应OC的宏，只不过是一些全局常量或者方法。下面列举一些我的在OC项目中映射多来的"宏"。

```
// MARK: -------------------------- 适配 --------------------------
/// 屏幕宽度
let kScreenHeight = UIScreen.main.bounds.height
/// 屏幕高度
let kScreenWidth = UIScreen.main.bounds.width

/// 状态栏高度
let kStatusBarHeight = UIApplication.shared.statusBarFrame.size.height
/// 导航栏高度
let kNavigationbarHeight = (kStatusBarHeight+44.0)
/// Tab高度
let kTabBarHeight = kStatusBarHeight > 20 ? 83 : 49

/// 屏幕比率
let kScreenWidthRatio = (kScreenWidth/375.0)
/// 调整
func AdaptedValue(x:Float) -> Float {
    return Float(CGFloat(x) * kScreenWidthRatio)
}
func ShiPei(x:Float) -> Float{
    return AdaptedValue(x: x)
}
/// 调整字体大小
func AdaptedFontSizeValue(x:Float) -> Float {
    return (x) * Float(kScreenWidthRatio)
}

// MARK: -------------------------- 颜色 --------------------------
// MARK: 请参考 UIColor_Extentsion.swift 文件

// MARK: -------------------------- 偏好 --------------------------
func SaveInfoForKey(_ value:String ,_ Key:String) {
    UserDefaults.standard.set(value, forKey: Key)
    UserDefaults.standard.synchronize()
}
func GetInfoForKey(_ Key:String) -> Any! {
   return UserDefaults.standard.object(forKey: Key)
}
func RemoveObjectForKey(_ Key:String){
    UserDefaults.standard.removeObject(forKey: Key)
    UserDefaults.standard.synchronize()
}

// MARK: -------------------------- 输出 --------------------------
// 带方法名、行数
func printLog<T>(_ message:T,method:String = #function,line:Int = #line){
    print("-[method:\(method)] " + "[line:\(line)] " + "\(message)")
}
// 只在Debug下输出，为了给习惯OC输出写法的同事
func DLog(_ format: String, method:String = #function,line:Int = #line,_ args: CVarArg...){
//    print("-[method:\(method)] " + "[line:\(line)] ", separator: "", terminator: "")
    #if DEBUG
        let va_list = getVaList(args)
        NSLogv(format, va_list)
    #else
    #endif
}
// 只在Debug下输出，为了为习惯PHP输出写法的同事
func echo(_ format: String,_ args: CVarArg...) {
    #if DEBUG
        let va_list = getVaList(args)
        NSLogv(format, va_list)
    #else
    #endif
}
```

使用【扩展】功能创建一些工具

颜色类

```
extension UIColor {
	// RGB颜色
    static func rgba(_ r:CGFloat,_ g:CGFloat,_ b:CGFloat,_ a:CGFloat) -> UIColor {
        return UIColor.init(red: (r)/255.0, green: (g)/255.0, blue: (b)/255.0, alpha: a)
    }
    static func rgb(_ r:CGFloat,_ g:CGFloat,_ b:CGFloat) -> UIColor {
        return rgba((r), (g), (b), 1)
    }
    /// 十六进制的色值 例如：0xfff000
    static func hex(_ value:Int) -> UIColor{
        return rgb(CGFloat((value & 0xFF0000) >> 16),
                   CGFloat((value & 0x00FF00) >> 8),
                   CGFloat((value & 0x0000FF)))
    }
    /// 十六进制的色值，例如：“#FF0000”，“0xFF0000”
    static func hexString(_ value:String) -> UIColor{
        var cString:String = value.trimmingCharacters(in: CharacterSet.whitespacesAndNewlines)
        cString = cString.lowercased()
        if cString.hasPrefix("#") {
            cString = cString.substring(from: cString.index(after: cString.startIndex))
        }
        if cString.hasPrefix("0x") {
            cString = cString.substring(from: cString.index(cString.startIndex, offsetBy: 2))
        }
        if cString.lengthOfBytes(using: String.Encoding.utf8) != 6 {
            return UIColor.clear
        }
        var startIndex = cString.startIndex
        var endIndex = cString.index(startIndex, offsetBy: 2)
        let rStr = cString[startIndex..<endIndex]
        startIndex = cString.index(startIndex, offsetBy: 2)
        endIndex = cString.index(startIndex, offsetBy: 2)
        let gStr = cString[startIndex..<endIndex]
        startIndex = cString.index(startIndex, offsetBy: 2)
        endIndex = cString.index(startIndex, offsetBy: 2)
        let bStr = cString[startIndex..<endIndex]
        var r :UInt32 = 0x0;
        var g :UInt32 = 0x0;
        var b :UInt32 = 0x0;
        Scanner.init(string: String(rStr)).scanHexInt32(&r);
        Scanner.init(string: String(gStr)).scanHexInt32(&g);
        Scanner.init(string: String(bStr)).scanHexInt32(&b);
        return rgb(CGFloat(r), CGFloat(g), CGFloat(b))
    }
    /// 随机色
    static var rand: UIColor {
        return rgb(CGFloat(arc4random_uniform(255)), CGFloat(arc4random_uniform(255)), CGFloat(arc4random_uniform(255)))
    }
}
```

字符转数字类

```
extension String {
    /// 将字符串转换为合法的数字字符
    ///
    /// - Returns: String类型
    func toPureNumber() -> String {
        let characters:String = "0123456789."
        var originString:String = ""
        for c in self {
            if characters.contains(c){
                if ".".contains(c) && originString.contains(c){}
                else{
                    originString.append(c)
                }
            }
        }
        return originString
    }
    /// 将字符串转换为 Double 类型的数字
    ///
    /// - Returns: Double类型
    func toDouble() -> Double {
        return Double(self.toPureNumber())!
    }
    /// 将字符串转换为 Float 类型的数字
    ///
    /// - Returns: Float类型
    func toFloat() -> Float {
        return Float(self.toDouble())
    }
    /// 将字符串转换为 Int 类型的数字
    ///
    /// - Returns: Int类型
    func toInt() -> Int {
        return Int(self.toDouble())
    }
    
    // 保留旧版本的字符转换为数字类型，例："123".floatValue
    var pureNumber: String {
        return self.toPureNumber()
    }
    var doubleValue: Double {
        return self.toDouble()
    }
    var floatValue: Float {
        return self.toFloat()
    }
    var intValue: Int {
        return self.toInt()
    }
}
```

以上介绍了几种工具类的创建方式，相信大家进过基础一节过后，都能看懂。

## 实战篇

在这一节中，我们利用之前的测试项目简单的演示Swift下如何搭建页面、和OC进行数据传递，其中会涉及到自定义数据模型、延迟加载、类型转换、协议、协议代理模式、弱引用、闭包、闭包传值、检测释放等等，希望能都帮助到第一次接触 Swift 的小白们。

Demo中，简单搭建一个Swift页面，页面中展示一个列表，给列表Cell有标题，自标题和一张图片，用户点击某个cell时，将数据回传到上一个OC页面。

a. 定义Model层

首先我们需要要一个自定义模型，以及一个用来获取、处理数据的View-Model。

首先是自定义模型：

```
import UIKit
// 列表数据源 
class AVTableModel: NSObject {
    var title: String?
    var subTitle: String?
    var image: String?
}
```

View-Model：

```
import UIKit

class AVTableManager: NSObject {
    // 定义数据请求，使用闭包回调
    class func requestData(response:@escaping (_ flag : Bool,_ resArray : [AVTableModel]?) -> Void) {
        // 模拟数据
        var resList = [AVTableModel]()
        for i in 0...30 {
	        // 使用自定义模型
            let avtm:AVTableModel = AVTableModel()
            avtm.title = "\(i) row"
            avtm.subTitle = [
                                "月光推开窗亲吻脸颊，沁得芬芳轻叩门，敛去湖中婆娑影，拈起肩上落梨花",
                                "屋檐下的风轻轻拂过了衣角，弄皱了桌上的画卷，月影疏疏，落花朵朵，不经意间，看成了风景",
                                "远处的烟穿过水路，哼着不知名的小调，踏碎了一方的圆月",
                                "谁家的酒酿醉了空气中潮湿的时光，睁眼瞬间，藏进了楼阁"
                            ][i%4]
            avtm.image = ["0.jpg","1.jpg","2.jpg"][i%3]
            resList.append(avtm)
        }
        // 回调数据和当前状态
        response(true, resList)
    }
}
```

b. 控制器

控制器需要完成UI的搭建，因此需要一个列表，完成相应的列表代理，再者就是定义数据，完成页面交互逻辑。

Swift 中列表是使用和 OC 非常类型，仅仅是方法的实现和调用方面是链式调用。

```
import UIKit

class AViewController: UIViewController,UITableViewDelegate,UITableViewDataSource {	// 实现列表的代理
    // 数据源，延迟属性
    lazy var dataArray = [AVTableModel]()
    // 列表视图
    @IBOutlet weak var table: UITableView!
    override func viewDidLoad() {
        super.viewDidLoad()
        self.title = "Swift页面";
        // 配置UI
        self.setUpUI()
        // 获取数据
        self.getData()  // 获取数据
    }
    
    // 配置UI
    func setUpUI() {
        self.table.tableFooterView = UIView()
    }
    
    // 获取数据
    func getData() {
        // 这里因为是类方法，不会产生循环引用
        AVTableManager.requestData { (flag, resList) in
            if flag {
                self.dataArray = resList!
                self.table.reloadData()
            }
        }
    }
    
    // tableView的代理
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.dataArray.count
    }
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        // 创建cell，并返回给tableView
    }
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 60.5
    }
    
    // 检测释放
    deinit {
        print("\(self.classForCoder) 释放了。。。")
    }
    
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }

}
```

c. 视图层

我这里使用的是 xib，使用方式和 OC 下并无多少差异，将一些必要的控件拉到类中做入口。

```
import UIKit
class AVTableViewCell: UITableViewCell {
    @IBOutlet weak var titleLabel: UILabel!      // 标题
    @IBOutlet weak var subTitleLabel: UILabel!   // 子标题
    @IBOutlet weak var imgView: UIImageView!     // 图片
}
```

在控制器中完成列表cell的注册，以及cell的赋值

```
import UIKit

class AViewController: UIViewController,UITableViewDelegate,UITableViewDataSource {
    // 数据源，延迟属性
    lazy var dataArray = [AVTableModel]()
    // 列表视图
    @IBOutlet weak var table: UITableView!
    let cellId = "cellId"
    
    override func viewDidLoad() {
        super.viewDidLoad()
        self.title = "Swift页面";
        // 配置UI
        self.setUpUI()
        // 获取数据
        self.getData()  // 获取数据
    }
    // 配置UI
    func setUpUI() {
        self.table.tableFooterView = UIView()
        // 注册cell
        self.table .register(UINib.init(nibName: "AVTableViewCell", bundle: nil), forCellReuseIdentifier: cellId)
    }
    // 获取数据
    func getData() {
        // 这里因为是类方法，不会产生循环引用
        AVTableManager.requestData { (flag, resList) in
            if flag {
                self.dataArray = resList!
                self.table.reloadData()
            }
        }
    }
    // tableView的代理
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.dataArray.count
    }
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        // 将UITableViewCell向下转换为AVTableViewCell
        let cell: AVTableViewCell = table.dequeueReusableCell(withIdentifier: cellId) as! AVTableViewCell
        let model = self.dataArray[indexPath.row]
        // 将数据赋值给cell
        cell.titleLabel.text = model.title
        cell.subTitleLabel.text = model.subTitle
        cell.imgView.image = UIImage.init(named: model.image!)
        return cell
    }
    func tableView(_ tableView: UITableView, heightForRowAt indexPath: IndexPath) -> CGFloat {
        return 60.5
    }
    deinit {
        print("\(self.classForCoder) 释放了。。。")
    }
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }
}
```

![当前样式](https://img-blog.csdn.net/20180827215154143?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xPTElUQTAxNjQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

d. 将数据回传到 OC 页面

这里演示代理和闭包两种方式

首先是代理，和 OC 一样，我们需要先定义一个协议用来约束遵循者完成该协议方法。

```
import Foundation
@objc protocol AViewControllerDelegate {
    // 默认是必须实现的，这里将其定义为可选类型
    @objc optional func aViewCtrlData(model: AVTableModel) -> Void
}
```

那么在控制器中，代理和闭包的声明就可以想下面那样

```
// 代理，将数据传递回其他类，可选类型
weak var delegate: AViewControllerDelegate?
// 闭包，将第一条数据传递回其他类, 该闭包和代理一样是可选类型
var returnModelBlock: ((_ model: AVTableModel) ->Void)?
```

我们在用户点击事件中将对应数据回传到 OC 页面中去

```
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        // 我们将前十项的数据使用代理方法回传，剩余的数据使用闭包方式回传
        if indexPath.row > 10 {
            // 将数据通过代理回传给 OC 页面
            let model = self.dataArray[indexPath.row]
            self.delegate?.aViewCtrlData!(model: model)
        }
        else {
            // 将数据回传给其他类
            if returnModelBlock != nil {
                returnModelBlock!(self.dataArray.first!)
            }
        }
        
        // 返回
        self.navigationController?.popViewController(animated: true)
    }
```

f. 在 OC 页面中，使用 Swift 页面中的block和代理和平常没有什么区别

```
AViewController *ctrl = [AViewController new];
ctrl.delegate = self;
// 这里并没有循环引用，因为self并没有持有ctrl，ctrl仅是被self.navigationController所持有
ctrl.returnModelBlock = ^(AVTableModel * model) {
        [self aViewCtrlDataWithModel:model];
};
[self.navigationController pushViewController:ctrl animated:YES];
```

演示：

![演示](https://img-blog.csdn.net/20180827221055912?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xPTElUQTAxNjQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

注：由于 OC 和 Swift 混编是由桥接文件实现桥接的，因此并不是你写完一个public属性在OC中就能够使用的，在使用前，你应该手动 command + B 进行编译完成桥接文件的内容。你可以使用 command + 点击TestDemo-Swift.h文件 进入桥接文件查看桥接内容。

![桥接内容](https://img-blog.csdn.net/20180827220806744?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xPTElUQTAxNjQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


## 声明

上述部分内容参考自：
[Swift 4.0 教程 - Swift编程](http://www.swift51.com/swift4.0/)
[Kenshin Cui's Blog](https://www.cnblogs.com/kenshincui/p/4717450.html) 

鸣谢
