![基于BPMN2.0的业务流程引擎.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/78c674e8f8a44be8bae8946f9fbee154~tplv-k3u1fbpfcp-zoom-1.image)

## BPMN 规范
### BPMN 2.0 是什么？

&ensp;&ensp;&ensp;&ensp; BPMN是英文Business Process Model and Notation的缩写,即业务流程模型注解, 是业务流程模型的一种标准图形注解。这个标准是由对象管理组（Object Management Group - OMG）维护的.与任何特定商业组织或工具是没有关系，无需为此付费.

&ensp;&ensp;&ensp;&ensp; BPMN 2.0即BPMN规范的 2.0 版本，当前版本是比较稳定的一个版本。允许在BPMN的图形和元素中添加精确的技术细节， 同时制定BPMN元素的执行语法。 通过使用XML语言来指定业务流程的可执行语法， BPMN规范已经演变为业务流程的语言，可以执行在任何兼容BPMN 2.0的流程引擎中, 比如:Activiti、jBPM、Bonita、Camunda、ActiveVOS(商业)，同时依然可以使用强大的图形注解。
### 发展历史

BPMN 标准发展版本历史如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f117e412d284f1fa565a9feb0ce0c7c~tplv-k3u1fbpfcp-zoom-1.image "BPMN发展历史图鉴")

> &ensp;&ensp;&ensp;&ensp; BPMN1.x被大多数的建模工具和BPMS厂商所支持。但是， BPMN1.x只是一些建模符号，不支持元模型，不支持存储和交换，也不支持执行。那么围绕着BPMN1.x的存储、交换和执行，必然会产生新的竞争，这次的主角换成了XPDL、BPEL和BPDM。
>
> &ensp;&ensp;&ensp;&ensp; XPDL作为WfMC(工作流管理联盟)提出的流程定义语言规范，本身就是一个元模型，可以存储，并且具备执行语义。如今有超过80个的不同公司的产品使用XPDL来交换流程定义，同时也有一些厂商在自己提供的BPMN工具中使用了XPDL作为交换和存储格式。
>
> &ensp;&ensp;&ensp;&ensp; 为了抗衡XPDL，OASIS组织(包括几个大的平台公司，Microsoft、 BEA、 IBM、 SAP 、Sun、Oracle)开发了BPEL规范。但BPMN到BPEL的转换存在着先天上的缺陷，原因是BPMN是基于图的，而BPEL是基于块的。这个缺陷导致有些BPMN建模的流程无法映射到BPEL，两者的双向工程更是存在问题。这个缺陷成为人们反复诟病的对象。许多支持BPEL的产品为了解决这一问题，不得不在用户建模时做出种种限制，让用户绘制不出无法转换的模型。
>
> &ensp;&ensp;&ensp;&ensp; 而BPDM（业务流程定义元模型）则是OMG组织自己提出来解决BPMN存储和交换问题的规范。于2007年7月形成初稿，2008年7月被OMG最终采用。BPDM是一个标准的概念定义，用来表达业务流程模型。元模型定义了用来交换的概念，关系和场景，可以使得不同的建模工具所建模出来的流程模型进行交换。BPDM超越了BPMN和BPEL所定义的业务流程建模的要素，它定义了编排和编制。
>
> &ensp;&ensp;&ensp;&ensp; 三者的竞争关系似乎还将继续，但，BPMN2.0出现了。BPMN2.0相比BPMN1.x，最重要的变化在于其定义了流程的元模型和执行语义，即它自己解决了存储、交换和执行的问题，BPMN由单纯的业务建模重新回归了它的本源，即作为一个对业务人员友好的标准流程执行语言的图形化前端。BPMN2.0一出手，竞争就结束了，XPDL、BPEL和BPDM各自准备回家钓鱼。看起来胜利者似乎是BPMN，但看看BPMN2.0的领导者，就会发现最后的胜利者还是IBM，Oracle和SAP这些大厂商们，他们提交的草案明确要赋予BPMN2.0以执行语义，这迫使BPDM团队撤回了其提交，并将他们的提议与BPDM团队想法合并，这就是BPMN2.0最后内容的由来。
> 
> —— 摘自CSDN[《BPMN2.0协议解析》](https://blog.csdn.net/liaoxiangui/article/details/80577721)

##    BPMNJS 简介

&ensp;&ensp;&ensp;&ensp; bpmn.js是一个BPMN 2.0渲染工具包和web建模器。它是用JavaScript编写的，将BPMN 2.0 图表嵌入在浏览器中, 并独立于后端, 这也使得将其嵌入到任务Web应用程序中变得很容易: 可以独立使用也可以集成到你的应用中。

&ensp;&ensp; 该库的构建方式既可以是查看器，也可以是Web建模器

   - 使用查看器(Viewer)将BPMN 2.0 嵌入到应用程序中，并用系统数据丰富其查看器。
   - 使用建模器(Modeler) 在应用程序中创建BPMN 2.0 图表。

###    diagram-js 和bpmn-moddle

&ensp;&ensp;&ensp;&ensp; BPMN 2.0 中 bpmn-js主要依赖的库有两个:  diagram-js 和bpmn-moddle.

&ensp;&ensp;&ensp;&ensp; bpmn.js是建立在 diagram-js和bpmn-moddle 两个库之上进行使用的。其中 diagram-js 是用来进行绘制形状和连接。它为我们提供了与这些图形元素交互的方法，以及帮助用户构建强大的BPMN 查看器等辅助工具. 对于建模它提供了上下文、调色板和重做/撤销等功能。bpmn-moddle 它允许我们读取和写入符合 BPMN 2.0 模式的XML 文档，并访问图表上绘制的形状和连接背后的BPMN相关信息。在这两个关联库之上，bpmn-js定义了BPMN的细节，例如外观、建模规则和工具(调色板)等等。

### diagram-js(图表交互/建模)

&ensp;&ensp;&ensp;&ensp; 模块系统：在底层 diagram-js使用依赖注入来连接和发现图表组件。在diagram-js的上下文中讨论模块时，指的是提供命名服务以及其实现的单元。"服务"是一个函数或实例，它可以使用其他服务在图表的上下文中做事。

&ensp;&ensp;&ensp;&ensp; 下面是与生命周期事件挂钩的一个服务，它通过 eventBus 代理一个事件来做到可以处理事件的功能。如下：       

```javascript
const MyPlugin = (eventBus) => {
  eventBus.on('element.changed', (event) => {
    console.log('element ', event.element, 'changed');
  });
}
MyPlugin.$inject = ['eventBus' ];
```

**diagram-js是围绕许多基本服务构建的：**

- Canvas- 提供用于添加和删除图形元素的 API；处理元素生命周期并提供 API 来缩放和滚动。
- EventBus- 事件总线模块来管理监听事件，相关方可以订阅各种事件，并在它们发出后对其采取行动。事件总线帮助我们解耦关注点并将功能模块化，以便新功能可以轻松地与现有行为挂钩。
- ElementFactory- 根据 diagram-js 的内部数据模型创建形状和连接的工厂。
- ElementRegistry- 了解添加到图表中的所有元素，并提供 API 以通过id检索元素及其图形表示。
- GraphicsFactory- 负责创建形状和连接的图形表示。实际的外观和感觉由渲染器定义，即DefaultRender在绘图模块内部。

**diagram-js还提供一些辅助工具箱：**

- CommandStack- 负责建模期间的重做和撤消。
- ContextPad- 提供围绕元素的上下文操作。
- Overlays- 提供用于将附加信息附加到图表元素的 API。
- Modeling- 提供用于更新画布上的元素（移动、删除）的 API
- Palette - 左侧和右侧工具面板等等一些可扩展的辅助工具；

### bpmn-moddle

&ensp;&ensp;&ensp;&ensp; bpmn-moddle是封装了BPMN 2.0模型，并提供了读写 BPMN 2.0 XML 文档工具。导入时 将 XML 文档解析为JavaScript 对象树，在建模时对该树进行编辑和验证，然后在保存图表时将其导出成 BPMN 2.0 XML协议文件。

&ensp;&ensp;&ensp;&ensp; bpmn-moddle 将 BPMN 2.0 规范添加为元模型，并为BPMN 2.0 模式验证提供了简单的接口，并提供如下API:

- formXML - 从给定的XML字符串创建 BPMN 树
- toXML - 将BPMN对象树写入BPMN 2.0 XML


## BPMN 2.0 基本结构

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7545e00fa6514375b010b3ad7348d810~tplv-k3u1fbpfcp-zoom-1.image "BPMN2.0基本元素")

###   事件

 事件通常是与活动和网关一起工作的，事件是用在实际的每个业务流程中重要的组成形式。
 事件让业务建模工具用很自然的方式描述业务流程，比如：
 
> 当我介绍到一个客户的准入信息，这个流程就启动.
>
> 如果两个小时内任务没有进行审核或者是进行相关操作，这个任务节点就超时提醒或者结束该流程
>
> 这就是通过事件进行驱动业务流程。事件又分很多种事件类型：

#### 空启动事件

&ensp;&ensp;&ensp;&ensp; 启动事件说明流程的开始(或子流程)，图形形式，看起来是一个圆(可能) 内部有一个小图标。图标指定了事件的事件类型 会在流程实例创建时被触发。

&ensp;&ensp;&ensp;&ensp; 空启动事件画出来的是一个空圆，内部么有图标，意思是这个触发器是未知或者未指定。一个空开始事件的定义如下：

```javascript
<startEvent id="start"  name="myStart" />
其中id是必须得，name是可选项
```

#### 空结束事件

&ensp;&ensp;&ensp;&ensp; 结束事件指定了流程实例中一个流程路径的结束。图形上，它看起来就是一个圆拥有厚边框（可能）内部有小图标。图标指定了结束的时候会执行哪种操作。

&ensp;&ensp;&ensp;&ensp; 空结束事件画出来是一个圆，拥有厚边框，内部没有图标，这意味着当流程到达事件时，不会抛出任何信号。一个空结束事件的定义如下：

```javascript
<endEvent id="end" name="myEnd" />
```

如下图是一个空的开始事件和空结束事件流程:
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/566b78e24bf743c99dcd483151dc7b32~tplv-k3u1fbpfcp-zoom-1.image)

#### 终止结束事件

&ensp;&ensp;&ensp;&ensp; 终止和空结束事件的区别是 实际中流程的路径是如何处理的（或者使用BPMN 2.0的术语叫做token）。终止结束事件会结束整个流程实例，而空结束事件只会结束当前流程路径。他们都不会抛出任何事情 当到达结束事件的时候。一个终止结束事件的定义如下：

```javascript
<endEvent id="terminateEnd" name="myTerminateEnd">
  <terminateEventDefinition/>
</endEvent>
```

&ensp;&ensp;&ensp;&ensp; 终止结束事件被描绘成结束事件一样（圆，厚边框），内部图标时一个完整的圆。在下面的例子中，完成任务A 会结束流程实例，当完成完成任务B时只会结束到达结束事件 的流程路径，只剩下任务A打开。示例如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64daee7f1ae94376a05b6d205f0d88c9~tplv-k3u1fbpfcp-zoom-1.image)
#### 定时启动事件

&ensp;&ensp;&ensp;&ensp; 定时启动事件用来表示流程需要在指定时间启动。可以指定一个特殊的时间点（比如，2010年10月10日下午5点），但是也可以用一个通常的时间（比如，每个周五的半夜）。
定时启动事件看起来是在圆圈中有一个表的图标。
使用定时启动事件，要添加一个timerEventDefinition元素在开始事件元素下面：

```javascript
<startEvent name="Every Monday morning" id="myStart">
  <timerEventDefinition/>
</startEvent>
```

图形如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/938c4264969247328a84b40bcbeb2f0e~tplv-k3u1fbpfcp-zoom-1.image)

#### 中间事件

&ensp;&ensp;&ensp;&ensp; 中间事件用来表示在流程执行过程中发生的事件（比如， 在流程启动之后，在它完成之前）。中间事件看起来就像一个有着双边线的圆圈，圆圈中的图标表示了事件的类型。

&ensp;&ensp;&ensp;&ensp; 这儿有好多种中间事件类型，比如定时器事件，触发事件，传播事件，等等。 中间事件既可以抛出也可以捕获：

> 抛出：当一个流程到达事件中，它会立刻触发一个对应的触发器（一个激活，一个错误，等等）。抛出事件用图形表示起来就是使用黑色填充的图标。

> 捕获：当一个流程到达事件中，它会等待一个对应的触发器发生（一个错误，一个定时器，等等）捕获事件用图形表示起来就是没有使用黑色填充的图标（比如，内部是白色的）。

**中间事件网关类型有如下几种：**

| 捕获事件 | 抛出事件 |
| --- |  --- | 
| 消息中间捕获事件 | 消息中间抛出事件 | 
| 定时中间捕获事件 | 升级中间抛出事件 | 
| 条件中间捕获事件 | 补偿中间抛出事件 | 
| 信号中间捕获事件 | 信号中间抛出事件 | 
| 链接中间捕获事件 |  |

### 顺序流

&ensp;&ensp;&ensp;&ensp; 顺序流是事件，活动和网关之间的连线，显示为一条实线 带有箭头，在BPMN图形中（jPDL中等效的是transition）。每个顺序流都有一个源头和一个目标引用，包含了活动，事件或网关的id。

```javascript
<sequenceFlow id="GEF" name="GEF"　 sourceRef="任务A" targetRef="空终止结束事件" />
```

&ensp;&ensp;&ensp;&ensp; 为了避免使用一个顺序流，必须添加condition条件到顺序流中。在运行时，只有当condition条件结果为true，顺序流才会被执行。
为了给顺序流添加condition条件，添加一个conditionExpression元素到顺序流中。条件可以放在${}中。

```javascript
<sequenceFlow id='111'>
  <conditionExpression xsi:type="tFormalExpression">${age >10}</conditionExpression>
</sequenceFlow>
<sequenceFlow id='22'>
  <conditionExpression xsi:type="tFormalExpression">${age <=10}</conditionExpression>
</sequenceFlow>
```

示例图如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27f91dc3097e48dd84acf1cc16eab00e~tplv-k3u1fbpfcp-zoom-1.image)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5c304b3e8ad4fbda8035c1bd91277ae~tplv-k3u1fbpfcp-zoom-1.image)

注意，当前必须把xsi:type="tFormalExpression"添加到conditionExpression中。一个条件性的顺序流可以看到一个小菱形图片 在顺序流的起点。记住表达式一直可以定义在顺序流上，但是一些结构不会解释它（比如，并行网关）。
活动（比如用户任务）和网关（比如唯一网关）可以用户默认顺序流。默认顺序流只会在活动或网关的 所有其他外向顺序流的condition条件为false时才会使用。默认顺序流图形像是顺序流多了一个斜线标记。

### 网关

&ensp;&ensp;&ensp;&ensp; BPMN中的网关是用来控制流程中的流向的。更确切的是， 当一个token（BPMN 2.0中execution的概念注解）到达一个网关， 它会根据网关的类型进行合并或切分。网关描绘成一个菱形，使用一个内部图标来指定类型 （唯一，广泛，其他）。
所有网关类型，有如下几种：

- 排他网关(唯一网关)：也称专用网关， 只有一条路径会被选

&ensp;&ensp;&ensp;&ensp; 当用作分支网关（将顺序流分成多个路径，一分为二）时，专用网关可以具有2个或更多个传出路径，当某个变量条件返回“真”时，它会专门只指向下一个路径，当使用专用网关时，对于某个流程实例，运行时只能在多个路径中使用其中任意一条，这就是使用术语“独占或排他”的意思，检查每个路径上的变量条件，直到有一个路径的变量条件评估为真，一旦条件评估为真，流程就沿着为真的路径前进，并且不再检查其他路基的条件。

- 并行网关：所有路径会被同时选择
- 包容网关：可以同时执行多条线路，也可以在网关上设置条件

但是每个网关都可以设置gatewayDirection。下面的值可以使用：

- unspecificed (默认)：网关可能拥有多个 进入和外出顺序流。
- mixed：网关必须拥有多个 进入和外出顺序流。
- converging：网关必须拥有多个进入顺序流， 但是只能有一个外出顺序流。
- diverging：网关必须拥有一个进入顺序流， 和多个外出顺序流。

比如下面的例子：并行网关的gatewayDirection属性为'converging'，会拥有json行为。

```javascript
<parallelGateway id="test" name="GFE" gatewayDirection="converging" />
```

示例图如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/19d2fbe0081440e7b54b53111df116a1~tplv-k3u1fbpfcp-zoom-1.image)

#### 1、排他网关(唯一网关)

&ensp;&ensp;&ensp;&ensp; 唯一网关表达了一个流程中的唯一决策。会有一个外向顺序流被使用，根据定义在顺序流中的条件。示例图如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e2bc55add19645869f228bc34412a638~tplv-k3u1fbpfcp-zoom-1.image)

&ensp;&ensp;&ensp;&ensp; 唯一网关需要所有外向顺序流上都定义条件。 对这种规则一种例外是默认顺序流。 使用default 属性来引用一个已存在的 顺序流的id。这个顺序流会被使用，当其他外向顺序流的条件都执行为false时。

```javascript
<exclusiveGateway id="decision" name="decideBasedOnAmountAndBankType" default="myFlow"/>

<sequenceFlow id="myFlow" name="fromGatewayToStandard"
    sourceRef="decision" targetRef="standard">
</sequenceFlow>
```

&ensp;&ensp;&ensp;&ensp; 唯一网关可以同时实现汇聚和发散功能。这个逻辑很容易理解： 对于每个到达这个网关的分支流程，都会选择一个外向顺序流来继续执行。 下面的图形在BPMN 2.0中是完全合法的 （忽略名称和声明的条件）。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3034b3e9215d4e9ab99264f9c7be235f~tplv-k3u1fbpfcp-zoom-1.image)

#### 2、并行网关

并行网关用来切分或同步相关的进入或外出顺序流。

- 并行网关拥有一个进入顺序流的和多于一个的外出顺序流叫做'并行切分或'AND-split'。所有外出顺序流都会 被并行使用。注意：像规范中定义的那样， 外出顺序流中的条件都会被忽略。
- 并行网关拥有多个进入顺序流和一个外出顺序流叫做'并行归并';所有进入顺序流需要到达这个并行归并，在外向顺序流使用之前。

```javascript
<parallelGateway id="GFE" name="GFE" />
```

下面的图形显示了一个并行网关可以如何使用如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dcfce30ff11349299f71b1aae888024d~tplv-k3u1fbpfcp-zoom-1.image)

&ensp;&ensp;&ensp;&ensp; 一个并行网关（其实是任何网关）可以同时拥有切分和汇聚行为。 下面的图形在BPMN 2.0中是完全合法的。 在流程启动之后，A和B任务都会激活。当A和B完成时，C,D和E 任务会被激活。示例图如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca64140dadb1452a826cef7308a91067~tplv-k3u1fbpfcp-zoom-1.image)

#### 3、包含网关

&ensp;&ensp;&ensp;&ensp; 一个包含网关 - 也叫做OR-gateway - 被用来进行“条件性”切分或汇聚顺序流。它基本的行为就和一个并行网关一样，但是它也可以统计条件，在外出顺序流上（切分行为）和计算，如果这儿有流程离开，可以到达网关（合并行为）。

&ensp;&ensp;&ensp;&ensp; 包含网关显示为一个典型的网关图形，里边有一个圆圈（参考'OR'的语法）。 和唯一网关不同，所有条件表达式被执行（发散或切分行为）。对于每个表达式结果为true时，一个新的子流程分支就会被创建。没有定义条件的顺序流会永远被选择（比如一个子流程在这种情况下总是会被创建）。

&ensp;&ensp;&ensp;&ensp; 一个收敛的包含网关（合并行为）有一个更困难的执行逻辑。当一个执行（在BPMN 2.0的语法中叫做Token）到达一个合并包含网关。就会进行下面的检测（引用规范的文字）：

```shell
对于每个空的进入顺序流，这里没有Token
在顺序流的图形上面，比如，这里有一个没有直接的路径
（由顺序流组成）从Token到这个顺序流，除非
a) 路径到达了一个包含网关，或
b) 路径到达了一个节点，直接到一个非空的
  进入顺序流的包含网关 "
```

&ensp;&ensp;&ensp;&ensp; 简单来说：当一个流程到达了这个网关，所有的激活流程会被检测它们是否可以到达包含网关，只是统计顺序流（注意：条件不会被执行！）。当包含网关被使用时，它通常用在一个切分/汇聚包含网关对中。在其他情况，流程行为足够简单，只要通过看图就可以理解了。

&ensp;&ensp;&ensp;&ensp; 当然，不难想象情况，当流程切分和汇聚在复杂的组合，使用大量的结构，其中包括包含网关。在那些情况，很可能出现实际的流程行为可能与建模者的期望不符。所以，当使用包含网关时，要注意通常的最佳实践是让包含网关成对使用。

&ensp;&ensp;&ensp;&ensp; 下面的图形演示了如何使用包含网关。 （例子来自于Bruce Silver的"BPMN method and style"）

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3ad6b79bc9b43a38e61f4d1a1fb9610~tplv-k3u1fbpfcp-zoom-1.image)

### 任务

&ensp;&ensp;&ensp;&ensp; 一个任务表示工作需要被外部实体完成， 比如人工或自动服务。在BPMN 2.0中，这里有很多任务类型，一些表示等待状态（比如，User Task一些表示自动活动（比如，Service Task。所以小心不要混淆了任务的概念，在切换语言的时候。任务被描绘成一个圆角矩形，一般内部包含文字。任务的类型（用户任务，服务任务，脚本任务，等等）显示在矩形的左上角，用小图标区别。根据任务的类型，引擎会执行不同的功能。

#### 1、用户任务（人工任务）

&ensp;&ensp;&ensp;&ensp; user task是典型的'人工任务'， 实际中的每个workflow或BPMN软件中都可以找到。当流程执行到达这样一个user task时， 一个新人工任务就会被创建，交给用户的任务列表。和manual task的主要区别是 （也与人工工作对应）是流程引擎了解任务。 引擎可以跟踪竞争，分配，时间，其他，这些不是manual task的情况。

user task描绘为一个圆角矩形，在左上角是一个小用户图标。

**user task被定义为下面的BPMN 2.0 XML：**

```javascript
<userTask id="myTask" name="My task" />
```

示例图如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f5f36ff5e79347bf8134aa86b145019a~tplv-k3u1fbpfcp-zoom-1.image)

&ensp;&ensp;&ensp;&ensp; 根据规范，可以使用多种实现（WebService, WS-humantask，等等）。通过使用implementation属性。当前，只有标准的jBPM任务机制才可以用，所以这里（还）没有定义'implementation'属性的功能。

&ensp;&ensp;&ensp;&ensp; BPMN2.0规范包含了一些方法把任务分配给用户，组，角色等等。当前的BPMN 2.0jBPM实现允许使用一个resourceAssignmentExpression来分配任务，结合humanPerformer or PotentialOwner结构。这部分希望在未来的版本里能够进一步演化。

&ensp;&ensp;&ensp;&ensp; potentialOwner用来在你希望确定用户，组，角色的时候。这是一个task的候选人。参考下面的例子。这里的'My task'任务的候选人组是'management'用户组。也要注意，需要在流程外部定义一个资源，这样任务分配器可以引用到这个资源。实际上，任何活动都可以引用一个或多个资源元素。目前，只需要定义这个资源就可以了（因为它是规范中的一个必须的元素），但是在以后的发布中会进行加强（比如，资源可以拥有运行时参数）。

#### 2、服务任务

&ensp;&ensp;&ensp;&ensp; Service Task是一个自动活动，它会调用一些服务，比如web service，java service等等。当前jBPM引擎 只支持调用java service，但是web service的调用已经在未来的版本中做了计划。

示例图如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23726714ec214d8692b9701684ae4ce2~tplv-k3u1fbpfcp-zoom-1.image)

&ensp;&ensp;&ensp;&ensp; 定义一个服务任务需要好几行XML（这里就可以看到BPEL的影响力）。当然，在不久的未来，我们希望有工具可以把这部分大量的简化。一个服务任务需要如下定义：

```javascript
<serviceTask id="MyServiceTask" name="My service task"
  implementation="Other" operationRef="myOperation" />
```

&ensp;&ensp;&ensp;&ensp; 服务任务需要一个必填的id和一个可选的name。implementation元素是用来表示调用服务的类型。可选值是WebService, Other或者Unspecified。因为我们只实现了Java调用，现在只能选择Other。

&ensp;&ensp;&ensp;&ensp; 服务任务将调用一个操作，operation的id 会在operationRef属性中引用。 这样一个操作就是下面实例的interface的一部分。每个操作都至少有一个输入信息，并且最多有一个输出信息。

```javascript
<interface id="myInterface"
    name="org.jbpm.MyJavaServicek">
    <operation id="myOperation2" name="myMethod">
      <inMessageRef>inputMessage</inMessageRef>
      <outMessageRef>outputMessage</outMessageRef>
    </bpmn:operation>
</interface>
```

对于java服务，接口的名称用来指定java类的全类名。操作的名称用来指定将要调用方法名。输入/输出信息表示着java方法的参数/返回值，定义如下所示：

```javascript
<message id="inputMessage" name="input message" structureRef="myItemDefinition1" />
```

#### 3、脚本任务

&ensp;&ensp;&ensp;&ensp; 脚本任务是一个自动活动，当到达这个任务的时候流程引擎会执行一个脚本。脚本任务使用方式如下：

```javascript
<scriptTask id="scriptTask" name="Script Task" scriptLanguage="bsh">
  <script><![CDATA[
    for(int i=0; i < input.length; i++){
      System.out.println(input[i] + " x 2 = " + (input[i]*2));
    }]]>
  </script>
</scriptTask>
```

&ensp;&ensp;&ensp;&ensp; 脚本任务，除了必填id和可选的name之外，还允许指定scriptLanguage和script。因为我们使用了JSR-223（java平台的脚本语言）修改脚本语言就需要：

- 把scriptLanguage 属性修改为JSR-223兼容的名称
- 在classpath下添加JSR规范的ScriptEngine实现

上面的XML对应图形如下所示（添加了空开始和结束事件）。
图形如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9363330fbe264b20a210aed37d6e62e7~tplv-k3u1fbpfcp-zoom-1.image)

#### 4、手工任务

&ensp;&ensp;&ensp;&ensp; 手工任务是一个由外部人员执行的任务，但是没有指定是一个BPM系统或是一个服务会被调用。在真实世界里，有很多例子：安装一个电话系统，使用定期邮件发送一封信，用电话联系客户，等等。

```javascript
<manualTask id="myManualTask" name="Call customer" />
```

&ensp;&ensp;&ensp;&ensp; 手工任务的目标更像 文档/建模提醒的，因为它对流程引擎的运行没有任何意义。因此，当流程引擎遇到一个手工任务时会简单略过。
图形如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/172d9994ad6e46c5834b3594d9fd4caf~tplv-k3u1fbpfcp-zoom-1.image)

#### 5、接收任务

&ensp;&ensp;&ensp;&ensp; receive task是一个任务会等到外部消息的到来。除了广泛使用的web service用例，规范在其他环境中的使用也是一样的 web service用例还没有实现，但是receive task已经可以在java环境中使用了。

&ensp;&ensp;&ensp;&ensp; receive task显示为一个圆角矩形（和task图形一样）在左上角有一个小信封的图标。

&ensp;&ensp;&ensp;&ensp; 在java环境中，receive task没有其他属性，除了id和name（可选），行为就像是一个等待状态。为了在你的业务流程中使用等待状态，只需要加入如下几行：

```javascript
<receiveTask id="receiveTask" name="wait" />
```

&ensp;&ensp;&ensp;&ensp; 流程执行会在这样一个receive task中等待。流程会使用熟悉的jBPM signal methods来继续执行。注意，这些可能在未来改变，因为'signal'在BPMN 2.0中拥有完全不同的含义。

```javascript
Execution execution = processInstance.findActiveExecutionIn("receiveTask");
executionService.signalExecutionById(execution.getId());
```

图形如下：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/165289f173a54163be17d33ea6e80130~tplv-k3u1fbpfcp-zoom-1.image)

### 数据（Data）：

数据主要通过四种元素表示

- 数据对象（Data Objects）
- 数据输入（Data Inputs）
- 数据输出（Data Outputs）
- 数据存储（Data Stores）

### 连接对象（Connecting Objects）

流对象彼此互相连接或者连接到其他信息的方法主要有三种

- 顺序流：用一个带实心箭头的实心线表示，用于指定活动执行的顺序
- 信息流：用一条带箭头的虚线表示，用于描述两个独立的业务参与者（业务实体/业务角色）之间发送和接受的消息流动
- 关联：用一根带有线箭头的点线表示，用于将相关的数据、文本和其他人工信息与流对象联系起来。用于展示活动的输入和输出

### 泳道（Swimlanes）

通过泳道对主要的建模元素进行分组，将活动划分到不同的可视化类别中来描述由不同的参与者的责任与职责。

## 案例

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aeba1165c0b9489697fbb6b36b00ea3b~tplv-k3u1fbpfcp-zoom-1.image "联盟货物购买审批流程案例")

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7e3f2ea12e5c419ba3e71c3b1a9709ee~tplv-k3u1fbpfcp-zoom-1.image "用户调整费用审批流程案例")


参考资料

1. [https://www.omg.org/spec/BPMN/2.0/](https://www.omg.org/spec/BPMN/2.0/)

2. [https://zhuanlan.zhihu.com/p/32625363](https://zhuanlan.zhihu.com/p/32625363)

# 源码

本文示例源码：[https://github.com/gfe-team/BPMN](https://github.com/gfe-team/BPMN)

## 团队介绍
高灯科技交易合规前端团队（GFE）, 隶属于高灯科技(北京)交易合规业务事业线研发部，是一个富有激情、充满创造力、坚持技术驱动全面成长的团队, 团队平均年龄27岁，有在各自领域深耕多年的大牛, 也有刚刚毕业的小牛, 我们在工程化、编码质量、性能监控、微服务、交互体验等方向积极进行探索, 追求技术驱动产品落地的宗旨，打造完善的前端技术体系。

- 愿景: 成为最值得信任、最有影响力的前端团队
- 使命: 坚持客户体验第一, 为业务创造更多可能性
- 文化: 勇于承担、深入业务、群策群力、简单开放

**Github：**[github.com/gfe-team](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fgfe-team)

**团队邮箱：**[gfe@goldentec.com](https://link.juejin.cn?target=mailto%3Agfe%40goldentec.com)

作者：GFE-刘连军
著作权归GFE(高灯科技交易合规团队)所有。商业转载请联系作者获得授权，非商业转载请注明出处。
