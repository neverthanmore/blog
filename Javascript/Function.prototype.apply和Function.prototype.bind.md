平时书写Javascript代码时，经常会用到`apply`和`bind`这两个方法。他们都可以动态的改变this，但是内部具体是怎么实现的，确是一无所知。

Javascript基础有一块重点就是this指针，理解了这两个方法内部如何实现，对我们理解this有很大的帮助，先看下bind和apply使用的例子：

```js
let a = {
  name: 'gbb'，
  say() {
    console.log(this.name)
  }
}
let b = {name: 'acy'}
a.say() // gbb
let fn = a.say.bind(b)
fn() // acy
a.say.apply(b) //acy
```

### Function.prototype.apply

`apply`方法的一种实现就是传入的第一个参数是一个对象，接着将fn定义在对象里面，实现this的动态传入，所以可以通过如下代码简单实现：

```js
Function.prototype.apply = function (t) {
  let args = Array.prototype.slice.call(arguments, 1, 2)
  t = t || {};
  t['fn'] = this;
  t['fn'](...args)
}

function a(){
  console.log(this.name)
}
a.apply({name: 'gbb'})  // "gbb"
```

说明：这边通过es6 spread方法实现简单的apply，通过es低版本实现的话，需要借助`new function的形式把参数传入`。

### Function.prototype.bind

MDN上对bind方法的说明如下：

```
The bind() method creates a new function that, when called, has its this keyword set to the provided value, with a given sequence of arguments preceding any provided when the new function is called.
//语法格式
fun.bind(thisArg[, arg1[, arg2[, ...]]])
```

中文解释为：

```js
bind()方法创建一个新的函数，当被调用时，它的this关键字被设置为提供的值，并且在调用新函数时提供的任何参数序列都在前面。
```

关于最后一句解释`在调用新函数时提供的任何参数序列都在前面`，这边说的任何参数就是bind方法除了第一个this，后面还能跟很多参数，用法如下：

```js
function list() {
  return Array.prototype.slice.call(arguments);
}

var list1 = list(1, 2, 3) //[1, 2, 3]
//使用bind创建一个新方法，并且传入args
var leadingThirtysevenList = list.bind(null, 37);
var list2 = leadingThirtysevenList()  //[37]
var list3 = leadingThirtysevenList(1, 2, 3) //[37, 1, 2, 3]
```

直观的例子能够看出bind除第一个参数之外后面的参数在新函数调用的时候都是在所有参数之前，这是如何实现的呢？我们看下bind的简单实现

```js
Function.prototype.bind = function (othis) {
  //获取bind除了othis之后的所有参数
  let agrs = Array.prototype.slice.call(arguments, 1)
  //this为需要绑定的函数
  let orginFn = this
  let emptyBind = function () {}
  //最终返回的新函数
  let fBind = function () {
    return orginFn.apply(
    	(
          //如果使用new操作符调用这个就不会传入othis
          this instanceof emptyBind && othis
          	? this
          	: othis
        ),
        //这句就是实现 "在调用新函数时提供的任何参数序列都在前面"
      	agrs.concat(Array.prototype.slice.call(arguments))
    )
  }
  
  // 返回的fBind继承原函数的原型
  emptyBind.prototype = origin.prototype
  fBind.prototype = new emptyBind;
  fBind.prototype.constructor = fBind;

  return fBind
}
```

总体来说，this属于Javascript基础，所以通过这两个方法的学习，能很好的掌握this的使用。

