# Promise 对象

标签（空格分隔）： ES6 笔记

---

Promise 是异步编程的一种解决方案，比传统的解决方案（回调函数和事件）更合理和更强大。
###优点：
1. **对象的状态不受外界影响**。`Promise`对象代表一个一步操作，有三种状态：`Pending`进行中 `Fulfilled`已完成 `Rejected`已失败。只有异步操作的结果可以决定当前是哪一种状态。
2. **一旦状态改变，就不会在变，任何时候都可以得到这个结果。**`Promise`对象的状态改变只在两种可能，从 Pending 到 Fulfilled 和从 Pending 到 Rejected。
有了 Promise 对象，就可以将一步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数。
此外 Promise 对象提供统一的借口，使得异步操作更加容易。
###缺点：
1. 无法取消 Promise，一旦新建就会立即执行，无法中途取消。
2. 如果不设置回调函数，Promise 内部跑出的错误不会反映到外部。
3. 当处于 Pending 状态时，物质得知目前进展到哪一个阶段（刚开始还是即将完成）。

###基本用法
用 Promise 实现 Ajax：
```
var getJson = function(url) {
    var promise = new Promise((resolve, reject) => {
        var handler = function() {
            if (this.readyState !== 4) {
                return
            }
            if(this.status === 200) {
                resolve(this.response)
            } else {
                reject(new Error(this.statusText)
            }
        }
        var client = new XMLHttpRequest()
        client.open("GET", url)
        client.onreadystatechange = handler
        client.responseType = 'json'
        client.setRequestHeader('Accept', 'application/json')
        client.send()
    }
    return promise
}
getJSON('post.json').then((json)=>{
    console.log('result:' + json)
}, (error)=>{
    console.error('error:', error)
})
```
`resolve`和`reject`由 JavaScript 引擎提供，不用自己部署。两者在改变 Promise 对象的状态的同时，将异步操作的结果或者报出的错误作为参数传出。

then方法可以接受两个回调函数作为参数（分别是处理 result 和 error 的回调）。第一个在 Promise 对象变为 Fulfilled 时调用，第二个（可省略）在 Promise 对象状态变为 Rejected 时调用，都接受 Promise 对象产出的值作为参数。

如果调用 resolve 函数和 reject 函数时带有参数，那么他们的参数会传递给回调参数。reject 函数的参数通常是 Error 对象的实例，表示抛出的错误；resolve 函数的参数除了正常的值以外，还可能是另一个 Promise 实例（决定前一个 Promise 对象的状态），表示异步操作的结果可能是一个值，也有可能是另一个异步操作。

###注意点
1. then() 方法会返回一个 Promise 实例
```
var outputPromise = promise.then(
    (fulfilled) => {...},
    (rejected) => {...},
)
```
这时`outputPromise`变成了受`function(fulfilled)`或者`function(rejected)`控制状态的`Promise`实例了：
若`function(fulfilled)`或者 `function(rejected)`返回一个值（字符串、数组、对象等），那么`outputPromise`的状态变为`Resolved`；
若`function(fulfilled)`或者`function(rejected)`抛出异常（throw new Error(...)），那么`outputPromise`的状态变为`Rejected`；
若`function(fulfilled)`或者`function(rejected)`返回一个`Promise`实例，那么`outputPromise`就成为这个新的`Promise`实例。

2. 当创建新的 Promise 实例时，作为参数传入的函数是会被立即执行（而不是调用 then 时才执行），只是其中的代码可以是异步代码
虽然 Promise 作为参数接收的参数是同步执行的，但是 then 方法的回调函数执行是异步的。
例子：
```
var p = new Promise((resolve, reject)=>{
    console.lig('create a promise')
    resolve('success')
})
console.log('after new Promise')
p.then((value)=>{
    console.log(value)
})
```
###实例方法
- Promise.prototype.then()
`then`方法返回一个新的`Promise`实例。链式写法调用`then` 方法时，前一个回调函数将**返回结果**作为参数，传入第二个回调函数（前一个回调函数没有用`return`返回结果时，默认返回`undefined`）
- Promise.prototype.catch()
`reject`方法的作用，等同于抛出错误。如果`Promise`状态已经变成 `Fulfilled`，在`resolve`语句之后在抛出错误时无效的，因为状态不会再改变了。
一般来说，不要在`then`方法定义`Rejected`状态的回调函数，而总是使用`catch`方法。因为`catch`可以捕获之前所有`then`方法执行中的错误，也更接近同步的`try/catch`写法。
`catch`方法返回的也是一个`Promise`对象.
- Promise.all()
`Promise.all()`方法将用于多个`Promise`实例(否则调用`Promise.resolve`方法再处理),包装成一个新的`Promise`实例,接受一个具有`Iterator`接口,且返回的每个成员都是`Promise`实例的参数(一般为数组).
```
var p = Promise.all([p1, p2, p3])]
```
只有在每个成员的状态都为`Fulfilled`时, p的状态才为`Fulfilled`, 所有返回值组成一个参数传递给p的回调函数;否则只要有一个成员状态变为`Rejected`,p的状态都会变为`Rejected`,第一个`reject`的实例的返回值被传递给 p 的回调函数.
**注意:**如果作为参数的`Promise`实例自身定义了`catch`方法，那么它一旦被`reject`,并不会触发`Promise.all()`的`catch`方法.所以不要使用`Promise.all`时给数组中的`promise`设置`catch`回调.
- Promise.race()
```
const p = Promise.race([p1, p2, p3])
```
p 的状态随第一个改变状态的成员而做相同改变，该成员返回值传递给 p 的回调函数。
**注意：**不是第一个 resolve 而是第一个改变的成员

- Promise.resolve()
将现有对象转为 Promise 对象.对象有以下四种情况
1. Promise 实例: 不作任何修改,直接返回
2. `thenable`对象(具有`then`方法的对象):转为` Promise`对象,然后立即调用其`then`方法的同时,状态变为`Resolved`
3. 不符合以上情况的任何参数:返回状态为`Fulfilled`的`Promise`对象,参数传回给回调函数
4. 不带有任何参数:返回状态为`Fulfilled`的`Promise`对象
**注意:**当`Promise.resolve()`的参数是`Promise`实例时,立即`resolve`的`Promise`对象，是在本轮“事件循环”（event loop）的结束时，而不是在下一轮“事件循环”的开始时。
例子:
```
setTimeout(function () {
  console.log('three');
}, 0); // setTimeout 放在下轮 event loop 里
Promise.resolve().then(function () {
  console.log('two') // 放在本轮 event loop 结尾
});
console.log('one') //立即执行
// one
// two
// three
```
- Promise.reject()
`Promise.reject(reason) 返回一个新的状态为`rejected`的`Promise` 实例.
**注意:** `Promise.reject()`方法的参数，会原封不动地作为`reject`的理由，变成后续方法的参数。这一点与`Promise.resolve`方法不一致。

##应用
- 加载图片: 加载完成时 `Promise` 的状态发生变化.
- `Generator` 函数与 `Promise` 的结合: 使用 `Generator` 函数管理流程,遇到异步操作的时候,通常返回一个 `Promise` 对象.

- `promise` 链式调用总结 
1. `promise`的`then`方法里面可以继续返回一个新的`promise`对象 
2. `then`方法的参数是上一个`promise`对象的`resolve`参数
3. `catch`方法的参数是其之前某个`promise`对象的`reject`参数
4. 一旦某个`then`方法里面的`promise`状态改变为了`rejected`，则`promise`方法连会跳过后面的`then`直接执行`catch`
5. `catch`方法里面依旧可以返回一个新的`promise`对象
6. 需要在某个`then`之中直接停止，那么返回一个`Promise.rejected()`，来跳过之后的`then`.