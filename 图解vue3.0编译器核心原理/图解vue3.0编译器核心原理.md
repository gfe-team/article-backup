<a name="86385379"></a>
## 概览

Vue.js作为目前最流行的前端框架之一，一些概念和原理还是需要我们前端开发人员了解与深入理解的。

Vue.js涉及的知识点很多，一些重要概念，例如：如何使用`proxy`实现响应式`effect`，虚拟`DOM`的`Diff`算法及演变过程（双端Diff算法、快速Diff算法等），渲染器原理的实现，编译器、解析器的工作原理，动态节点、静态提升等等；

现在重点采用图解步骤分析一下编译器的简单工作原理；

<a name="dc0842d1"></a>
## 编译器概念

编译器其实就是一段`JavaScript`代码程序，它将一种语言（`A`）编译成另外一种语言(`B`)，其中前者`A`通常被叫做源代码，后者`B`通常被叫做为目标代码。例如我们vue的前端项目的`.vue`文件一般即为源代码，而编译后`dist`文件里的`.js`文件即为目标代码；这个过程就被称为编译（`compile`）

<a name="3310cc3f"></a>
### 关键概念

主要涉及的概念：

- `DSL` 领域特定语言
- `AST` 抽象语法树（Abstract Syntax Tree）
- 有限状态机
- 深度优先算法

<a name="e9006bf0"></a>
## 简单流程

一个标准的编译器流程如下图所示：<br />![compiler.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1653450178366-f4a9c374-8a7a-4da9-b6ce-af22b5244714.png#clientId=u8cd2dfe7-b171-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=ub123470d&name=compiler.png&originHeight=457&originWidth=722&originalType=binary&ratio=1&rotation=0&showTitle=false&size=36347&status=done&style=none&taskId=uaa6986d3-d2e1-49b0-8eab-52e6830d145&title=)<br />`Vue.js`作为`DSL`，其编译流程会与上图有所不同，对于`Vue.js`来说，源代码就是组件的模板代码，而目标代码就是能够在浏览器（或其他平台）平台上运行的`JavaScript`代码。

<a name="40ce000d"></a>
## Vue的编译器

`Vue.js`的目标代码其实就是渲染函数（`render`函数）。概况而言，`Vue.js`编译器首先对模板进行词法分析、语法分析，然后得到模板的抽象语法树（`AST`）。随后将模板`AST`转换成`JavaScript` AST，最后再转换成`JavaScript`代码，及渲染函数。一个简单的Vue.js模板编译器的工作流如下：<br />![vue_compiler.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1653450197806-f5a7e695-28ff-493a-a9e5-645b805bcca9.png#clientId=u8cd2dfe7-b171-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u66b1bbe3&name=vue_compiler.png&originHeight=256&originWidth=722&originalType=binary&ratio=1&rotation=0&showTitle=false&size=27956&status=done&style=none&taskId=ud4f6bfdb-40ba-4545-82c9-8fc7446cc37&title=)

简单如下：<br />模板代码

```html
<div>
    <h1 id="vue">vue_compiler</h1>
</div>
```

目标的AST

```javascript
const ast = {
    type: 'Root',
    children: [
        {
            type: 'Element',
            tag: 'div',
            children: [
                {
                    type:'Element',
                    tag: 'h1',
                    props: [
                        {
                            type: 'Attribute',
                            name: 'id',
                            content: 'vue'
                        }
                    ],
                    children: [
                        {
                            type: 'Text',
                            content: 'vue_compiler'
                        }
                    ]
                }
            ]
        }
    ]
}
```

目标代码

```javascript
function render() {
    return h('div', [
        h('h1', {id: 'vue'}, 'vue_compiler')
    ])
}
```

由以上代码可以看出，`AST`其实就是一个具有层级结构的对象，模板的`AST`与模板具有相同的嵌套结构。每一颗`AST`都有一个逻辑上的根节点，其类型为`Root`，而模板中真正的根节点则作为`Root`节点的`children`存在。

观察`AST`可知：

- 不同类型的节点是通过节点的`type`属性进行区分的。
- 标签节点的子节点存储在其`children`数组中。
- 标签节点的属性节点会存储在`props`数组中。
- 不同类型的节点会使用不同的对象属性进行描述。

<a name="bde58c19"></a>
## 编译过程

<a name="35530dc0"></a>
### parse函数

`Vue.js`通过封装`parse`函数，实现对模板的词法分析和语法分析，最终得到模板的AST。`parse`函数接收模板字符串作为参数，并将解析后的`AST`作为返回值返回；

```javascript
const template = `
    <div>
        <h1>vue<h1>
    </div>
`
const templateAst = parse(template)
```

![parse.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1653450245393-8e0f1932-0b36-4843-825a-8e6dc923ed52.png#clientId=u8cd2dfe7-b171-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=uda2507b0&name=parse.png&originHeight=522&originWidth=1064&originalType=binary&ratio=1&rotation=0&showTitle=false&size=45316&status=done&style=none&taskId=u42fa411b-0a0e-4351-aa02-ab2df3449b7&title=)

解析器是如何对模板字符串进行分割的呢，此处就需要用到有限状态自动机。指的是在有限个状态之间，随着字符的输入，解析器会自动地在不同的状态之间进行切换。（实际上有限状态机是可以使用正则表达式来实现的）。

简单的状态机流程图：<br />![status.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1653450257983-2fe61bb6-4a98-43b9-b044-0312b86eba0f.png#clientId=u8cd2dfe7-b171-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=uca92fa3b&name=status.png&originHeight=625&originWidth=714&originalType=binary&ratio=1&rotation=0&showTitle=false&size=61977&status=done&style=none&taskId=ufa1952a2-93a4-4e35-a547-7f627d73c5f&title=)<br />通过有限状态机原理，可以帮助我们完成对模板的标记，最终将得到一系列`Token`（词法标记号）。

假设有如下代码：

```javascript
const template = `<div><span>Vue</span><p>Vue Compiler</p></div>` // 模板字符串

// 通过有限状态机原理实现词法分解得到三个Token
// 开始标签 <div>
// 文本节点 vue
// 结束标签 </div>

// 最终值为
const tokens = tokenize(template);
// [
//     {
//         type: 'tag', name: 'div'
//     },
//     {
//         type: 'tag', name: 'span'
//     },
//     {
//         type: 'text', name: 'Vue'
//     },
//     {
//         type: 'tagEnd', name: 'span'
//     },
//     {
//         type: 'tag', name: 'p'
//     },
//     {
//         type: 'text', name: 'Vue Compiler'
//     },
//     {
//         type: 'tagEnd', name: 'p'
//     },
//     {
//         type: 'tagEnd', name: 'div'
//     }
// ]


// 此代码需要生成的AST应为
const ast = {
    type: 'Root',
    children: [
        {
            // 实际的根节点
            type: 'Element',
            tag:: 'div',
            children: [
                {
                    type: 'Element',
                    tag:: 'span',
                    children: [
                        {
                            type: 'Text',
                            content: 'Vue'
                        }
                    ]
                },
                {
                    type: 'Element',
                    tag:: 'p',
                    children: [
                        {
                            type: 'Text',
                            content: 'Vue Compiler'
                        }
                    ]
                }
            ]
        }
    ]
}
```

以上代码生成的AST数据结构HTML结构相同，都是树状结构

![ast.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1653450282001-7642f86d-7d98-4c6b-8a9f-f3ddf24ea9de.png#clientId=u8cd2dfe7-b171-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=ua7339bfd&name=ast.png&originHeight=540&originWidth=612&originalType=binary&ratio=1&rotation=0&showTitle=false&size=31154&status=done&style=none&taskId=ucc15f8ba-900a-4add-9260-ace9c3e1caa&title=)

接下来要做的就是将生成的`tokens`转换成`AST`，在转换过程中需要维护一个`Stack`,这个栈将用来维护元素间的父子关系。每到遇到一个开始标签，就创建一个`Element`类型的AST节点，并将其压入栈内，类似的，每当遇到一个结束标签节点，我们就将当前栈顶的节点弹出。这样栈顶的节点将始终充当父节点的角色。转换过程中的所有节点，都将作为当前栈顶节点的子节点，并添加到栈顶节点的`children`属性下。流程如下图示：

最初节点只有根节点Root

![ast1.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1653450292739-4e30eae8-3134-4062-b652-c87727066a4f.png#clientId=u8cd2dfe7-b171-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=ue73040ce&name=ast1.png&originHeight=606&originWidth=1040&originalType=binary&ratio=1&rotation=0&showTitle=false&size=53559&status=done&style=none&taskId=ueb7c5afa-bc5e-4ede-8ec4-d7d5d57bd2c&title=)

当扫描到第一个标签是开始节点时，因此我们创建一个类型为`Element`的AST节点`Element（div）`，并将该节点作为当前节点的子节点。由于当前的栈顶节点是`Root`节点，所以新创建的`Element（div）`节点作为`Root`节点的子节点被添加到`AST`中，最后将新建的`Element（div）`节点压入栈中。

![ast7.jpg](https://cdn.nlark.com/yuque/0/2022/jpeg/26114887/1653450304644-a4e5b697-caee-48a8-b23a-352adbad1b90.jpeg#clientId=u8cd2dfe7-b171-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u08e7008d&name=ast7.jpg&originHeight=909&originWidth=1560&originalType=binary&ratio=1&rotation=0&showTitle=false&size=106260&status=done&style=none&taskId=ud5546229-756a-4b90-b614-b18e4d97179&title=)

由于第二个节点也是一个开始标签，所以流程同上一步，只不过当前的栈顶节点为`Element（div）`，所以将当前的节点`Element（span）`作为其子节点添加到`AST`中，最后将`Element（div）`节点压入栈中。

![ast2.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1653450313835-78e5e490-bea6-4817-80d1-4afbc7c38725.png#clientId=u8cd2dfe7-b171-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u70ddd79b&name=ast2.png&originHeight=606&originWidth=1040&originalType=binary&ratio=1&rotation=0&showTitle=false&size=70711&status=done&style=none&taskId=u81de6db4-43ee-42eb-b038-f34ff8aef11&title=)

接下来的节点是一个文本节点，所以需要创建一个`Text`类型的`AST`节点，并将其作为栈顶节点`Element（span）`的子节点加入到AST中，不同的时，当前接待不是`Element`类型，所以不需要压入栈中；

![ast3.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1653450324718-0d92f180-8e86-465d-b588-6441a3918d68.png#clientId=u8cd2dfe7-b171-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=ubcb50b6d&name=ast3.png&originHeight=606&originWidth=1040&originalType=binary&ratio=1&rotation=0&showTitle=false&size=75248&status=done&style=none&taskId=uf59d8fb9-e08e-4b2e-96f6-aa0e56183cb&title=)

下面是一个结束标签节点，根据规则，则需要将当前栈顶的节点弹出。

![ast4.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1653450335175-ef42e0af-6d2c-447e-940f-235b77ecd6de.png#clientId=u8cd2dfe7-b171-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=uca8e9068&name=ast4.png&originHeight=606&originWidth=1040&originalType=binary&ratio=1&rotation=0&showTitle=false&size=75021&status=done&style=none&taskId=u57a6c679-2f18-4d98-a65f-0bfa15a5cad&title=)

后面的流程此处就不再累述

![ast5.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1653450343557-56273373-4e56-478d-8582-f006aa7a1d6d.png#clientId=u8cd2dfe7-b171-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=uba03b244&name=ast5.png&originHeight=606&originWidth=1193&originalType=binary&ratio=1&rotation=0&showTitle=false&size=83359&status=done&style=none&taskId=u190fe0fc-8fd2-49c2-a3b4-f1e13b07b28&title=)

最终完成后的效果如下：

![ast6.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1653450351561-6ff538e2-7d05-48fe-89e7-c01cde248652.png#clientId=u8cd2dfe7-b171-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u29387d0d&name=ast6.png&originHeight=606&originWidth=1193&originalType=binary&ratio=1&rotation=0&showTitle=false&size=77293&status=done&style=none&taskId=uf0375e24-3660-4ff5-b0c1-acd9f6e6476&title=)

现在我们来实现parse函数

```javascript
function parse(str) {
    // 对模板进行词法分析，得到节点list
    const tokens = okenize(template);
    // 创建跟节点
    const root = {
        type: 'Root',
        children: []
    };
    // 创建节点栈,root节点作为栈的根节点
    const stack = [root];
    while(tokens.length) {
        const parent = stack[stack.length - 1];
        const token = tokens[0] // 从第一个点开始
        switch(t.type) {
            case 'tag':
                const eleNode = {
                    type: 'Element',
                    tag: t.name,
                    children: []
                }
                parent.children.push(eleNode);
                stack.push(eleNode);
                break;
            case 'text':
                const textNode = {
                    type: 'Text',
                    content: t.content
                }
                parent.children.push(textNode);
                break;
            case 'tagEnd':
                // 结束标签，将栈顶节点弹出栈
                stack.pop();
                break;
        }
        // 消费掉已处理的节点
        tokens.shift()
    }
    return root
}
```

以上就是一个简版的parse函数的实现，当然相对于Vue.js的源码还有很多差异，但基本原理大致相同。

下面关于`transform`函数和`generate`函数仅做了简要说明，具体实现原理敬请期待；

<a name="30a21103"></a>
### transform函数

```javascript
const template = `
    <div>
        <h1>vue<h1>
    </div>
`
const templateAst = parse(template)
const jsAst = transform(templateAst)
```

![transform.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1653450364578-073e7850-8931-4586-ab28-04b065679617.png#clientId=u8cd2dfe7-b171-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=uf00406b1&name=transform.png&originHeight=424&originWidth=1118&originalType=binary&ratio=1&rotation=0&showTitle=false&size=37320&status=done&style=none&taskId=u70bcc472-16bd-4443-8d78-0e81636f370&title=)

<a name="5c489d8a"></a>
### generate函数

```javascript
const template = `
    <div>
        <h1>vue<h1>
    </div>
`
const templateAst = parse(template)
const jsAst = transform(templateAst)
const code = generate(jsAst)
```

![generate.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1653450373172-65298b02-512e-420c-924c-0f7271bf833e.png#clientId=u8cd2dfe7-b171-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=ubccafafa&name=generate.png&originHeight=372&originWidth=1078&originalType=binary&ratio=1&rotation=0&showTitle=false&size=36723&status=done&style=none&taskId=u53ed8c95-291d-4e8a-800b-ee0b98e1e75&title=)

<a name="10476cf7"></a>
### 完整流程

以上就是`Vue`模板编译器的基本结构和工作流程，它主要有三个部分组成：

- 用来将模板字符串解析为模板`AST`的解析器（`parser`）;
- 用来将模板`AST`解析成`JavaScript AST`的转换器（`transformer`）；
- 用来根据`JavaScript AST`生成渲染函数代码的生成器（`generator`）；

本文章主要讨论了parser的基本实现原理（实际上Vue.js的真正实现要复杂的多，比如正则解析、Vue语法解析v-if、v-show、内插值{{}}等等），以及如何使用有限状态自动机来构造一个词法分析器，其过程就是状态机在不同的状态之间进行迁移的过程，并生成一个Token列表集合。然后使用Token列表集合和顶节点元素栈来构造一个可以用来描述模板的AST，最后使用模板AST来解析成JavaScript AST和渲染函数。

![compiler2.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1653450389477-4d86aa0c-b9a6-4e01-99f2-538ddc4cb681.png#clientId=u8cd2dfe7-b171-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u74658f93&name=compiler2.png&originHeight=166&originWidth=1277&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28459&status=done&style=none&taskId=uda37481c-56ee-4e13-b5a9-b570affdb32&title=)
<a name="LgXXN"></a>
## 团队介绍
高灯科技交易合规前端团队（GFE）, 隶属于高灯科技(北京)交易合规业务事业线研发部，是一个富有激情、充满创造力、坚持技术驱动全面成长的团队, 团队平均年龄27岁，有在各自领域深耕多年的大牛, 也有刚刚毕业的小牛, 我们在工程化、编码质量、性能监控、微服务、交互体验等方向积极进行探索, 追求技术驱动产品落地的宗旨，打造完善的前端技术体系。

- 愿景: 成为最值得信任、最有影响力的前端团队
- 使命: 坚持客户体验第一, 为业务创造更多可能性
- 文化: 勇于承担、深入业务、群策群力、简单开放



**Github：**[github.com/gfe-team](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fgfe-team)

**团队邮箱：**[gfe@goldentec.com](https://link.juejin.cn?target=mailto%3Agfe%40goldentec.com)

作者：GFE-绝对零度<br />著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
<a name="d17a0f0b"></a>
## 参考

Vue.js源码；

Vue.js设计与实现；
