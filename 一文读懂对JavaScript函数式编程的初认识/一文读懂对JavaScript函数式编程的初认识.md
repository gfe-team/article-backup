![222](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92f808c167094496b90a8a010e393e96~tplv-k3u1fbpfcp-zoom-1.image)

## 背景

&ensp;&ensp;&ensp;&ensp; 函数式编程可以说是非常古老的编程方式，但是近几年变成了一个非常热门的话题。不管是Google力推的Go、学术派的Scala与Haskell，还是Lisp的新语言Clojure，这些新的函数式编程语言越来越受到人们的关注。函数式编程思想对前端的影响很大，Angular、React、Vue等热门框架一直在不断通过该思想来解决问题。

&ensp;&ensp;&ensp;&ensp; 函数式编程作为一种高阶编程范式，更接近于数学和代数的一种编程范式，与面向对象的开发理念和思维模式截然不同，深入理解这种差异性，是程序员进阶的必经之路。

## 编程范式

&ensp;&ensp;&ensp;&ensp; 编程范式（Programming Paradigm）是编程语言领域的模式风格，体现了开发者设计编程语言时的考量，也影响着程序员使用相应语言进行编程设计的风格。大体分为两大类，具体内容如下图所示：

![截屏2022-04-15 上午9.44.26](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/11fee527e7454c398048a2c4758aee4b~tplv-k3u1fbpfcp-zoom-1.image)

## 函数式概念与思维

&ensp;&ensp;&ensp;&ensp; 函数式编程(Functional Programming)是基于λ演算（Lambda Calculus）的一种语言模式，它的实现基于λ演算和更具体的α-等价、β-归约等设定 。这是一个较官方的解释，大家不要被这种概念吓到，很有可能你已经在日常开发中使用了大量的函数式编程概念和工具。如越来越函数式的ES6，新的规范有非常多的新特性，其中不少借鉴其他函数式语言的特性，给JavaScript语言添加了不少函数式的新特性。箭头函数就是ES6发布的一个新特性，箭头函数也被叫做肥箭头（Fat Arrow），大致是借鉴自CoffeeScript或者Scala语言。箭头函数是提供词法作用域的匿名函数。

***函数式编程思维的目标：程序执行时，应该把程序对结果以外的数据的影响控制到最小***。

### 函数式编程的特点

1.  声明式（Declarative）
2.  纯函数（Pure Function）

-   函数的执行过程完全由输入参数决定，不会受除参数之外的任何数据影响。
-   函数不会修改任何外部状态，比如修改全局变量或传入的参数对象。

3.  数据不可变性（Immutability）

    当我们需要数据状态发生改变时，保持原有数据不变，产生一个新的数据来体现这种变化。不可改变的数据就是Immutable数据，一旦产生，可以肯定它的值永远不会变，这非常有利于代码的理解。

下面用一段对比代码解释**命令式编程与函数式编程**：

```JavaScript
// 计算传入数据乘以2

// 命令式编程
function double(arr) {
  const results = []
  for (let i = 0; i < arr.length; i++){
    results.push(arr[i] * 2)
  }
  return results
}

console.log(double([1, 2, 3]));// [2, 4, 6]

// 函数式编程
function double(arr) {
  return arr.map(item => item * 2);
}

const oneArray = [1, 2, 3];
const anotherArray = double(oneArray);

console.log(oneArray); // [1, 2, 3]
console.log(anotherArray);// [2, 4, 6]
```

### 函数是一等公民

数字在JavaScript里就是一等公民，同样作为一等公民的函数就会拥有类似数字的性质。

1.  函数与数字一样可以存储为变量

```JavaScript
let one = function() { return 1 };
```

2.  函数与数字一样可以存储为数组的一个元素

```JavaScript
let ones = [1, function() { return 1 }];
```

3.  函数与数字一样可以被传递给另一个函数

```JavaScript
function numAdd(n, f) { return n + f()};
numAdd(1, function() { return 1}); // 2
```

4.  函数与数字一样可以被另一个函数返回

```JavaScript
return 1;
return function() { return 1 };
```

最后两点其实就是“高阶”函数的定义；一个高阶函数应该可以至少执行一项，以一个函数作为参数或者返回一个函数作为结果。

### 高阶函数（High Order Function）

&ensp;&ensp;&ensp;&ensp; 高阶函数，通俗来说，就是以其他函数为参数的函数，返回其他函数的函数。我们称函数的嵌套高阶调用为高阶函数，高阶函数可以说是编程语言便捷践行函数式的基础。比如在React中我们会遇到的高阶组件HOC。

**以数字添加千分位符号为demo的代码如下：**

```JavaScript
const addThousandSeprator = (strOrNum) => {
    return parseFloat(strOrNum).toString().split('.').map((x,idx) => {
        if(!idx) {
            return x.split('')
                    .reverse()
                    .map((xx,idxx) => (idxx && !(idxx % 3)) ? (xx + ',') : xx )
                    .reverse()
                    .join('')
        } else {
            return x;
        }
    }).join('.')
}
```

#### 高阶函数应用之柯里化（Currying）

&ensp;&ensp;&ensp;&ensp; 柯里化函数为每一个逻辑参数返回一个新的函数，会逐渐返回已配置的函数，直到所有的参数用完。

```JavaScript
function curry(fun) {
    return function(arg) {
        return fun(arg)
    }
}
​
const arr = ['1', '2', '3', '4'].map(curry(parseInt));
console.log(arr) // [ 1, 2, 3, 4 ]
```

&ensp;&ensp;&ensp;&ensp; 使用柯里化比较容易产生流利的函数式API。在Haskell编程语言中，函数式默认柯里化。但在JavaScript中，函数式API的设计必须利用柯里化，而且必须文档化。

### 递归

&ensp;&ensp;&ensp;&ensp; 程序调用自身的编程技巧称为递归（ recursion）。递归作为一种算法在程序设计语言中广泛应用。 递归是一种解决过程堆叠的方法，在运行时承担了更多的工作。递归的能力在于用有限的语句来定义对象的无限集合。一般来说，递归需要有边界条件、递归前进段和递归返回段。当边界条件不满足时，递归前进；当边界条件满足时，递归返回。

&ensp;&ensp;&ensp;&ensp; 说起递归，不得不谈起尾递归。早期的浏览器引擎是不支持尾递归，所以当我们计算经典的斐波那契数列或进行其他递归操作时，可能会触发堆栈调用超限的提醒。如果每次递归尾部返回的内容都是一个待计算的表达式，那么运行时的内存栈中会一直压入等待计算的变量和环境，这就是产生超限的根本原因。而如果我们使用新的递归方法，若运行环境支持优化，则立即释放被替换的函数负载。

```JavaScript
// 递归：将外层调用保存在内存堆栈中
const factorialFn = (n) =>  {
  if (n <= 1) {
    return 1;
  } else {
      return n + factorialFn(n - 1);
  }
}
console.log('factorialFn:  ', factorialFn(30))
​
// 返回函数调用；尾递归优化
const factorialFun = (n, acc) => {
    if(n <= 1) {
        return acc;
    } else {
        return factorialFun(n - 1, n + acc)
    }
}
​
console.log('factorialFun: ', factorialFun(30, 1))
```

**运行结果如下：**

![截屏2022-05-18 下午6.26.53](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c093558a1c3f420da40462ceef341fdd~tplv-k3u1fbpfcp-zoom-1.image)

### 基于流的编程

&ensp;&ensp;&ensp;&ensp; 在前端领域中，「流」的经典代表之一「RxJS」。

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  在Rx官网<https://reactivex.io/> 上，有一段介绍文字：

&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;  An API for asynchronous programming with observable streams.

&ensp;&ensp;&ensp;&ensp; 翻译过来就是：Rx是一套通过可监听流来做异步编程的API。老实说，这句描述并没有把概念解释清楚，所以在下面我们就用普通的语言来解释Rx。

#### RxJS初认识

**RxJS是Reactive Extension模式的JavaScript语言实现**。

&ensp;&ensp;&ensp;&ensp; RxJS是一个使用可观察序列组成异步和基于事件的程序库。它提供了一种核心类型，Observable，广播类型（Observer，Schedulers，Subjects）和操作符（map，filter，reduce等），允许将异步事件作为集合处理。

&ensp;&ensp;&ensp;&ensp; RxJS的运行就是Observable和Observer之间的互动游戏。

&ensp;&ensp;&ensp;&ensp; RxJS中的数据流就是Observable对象，Observable实现了两种设计模式：**观察者模式（Observer Pattern）、迭代器模式（Iterator Pattern）** 。

&ensp;&ensp;&ensp;&ensp; **Observable和Observer的关系是观察者模式和迭代器模式的结合，通过Observable对象的subscribe函数，可以让一个Observer对象订阅某个Observable对象的推送内容，可以通过unsubscribe函数退订内容。**

#### RxJS核心概念

**Observable**：可观察者对象，表示可以调用的未来值或事件集合的方法。

**Observer**： 观察者，是一组回调函数，处理Observable提供的值。

![image-20220518105003037](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23ed1341935c48b9b07d6894ef9f8683~tplv-k3u1fbpfcp-zoom-1.image)

```JavaScript
​
/**
 * Observable对象（source$）就是一个发布者，通过Observable对象的subscribe函数，把发布者和观察者连接起来
 * 扮演观察者的是console.log，不管传入什么“事件”，它只管把“事件”输出到console上
 */
const source$ = of(1, 2, 3);  // 发布者
source$.subscribe(console.log); // 观察者
```

**这段代码输出结果如下：**

![截屏2022-05-18 下午6.33.41](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a1593de514af4c43af0d35e95cf6aa2d~tplv-k3u1fbpfcp-zoom-1.image)

**Subscription**：订阅关系，表示Observable执行，主要用于取消执行。

```JavaScript
import {Observable} from 'rxjs/Observable';
​
const onSubscribe = observer => {
  let number = 1;
  const handle = setInterval(() => {
    console.log(`onSubscirbe: ${number}`)
    observer.next(number++);
  }, 1000);
​
  return {
    unsubscribe: () => {
      clearInterval(handle);
    }
  };
};
​
const source$ = new Observable(onSubscribe);
const subscription = source$.subscribe(item => console.log(`第${item}次调用`));
​
setTimeout(() => {
  subscription.unsubscribe();
}, 5500);
```

**这段代码输出结果如下：**

![截屏2022-05-18 下午6.32.41](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff00d3624fcc4c469aa1577052d0a049~tplv-k3u1fbpfcp-zoom-1.image)

**该行代码被注释后 clearInterval(handle)，代码输入结果如下：**

![截屏2022-05-18 上午11.30.31](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b593b8d9be6b433e8bf495de20203143~tplv-k3u1fbpfcp-zoom-1.image)

当unsubscribe函数中的clearInterval被注释掉后，也就是setInterval不被打断，setInterval的函数参数中输出当前number，修改之后的程序会不断的输出 onSubscirbe： n。

**由此可见，Observable产生的事件，只有Observer通过subscribe订阅之后才会收到，在unsubscribe之后就不会再收到**。

&ensp;&ensp;&ensp;&ensp; **Operators**：操作符，纯粹的函数，一个操作符是返回一个Observable对象的函数。

&ensp;&ensp;&ensp;&ensp; 说起操作符，不得不说的就是**弹珠图**，弹珠图可以通过动画很直白的向我们展示操作过程，动态：[​ https://reactive.how/rxjs/](https://rxmarbles.com/#interval) ， 静态：<https://rxmarbles.com/#interval>。

&ensp;&ensp;&ensp;&ensp; 在所有操作符中最容易理解的可能就是**map**和**filter**，因为JavaScript的数组对象有两个同名的函数map和filter。

**JavaScript写法：**

```JavaScript
const source = [1,2,3,4,5,6];
const result = source.filer(x => x % 2 === 0).map(x => x * 2);
console.log(result);
```

**RxJS写法：**

```JavaScript
const result$ = of(1,2,3,4,5,6).filter(x => x % 2 === 0).map(x => x * 2);
result$.subscribe(console.log);
```

**按功能分类，大致可以分为9大类：**

-   创建类（creation）
-   转化类（transformation）
-   过滤类（filtering）
-   合并类（conbination）
-   多播类（multicasting）
-   错误处理类（error Handling）
-   辅助工作类（untility）
-   条件分支类（conditional & boolean）
-   数据和合计类（mathmatical & aggregate）

**Subject**：主题，相当于EventEmitter，将值或事件广播到多个Observer的唯一方法。

```JavaScript
import {Observable} from 'rxjs/Observable';
import {Subject} from 'rxjs/Subject';
import 'rxjs/add/observable/interval';
import 'rxjs/add/operator/take';
​
const tick$ = Observable.interval(1000).take(3);
const subject = new Subject();
tick$.subscribe(subject);
​
subject.subscribe(value => console.log('observer 1: ' + value));
setTimeout(() => {
  subject.subscribe(value => console.log('observer 2: ' + value));
}, 1500);
```

这段代码的执行结果如下：

![截屏2022-05-18 下午6.28.24](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5d888460f62245e4aa00a9dfd0432791~tplv-k3u1fbpfcp-zoom-1.image)

以上代码可以看出，**Subject兼具Observable和Observer的性质**，就像有两副面孔，可以左右逢源。

日常常用场景如浏览器中鼠标的移动事件、点击事件，浏览器的滚动事件，来自WebSocket的推送消息，还有Node.js支持的EventEmitter对象消息，及微服务系统中主应用与各个子应用之间的通信等。

**Scheduler**：控制并发的集中调度器，使我们能够协调发生在setTimeout或其他的事件。

**Scheduler实例：**

- undefined/null：也就是不指定Scheduler，代表同步执行的Scheduler。
- asap：尽快执行的Scheduler。
- async：利用setInterval实现的Scheduler，用于基于时间吐出数据的场景。
- queue：利用队列实现的Scheduler，用于迭代一个大的集合的场景。
- animationFrame：用于动画场景的S cheduler。

&ensp;&ensp;&ensp;&ensp; RxJS默认选择Scheduler的原则是：尽量减少并发运行。所以，对于range，就选择undefined，指的是同步执行的Scheduler；对于很大的数据，就选择queue；对于时间相关的操作符比如interval，就选择async。

```JavaScript
import {Observable} from 'rxjs/Observable';
import 'rxjs/add/observable/range';
import {asap} from 'rxjs/scheduler/asap';
​
const source$ = Observable.range(1, 3, asap);
​
console.log('before subscribe');
source$.subscribe(
  value => console.log('data: ', value),
  error => console.log('error: ', error),
  () => console.log('complete')
);
console.log('after subscribe');
```

这段代码的执行结果如下：

![截屏2022-05-18 下午6.28.24](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f71a89812a9f46449dff8d949016e561~tplv-k3u1fbpfcp-zoom-1.image)

## 函数式在前端的积极作用

&ensp;&ensp;&ensp;&ensp; web开发时，我们会在服务端管理大量的系统状态和系统数据，可以看到随着前端工作流逐渐增多，事件和远程状态响应都会变得错综复杂。对于查看一个多于10个页面或组件复杂的项目代码时，我们会发现相比于后端，很难通过前端代码读懂整个业务链路。如果我们将核心代码更换成较为合理的函数式逻辑，或者使用函数式工具和规范对已有逻辑进行归纳，就可以明显提高代码的可读性和代码运行时的可调试性，这也是对历史代码进行升级、改造的方法之一。

&ensp;&ensp;&ensp;&ensp; 前端函数式的初衷是我们希望能更好、更快、更强地解决开发过程中遇到的问题。与其等待后续的治理，不如在日常开发中进行合理的规划，养成良好的开发习惯。