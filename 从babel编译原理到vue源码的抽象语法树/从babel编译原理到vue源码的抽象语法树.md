![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/63d17bd7905e4e29a7dcbd0caad9bb15~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)



babel是什么
--------

*   babel是一个javascript编译器，从ECMAScript的诞生后，它便充当了代码和运行环境的翻译官，让我们随心所欲的使用js的新语法进行代码编写。
*   babel 是一个工具链
*   babel能为开发者做什么？
    1.  语法转换；
    2.  源码转换；
    3.  通过polyfill方式在目标环境中添加缺失的特性（ 通过引入第三方polyfill模块实现转换和polyfill）；
    4.  利用Babel对源码的操作，可以用来静态分析。实战中经常会用到的有：自动国际化、自动生成文档、自动埋点、js解释器等；

babel的编译过程
----------



### Parse（解析）

*   执行
*   调用parser遍历所有插件
*   找寻parserOverride编译器
*   将js代码转换成抽象语法树的解析器（简称AST）

### Transform（转换）

*   pre（进入AST时初始化操作对象）
*   visitor（在初始化操作对象以后执行visitor中的方法，使用traverse遍历AST节点，在遍历过程中执行插件）
*   post（离开AST时删除操作对象）

### Generate（生成）

*   AST转换完毕通过generate重新生成code
*   生成目标代码后，还会同时生成sourceMap相关的代码



实战 vue 源码的抽象语法树
---------------

正常vue，咱们都写的是模板语法，想要编译成正常的html语法，直接编译的话是很困难的，所以需要借助 _抽象语法树_ 来周转，也就是先得把 \_模板语法 \_变成 _AST_，完了以后再把AST变成正常的 _HTML语法_，那抽象语法树就是起到了一个中间过渡的作用，让编译工作变得简单。  
这是模板语法  
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/382827254c954cc8aaad9edabdddfa95~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)  
这是正常的HTML语法  
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/81da098fa90142d38800f81097f65da8~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)  
这是需要解析成的抽象语法树  
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0e5b85b4f4f04b55923666af4ea2f0f4~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)  
vue底层是以字符串的视角审视所有的模板，这样看的话，解析出来的AST就像是一个js版本的html对象，抽象语法树被广泛的应用在编译模板方面，就是只要有编译模板的地方就一定会用到抽象语法树。  
抽象语法树和虚拟节点有什么关系？  
**模板语法 => 抽象语法树 => 渲染函数（h函数）=> 虚拟节点 => 界面**



### 需要的算法储备



#### AST指针思想

```
【例1】试寻找字符串中，连续重复次数最多的字符‘aaaaaaaaaabbbbbbbbcccddd’
解：
var str = 'aaaaaaaaaaabbbccccccddd';
//指针
var i = 0;
var j = 1;
// 当前重复次数最多的次数
var maxRepeatCount = 0;
// 重复次数最多的字符串
var maxRepeatChar = '';
//当i还在范围内的时候，应该继续寻找
while (i <= str.length - 1) {
  //看i指向的字符和j指向的字符是不是不相同
  if (str[i] != str[j]) {
    console.log(i + '和' + j + '之间文字连续相同' + '字母' + str[i] + '它重复了' + (j - i) + '次')
    //和当前重复次数最多的进行比较
    if (j - i > maxRepeatCount) {
      // 如果当前文字重复次数（j - i）超过了此时的最大值
      // 就让它成为最大值
      maxRepeatCount = j - i;
      // 将i指针只想的字符串存为maxRepeatChar
      maxRepeatChar = str[i];
    }
    // 让指针i追上指针j
    i = j;
  }
  //不管相不相同，j永远要后移
  j++;
}

```

解题思路：  
第一时间想到的就是把所有子字符串拿出来然后比较长度和是不是都一样，但是这样的话 循环次数就比较多，而且会浪费效率，很多已经不是重复的字符串也会计算进去进行比较，所以就介绍指针法。  
指针就是下标，不是c语言中的指针，c语言中的指针可以操作内存，js中的指针就是一个下表位置。  
i:0  
j:1  
如果i和j指向的字一样，那么i不动，j后移  
如果i和j指向的字不一样，此时说明它们之间的字都是连续相同的，让i追上j，j后移。  
运行结果：

> 0和11之间文字连续相同字母a它重复了11次 11和14之间文字连续相同字母b它重复了3次 14和20之间文字连续相同字母c它重复了6次 20和23之间文字连续相同字母d它重复了3次



#### 递归思想

```
【例1】斐波那契数列，求前N项的和
解：
function fib(n) {
  return n == 0 || n == 1 ? 1 : fib(n - 1) + fib(n - 2)
}
这其实很容易，但是衍生出来一个思考，代码是否有大量重复计算？应该怎样解决重复计算的问题。
缓存思想用hasOwnProperty方法判断
这样的话总的递归次数减少了，只要命中了缓存，就直接读缓存，不会再引发下一次递归了
解：
var cache = {}
function fib(n) {
  if (cache.hasOwnProperty(n)) {
    return cache[n]
  }
  var v = n == 0 || n == 1 ? 1 : fib(n - 1) + fib(n - 2)
  cache[n] = v;
  return v;
}

```
```
【例2】将高维数组 [1, 2, [3, [4, 5], 6], 7, [8], 9] 转换为以下这个对象
{
  children: [
    { value: 1 },
    { value: 2 },
    {
      children: [
        { value: 3 },
        {
          children: [
            { value: 4 },
            { value: 5 }
          ]
        },
        { value: 6 }
      ]
    },
    { value: 7 },
    {
      children: [
        { value: 8 }
      ]
    },
    { value: 9 }
  ]
}

解：
var arr = [1, 2, 3, [4, 5, 6]]
function convert(arr) {
  var result = [];
  for (let i = 0; i < arr.length; i++) {
    if (typeof arr[i] === 'number') {
      result.push({
        value: arr[i]
      })
    } else if (Array.isArray(arr[i])) {
      result.push({
        children: convert(arr[i])
      })
    }
  }
  return result;
}

// 还有一种更秒的写法不用循环数组，大大减少了递归次数
function convert(item) {
  var result = [];
  if (typeof item === 'number') {
    return {
      value: item
    }
  } else if (Array.isArray(item)) {
    return {
      children: item.map(_item => convert(_item))
    }
  }
  return result;
}

```



#### 栈结构

我们都知道数组是一种线性结构，并且可以在任意位置插入和删除数据。但是有时为了实现某些功能，我们必须对这种任意性加以限制。而栈和队列就是常见的受限的线性结构。  
栈是一种受限的线性表，后进先出。其限制是仅允许在表的一端进行插入和删除操作，这一端被称为栈顶，另一端称为栈底。LIFO（last in first out）表示就是后进入的元素，第一个弹出栈空间。向一个栈插入新元素又称为进栈、入栈或压栈，它是把新元素放到栈顶元素的上面，使之成为新的栈顶元素。从一个栈删除元素又称作出栈或退栈，它把栈顶元素删除掉，使其相邻的元素成为新的栈顶元素

```
【例1】smartRepeat智能重复字符串问题
将 '3[abc]' 变为 'abcabcabc'
将 '3[2[a]2[b]]' 变成 'aabbaabbaabb'
将 '2[1[a]3[b]2[3[c]4[d]]]' 变成 'abbbcccddddcccddddabbbcccddddcccdddd'
function smartRepeat(templateStr) {
  // 指针下标
  let index = 0
  // 栈一，存放数字
  let stack1 = []
  // 栈二，存放需要重复的字符串
  let stack2 = []
  let tailStr = templateStr
  // 为啥遍历的次数为 length - 1 ? 因为要估计忽略最后的一个 ] 字符串
  while (index < templateStr.length - 1) {
    // 剩余没处理的字符串
    tailStr = templateStr.substring(index)
    if (/^\d+\[/.test(tailStr)) {
      // 匹配 "[" 前的数字
      let word = tailStr.match(/^(\d+)\[/)[1]
      // 转为数字类型
      let num = Number(word)
      // 入栈
      stack1.push(num)
      stack2.push('')
      index++
    } else if (/^\w+\]/.test(tailStr)) {
      // 匹配 "]" 前的需要重复的字符串
      let word = tailStr.match(/^(\w+)\]/)[1]
      // 修改栈二栈顶的字符串
      stack2[stack2.length - 1] = word
      // 让指针后移，word的长度，避免重复计算字符串
      index += word.length
    } else if (tailStr[0] === ']') {
      // 遇到 [ 字符串就需要出栈了，栈一和栈二同时出栈，栈二出栈的字符串重复栈一出栈的 数字的次数，并赋值到栈二的新栈顶上
      let times = stack1.pop()
      let word = stack2.pop()
      stack2[stack2.length - 1] += word.repeat(times)
      index++
    } else {
      index++
    }
    // console.log('tailStr', tailStr)

    // console.log('index', index)

    // console.log('stack1', stack1)

    // console.log('stack2', stack2)

  }
  // while结束之后, stack1 和 stack2 中肯定还剩余1项，若不是，则用户输入的格式错误
  if (stack1.length !== 1 || stack2.length !== 1) {
    throw new Error('输入的字符串有误，请检查')
  } else {
    return stack2[0].repeat(stack1[0])
  }
}

```

解题思路：

遍历每一个字符

*   创建两个栈
*   如果这个字符是数字，那么就把数字压入 栈1，把空字符串压入 栈2
*   如果这个字符是字母，那么此时就把 栈2 的栈顶这项改为这个字母
*   如果这个字符是 \] ，那么就将数字从 栈1 弹栈，就把 栈2 的栈顶的元素重复 栈1 弹出数字的次数，栈2 弹栈，拼接到 栈2 的新的栈顶

  
前期铺垫结束，让我们用“梦想照亮现实”，将 \*\*模板字符串 \*\*转换为 **AST树形结构**



### 手写实现AST抽象语法树

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/697c2ba1ad2c49cea8297681d090bea6~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)  
我们需要将模板字符串解析为AST  
解题思路：

1.  解析html标签中的attributes属性，栈的思维在对模板字符串进行解析的时候很有用，能对嵌套 HTML 进行快速的解析
2.  将模板字符串转换为AST树形结构，使用到算法就是栈，利用到了算法储备中栈的思路



#### 解析html标签中的attributes属性

```
export default function (attrsString) {
  let result = []
  if (!attrsString) {
    return result
  } else {
    // console.log('attrsString', attrsString)
    // 案例 'class="box" title="标题" data-type="3"'
    let isMatchQuot = false // 是否遇到引号
    // 改变了一下写法，采用双指针来记录 "" 之间走过的字符串
    let i = 0
    let j = 0

    while (j < attrsString.length) {
      // 当前指针指向的这一项
      const char = attrsString.charAt(j)
      if (char === '"') {
        // 匹配 " 字符
        isMatchQuot = !isMatchQuot
      } else if (!isMatchQuot && char === ' ') {
        // 没匹配到 " 字符，而且当前项是空格

        // 尝试拿到 i 和 j 指针之间的目标字符串
        const target = attrsString.substring(i, j).trim()
        // console.console.log(target);
        if (target) {
          result.push(target)
        }

        // 让指针i 追上 指针j
        i = j
      }
      j++
    }
    // 循环结束，还剩下最后一项属性
    result.push(attrsString.substring(i).trim())

    // filter过滤空字符 
    return result.filter(item => item).map(item => {
      const res = item.match(/^(.+)="(.+)"$/)
      return {
        name: res[1],
        value: res[2]
      }
    })
  }
}

```

思路：  
对于attrs属性，则去除两侧空格，然后同样通过指针做判断依次截取各项属性的内容，以对象形式返回键值。



#### 将模板字符串转换为AST树形结构

```
import parseAttribute from './parseAttribute'

export default function parse(templateString) {
  let index = 0
  // 未处理的字符串
  let tailStr = templateString
  // 匹配开始的html标签
  const startTagRegExp = /^\<([a-z]+[1-6]?)(\s[^\<]+)?\>/
  // 匹配结束的html标签
  const endTagRegExp = /^\<\/([a-z]+[1-6]?)\>/
  // 抓取结束标签前的文字
  const wordRegExp = /^([^\<]+)\<\/[a-z]+[1-6]?\>/

  // 准备两个栈
  let stack1 = [] // 存储匹配到的开始html标签
  let stack2 = []
  let result = null

  while (index < templateString.length - 1) {
    tailStr = templateString.substring(index)

    if (startTagRegExp.test(tailStr)) {
      // 匹配开始标签
      const res = tailStr.match(startTagRegExp)
      const startTag = res[1]
      const attrsString = res[2]
      // 开始将标记放入到栈1中
      stack1.push(startTag)
      // 将对象推入数组
      stack2.push({ tag: startTag, children: [], attrs: parseAttribute(attrsString)})
      // 得到attrsString的长度
      const attrsStringLength = attrsString ? attrsString.length : 0
      // 指针移动标签的长度 + 2 + attrsString.length
      // 为什么 +2,因为 <> 也占两个长度
      index += startTag.length + 2 + attrsStringLength
    } else if (endTagRegExp.test(tailStr)) {
      // 匹配结束标签
      const endTag = tailStr.match(endTagRegExp)[1]
      // 栈1和栈2都需要弹栈
      const pop_tag = stack1.pop()
      const pop_obj = stack2.pop()
      // 此时栈1的栈顶的元素肯定和endTag相同
      if (endTag === pop_tag) {
        if (stack2.length > 0) {
          stack2[stack2.length - 1].children.push(pop_obj)
        } else if (stack2.length === 0) {
          // 匹配到结束标签，且stack2出栈完毕，证明已经遍历结束，那么结果就是stack2最后出栈的那一项
          result = pop_obj
        }
      } else {
        throw new Error(`${pop_tag}便签没有封闭！！`)
      }
      // 指针移动标签的长度 + 3,为什么 +3,因为 </> 也占三个长度
      index += endTag.length + 3
    } else if (wordRegExp.test(tailStr)) {
      // 识别遍历到的这个字符，是不是文字，并且不能全是空字符
      const word = tailStr.match(wordRegExp)[1]
      if (!/^\s+$/.test(word)) {
        stack2[stack2.length - 1].children.push({
          text: word,
          type: '3'
        })
      }
      index += word.length
    } else {
      index++
    }
  }

  return result
}

```

思路：  
设置移动指针，判断剩余字符串是开始标签还是结束标签或文字，通过两个栈来保存标签名与容器，遇到开始标签则标签入栈1，生成容器入栈2，遇到文字填充栈2顶内容，闭合则将栈2顶内容移入栈顶前一容器。

```
import parse from './parse'

const templateString = `
<div>
    <h3 class="box" title="标题" data-id="1">模拟字符串</h3>
    <ul>
        <li>A</li>
        <li>B</li>
        <li>C</li>
    </ul>
</div>
`
console.log('输入的模板字符串', templateString);
const ast = parse(templateString)
console.log('生成的AST\n', ast)

```



总结
--

本文以上就是 Vue 中 AST抽象语法树 的主干部分，最精彩的部分莫过于是 栈 和 双指针的解决问题的思路，希望读完这篇文章，你对栈和指针的算法会有一个新的认识。