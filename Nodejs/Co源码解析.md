### 一、为什么学习CO

nodejs属于异步编程模型，所以在实现业务的时候，需要掌握异步编程，但是在真实场景中开发人员还是习惯线性思维去编写业务。针对这种情况，nodejs在发展的数年中出现了几种方案：

- async/await
- co+generator
- Promise chains
- callback

当然node8中可以使用的async/await是最佳方案，由于目前部门使用的koa1.x框架，Koa中间件的洋葱圈模型就是基于co+generator实现的，所以需要深入学习下co的实现。简化Koa1.x callback核心实现:

```js
//co.wrap 将generator-function转换成普通的函数并且返回一个promise
var fn = co.wrap(compose(this.middleware))
if (!this.listeners('error').length) this.on('error', this.onerror);
return function handleRequest(req, res){
    res.statusCode = 404;
    var ctx = self.createContext(req, res);
    onFinished(res, ctx.onerror);
    //问题：如何catch到异常？
    fn.call(ctx).then(function handleResponse() {
      respond.call(ctx);
    }).catch(ctx.onerror);
}
```



### 二、Generator函数

Javascript-MDN上对generator的解释是：The Generator object is returned by a [generator function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*) and it conforms to both the [iterable protocol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#iterable) and the [iterator protocol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#iterator).

中文解释：Generator对象由一个生成器函数生成，它符合iterable协议和iterator协议。

我们先来实现一个最基本的迭代器：

```js
function iteratorMaker(list) {
    return {
        i: 0,
        length: list.length,
        next() {
            return this.i === this.length 
                ? {value: undefined, done: true}
                : {value: list[this.i++], done: false}
        }
    }
}
let iterator = iteratorMaker(['g', 'b', 'b'])
let r
do {
    r = iterator.next()
} while(!r.done)
// { value: 'g', done: false }
// { value: 'b', done: false }
// { value: 'b', done: false }
// { value: undefined, done: true }
```

会发现迭代器有一个next()函数，并且需要我们一步一步的执行下去，直到迭代器的实例被消耗完，es6中一直有着迭代器的存在，如下：

```js
//[Symbol.iterator]
let arr = ['g', 'b', 'b']
let arrIterator = arr[Symbol.iterator]()
arrIterator + '' // [object Array Iterator]
[...arrIterator] // ['g', 'b', 'b']
let t
do{
    t = arrIterator.next()
}while(!t.done)
// { value: 'g', done: false }
// { value: 'b', done: false }
// { value: 'b', done: false }
// { value: undefined, done: true }

```

他实现的原理和上面自己封装的方法几乎一致。熟悉了迭代器，下面我们来看一下Generator是如何使用的，如下:

```js
function* gen(val) {
    let index = 0
    const max = val.length
    while (index < max) {
        yield val[index++]
    }
}

let generator = gen(['g', 'b', 'b'])
let rt
do{
    rt = generator.next()
    console.log(rt)
}while(!rt.done)
// { value: 'g', done: false }
// { value: 'b', done: false }
// { value: 'b', done: false }
// { value: undefined, done: true }
```

可以发现Generator函数也是拥有next()方法，运行yield全部完成或者遇到return才会结束，所以上面的解释才会说，它符合iterator协议，当然它也是协程，可以控制函数的执行。

### 三、Generator函数的数据获取

在分析co源码之前还需要知道generator函数中数据如何获取, 例如`let y = yield x+2`，其实这就涉及到如何给y赋值? next方法可以获取value值，也可以传入参数，将值注入Generator函数，如下:

```js
function* generator() {
    let y = yield 'gbb'
    console.log(y)
    yield 'done'
}
let gen = generator()
gen.next()
gen.next('acy')
// { value: 'gbb', done: false }
// "acy"
// { value: 'done', done: false }
```



### 四、控制异步流程之thunk

通过do while可以控制一般generator函数流程， 但是当遇到异步操作时需要保证前一步执行完再执行下一步，显然此时do的方法无法实现。此时thunk函数就派上用场了，什么是thunk函数呢？

thunk函数是把一个多参数函数改装成一个接受回调函数的单参数函数，如下：

```js
let fs = require('fs')
let thunk = function (fn) {
    return function(...args){
        return function(callback) {
            fn.call(this, ...args, callback)
        }
    }
}
let readFileThunk = thunk(fs.readFile)

function* generator () {
  	let doc1 = yield readFileThunk('../node-thrift-test/client.js')
    console.log(doc1)
    let doc2 = yield readFileThunk('../node-thrift-test/server.js')
    console.log(doc2)
}
let g = generator()
//g.next().value 返回一个参数为callback函数
g.next().value((err, data) => {
  if (err) g.throw(err)
  g.next(data).value((err, data) => {
    if (err) g.throw(err)
    g.next(data)
  })
})
```

上面这个例子最终能输出文件的内容，`g.next().value`返回一个以callback为入参的函数，然后在callback中反复调用`g.next()`。这样的形式可以用递归来自动控制流程，如下：

```js
function run (g) {
  let gen = g()
  function next(err, data) {
    let r = gen.next(data)
    if (r.done) return
    r.value(next)
  }
  next()
}
run(generator)
```



### 五、异步流程控制之promise

除了上述所说的thunk函数可以控制异步流程，promise也是可以控制异步流程，如下

```js
let fs = require('fs')

let promiseMaker = function(path) {
    return new Promise((resolve, reject) => {
        fs.readFile(path, (err, data) => {
            resolve(data)
        })
    })
}

function* generator () {
    let doc1 = yield promiseMaker('../node-thrift-test/client.js')
    console.log(doc1.toString())
    let doc2 = yield promiseMaker('../node-thrift-test/server.js')
    console.log(doc2.toString())
}

function run(g, res) {
    let r = g.next(res)
    if (r.done) return
    r.value.then((data) => {
        run(g, data)
    })
}

run(generator())
```

promise和thunk原理差不多，都是能在异步函数执行完成之后通过递归方式不断调用next()，thunk通过callback方式，而promise通过resolve的方式。

### 六、CO源码

介绍完上面的内容，可以开始正式的head first co 了， 打开`CO`的`github`地址，上面有一句简单明了的介绍：

```
The ultimate generator based flow-control goodness for nodejs (supports thunks, promises, etc)
```

大意就是帮助generator控制流程，支持thunks和promise等。

```js
//co函数接收一个Generator函数
function co(gen) {
  var ctx = this;
  var args = slice.call(arguments, 1);
  //返回一个promise
  return new Promise(function(resolve, reject){
    // 判断gen函数，如果是function，执行函数传入args，获得generator对象
    if(typeof gen === 'function') gen = gen.apply(this, args)
    // gen对象如果不是generator执行生成的，直接把promise的状态改为resolved
    if (!gen || typeof gen.next !== 'function') return resolve(gen);
   	
    //执行onFulfilled
    onFulfilled();
    //onFulfilled函数执行gen.next(res),通过trycatch捕获异常
    function onFulfilled(res) {
      var ret;
      try {
        ret = gen.next(res);
      } catch (e) {
        return reject(e);
      }
      next(ret);
    }
   	//捕获yield promise中reject状态
    function onRejected(err) {
      var ret;
      try{
        ret = gen.throw(err) 
      }catch(e) {
        return reject(e)
      }
      next(ret)
    }
    //next函数通过promise.then的onFulfilled函数实现重复调用
    function next(ret){
      //gen.next()执行完成把promise状态改为resolved
      if(ret.done) return resolve(ret.value)
      //把ret.value的值转换为promise,toPromise里定义了一些yield支持的格式
      var value = toPromise.call(ctx, ret.value);
      //转换为promise后，实现promise.then方法，onFulfilled实现generator重复执行
      if (value && isPromise(value)) return value.then(onFulfilled, onRejected);
      return onRejected(new TypeError('You may only yield a function, promise, generator, array, 		or object, '
        + 'but the following object was passed: "' + String(ret.value) + '"'));
    }
  })
}
```

toPromise方法:

```js
//支持promise、thunk、promise数组，对象value值为promise, generator
function toPromise(obj) {
  if (!obj) return obj;
  if (isPromise(obj)) return obj;
  if (isGeneratorFunction(obj) || isGenerator(obj)) return co.call(this, obj);
  if ('function' == typeof obj) return thunkToPromise.call(this, obj);
  if (Array.isArray(obj)) return arrayToPromise.call(this, obj);
  if (isObject(obj)) return objectToPromise.call(this, obj);
  return obj;
}
```

在koa1.x中，组装中间件时，有一个写法`yield next`，而在原生中实现需要`yield* generator对象`, 这个是如何实现的呢？在toPromise方法中有一句：

```js
if (isGeneratorFunction(obj) || isGenerator(obj)) return co.call(this, obj);
```

判断是否是Generator函数或者是Generator对象，则递归调用，这就是koa1.x洋葱圈模型的实现。

在最上面koa提了一个问题，koa如何catch异常？gen.next()异常，会调用reject(e)，这样promise的catch就能捕获错误啦。或者yield之后任何一个promise调用reject方法，会调用onRejected函数，onRejected函数有这么几句：

```js
try{
  ret = gen.throw(err) 
}catch(e) {
  return reject(e)
}
```

通过gen抛出错误，trycatch捕捉后，`return reject(e)`, 此时promise的catch就能捕获错误啦





