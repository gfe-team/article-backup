<a name="fc132856"></a>
## 前言 Web 安全的兴起

Web 攻击技术的发展也可以分为几个阶段。在 `Web 1.0` 时代，人们更多的是关注服务器端<br />动态脚本的安全问题，比如将一个可执行脚本（俗称 `webshell`）上传到服务器上，从而获得权<br />限。后续有出现了`SQL`注入，`SQL`注入的出现是`Web`安全史上的一个里程碑，`SQL`注入漏洞至今仍然是`Web`安全领域中的一个重要组成部分。再后续另一个里程碑的安全问题问世--`XSS`（跨站脚本攻击）。伴随着 `Web 2.0` 的兴起，`XSS`、`CSRF` 等攻击已经变得更为强大。`Web` 攻击的思路也从服务器端转向了客户端，转向了浏览器和用户。

`Web` 技术发展到今天，构建出了丰富多彩的互联网。互联网业务的蓬勃发展，也催生出了<br />许多新兴的脚本语言，比如 `Python`、`Ruby`、`NodeJS` 等，敏捷开发成为互联网的主旋律。而手机技术、移动互联网的兴起，也给 `HTML 5` 带来了新的机遇和挑战。与此同时，`Web` 安全技术，也将紧跟着互联网发展的脚步，不断地演化出新的变化。

跨站脚本攻击（`XSS`）是客户端脚本安全中的头号大敌。`OWASP TOP 10` 威胁多次把 `XSS`<br />列在榜首，该篇文章将重点介绍`XSS`的攻防问题

<a name="fb73a10e"></a>
## 初探`XSS`

**跨站脚本攻击**，英文全称是 `Cross Site Script`，本来缩写是 `CSS`，但是为了和层叠样式表（`Cascading Style Sheet`，`CSS`）有所区别，所以在安全领域叫做“`XSS`”。

`XSS`攻击，通常指黑客通过`HTML`注入 篡改网页，插入恶意脚本，从而在用户浏览网页时，控制用户浏览器的一种攻击行为。在这种行为最初出现之时，所有的演示案例全是跨域行为，所以叫做 "跨站脚本" 。时至今日，随着`Web`端功能的复杂化，应用化，是否跨站已经不重要了，但 `XSS`这个名字却一直保留下来。

随着`Web`发展迅速发展，`Web`开发已经被应用的非常广泛了，由之前的单一`PC`端扩展到现在的移动端（`APP`、`H5`），甚至包括桌面工具、设备大屏等等，所以在产生的应用场景越来越多，越来越复杂的情况下，同时大多数互联网（尤其是传统行业）的产品开发版本迭代上线时间非常短，一周一版本，两周一大版本的情况下，忽略了安全这一重要属性，一旦遭到攻击，后果将不堪设想。

<a name="82fb4b68"></a>
## `XSS`攻击类型分类

`XSS`攻击可以分为3类：反射型（非持久型）、存储型（持久型）、基于`DOM XSS`；

<a name="9dde8bab"></a>
### 反射型

反射型`XSS`只是简单地把用户输入的数据“反射”给浏览器。也就是说，黑客往往需要诱使用户“点击”一个恶意链接，才能攻击成功。反射型`XSS`也叫做 **“非持久型 **`**XSS**`**”（**`**Non-persistent XSS**`**）**。

通常反射型`XSS`的恶意代码存在`URL`里，通过`URL`传递参数的功能，如网站搜索、跳转等。由于需要用户主动打开恶意的`URL`才能生效，攻击者往往会结合多种手段诱导用户点击。

一个最初级的反射型攻击是：我们对网页数据进行获取:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>XSS攻防演练</title>
</head>
<body>
    <div id="t"></div>
    <input id="s" type="button" value="这是一个按钮" onclick="test()">
</body>
<script>
    function test() {
        const arr = ['自定义的数据1', '自定义的数据2', '自定义的数据3', '<img src="11" onerror="console.log(window.localStorage)" />']
        const t = document.querySelector('#t')
            arr.forEach(item => {
            const p = document.createElement('p')
            p.innerHTML = item
            t.append(p)
        })
    }
</script>
</html>
```

当黑客点击`这是一个按钮`时，即可轻松获取本地`localStorage`数据，从而获取关键信息。

<a name="86228154"></a>
### 存储型

存储型 `XSS` 会把用户输入的数据“存储”在服务器端。这种 `XSS` 具有很强的稳定性。

比较常见的一个场景就是，黑客写下一篇包含有恶意 `JavaScript` 代码的博客文章，文章发表后，所有访问该博客文章的用户，都会在他们的浏览器中执行这段恶意的 `JavaScript` 代码。黑客把恶意的脚本保存到服务器端，所以这种 `XSS` 攻击就叫做 **“存储型 **`**XSS**`**”**。

```html
<!-- 例如我们分别在网站中的输入框中输入以下信息，并保存到远程数据库 -->
<img src="11" onerror="console.log(window.localStorage)" />
<img src="11" onerror="alert(111)" />
```

页面输入

![1.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1666237310004-38c8cef8-a830-41df-8ea9-47bb24d203b9.png#clientId=u3a729705-4038-4&crop=0&crop=0&crop=1&crop=1&errorMessage=unknown%20error&from=ui&id=u99baa381&name=1.png&originHeight=767&originWidth=716&originalType=binary&ratio=1&rotation=0&showTitle=false&size=21869&status=error&style=shadow&taskId=u3a7ae128-442f-4bd0-be42-ffc74de9006&title=)

---

![2.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1666237320871-f9ba41b2-ed5d-4083-94a0-d5faffc84231.png#clientId=u3a729705-4038-4&crop=0&crop=0&crop=1&crop=1&errorMessage=unknown%20error&from=ui&id=ue6fcecd1&name=2.png&originHeight=690&originWidth=700&originalType=binary&ratio=1&rotation=0&showTitle=false&size=19929&status=error&style=shadow&taskId=ub44e845b-e234-4816-bc05-b66ec35182f&title=)

使用者浏览页面，分别先后触发了`alert`弹框和`localStorage`获取本地数据：

![3.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1666237334372-ffdde9a5-f47f-4658-9123-4e9276bad7c0.png#clientId=u3a729705-4038-4&crop=0&crop=0&crop=1&crop=1&errorMessage=unknown%20error&from=ui&id=uf40360a6&name=3.png&originHeight=608&originWidth=1096&originalType=binary&ratio=1&rotation=0&showTitle=false&size=36245&status=error&style=shadow&taskId=u9a23369d-4670-420b-8222-edbea650c85&title=)

---

![4.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1666237347272-3a535e41-4b97-4336-9fc3-04569992447e.png#clientId=u3a729705-4038-4&crop=0&crop=0&crop=1&crop=1&errorMessage=unknown%20error&from=ui&id=uba39037f&name=4.png&originHeight=167&originWidth=1041&originalType=binary&ratio=1&rotation=0&showTitle=false&size=35629&status=error&style=none&taskId=ua973d859-a068-4643-bde4-67a049ced04&title=)<br />以上就是一个典型的**存储型**攻击。

<a name="8807bb11"></a>
### 基于`DOM XSS`

实际上，这种类型的`XSS`并非按照“数据是否保存在服务器端”来划分，`DOM Based XSS`从效果上来说也是反射型`XSS`。单独划分出来，是因为`DOM Based XSS` 的形成原因比较特别，发现它的安全专家专门提出了这种类型的`XSS`。`DOM 型 XSS`跟前两种`XSS`的区别：`DOM 型 XSS`攻击中，取出和执行恶意代码由浏览器端完成，属于前端`JavaScript`自身的安全漏洞，而其他两种`XSS`都属于服务端的安全漏洞。

接下来我们来看一个简单的示例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>XSS攻防演练</title>
</head>
<body>
    <h3>基于DOM的XSS</h3>
    <input type="text" id="input">
    <button id="btn">提交内容</button>
    <div id="div"></div>
</body>
<script>
    const input = document.getElementById('input');
    const btn = document.getElementById('btn');
    const div = document.getElementById('div');

    let inputValue;
     
    input.addEventListener('change', (e) => {
        inputValue = e.target.value;
    }, false);

    btn.addEventListener('click', () => {
        div.innerHTML = `<a href=${inputValue}>链接地址</a>`
    }, false);
</script>
</html>
```

我们再页面输入框中输入以下文本`'' onclick=alert(/xss/)`，这里的`''`引号是为了关闭掉`href`属性，给它赋予了一个空值。然后点击`提交内容`按钮，则页面中的`<div id="div"></div>`标签包含了一下`html`内容

```html
<a href onlick="alert(/xss/)">链接地址</a>
```

---

![7.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1666237393329-6f6083a5-60b0-469b-8fd7-c444c8e868ff.png#clientId=u3a729705-4038-4&crop=0&crop=0&crop=1&crop=1&errorMessage=unknown%20error&from=ui&id=u5cb0081a&name=7.png&originHeight=299&originWidth=1483&originalType=binary&ratio=1&rotation=0&showTitle=false&size=24031&status=error&style=shadow&taskId=uf9331afe-e256-40d3-b7b6-724db834159&title=)
<a name="cafc29de"></a>
## `XSS`攻击防御

关于`XSS`的防御是非常复杂的，值得幸运的是现代浏览器、前端框架/库已经帮我们做了相当大的一部分工作。

<a name="26cfdeb0"></a>
### `HttpOnly`

`HttpOnly` 最早是由微软提出，并在`IE 6`中最先实现的，至今已经逐渐成为一个标准。浏览器将禁止页面的`JavaScript`访问带有`HttpOnly`属性的`Cookie`。所以我们需要在`http`的响应头`set-cookie`时设置`httpOnly`，让浏览器知道不能通过`document.cookie`的方式获取到`cookie`内容。

严格地说，`HttpOnly` 并非为了对抗 `XSS——HttpOnly` 解决的是`XSS`后的 `Cookie` 劫持攻击。所以说使用`HttpOnly`有助于缓解`XSS`攻击，但仍然需要其他能够解决`XSS`漏洞的方案；

<a name="cbf7cd7f"></a>
### 输入检查

对于用户的输入内容我们需要持怀疑态度。在对输入不做任何过滤检查的情况下用户可能输入任何字符串。比如我们期望输入的内容是：`hello word`, 也许我们收到的内容是`onclick=alert(/xss/)`。

在`XSS`的防御上，输入检查一般是检查用户输入的数据中是否包含一些特殊字符，如`＜、＞、’、”`等。如果发现存在特殊字符，则将这些字符过滤或者编码。这种输入检查的方式，可以称为`“XSS Filter”`。互联网上有很多开源的`“XSS Filter”`的实现。比如一个简单的`htmlencode`转义：

```javascript
const htmlEncode = function (handleString){
    return handleString
    .replace(/&/g,"&amp;")
    .replace(/</g,"&lt;")
    .replace(/>/g,"&gt;")
    .replace(/ /g,"&nbsp;")
    .replace(/\'/g,"&#39;")
    .replace(/\"/g,"&quot;");
}
```

但是**输入检查**也有弊端，比如

- 攻击者绕过前端页面直接使用接口就可以提交恶意代码到远程库中。
- 输入数据，还可能会被展示在多个地方，每个地方的语境可能各不相同，如果使用单一的替换操作，则可能会出现问题。输入检查也需要有针对性，如果我们想表达的意思是一个数小于另一个数（ `3 < 4`）,前端转义后的字符就变成`3 &lt; 4`，当这个值被存到远端时后，再通过`AJAX`获取使用就会造成不必要的麻烦，比如我就进行数值计算等等。

<a name="ad97d181"></a>
### 输出检查

一般来说，除富文本的输出外，在变量输出到`HTML`页面时，可以使用编码或转义的方式来防御`XSS`攻击。

XSS的本质还是一种“HTML 注入”，用户的数据被当成了HTML代码一部分来执行，从而混淆了原本的语义，产生了新的语义。

如同输入检查一样，我们可以对输出进行编码转义。

<a name="9f4f1c0c"></a>
#### 1.在`HTML`中输出

比如我们的html代码中有这样一段代码：

```html
<div>$htmlVar</div>
<a href="">$htmlVar</a>
```

如果输出的变量没有进行安全处理，直接使用并渲染在页面中，都能导致直接产生`XSS`。最终的结果可能生成一下代码：

```html
<div><script>alert('我是一个XSS攻击者')</script></div>
<a href="#"><img href="" onclick="alert('我是另外一个XSS攻击者')"></a>
```

这个预防的方法就是对html进行转义检查

<a name="f8e1304d"></a>
#### 2. 在`HTML`属性中输出

如果我们的html属性时动态值，那么利用属性也可以被攻击；

```html
<div id="testXSS" data-name=""></div>
```

现在往`data-name`属性中插入一段未转义的代码`"><script>alert('我是一个XSS攻击者')</script><"`,结果如下：

```html
<div id="testXSS" data-name=""><script>alert('我是一个XSS攻击者')</script><""></div>
```

<a name="0e6c716a"></a>
#### 3. 在`<script>`标签中输出

在`<script>`标签中输出时，首先应该确保输出的变量在引号中。

```html
<script>
  // 假设userData是攻击者注入的数据
  let xssVar = userData;
</script>
```

攻击者需要先闭合引号才能实施XSS攻击：

```html
<script>
  // 假设userData是攻击者注入的数据
  let xssVar = "";alert('我是一个script XSS攻击者');
</script>
```

<a name="2f70e748"></a>
#### 4. 在`CSS`中输出

在 `CSS` 和 `style`、`style attribute` 中形成 `XSS` 的方式非常多样化，所以，一般来说，尽可能禁止用户可控制的变量在“`<style>`标签”、“`HTML`标签的`style`属性”以及“`CSS` 文件”中输出。如果一定有这样的需求，则推荐使用一个关于CSS转义库。

<a name="c7bb0def"></a>
### 防御`DOM Based XSS`

`DOM Based XSS`是一种比较特别的`XSS`漏洞，前文提到的几种防御方法都不太适用，需要特别对待。这个本质上，实际上就是网站前端JavaScript代码本身不够严谨，把不可信的数据当作代码执行了。

如果用 `Vue/React` 技术栈，并且不使用 `v-html`/`dangerouslySetInnerHTML`功能，就在前端`render`阶段避免`innerHTML`、`outerHTML`的 `XSS`隐患。稍后会有专门的`Vue`关于`XSS`的防御段落。

会触发`DOM Based XSS`的地方有很多，以下几个地方是`JavaScript`输出到`HTML`页面的必经之路。

- `document.write()`;
- `document.writeln()`;
- `xxx.innerHTML()`;
- `xxx.outerHTML()`;
- `xxx.innerHTML.replace()`;
- `document.attachEvent()`;
- `window.attachEvent()`;
- `window.location()`;
- `window.name`;

所以开发者需要重点关注这几个地方的参数是否可以被用户控制。如果项目中有用到这些的话，一定要避免在字符串中拼接不可信数据。

<a name="68bdc952"></a>
## `Vue`中的`XSS`防御

如果你在项目中使用了`Vue`作为前端开发框架，恭喜你，`Vue`将为你解决绝大多数的`XSS`攻击问题，但是`Vue`不是一个预防`XSS`攻击的框架，在开发使用的时候还是有被攻击的漏洞存在；

<a name="533572b2"></a>
### `Vue`中的防御措施

不论使用模板还是渲染函数，`Vue`都会将插值的内容都会自动转义，就像下面这段模板代码：

```html
<template>
    <p>{{userData}}</p>
</template>

<script>
    // 从远程获取的数据
    userData = "<script>alert('xss')</script>"
</script>
```

最终编译后页面显示的`html`源码内容如下：

```html
<p>
    <script>alert('xss')</script>
</p>
```

原因是`Vue`帮我们对数据进行了转义，因此避免了脚本注入。该转义通过诸如 textContent 的浏览器原生的 API 完成，所以除非浏览器本身存在安全漏洞，否则不会存在安全漏洞。转义后的内容如下：

```html
&lt;script&gt;alert(&quot;xss&quot;)&lt;/script&gt;
```

<a name="ff1f32cd"></a>
### 注入`HTML`

如果你要动态注入远程的`HTML`内容，首先你应该确保这些内容是安全有效的，否则你应该采取一些防御措施，去过滤或转义掉一些危险的标签符号；例如你可以这样显示的渲染`HTML`：

```html
<!-- 当使用模版时 -->
<div v-html="userProvidedHtml"></div>

<!-- 当使用渲染函数时 -->
<script>
    h('div', {
    domProps: {
        innerHTML: this.userProvidedHtml
    }
    })
</script>
<!-- 当使用JSX 的渲染函数时 -->
<div domPropsInnerHTML={this.userProvidedHtml}></div>
```

例如我们可以使用一个简单的方法（或者引用一个更加健壮的库/插件[XSS](https://jsxss.com/zh/index.html)来过滤一遍这个远程的`userProvidedHtml`数据内容，以确保安全；

```javascript
// 一个简单的函数，通过转义<为&lt以及>为&gt来实现防御HTML节点内容
const escape = function(str){
    return str.replace(/</g, '&lt;').replace(/>/g, '&gt;')
}
```

<a name="ef397f4b"></a>
### 样式注入

在使用`Vue` 要在模板内避免渲染 `style` 标签:

```html
<style>{{ userProvidedStyles }}</style>
```

这是因为，一但通过`userProvidedStyles`，恶意用户仍可以提供 `CSS` 来进行“点击诈骗”，例如将链接的样式设置为一个透明的方框覆盖在“登录”按钮之上。然后再把`https://user-XSS-website.com/` 做成你的应用的登录页的样子，它们就可能获取一个用户真实的登录信息，所以Vue推荐使用`对象语法`且只允许用户提供特定的可以安全控制的`property`的值:

```html
<!-- sanitizedUrl应为受控的地址 -->
<a
  v-bind:href="sanitizedUrl"
  v-bind:style="{
    color: userProvidedColor,
    background: userProvidedBackground
  }"
>
  click me
</a>
```

<a name="c6090a14"></a>
## 安全问题“没有银弹”

> 在解决安全问题的过程中，不可能一劳永逸，也就是说“没有银弹”。


> 一般来说，人们都会讨厌麻烦的事情，在潜意识里希望能够让麻烦越远越好。而安全，正是一件麻烦的事情，而且是无法逃避的麻烦。任何人想要一劳永逸地解决安全问题，都属于一相情愿，是“自己骗自己”，是不现实的。


<a name="34062b25"></a>
## 最佳实践

通用的规则是只要允许执行未过滤的用户提供的内容 (不论作为 `HTML`、`JavaScript` 甚至 `CSS`)，你就可能令自己处于被攻击的境地。这些建议实际上不论使用 `Vue`、`React`还是别的框架甚至不使用框架，都是成立的。

<a name="ZTFrP"></a>
## 参考文献

- [白帽子讲Web安全](https://baike.baidu.com/item/%E7%99%BD%E5%B8%BD%E5%AD%90%E8%AE%B2Web%E5%AE%89%E5%85%A8/7992696?fr=aladdin)
- [Vue官方文档](https://vuejs.org/guide/best-practices/security.html#what-vue-does-to-protect-you)
