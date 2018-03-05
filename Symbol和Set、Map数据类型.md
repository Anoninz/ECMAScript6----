# Symbol 和 Set、Map 数据类型

标签（空格分隔）： ES6 笔记

---

##Symbol
### ES6引入的新的原始数据类型，表示独一无二的值。
Symbol 值通过`Symbol`函数生成。即对象的属性名有两种类型：1.原来就有的字符串，和新增的 Symbol 类型。凡是属性名属于 Symbol 类型，就是独一无二的，不会与其他属性名产生冲突。
```
let s = Symbol()
typeof s
// "symbol"
```
生成的 Symbol 是一个原始类型的值，不是对象。因此`Symbol`函数前不能使用new命令（否则报错），Symbol 值也不能添加属性。
`Symbol`函数可以接受一个字符串作为参数，表示对 Symbol 实例的描述。但相同参数的`Symbol`函数的返回值是不相等的。
```
var s1 = Symbol('foo');
var s2 = Symbol('bar');
s1  // Symbol(foo)
s2  // Symbol(bar)
s1.toString()  // "Symbol(foo)"
s2.toString()  // "Symbol(bar)"
```
如果 Symbol 的参数是一个对象，就会调用该对象的`toString()`方法，将其转换为字符串，然后生成一个Symbol值。
```
const obj = {
  toString(){
    return 'kyon';
  }
};
const sym = Symbol(obj);
sym  // Symbol(abc)
```
Symbol 值不能与其他类型的值进行运算（会报错），但可以显式转为字符串和布尔值。
###作为属性名的 Symbol
```
var mySymbol = Symbol();
// 第一种写法
var a = {};
a[mySymbol] = 'Hello!';
// 第二种写法
var a = {
  [mySymbol]: 'Hello!'
};
// 第三种写法
var a = {};
Object.defineProperty(a, mySymbol, { value: 'Hello'! });
// 以上写法都得到同样结果
a[mySymbol]  // "Hello!"
```
Symbol 值作为对象属性名时，不能用点运算符。
```
var mySymbol = Symbol();
var a = {};
a.mySymbol = 'Hello!'
a[mySymbol]  // undefined
a['mySymbol']  // "Hello!"
```
Symbol 值作为属性名时，该属性是公开属性而非私有属性。
###魔术字符串：在代码中多次出现、与代码形成强耦合的某一个具体的字符串或数值。应尽量消除而改用含义清晰的变量代替。
Symbol可用于消除魔术字符串。
###属性名的遍历
- Symbol作为属性名，该属性不会出现在`for...in`、`for...of`循环中，也不会被`Object.keys()`、`Object.getOwnPropertyNames()`、`JSON.stringify()`返回。这个特性可被用于为对象定义一些非私有的、但又希望只用于内部的方法。
- 需要显示用作属性名的 Symbol 值时
    - `Object.getOwnPropertySymbols`方法，返回一个数组，成员是当前对象的所有用作属性名的 Symbol 值
    - `Reflect.ownKeys(obj)`方法可以返回所有类型的键名，包括常规键名和 Symbol 键名。
```
let obj = {
  [Symbol('my_key')]: 1,
  enum: 2,
  nonEnum: 3
};
Reflect.ownKeys(obj)
// ["enum", "nonEnum", Symbol(my_key)]
```
###Symbol.for()
接受一个字符串作为参数，然后搜索有没有以该参数作为名称的参数值，如果有，就返回这个 Symbol 值，否则就新建并返回一个以该字符串作为名称的 Symbol 值。
```
var s1 = Symbol.for('foo');
var s2 = Symbol.for('foo');
s1 === s2;  // true
```
Symbol.for()与Symbol()这两种写法，都会生成新的Symbol。它们的区别是，前者会被登记在全局环境中供搜索，后者不会。
```
Symbol.for("bar") === Symbol.for("bar")
// true
Symbol("bar") === Symbol("bar")
// false
```
###Symbol.keyFor()
返回一个已登记(使用`Symbol.for()`登记)的 Symbol 类型值的 key
```
var s1 = Symbol.for("foo");
Symbol.keyFor(s1);  // "foo"
var s2 = Symbol("foo")
Symbol.keyFor(s2);  // undefined
```
###实例：模块的 Singleton 模式（单个物件模式）
Singleton 模式指的是调用一个类，任何时候返回的都是同一个实例。
Node 中，模块文件可以看作一个类。可以使用 Symbol，通过把实例放到顶层对象global来实现 Singleton 模式。
```
// mod.js
const FOO_KEY = Symbol.for('foo');
function A() {
  this.foo = 'hello';
}
if (!global[FOO_KEY]) {
  global[FOO_KEY] = new A();
}
module.exports = global[FOO_KEY];

// 使用
global[Symbol.for('foo')] = { foo: 'world' };
const a = require('./mod.js');
```
###内置的 Symbol 值
- Symbol.hasInstance 
- Symbol.isConcatSpreadable
- Symbol.species
- Symbol.match
- Symbol.replace
- Symbol.search
- Symbol.split
- Symbol.iterator
- Symbol.toPrimitive
- Symbol.toStringTag
- Symbol.unscopables


## Set
###基本用法：Set 结构不会添加重复的值
```
const s = new Set();
[2, 3, 5, 4, 5, 2, 2].forEach(x => s.add(x));
for(let i of s){
  console.log(i);
}
// 2 3 5 4
```
可以用来对数组去重
```
[...new Set(array)]
```
Set 内部判断两个值是否不同，使用的算法叫做“Same-value equality”，类似精确相等运算符（===），主要的区别是NaN等于自身（===认为NaN不等于自身）。另外，两个对象总是不相等的。
###Set 的属性和方法
- 原型属性
    - Set.prototype.constructor: 构造函数，默认就是 `Set` 函数。
    - Set.prototype.size: 返回 Set 实例的成员总数
- 实例方法-常用
    - add(value) 添加某个值，然后返回 Set 本身。
    - delete(value) 删除某个值，返回一个表示是否成功删除的布尔值
    - has(value) 返回一个表示 Set 内是否包含 value 的布尔值
    - clear() 清除所有成员，没有返回值。
- 实例方法-遍历（Set 结构中，键名和键值是同一个值）（Set的遍历顺序就是插入顺序）
    - keys() 返回键名的遍历器
    - values() 返回键名的遍历器
    - entries() 返回键值对的遍历器
    - forEach(f) 使用回调函数遍历每个成员
###应用
扩展运算符（`...`）内部使用for...of循环，所以也可以用于 Set 结构。结合扩展运算符可以实现很多功能
```
// 对 array 去重
let unique = [...new Set(array)]
// 对 Set 使用数组的 map 与 filter 方法
let set = new Set([1, 2, 3]);
set = new Set([...set].map(x => x * 2));

let set = new Set([1, 2, 3, 4, 5]);
set = new Set([...set].filter(x => (x % 2) == 0));

// 取数组的并集、交集、差集
let a = new Set([1, 2, 3]);
let b = new Set([4, 3, 2]);
// 并集
let union = new Set([...a, ...b]);
// Set {1, 2, 3, 4}
// 交集
let intersect = new Set([...a].filter(x => b.has(x)));
// set {2, 3}
// 差集
let difference = new Set([...a].filter(x => !b.has(x)));
// Set {1}
```
##WeekSet
###定义
WeakSet 与 Set 类似，但成员只能是对象，并且都是弱引用，即垃圾回收机制不考虑 WeakSet 对该对象的引用。如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中。
ES6 规定，WeakSet 不可遍历。
###语法
有add、delete、has方法；没有size属性。
###用处
是储存 DOM 节点，而不用担心这些节点从文档移除时，会引发内存泄漏。

##Map
###定义
为了解决 Object 的只能用字符串当作键名的问题，ES 6 提供了 Map 数据结构。各种类型的值（包括对象）都可以当作键。
###常用属性与方法
- map.size 返回 Map 结构的成员总数
- set(key, value) 设置键名key 对应的键值为 value，然后返回整个 Map 结构。如果原 key 已有值，则更新键值
- get(key) 读取键名 key 对应的键值，没有则返回 undefined
- has(key) 返回一个布尔值，表示某个键是否在当前 Map 对象中
- delete(key) 删除某个键，返回 true，如果删除失败，返回 false
- clear() 清除所有成员，没有返回值
### 遍历方法
- keys()
- values()
- entries()
- map.forEach() 
###与其他数据结构的互相转换
- Map 转为数组：使用扩展运算符(`...`)
- 数组转为 Map：使用 Map 构造函数
- Map 转为对象：如果所有 Map 的 key 都是字符串，可以转为对象
```
function strMapToObj(strMap){
  let obj = Object.create(null);
  for(let [k, v] of strMap){
    obj[k] = v;
  }
  return obj;
}
const myMap = new Map().set('yes', true).set('no', false);
strMapToObj(myMap);
// { yes: true, no: false }
```
- 对象转为 Map
```
function objToStrMap(obj){
  let strMap = new Map();
  for(let k of Object.keys(obj)){
    strMap.set(k, obj[k]);
  }
  return strMap;
}
```
- Map 转为 JSON：
```
// 情况一：Map 的键名都是字符串
// 可以转换为对象 JSON
function strMapToJson(strMap){
  return JSON.stringify(strMapToObj(strMap));
}
// 情况二：Map 的键名有非字符串
// 可以转换为数组 JSON
function mapToArrayJSON(map){
  return JSON.stringify([...map]);
}
```
##WeakMap
###含义：
与 Map 类似，用于生成键值对。但只接受对象作为键名（null除外），且键名所指向的对象不计入垃圾回收机制。
###专用场合
它的键所对应的对象可能会在将来消失。WeakMap结构有助于防止内存泄漏。
###语法：
没有遍历操作，无法清空。只有四个方法可用：get()、set()、has()、delete()。
###用处：DOM 节点作为键名。
```
// 一个例子
let myElement = document.getElementById('logo');
let myWeakmap = new WeakMap();
myWeakmap.set(myElement, {timesClicked: 0});
myElement.addEventListener('click', function(){
  let logoData = myWeakmap.get(myElement);
  logoData.timesClicked++;
}, false);
// 一旦这个DOM节点删除，该状态就会自动消失，不存在内存泄漏风险
```
注册监听事件的listener对象，就很适合用 WeakMap 实现。
```
const listener = new WeakMap();
listener.set(element1, handler1);
listener.set(element2, handler2);
element1.addEventListener('click', listener.get(element1), false);
element2.addEventListener('click', listener.get(element2), false);
// 一旦DOM对象消失，跟它绑定的监听函数也会自动消失
```
WeakMap 的另一个用处是部署私有属性。
```
const _counter = new WeakMap();
const _action = new WeakMap();

class Countdown {
  constructor(counter, action) {
    _counter.set(this, counter);
    _action.set(this, action);
  }
  dec() {
    let counter = _counter.get(this);
    if (counter < 1) return;
    counter--;
    _counter.set(this, counter);
    if (counter === 0) {
      _action.get(this)();
    }
  }
}

const c = new Countdown(2, () => console.log('DONE'));

c.dec()
c.dec()
// DONE
```

