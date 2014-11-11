# Chapter 2. Grammar

## Number
* Only one data type: 64bit float. Like double in java.
* NaN is a number, number value that is the result of operation that can not produce a normal result. isNaN() true if is not number, false if it is number
* Infinity
Max value

# String
* Immutable.
* `\u`, convert number to char

# Block
* In js, block will not create new scope, so variable defined in block could be
access from outer scope, if in the same function.
* Default value for function return: `undefined`


# Chapter 3. Objects
* Base type: number, string, boolean, null, undefined. These are all objects.
* An object is a container of properties, where a property has a name and a value. name可以是empty string, value可以是任意object, 除了undefined

## Retrieval
取不存在的数值，返回undefined

可以用 || 操作符来赋值默认值
```js
var middle = stooge["middle-name"] || "none"
var status = flight.status || "unknow"
```

当试图从一个undefined的object取值的时候，会抛出TypeError异常，可以通过&&操作符避免

```js
flight.equipment  // undefined
flight.equpment.model // throw "TypeError"
flight.equipment && flight.equipment.model // undefined
```

## Reference
Object都是通过引用传递，从来不会被copied.
```js
a = b = c = {} // a,b,c 指向同一个empty objct
x = "value"
var y = x; // x和y指向同一个object
// 在string的时候没有问题，如
var x = { "name": "value"}
var y = x
// y和x是指向同一个object的，当x.name的数值变化的时候，y.name的数值也会跟着变化
```



## Prototype 继承关系
Every object is linked to a prototype object from which it can inherit properties.

可以实现继承，当new object的时候，可以选择一个object作为它的prototype.
当new结束之后，会产生一个叫做prototype link的关系。

在update的时候，prototype linke没有影响。当我们改变一个子object的数值时候，
the object's prototype is not touched.
prototype会再retrieval时候使用，当子object数值还没有被改变过，lacking the property.

会继续向其父类获取数值。一直到找不到为止。返回undefined. this is called delegation.

prototype relationship is a dynamic relationship. if we add a new property to  a prototype.
will immediately be visible in all the objects that are based on that prototype.
the same, when update the value, will be show in sub class immediately.
```js
f = function() {};
function () {}
f.prototype
f {}
f.prototype = x;
Object {name: "value2"}
tmp = new f()
f {name: "value2"}
```
## Reflection
如果想看某个object是不是有property, 或者property的值，可以直接调用。
但是由于prototype link的关系，可能会访问到父类的property.

可以通过hasOwnProperty方法，true如果object有一个特定property, 不会访问prototype chain.

反射，指在不知道一个类有什么成员的时候，可以通过枚举for in来访问获取全部方法属性。
同时可以进行修改。

反射机制是指程序在运行期间能够获取自身的信息，例如一个对象能够在运行时知道自己拥有哪些方法和属性，并且可以调用这些方法和属性。在C#和Java中都提供了反射机制，能够在运行时动态判断和调用自己的属性或方法。



## Enumeration
当用`for in`遍历object时候，会显示object中的所有property name， 使用prototype chain.
用for 的时候，不会通过prototype chain

## Delete
会删除当前object的property, 不会通过prototype linkage访问父类


## Global Abatement
javascript make it easy to define global var that can hold all of the assets of your application.
unfortunately, global variables weaken the resiliency of programs and should be avoid.

minimise the use of global variables is to create a single global variable for you application.
```js
var MYAPP = {};
MYAPP.flight = {
    // define sth
};
```

可以significantly reduce the chance of bad interactions with other applications, widgets or libraries.

# Chapter 4. Function

function 有
Function.prototype -> Object.prototype

## Invocation
有四种不同类型的invocation，它们的不同点就是this变量的初始化

调用时候, (var1, var2)
不会做类型检查，如果少于变量，不足的用undefined代替；如果多于要求变量，则多余部分被自动忽略。

### The method invocation pattern
When a function is stored as a property of an object, we call it a method.
when a method is invoked, this is bound to that object

### The function invocation pattern
When a function is not the property of an object , then it is invoked as a function;
this is bound to the global object

that was a mistake in the design of the language.
when create an inner function in an object's method, the this is still the global one, will cause some unexpected mistake
```js
var value =500; //Global variable
var obj = {
    value:0,
    increment: function() {
        this.value++;
        var innerFunction=function() {
            alert(this.value);
        }
        innerFunction(); //Function invocation pattern
    }
};
obj.increment(); //Method invocation pattern
```

```js
var value =500; //Global variable
var obj = {
    value:0,
    increment: function() {
        var that =this;
        that.value++;
        var innerFunction=function() {
            alert(that.value);
        }
        innerFunction(); //Function invocation pattern
    }
};
obj.increment();
```


### The constructor invocation pattern
js is a prototypal inheritance language. means object can inherit properties directly from other object.
class-free.

if a function is invoke with the new prefix, then a new object will be created with a hidden link to the value of
the function's prototype member, and this will be bound to that new object.

Called without the new prefix, very bad thins can happen without a compile-time or runtime warning.

`this` will be bound to the global one，

### The apply invocation pattern
Apply lets us construct an array of arguments to use to invoke a function.
it also lets us choose the value of this.
the apply take two params, first is the value that should would to this, second is an array of parameters.

```js
var obj = {
    data:'Hello World'
};
var displayData=function() {
    alert(this.data);
};
displayData(); //undefined
displayData.apply(obj); //Hello World
```
Throught the apply invoke, the `this` of the displayData been assign with `obj`

https://github.com/CoffeeXu/Front-end/blob/master/JavaScript%20Function%20Invocation%20Patterns.md




## Exception
通过throw{ xxx:aaa, …} 形式调用
通过
``` js
try {

} catch(e) {

}
```
## Scope
Function scope works, the variables defined in the function is invisible ouside of the function scope.

But block has no scope. the variables defined in the block scope, is visible in the whole function scope.


## Closure - TODO

it is important to understand that the inner function has access to the actual variables of the
outer functions and not copies in order to avoid :

Avoid creating functions within a loop, it can be wasteful computationally, and it can cause confusion,


## Cascade
链式表达式

## Curry - TODO
```js
var add1 = add.curry(1);
document,written(add1(6));
// a curried add()  
// accepts partial list of arguments  
function add(x, y) {  
    var oldx = x, oldy = y;  
    if (typeof oldy === "undefined") { // partial  
        return function (newy) {  
            return oldx + newy;  
        };  
    }  
    // full application  
    return x + y;  
}  
// test  
typeof add(5); // "function"  
add(3)(4); // 7  
// create and store a new function  
var add2000 = add(2000);  
add2000(10); // 2010  
```

当你发现你调用同一个的函数，而传递的参数大部分都是一样的时候，那么这个参数就是一个很好的可以柯里化的候选函数。
你可以动态创建一个新函数，部分应用一组参数到你的函数。接着新函数会存储重复的参数(那么你就不用每次都传递)，并且会用它们去填充原始函数需要的参数列表。



通过闭包实现。原理？


## Memoization - TODO
缓存函数

# Chapter 5. Inheritance
可以通过Object.create(obj)创建对象，新对象会继承obj里面定义的属性方法

# Chapter 6. Array
初始化
```js
number = ['one','two'];
number_object = {'0':'one', '1':'two'};
```

产生相同的结果，某些key/value是相同的。但是两个类型的property有本质区别。
number 是继承了Array.prototype, 包含一些Array的有用方法，比如length
number_object是继承了Object.prototype. 方法很少。

array.length可以显示设置，设置大了，并不会导致allocate more space
设置小了，会丢失数据。

Remove element from array.
`array.spice(start, num)`

遍历：`for...in`会变量元素，但是不能保证顺序

javascript都是一维矩阵 one dimension



# Chapter 8. Methods
```js
string.toLocalLowerCase()
string.toLocalUpperCase():
```
This is make by converting this string to uppercase/lowercase using the rules for the locale.
this is primarily for the benefit of Turkish, case 'i' and 'I'

# Awful parts
## Global variables
Fit for small program. but a nightmare for big project

## Scope
没有block scope.
js used the block syntax, but does not provide block scope: a variable declared in a block is visible everywhere
in the function containing the block.

## Semicolon insertion
js会自动插入一些分号，
比如
```js
return
{
     status:true
}
```
编译器不会报错，但是会在return后面自动加上分号，程序返回值是undefined.
正确写法是
```js
return {
     status:true
}
```

`{` is placed at the end of the previous line and not at the beginning of the next line.

## Reserved words
```js
object = {box:value} // ok
object = {case:value} // illegal
object = {'case':value} // ok
object.box = value // ok
object.case = value; // illegal
object['case'] = value; // ok
```

## Unicode

## ParseInt
parse is a function that converts a string into an integer it stops when it sees a non digit, so
parseInt('16') and parseInt('16 tons') product the same result.

if the first character of the string is o, then the string is evaluated in base 8 instead of base 10.
in base 8, 8 and 9 are not digits, so parseInt('08') and parseInt('09') product o as there result.

Fortunately, parseInt can take a radix parameter, so that parseInt('08', 10) produces 8.
parseInt(string, radix);

## Floating point
浮点数计算， 比较时候会有精度误差。
要通过误差进行比较。


## NaN
```
NaN !== NaN // true
NaN === NaN // false

+ '0' // 0
+ 'oops' // NaN
```
the isFinite function is the best way of determining whether a value can be used as a number because it
rejects NaN and Infinity. Unfortunately, isFinite will attempt to convert its operand to a number, so it's not a
good test if a value is not actually a number.
```js
isNumber = function isNumber(value) {
     return (typeof value === 'number' && isFinite(value) );
}
```

## Phony arrays.
假冒的数组

Javascript does not have real arrays.

the typeof operator does not distinguish between arrays and objects.
To determine that a value is an array, you also need to consult its constructor property.
```js
if (Object.prototype.toString.apply(my_value) === '[Object Array]') {
     // my_value is truly an array
}
```
the test will give a false negative if an array was created in a different frame of window.
better this way:

```js
if (my_value && typeof my_value === 'object' &&
typeof my_value.length === 'number' &&
!(my_value.propertyIsEnumerable('length'))) {
     // is truly an array.
}
```

obj.propertyIsEnumerable
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/propertyIsEnumerable
判断property是否可以列举，在for...in中被列举出来


#Bad parts
## ++ —
不要用++, —

## bitwise operation
因为js里面只有double类型，所以会强制转换成int。
在很多语言中，这些操作符are very close to the hardware and very fast. In js,
they are very far from the hardware and very slow. Javascript is rarely used for doing bit
manipulation.


# JSLint
http://www.jslint.com/
js code quality tool
