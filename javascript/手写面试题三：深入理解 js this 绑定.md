### 手写面试题三：深入理解 js this 绑定

> 转载请注明原文链接。[原文链接](https://immortalboy.cn/article-detail/30)

手写面试题系列是我为了准备当下和以后的面试而编写的文章系列，当然对于前端小伙伴也有帮助。我建议读完之后，自己动手敲代码或者手写一遍才能更好地掌握。

参考文献：[深入理解 js this 绑定](https://segmentfault.com/a/1190000011194676)

#### 一、什么是 this？

首先，我给出结论。

1. `this`是一个属性，它总是指向一个对象（非严格模式）；
2. 全局环境（全局上下文）中，`this`指向的是全局对象，浏览器中它指向`window`对象，node 环境中它指向`global`对象；
3. 函数环境（函数内部）中，**`this`的值取决于函数被调用的方式**；
4. this 不能在执行期间被赋值；

根据上面的结论，读者应该很好地理解了第一点和第二点，对于第三点可能抱有疑问。第三点也是本文的关键，同时也是实际工作中、面试中重点会设计到的。

#### 二、函数中的 this

首先，先思考下下面的代码的输出情况：

```
var person = {
 name: "承太郎",
 getName: function() {
  console.log(this.name);
 },
};

person.getName(); // 输出？

var person1 = person.getName();
person1(); // 输出？
```

要很好地理解`this`到底指向什么，就不得不提到`this`的绑定规则，掌握了以下 4 种绑定规则，就可以掌握函数调用时`this`的指向。

##### 1. 默认绑定

看下下面的例子：

```
function foo(){
    var a = 1 ;
    console.log(this.a);
}
var a = 10;
foo(); // 10
```

这里的例子就是典型的默认绑定，直接调用时，`this`指向了 window。所以**this.a**实际上就是**window.a**，而在函数外部执行的代码`var a = 10;`实际上也把 a 设置到了 window 对象上。所以，最后输出的结果为 10。
换个角度来说，可以认为`foo()`实际上调用的是`window.foo()`，而 foo 函数内部的`this`指向的是`window`，所以`this.a`指的是`window.a`，也就是 10。

简单理解就是：像 这种直接使用而不带任何修饰的函数调用 ，就 默认且只能 应用 默认绑定；默认绑定一般绑定到`window`上，严格模式下绑定到`undefined`。

把上面的例子改造一下，再思考一下输出会是怎样的：
_注意：把下面的代码复制到浏览器控制台运行是，请刷新页面，避免之前运行的代码造成的影响。_

```
function foo(){
    var a = 1 ;
    console.log(this.a);
}
let a = 10; // var 改成 let
foo(); // 输出？
```

相信部分读者应该能得出结论，输出的是`undefined`。我们知道运行`foo()`时，`this.a`实际访问的是`window.a`，但是运行代码`let a = 10; `时并不会把 a 赋值到`window`上（let 和 const 声明的变量不会同时赋值到 window 上，var 声明或者不声明的变量会挂在到 window 上），所以`window.a`并不存在。

换成严格模式呢？

```
"use strict";
function foo(){
    var a = 1 ;
    console.log(this.a);
}
var a = 10;
foo();
```

继续按照上面的思路分析。首先严格模式下 this 指向的是`undefined`，所以`this.a`实际上访问的是`undefined.a`，undefined 上不存在 a，所以会报错。

##### 2. 隐性 绑定

再看下这个例子：

```
function foo() {
 console.log(this.a);
}
var obj = {
    a : 10,
    foo : foo
}
foo();                // ?

obj.foo();            // ?
```

输出结果是：undefined、10。

结合上面的理解，我们知道`foo()`运行时，相当于运行了`window.foo()`，函数内部的`this`指向的是`window`，而 `window`上并没有属性`a`，所以结果为 undefined。

而`obj.foo()`呢？我们结合结果推理一下，就能知道，`obj.foo()`运行时，函数内部的`this`指向的是`obj`对象，所以`this.a`实际上指的是`obj.a`。我们可以在上面的例子中加上一些代码，验证我们的推理。

再看下这个例子：

```
function foo() {
 console.log(this.a);
}
var obj = {
    a : 10,
    foo : foo
}
foo();                // undefined

obj.foo();            // 10

obj.a = 20;
obj.foo();            // ?
```

按照猜想，函数内部的`this.a`指向的是`obj.a`，这时候给`obj.a`赋值为 20，所以`obj.foo()`输出的结果也是 20。实际运行，结果也符合我们的猜想。

上面的例子中`obj.foo()`就是`隐性绑定`。函数中`this`的指向了函数的上级。如果是类似于`a.b.c.foo()`的链式关系呢，此时的是指向它的直接上级 c 还是上级 a？

同样可以举例去求证：

```
function foo() {
 console.log(this.a);
}
var obj = {
    a : 10,
    b: {
        a: 100,
        foo: foo,
    },
    foo : foo
}
foo();                // ?

obj.foo();            // ?
obj.b.foo();          // ?
```

输出结果是：undefined、10、100。

这里，做一个简单的推理来分析下这段代码。经过前面例子的分析，我们知道`obj.foo()`运行时，函数 foo 内部的 this 指向的是`obj`，所以`this.a`就是`obj.a`，输出 10。
而`obj.b.foo()`运行时，输出的结果是 100，正好对应 obj.b 内部的变量 a 的值。也就是说，函数内部的`this.a` === obj.b.a，所以这时，`this`指向的是`obj.b`。

可以看到。对于链式关系的函数调用，函数内部的`this`指向的是它的直接上级。

结合默认绑定和隐性绑定，我们可以的出结论：

1. 函数运行时，函数内部的 this 指向的是调用它对象，也就是说 a.foo()内部的 this 指向的是 a，而 foo()内部的 this 指向的是 window；
2. 链式关系的函数调用时，函数内部的 this 指向的是函数的直接上级；

##### 3. 显示绑定

所谓显示绑定就是通过内置的`call()、apply()、bind()`方法来主动改变函数中 this 的指向。

这里，我不会去讨论`call()、apply()、bind()`的用法，具体用法参看官方文档：

1. [Function.prototype.call()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)
2. [Function.prototype.apply()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)
3. [Function.prototype.bind()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

以`call()`的用法为例子：

```
function foo(){
 console.log(this.a);
};

var obj = {
    a : 10
};
foo.call(obj);        // 10
```

上面的例子中，如果直接运行`foo()`的话，函数内部的`this`指向的是 window，使用`foo.call(obj)`改变了`foo()`函数内部`this`的指向，让 this 指向了对象`obj`，所以`this.a`等价于`obj.a`，输出结果为 10。

##### 4. new 绑定

在解释 new 绑定之前，先简单解释下 new。

我们知道`new`关键字的常用用法是：`new constructor[([arguments])]`。

**new**关键字会进行如下的操作：

1. 创建一个空的简单 JavaScript 对象（即{}）；
2. 为步骤 1 新创建的对象添加属性**proto**，将该属性链接至构造函数的原型对象 ；
3. 将步骤 1 新创建的对象作为 this 的上下文 ；
4. 如果该函数没有返回对象，则返回 this。

再来谈谈 new 绑定，也就是使用`new`关键字实例化对象后，创建的对象中的 this 指向什么？
举例说明：

```
function foo(){
    this.a = 10;
    console.log(this);
}
foo();                    // window对象
console.log(window.a);    // 10   默认绑定

var obj = new foo();      // foo{ a : 10 }  创建的新对象的默认名为函数名
                          // 然后等价于 foo { a : 10 };  var obj = foo;
console.log(obj.a);       // 10    new绑定
```

在此之前，结合一下上面的例子来解释下`new`关键字进行的几步操作。

首先`var obj = new foo(); `这行代码中，`new`关键字做了什么？

1. 创建了一个空对象{}，最终赋值给了变量`obj`（实际上是对象创建完毕之后才赋值给的变量`obj`，这样解释便于理解）；
2. 给空对象（可以理解为`obj`）添加`__proto__`（隐式原型），并把`__proto__`指向构造函数`foo()`的原型对象（foo.prototype）（也就是说，`obj.__proto__ === foo.prototype`，这意味着 obj 可以通过原型链使用 foo.prototype 上的属性）;
3. 把`obj`作为函数`foo()`中`this`的上下文，也就是说函数`foo()`中的`this`指向的是`obj`，那么`this.a`就是`obj.a`；
4. 如果函数`foo()`主动返回一个对象，那么就将第三部的`this`进行覆盖，把这个对象指向`obj`（具体请看下面的例子）；

```
function foo(){
    this.a = 10;
    console.log(this);
    return {
   		a: 100
    }
}
var obj = new foo();
console.log(obj.a); //输出：100
```

按照原型链，访问 obj 上不存在的属性时，会沿着原型链去寻找。这意味，下面的代码时成立的。

```
function foo(){
    this.a = 10;
    // return {
   	//	 a: 100
    // }
}
var obj = new foo();
console.log(obj.a); //输出：10
console.log(obj.b); // 输出： undefined
foo.prototype.b = 200;
console.log(obj.b); // 输出： 200
```

可以看到，上面的代码中在没有运行`foo.prototype.b = 200;`之前，`obj.b`的值为 undefined，在运行之后值为 200。验证了，`obj.__proto__ === foo.prototype`，obj 上没有属性 b 就会沿着原型链到 foo 的原型对象（`foo.prototype`）上寻找属性 b。

细心的同学可能会发现，我把`return {a: 100 }`给注释起来了。因为不注释的话，就相当于把`{a: 100 }`赋值给了`obj`。那么`obj`的隐式原型（`__proto__`）其实和对象`{a: 100 }`的隐式原型时一样的。我们可以换个写法验证一下。

```
const objA = {a: 100};
function foo(){
    this.a = 10;
	return objA;
}
var obj = new foo();
var obj1 = new foo();
console.log(obj === obj1); // true
```

在上面的例子中，每次`new foo()`都会返回对象`objA`，并分别赋值给了变量`obj`和`obj1`。但实际上，`obj`和`obj1`都指向了`objA`，其实是相等的。
如果函数`foo()`最终返回的是`{a: 100}`而不是`objA`，那么`obj`和`obj1`实际指向的实例化时返回的`{a: 100}`。但是，2 次实例化中创建了 2 次`{a: 100}`，所以这个时候`obj`不等于`obj1`。

如果读者可以理解上面的解释，那么`new`执行的操作就很好理解了，简单来说就是：

`new`关键字会创建一个空对象(`{}`)，然后让 this 指向这个空对象，继而给 this 的属性赋值也就是给这个空对象赋值（如上面例子中的`this.a = 10;`就是给空对象添加了一个属性 a，并给 a 赋值为 10），再然后就是把赋值后的对象的`__proto__`指向构造器的原型对象（上面的例子中就是函数`foo().prototype`）；如果函数最终返回了一个对象，那么前面创建的空对象并通过 this 给空对象赋值的操作都不会是最终返回的对象，最终返回的是函数 return 的那个对象。这意味着，上面的例子中实例化函数 foo 是没有意义的。

解释了那么多`new`关键字所做的事情，读者也一定能理解所谓的`new 绑定`就是实例化后的对象从被实例化的函数的`this`上“继承”了`this`的属性。

##### 5. this 绑定优先级

> new 绑定 > 显示绑定 > 隐式绑定 > 默认绑定

#### 三、面试题解析

**第一题**

```
var x = 10;
var obj = {
    x: 20,
    f: function(){
        console.log(this.x);        // ?
        var foo = function(){
            console.log(this.x);
            }
        foo();                      // ?
    }
};
obj.f();
```

答案是：20 10

第一个`this.x`就是上面所说的`隐式绑定`，`this`指向的就是调用函数`f()`的对象`obj`，所以`this.x`就是`obj.x`，所以输出 20。
第二个`this.x`就是`默认绑定`，第二个`this`是再内部函数 foo()调用的时候输出的，foo()的前面没有任何对象，也就是说 foo()中的`this`指向的是 window，所以`this.x`就是`window.x`，所以输出 10。

**第二题**

```
var x = 10;
var obj = {
    x: 20,
    f: function(){ console.log(this.x); }
};
var bar = obj.f;
var obj2 = {
    x: 30,
    f: obj.f
}
obj.f();
bar();
obj2.f();
```

答案：20 10 30

这一题和上面一样。`obj.f()`运行时，函数`f()`中的 this 指向了`obj`，所以`this.x`等价于`obj.x`，输出 20；
`bar()`运行时，等价于`window.bar()`，所以`this`指向 window，所以`this.x`等价于`window.x`，输出 10；
`obj2.f()`运行时，this 指向`obj2`，所以`this.x`等价于`obj2.x`，输出 30。

**第三题**

```
function foo() {
    getName = function () { console.log (1); };
    return this;
}
foo.getName = function () { console.log(2);};
foo.prototype.getName = function () { console.log(3);};
var getName = function () { console.log(4);};
function getName () { console.log(5);}

foo.getName();                // ?
getName();                    // ?
foo().getName();              // ?
getName();                    // ?
new foo.getName();            // ?
new foo().getName();          // ?
new new foo().getName();      // ?
```

这一题还是笔者在实际面试中遇到过的真题，而且当时确实不懂挂掉了。
答案是：2 4 1 1 2 3 3

`foo.getName()`很好解释，运行的第二个函数`foo.getName`，所以输出 2。

`getName()`，这里的考点在于函数声明会得到提升，所以尽管`function getName () { console.log(5);}`在`foo.prototype.getName = function () { console.log(3);};`后面，先执行的是函数声明，后执行的是函数表达式，所以`var getName = function () { console.log(4);};`会覆盖后面的函数声明，所以输出 4。

`foo().getName()`，`foo()`中的`this`就是`window`，所以`foo().getName()`就是`window.getName()`，但别忘了函数内部执行了`getName = function () { console.log (1); };`等价于`window.getName = function () { console.log (1); };`，所以这个时候全局环境中的`getName`已经被覆盖了。所以输出 1。

`getName(); `，由于上一步的代码覆盖了全局的`getName`，所以此时也输出 1。

`new foo.getName();`，我们得知道`new`关键字，最终得实例化的是一个函数，所以这里`new`关键词实例化的是函数`foo.getName()`，所以输出 2。

`new foo().getName();`，不同于上一个，这里`new`关键字先实例化了`foo()`，然后实例化后的对象调用了函数`getName()`；我们知道实例化后的对象上并没有函数`getName()`，所以就往`foo.prototype`上找，也就是`foo.prototype.getName`，所以输出 3。

`new new foo().getName()`，仔细分析一下，可以把代码拆分成两步，`var obj = new foo(); new obj.getName();`，我们知道`new foo()`返回的是一个普通对象，普通对象是不能像函数一样加上`()`运行的也不能通过`new`关键字实例化，所以最终一定是我拆分的 2 步；这里最终也输出 3，不同于上一步是运行了`getName()`,这一步是实例化了`getName()`，都是`foo.prototype.getName()`。

#### 四、箭头函数的 this 绑定

> 箭头函数不会创建自己的 this,它只会从自己的作用域链的上一层继承 this。

举例说明：

```javascript
var foo = () => {
  console.log(this);
};
foo(); // window对象
```

回顾一下我们最初的例子，

```
var person = {
 name: "承太郎",
 getName: function() {
  console.log(this.name);
 },
};

person.getName(); // 输出？

var person1 = person.getName();
person1(); // 输出？
```

现在我们知道，第一个输出的是"承太郎"，第二个是 undefined。

我们尝试着使用箭头函数来改造一下这个例子。

```
var person = {
 name: "承太郎",
 getName: function() {
	return () => {
		console.log(this.name);
	};
 },
};

person.getName()(); // 输出“承太郎”

var person1 = person.getName();
person1(); // 输出“承太郎”
```

对比一下，我们在函数`getName`内部返回了一个箭头函数，然后`person1()`就输出了预期的结果“承太郎”。结合上面的官方解释，箭头函数`会从自己的作用域链的上一层继承this`。所以，改造后的代码中，箭头函数内部的`this`会继承上一层的`person`对象上的 this，所以此时`this.name`就是`person.name`。

#### 五、总结

1. 全局环境的 this 指向 window；
2. 函数内部的 this 默认指向全局环境的 this，也就是 window；
3. a.foo()类似调用函数时，foo()内部的 this 指向 a，a.b.foo()中的 this 指向 b；
4. new 关键字实例化时，会创建一个新对象，this 会指向这个新创建的对象，所以实例化后的对象；
5. 箭头函数内部没有 this，但它会继承上一级作用域中的 this。
