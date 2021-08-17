## 一、什么是柯里化

> **Currying** ——只传递给函数一部分参数来进行调用，并让它返回一个函数去处理剩下的参数。

柯里化即 **Currying**，是一门编译原理层面的技术，用途是**实现多参函数**，其为实现多参函数提供了一个递归降解的实现思路——把**接受多个参数的函数**变换成**接受第一个参数的函数**，并且返回**接受剩余参数且返回结果的新函数**。

某些编程语言中（如 Haskell）就是通过柯里化技术支持多参函数这一语言特性的


## 二、JS 柯里化的实现

先来写一个实现加法的函数 `add`：

```js
function add(x, y) {
  return (x + y)
}
```

现在我们直接实现一个被柯里化的 `add` 函数，该函数名为 `curriedAdd`，则根据上面的定义，`curriedAdd` 需要满足以下条件：

```js
curriedAdd(1)(3) === 4
// true

var increment = curriedAdd(1)
increment(2) === 3
// true

var addTen = curriedAdd(10)
addTen(2) === 12
// true
```

满足以上条件的 `curriedAdd` 的函数可以用以下代码段实现：

```js
function curriedAdd(x) {
  return function (y) {
    return x + y
  }
}
```

这种实现方式并不通用，但表明了实现柯里化的一个基础——柯里化延迟求值的特性需要用到 JavaScript 中的作用域——使用作用域来保存上一次传进来的参数。

对 `curriedAdd` 进行抽象，可以得到如下函数 `currying` ：

```js
function currying(fn, ...args1) {
  return function (...args2) {
    return fn(...args1, ...args2)
  }
}

var increment = currying(add, 1)
increment(2) === 3
// true

var addTen = currying(add, 10)
addTen(2) === 12
// true
```

在此实现中，`currying` 函数的返回值其实是一个**接收剩余参数并立即返回计算值的函数**，即它的返回值并没有自动被柯里化 。
所以我们可以通过递归来将柯里化返回的函数也自动柯里化。

```js
function trueCurrying(fn, ...args) {
  if (args.length >= fn.length) {
    return fn(...args)
  }

  return function (...args2) {
    return trueCurrying(fn, ...args, ...args2)
  }
}
```

以上函数实现了柯里化的核心思想。

JavaScript 中的常用库 Lodash 中的 curry 方法，其核心思想和以上相似，都是对比**多次接受的参数总数**与**函数定义时的入参数量**，当**接受参数的数量**大于或等于**被柯里化函数的传入参数数量**时，就返回计算结果，否则返回一个继续接受参数的函数。

## 三、柯里化的应用

###  1、参数复用

固定不变的参数，实现参数复用是柯里化的主要用途之一：
```js
var increment = curriedAdd(1)
increment(2) === 3
// true

var addTen = curriedAdd(10)
addTen(2) === 12
// true
```
上面的 `increment`, `addTen` 就是进行参数复用

### 2、延迟执行

延迟执行也是柯里化的一个重要使用场景，而 bind 和箭头函数也能实现同样的功能。

一个常见的场景就是为标签绑定 onClick 事件，同时考虑为绑定的方法传递参数。

以下列出了几种常见的方法，来比较优劣：

#### ① 通过 data 属性
```jsx
<div data-name="name" onClick={handleOnClick} />
```
通过 data 属性本质只能传递**字符串**的数据，如果需要传递复杂对象，只能通过 `JSON.stringify(data)` 来传递满足 JSON 对象格式的数据，但对更加复杂的对象无法支持
    
#### ② 通过 bind 方法
```jsx
<div onClick={handleOnClick.bind(null, data)} />
```
bind 方法和以上实现的 `currying` 方法，在功能上有极大的相似，在实现上也几乎差不多。

唯一的不同就是 bind 方法需要强制绑定 context，即 bind 的第一个参数会作为原函数运行时的 this 指向，
而 `currying` 不需要此参数。

#### ③ 通过箭头函数
```jsx
<div onClick={() => handleOnClick(data)} />
```
箭头函数能够实现延迟执行，同时也不像 bind 方法必需指定 context。

但在 react 中，不建议直接在 jsx 标签内写箭头函数（直接在 jsx 标签内写业务逻辑）。
    
#### ④ 通过 currying
```jsx
<div onClick={currying(handleOnClick, data)} />
```

#### ⑤ 性能对比

<img src="https://img2020.cnblogs.com/blog/1731684/202108/1731684-20210809194450135-20505495.png" width = "800" />

通过 `jsPerf` 测试四种方式的性能，结果为：`箭头函数`\>`bind`\>`currying`\>`trueCurrying`

currying 函数相比 bind 函数，其原理相似，但是性能相差巨大，其原因是 bind 由浏览器实现，运行效率有加成。

从这个结果看柯里化性能是最差的，但另一方面就算最差的 `trueCurrying` 的实现，也能达到 50w Ops/s，说明这些性能其实影响非常小。

而 `trueCurrying` 方法中实现的自动柯里化，是另外三个方法所不具备的。

## 四、柯里化的优劣势

###  1、优势

#### ① 为了多参函数复用性
柯里化让人眼前一亮的地方在于，让人觉得函数还能这样子复用。
通过一行代码，将 add 函数转换为 increment，addTen 等。
对于柯里化的复杂实现中，以 Lodash 为列，提供了 placeholder 对多参函数的复用玩出花样：
```js
import _ from 'loadsh'

function abc(a, b, c) {
  return [a, b, c];
}

var curried = _.curry(abc)

// Curried with placeholders.
curried(1)(_, 3)(2)
// => [1, 2, 3]
```
    
#### ② 为函数式编程而生
柯里化是为函数式而生的东西，应运着有一整套函数式编程的东西，纯函数、compose、container 等等。

假如要写 Pointfree Javascript 风格的代码，那么柯里化是不可或缺的。使用 compose、container 等也需要柯里化
    

###  2、劣势

#### ① 柯里化的一些特性有其他解决方案
    
如果我们只是想提前绑定参数，那么我们有很多好几个现成的选择，bind，箭头函数等，而且性能比柯里化更好
    
#### ② 柯里化陷于函数式编程
    
上面 trueCurrying 的实现是最符合柯里化定义的，也提供了 bind，箭头函数等不具备的“新奇”特性——可持续的柯里化。

但柯里化是函数式编程的产物，它生于函数式编程，也服务于函数式编程，而 JavaScript 并非真正的函数式编程语言，相比 Haskell 等函数式编程语言，JavaScript 使用柯里化等函数式特性有额外的性能开销，也缺乏类型推导。

假如没有准备好去写函数式编程规范的代码，仅需要在 JSX 代码中提前绑定一次参数，那么 bind 或箭头函数就足够了。

## 五、总结

1、柯里化在 JavaScript 中是“低性能”的，但是这些性能在绝大多数场景，是可以忽略的。
2、柯里化的思想极大地助于提升函数的复用性。
3、柯里化生于函数式编程，也陷于函数式编程。