### 手写面试题一：值类型、引用类型与typeof

> 转载请注明原文链接。[原文链接](https://immortalboy.cn/article-detail/28)

手写面试题系列是我为了准备当下和以后的面试而编写的文章系列，当然对于前端小伙伴也有帮助。我建议读完之后，自己动手敲代码或者手写一遍才能更好地掌握。

#### 一、值类型、引用类型

JavaScript中的数据类型按照变量存储的值的不同可以分为：**值类型**和**引用类型**。

##### 概念

- 值类型：值类型变量存储的值本身；
- 引用类型：引用类型变量存储的是值的存储地址的引用；

##### 举例
- 值类型包括：一般也被成为基本数据类型，在最新的标准中包括 6 种：Number(整数或者浮点数)、BigInt(任意精度的整数)、String(字符串)、Boolean(布尔类型，包括 true、false)、undefined、Symbol。
- 引用类型包括：引用类型包括：对象（包含Map、Set等等）、数组、null、函数。

##### 区别
1. 值类型存储的是变量的值，引用类型存储的是变量的引用（内存地址，内存地址指向真正变量值）；
2. 两个值类型变量进行比较，存储的值相等变量就想等，两个引用类型变量进行比较时，比较的是变量的引用（内存地址），只有两个变量都指向同一块内存才会想等。

##### 代码示例

```javascript
// 以Number类型来为值类型举例
const num1 = 5;
const num2 = 5;
console.log(num1 === num2); // 输出 true

// 以数组来为引用类型距离
const arr1 = [1,2,3];
const arr2 = [1,2,3];
console.log(arr1 === arr2); // 输出 false
// 以对象来为引用类型举例
const obj1 = { a: 1 };
const obj2 = { a: 1 };
console.log(arr1 === arr2); // 输出 false
```

说到最后，如何区分值类型和引用类型呢？答案是`typeof`。

#### 二、typeof

##### 概念
`typeof`操作符返回一个字符串，表示未经计算的操作数的类型。

##### 语法
`typeof` 运算符后接操作数：
> **typeof operand**
> **typeof(operand)**
##### 参数
`operand`
一个表示对象或原始值的表达式，其类型将被返回。

以上概念来源于MDN的官方文档，可以得出以下几点结论：

1. `typeof`有两种语法，作为关键词使用和作为函数使用；
2. `typeof`接的参数可以是变量或者表达式；

这里我列出官方文档是为了向读者全面介绍`typeof`的用法。实际工作中，也可以一律使用 **typeof operand**的语法。
关键点来了，或者说要记忆的点来了，**typeof operand**的返回值有哪些，如何根据返回值区分`值类型`和`引用类型`？

首先，看一下下面表格中列出的`typeof`所有可能的返回值。

| 类型       | 示例      |   结果   |
| :-------- | :--------:| :------: |
| Undefined | typeof undeifned |  "undefined"  |
| Number | typeof 5 |  "number"  |
| String | typeof 'apple' |  "string"  |
| Boolean | typeof true |  "boolean"  |
| BigInt | typeof 9007199254740991n |  "bigint"  |
| Symbol | typeof Symbol('dog') |  "symbol"  |
| Function对象（函数） | var fn1 = ()=>{}; typeof fn1;  |  "function"  |
| 其他任何对象 |   |  "object"  |

上面的表格中，`typeof`关键词返回值为："undefined"、“number”、"string"、“boolean”、“bigint”、"symbol"的都是值类型，而返回值为"function"和"object"，都是引用类型。

注意：symbol类型也是值类型，但是 Symbol(“dog”) !== Symbol("dog")，可以理解为Symbol(“dog”)和Symbol("dog")创建后存储的值其实并不相同；因为symbol类型创建的每一个实例都是唯一且不可改变的，所以Symbol(“dog”)和Symbol(“dog”)都是唯一的，无法进行值比较。

#### 三、总结记忆

1. JavaScript中的变量类型可以分为值类型和引用类型，值类型存储的值本身，引用类型存储的是值的内存地址的引用（指向变量值实际的存储空间）；
2. JavaScript中的值类型包括：undefined、number、string、boolean、bigint、symbol，引用类型包括：对象（包括数组和其他任何对象）和函数；
3. JavaScript可以用`typeof`关键词来区分值类型和引用类型，`typeof`关键词还可以区分变量属于哪种值类型，可以区分引用类型中变量是不是函数；