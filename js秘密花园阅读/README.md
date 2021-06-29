# js秘密花园阅读笔记

## 对象
> 一切变量皆可当成对象使用。（undedined、null例外）
```js
false.toString() //"false"
[1, 2, 3].toString(); // '1,2,3'

function Foo(){}
Foo.bar = 1;
Foo.bar; // 1
```
> 数字使用报错。原因：2.toString() => 2.(当成浮点数了) + toString()
```js
2..toString() //"2"
(2).toString() //"2"
2 .toString() //"2"
```
> 删除对象的属性
```js
var foo = {name: "qyl"}
foo.name
foo['name'] ==> "name"可替换为属性名
delete foo.name //删除
```
> 对象的属性名可以是字符串或普通字符声明。
```js
var foo = {
  case: "I am a keyword so I must be notated as a string",
  "delete": "I am a keyword too so me" //ES5 关键字做属性名必须使用字符串声明，现在不会报错，尽量避免使用关键字。
}
```

## 原型
> 使用`prototype`实现继承。
```js
function foo() {
  this.name = "yuliwei"
}
foo.prototype = {
  age: 12
  sex: 1
}

function fn() {}
fn.prototype = new foo(); //实现继承。new fn()的原型为 new foo(),可使用new foo()的所有属性，且可使用foo.prototype的所有属性。
new fn() // {}
new fn().name // "yuliwei"
new fn().age // 12
new fn().sex // 1  存在foo的rpototype上，因此new foo()为空但是可访问name属性
```
> `for in`遍历对象时，原型链上的所有属性都将被访问。
```js
...
var s = new fn() 
console.log(s) // {}
for(var key in s) {
  console.log(key) // name, age, sex
}
```
解决：
```js
...
var s = new fn()
for(var key in s) {
  if(s.hasOwnProperty(key)) {
    console.log(key) // 拿到独有的属性
  }
}
```
> hasOwnProperty检验一个对象是否包含自定义属性而不是原型链上的属性
```js
// 修改Object.prototype
Object.prototype.bar = 1; 
var foo = {goo: undefined};

foo.bar; // 1
'bar' in foo; // true

foo.hasOwnProperty('bar'); // false
foo.hasOwnProperty('goo'); // true
```
总结：hasOwnProperty 是 JavaScript 中唯一一个处理属性但是不查找原型链的函数。
> 使用`prototype`重写内置类型的函数。
```js
// 实现forEach方法
Array.prototype.forEach = function(fn) {
  let arr = this;
  for(var i = 0; i < arr.length - 1; i++) {
    fn(arr[i], i, arr)
  }
}
```

## 函数

### this指向问题
> JavaScript 有一套完全不同于其它语言的对 this 的处理机制。 在五种不同的情况下 ，this 指向的各不相同。
全局范围内
```js
this; //window
```
函数调用
```js
foo() //window
```
方法调用
```js
test.foo() //test
```
调用构造函数
```js
let obj = new foo() //指向新创建的对象(obj)
```
改变this指向
```js
function foo(a, b, c) {}

var bar = {};
foo.apply(bar, [1, 2, 3]); // 数组将会被扩展，如下所示
foo.call(bar, 1, 2, 3); // 传递到foo的参数是：a = 1, b = 2, c = 3
foo.bind(bar,1,2,3) //本次不执行，下次调用执行，且永久改变this指向
```
#### 常见误解
一个常见的误解是 test 中的 this 将会指向 Foo 对象，实际上不是这样子的。
```js
Foo.method = function() {
    function test() {
        // this 将会被设置为全局对象（译者注：浏览器环境中也就是 window 对象）
        console.log(this)
    }
    test();
}
Foo.method() 
```
为了在 test 中获取对 Foo 对象的引用，我们需要在 method 函数内部创建一个局部变量指向 Foo 对象。
```js
Foo.method = function() {
  const that = this;
  function test() {
    // this 将会被设置为全局对象（译者注：浏览器环境中也就是 window 对象）
    console.log(that)
  }
  test();
}
Foo.method() 
```
### 工厂模式
不使用 new 关键字，达到构造函数一样的效果，函数必须显式的返回一个值。
```js
function Bar() {
  var value = 1;
  return {
    method: function() {
      return value;
    }
  }
}
Bar.prototype = {
    foo: function() {}
};

new Bar();
Bar();  //两者效果相同
```
缺点：工厂模式无法实现继承
### 作用域与命名空间
js仅函数有作用域
let const 块级作用域
>隐式全局变量 略

>局部变量 略

>变量提升 略

>名称解析排序
1. 当前作用域内是否有 var foo 的定义。
2. 函数形式参数是否有使用 foo 名称的。
3. 函数自身是否叫做 foo。
4. 回溯到上一级作用域，然后从 #1 重新开始

## 数组
> 略

## 类型
> 相等
```js
""           ==   "0"           // false
0            ==   ""            // true
0            ==   "0"           // true
false        ==   "false"       // false
false        ==   "0"           // true
false        ==   undefined     // false
false        ==   null          // false
null         ==   undefined     // true
" \t\r\n"    ==   0             // true
```
> 全等
```js
""           ===   "0"           // false
0            ===   ""            // false
0            ===   "0"           // false
false        ===   "false"       // false
false        ===   "0"           // false
false        ===   undefined     // false
false        ===   null          // false
null         ===   undefined     // false
" \t\r\n"    ===   0             // false
```
>`typeof`

它是个垃圾

> `instanceof`

 操作符用来比较两个操作数的构造函数。只有在比较自定义的对象时才有意义。 如果用来比较内置类型，将会和 typeof 操作符 一样用处不大。
```js
function Foo() {}
function Bar() {}
Bar.prototype = new Foo();

new Bar() instanceof Bar; // true
new Bar() instanceof Foo; // true

// 如果仅仅设置 Bar.prototype 为函数 Foo 本身，而不是 Foo 构造函数的一个实例
Bar.prototype = Foo;
new Bar() instanceof Foo; // false

[] instanceof Array //true
```
### 类型转换
```js
// 下面的比较结果是：true
new Number(10) == 10; // Number.toString() 返回的字符串被再次转换为数字

10 == '10';           // 字符串被转换为数字
10 == '+10 ';         // 同上
10 == '010';          // 同上 
isNaN(null) == false; // null 被转换为数字 0
                      // 0 当然不是一个 NaN（译者注：否定之否定）

// 下面的比较结果是：false
10 == 010;
10 == '-10';
```
### 内置类型的构造函数
> 内置类型（比如 Number 和 String）的构造函数在被调用时，使用或者不使用 new 的结果完全不同。
```js
new Number(10) === 10;     // False, 对象与数字的比较
Number(10) === 10;         // True, 数字与数字的比较
new Number(10) + 0 === 10; // True, 由于隐式的类型转换
```
使用内置类型 Number 作为构造函数将会创建一个新的 Number 对象， 而在不使用 new 关键字的 Number 函数更像是一个数字转换器。


## 核心
分号还是很有必要写的

## 定时器
不推荐使用setInterval,二十使用回调函数+setTimeout代替
```js
function foo(){
    // 阻塞执行 1 秒
    setTimeout(foo, 100);
}
foo();
```
