与传统 SPA 相比，服务器端渲染 (SSR) 对SEO更加友好，方便搜索引擎爬虫抓取，可以直接查看完全渲染的页面，除此之外，SSR能够在更短的时间内渲染出页面内容，让用户有更好的用户体验。

<a name="df368884"></a>
# 前言

本文将从以下三个模块介绍服务端渲染：

- 什么是客户端渲染？
- 什么是服务端渲染？
- 如何实现服务端渲染？希望看完后对你有所帮助！

<a name="03c6e4c0"></a>
# 客户端渲染

![image-20200731142605631.png](https://cdn.nlark.com/yuque/0/2022/png/26115280/1662114496991-3b6486e2-c72f-4c3e-af42-09ebe676d31f.png#clientId=u22f468bb-38d8-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u26dc3556&margin=%5Bobject%20Object%5D&name=image-20200731142605631.png&originHeight=876&originWidth=1424&originalType=binary&ratio=1&rotation=0&showTitle=true&size=157392&status=done&style=none&taskId=u0d279e6d-7fa8-4040-84ba-170dc5bba6f&title=%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%B8%B2%E6%9F%93%E6%B5%81%E7%A8%8B%E5%9B%BE#crop=0&crop=0&crop=1&crop=1&id=cUwSL&originHeight=876&originWidth=1424&originalType=binary&ratio=1&rotation=0&showTitle=true&status=done&style=shadow&title=%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%B8%B2%E6%9F%93%E6%B5%81%E7%A8%8B%E5%9B%BE "客户端渲染流程图")

<a name="46b032ab"></a>
## 1、概念

客户端渲染（CSR），即传统的单页面应用(SPA)模式，Vue构建的应用程序默认情况下是一个HTML模板页面，只有一个id为app的根容器，然后通过webpack打包生成css、js等资源文件，浏览器加载、解析来渲染HTML。

右键查看一个标准的Vue项目网页源代码，可以看出源代码并没有页面中实际渲染的相关内容，只有一个id为app的根元素，所以说网页展示出来的内容是通过 JavaScript 动态渲染出来的。这种通过浏览器端的 JavaScript 为主导来渲染网页的方式就是**客户端渲染**。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/26115280/1662452348780-721dc209-532e-408b-8bf6-774bf719d8d8.png#clientId=u470e72a7-5795-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=304&id=u9321b2c7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=304&originWidth=1165&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28311&status=done&style=none&taskId=uba3cbef2-b60b-4048-b48b-44a097e01de&title=&width=1165#crop=0&crop=0&crop=1&crop=1&id=iwKhC&originHeight=304&originWidth=1165&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=shadow&title=)

<a name="d8998abd"></a>
## 2、优缺点

<a name="9ffb436f"></a>
### 2.1、优点：

前后端分离；体验更好；

<a name="41ac5eed"></a>
### 2.2、缺点：

首屏渲染慢；SEO不友好；

<a name="85476e85"></a>
# 服务端渲染

服务端渲染（Server Side Render ）就是将一个Vue组件在服务器端渲染为HTML字符串并发送到浏览器，最后将这些静态标记“激活”为可交互应用程序的过程称为服务端渲染。

**简言之，在浏览器请求页面URL的时候，服务端将我们需要的HTML文本组装好，并返回给浏览器，这个HTML文本被浏览器解析之后，不需要经过 JavaScript 脚本的执行，即可直接构建出希望的 DOM 树并展示到页面中。这个服务端组装HTML的过程，叫做服务端渲染。**

<a name="38164c8b"></a>
# 实现

<a name="f0808a0a"></a>
## 1、小试牛刀

我们先简单实现服务端渲染，通过Vue3自带的 server-renderer 异步生成我们需要的HTML代码。

```javascript
// nodejs服务器  express koa
const express = require('express');
const { createSSRApp } = require('vue');
const { renderToString } = require('@vue/server-renderer');

// 创建express实例
let app = express();

// 通过渲染器渲染page可以得到html内容
const page = createSSRApp({
    data: () => {
        return {
            title: 'ssr',
            count: 0,
        }
    },
    template: `<div><h1>{{title}}</h1>hello world!</div>`,
});

app.get('/', async function (req, res) {
    try {
        // 异步生成html
        const html = await renderToString(page);
        res.send(html);
    } catch (error) {
        res.status(500).send('系统错误');
    }
});

app.listen(9001, () => {
    console.log('9001 success');
});
```

然后通过 node 命令启动，也可以通过 nodemon 启动（关于 nodemon 使用大家可以自行百度）。

```nginx
node .\server\index.js
// or
nodemon .\server\index.js
```

![image.png](https://cdn.nlark.com/yuque/0/2022/png/26115280/1662459254239-e4b81f6b-fed2-4db7-9823-ee46056eff6a.png#clientId=u470e72a7-5795-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=47&id=ueac7dcfb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=47&originWidth=649&originalType=binary&ratio=1&rotation=0&showTitle=true&size=5356&status=done&style=none&taskId=ueb8e12ab-60f5-4cd9-87c1-b2f60747685&title=node%E5%90%AF%E5%8A%A8&width=649#crop=0&crop=0&crop=1&crop=1&id=DnmAj&originHeight=47&originWidth=649&originalType=binary&ratio=1&rotation=0&showTitle=true&status=done&style=none&title=node%E5%90%AF%E5%8A%A8 "node启动")<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/26115280/1662459280999-d0be674b-71e3-49b3-8fd3-eae79a8b60fb.png#clientId=u470e72a7-5795-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=150&id=ua0b11e2d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=150&originWidth=588&originalType=binary&ratio=1&rotation=0&showTitle=true&size=14580&status=done&style=none&taskId=u23823296-0027-4c07-8208-894aee08fdf&title=nodemon%E5%90%AF%E5%8A%A8&width=588#crop=0&crop=0&crop=1&crop=1&id=V8k4R&originHeight=150&originWidth=588&originalType=binary&ratio=1&rotation=0&showTitle=true&status=done&style=shadow&title=nodemon%E5%90%AF%E5%8A%A8 "nodemon启动")

然后打开 [http://localhost:9001/](http://localhost:9001/) 就可以看到：

![image.png](https://cdn.nlark.com/yuque/0/2022/png/26115280/1662459441928-4ad27840-cc43-4c2b-aac3-ed69efdfaf10.png#clientId=u470e72a7-5795-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=171&id=u9dc0eb0a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=171&originWidth=402&originalType=binary&ratio=1&rotation=0&showTitle=false&size=6724&status=done&style=none&taskId=u753eafaa-5e98-4c1b-83b3-03265c0b92b&title=&width=402#crop=0&crop=0&crop=1&crop=1&id=gdyi5&originHeight=171&originWidth=402&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=shadow&title=)<br />右击查看网页源代码后：

![image.png](https://cdn.nlark.com/yuque/0/2022/png/26115280/1662459516858-3c5f84a7-45ff-49dd-8d28-a4c107b4c155.png#clientId=u470e72a7-5795-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=168&id=ueccf9c03&margin=%5Bobject%20Object%5D&name=image.png&originHeight=168&originWidth=560&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7491&status=done&style=none&taskId=u9502a5e0-693a-4bad-bcea-ba9c30aebd8&title=&width=560#crop=0&crop=0&crop=1&crop=1&id=R8nsf&originHeight=168&originWidth=560&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=shadow&title=)<br />从网页源代码可以看出，当浏览器从服务端直接拿到HTML代码后，不需要执行 JavaScript 代码也可以将 **hello world!** 显示在页面上。这就是简单实现SSR的全部过程了。

:::tips
大家可以用 vue-cli 新建一个vue项目，在页面显示 hello world ，然后通过查看网页源代码对比区别！
:::

<a name="b365d999"></a>
## 2、同构项目

前面已经通过简单的案例来演示了SSR，那么如何应用在我们的项目中呢？这里需要引入一个概念：同构。所谓同构，就是让一份代码，**既可以在服务端中执行，也可以在客户端中执行**，并且执行的效果都是一样的，都是完成这个 HTML 的组装，正确的显示页面。也就是说，一份代码，既可以客户端渲染，也可以服务端渲染。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/26115280/1662460055777-742662a5-6298-4365-ba67-84d785daa464.png#clientId=u470e72a7-5795-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=403&id=uc9463d24&margin=%5Bobject%20Object%5D&name=image.png&originHeight=403&originWidth=619&originalType=binary&ratio=1&rotation=0&showTitle=false&size=30088&status=done&style=none&taskId=u77b527b9-e847-4154-b9a4-81931837032&title=&width=619#crop=0&crop=0&crop=1&crop=1&id=VFkJn&originHeight=403&originWidth=619&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=shadow&title=)

<a name="3f376e21"></a>
### 2.1、服务端、客户端配置文件

在根目录新建 webpack 目录，此文件夹主要存放打包配置文件。<br />新建服务端配置文件：server.config.js

```javascript
const base = require('./base.config.js');
const path = require('path');
// webpack插件
const { default: merge } = require('webpack-merge');
const nodeExternals = require("webpack-node-externals");
const { WebpackManifestPlugin } = require('webpack-manifest-plugin');

module.exports = merge(base, {
    mode: "production",
    // 将 entry 指向应用程序的 server 文件
    entry: {
        'server': path.resolve(__dirname, '../entry/server.entry.js')
    },
    // 对 bundle renderer 提供 source map 支持
    devtool: 'source-map',
    // 外置化应用程序依赖模块。可以使服务器构建速度更快，并生成较小的 bundle 文件。
    externals: nodeExternals({
        allowlist: [/\.css$/],
    }),
    output: {
        path: path.resolve(__dirname, './../dist/server'),
        filename: '[name].server.bundle.js',
        library: {
            type: 'commonjs2'   // 构建目标加载模式 commonjs
        }
    },
    // 这允许 webpack 以 Node 适用方式处理动态导入
    // 并且还会在编译 Vue 组件时告知 `vue-loader` 输送面向服务器代码(server-oriented code)。
    target: 'node',
    module: {
        rules: [
            {
                test: /\.js$/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['@babel/preset-env'],
                        plugins: [["@babel/plugin-transform-runtime", {
                            "corejs": 3
                        }]]
                    },

                },
                exclude: /node_modules/
            }
        ]
    },
    // 这是将服务器的整个输出构建为单个 JSON 文件的插件。
    // 服务端默认文件名为 `ssr-manifest.json`
    plugins: [
        new WebpackManifestPlugin({ fileName: 'ssr-manifest.json' }),
    ],
})
```

新建客户端配置文件：client.config.js

```javascript
const base = require('./base.config.js');
const path = require('path');
// webpack插件
const { default: merge } = require('webpack-merge');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CopyPlugin = require("copy-webpack-plugin");

module.exports = merge(base, {
    mode: "production",
    // 对 bundle renderer 提供 source map 支持
    devtool: 'source-map',
    // 将 entry 指向应用程序的 client 文件
    entry: {
        'client': path.resolve(__dirname, '../entry/client.entry.js')
    },
    output: {
        path: path.resolve(__dirname, './../dist/client'),
        clean: true,
        filename: '[name].client.bundle.js',
    },
    plugins: [
        // 通过 html-webpack-plugin 生成client index.html
        new HtmlWebpackPlugin({
            filename: 'index.html',
            template: path.resolve('public/index.html')
        }),
        // 图标
        new CopyPlugin({
            patterns: [
                { from: path.resolve(__dirname, "../public/favicon.ico"), to: path.resolve(__dirname, './../dist/client') },
            ],
        }),
    ]
})
```

最后新建 base 配置文件：base.config.js

```javascript
const path = require('path');
const { VueLoaderPlugin } = require('vue-loader');
module.exports = {
    // 输出
    output: {
        path: path.resolve(__dirname, './../dist'),
        filename: '[name].bundle.js',
    },
    //  loader
    module: {
        rules: [
            { test: /\.vue$/, use: 'vue-loader' },
            {
                test: /\.css$/, use: [
                    'vue-style-loader',
                    'css-loader'
                ]
            },
            {
                test: /\.s[ac]ss$/i,
                use: [
                    "vue-style-loader",
                    "css-loader",
                    "sass-loader",
                ],
            },
            {
                test: /\.js$/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['@babel/preset-env']
                    },

                },
                exclude: /node_modules/
            }
        ],
    },
    plugins: [
        new VueLoaderPlugin(),
    ]
}
```

到此就完成了配置文件，最后贴出打包命令（package.json）

```javascript
{
  "scripts": {
    "build:client": "webpack --config ./webpack/client.config.js",
    "build:server": "webpack --config ./webpack/server.config.js",
    "build": "npm run build:client && npm run build:server"
  },
}
```

<a name="92dcf8bd"></a>
### 2.2、入口（entry）文件

在根目录下创建 entry 目录，此文件夹主要存放入口文件。<br />新建通用入口文件：app.js

```javascript
// 通用入口
// 创建VUE实例
import { createSSRApp } from 'vue';
import App from './../src/App.vue';


export default function () {
    return createSSRApp(App);
}
```

新建服务端入口文件：server.entry.js

```javascript
import createApp from './app';
// 服务器端路由与客户端使用不同的历史记录
import { createMemoryHistory } from 'vue-router'
import createRouter from './router.js'
import createStore from './store';
import { renderToString } from '@vue/server-renderer'

export default context => {
    return new Promise(async (resolve, reject) => {
        const app = createApp();
        const router = createRouter(createMemoryHistory())
        const store = createStore();
        app.use(router);
        app.use(store);

        // 设置服务器端 router 的位置
        await router.push(context.url);
        // isReady 等到 router 将可能的异步组件和钩子函数解析完
        await router.isReady();
        // 匹配路由是否存在
        const matchedComponents = router.currentRoute.value.matched.flatMap(record => Object.values(record.components))
        // 不存在路由，返回 404
        if (!matchedComponents.length) {
            return reject({ code: 404 });
        }
        // 对所有匹配的路由组件调用 `asyncData()`
        Promise.all(matchedComponents.map(component => {
            if (component.asyncData) {
                return component.asyncData({
                    store,
                    route: router.currentRoute.value
                });
            }
        })).then(async (res) => {
            let html = await renderToString(app);

            html += `<script>window.__INITIAL_STATE__ = ${replaceHtmlTag(JSON.stringify(store.state))}</script>`

            resolve(html);
        }).catch(() => {
            reject(html)
        })
    })
}

/**
 * 替换标签
 * @param {*} html 
 * @returns 
 */
function replaceHtmlTag(html) {
    return html.replace(/<script(.*?)>/gi, '&lt;script$1&gt;').replace(/<\/script>/g, '&lt;/script&gt;')
}
```

新建客户端入口文件：client.entry.js

```javascript
// 挂载、激活app
import createApp from './app';
import { createWebHistory } from 'vue-router'
import createRouter from './router.js'
import createStore from './store';
const router = createRouter(createWebHistory())

const app = createApp();
app.use(router);
const store = createStore();

// 判断window.__INITIAL_STATE__是否存在，存在的替换store的值
if (window.__INITIAL_STATE__) {
    // 激活状态数据
    store.replaceState(window.__INITIAL_STATE__);
}
app.use(store)

// 在客户端和服务端我们都需要等待路由器先解析异步路由组件以合理地调用组件内的钩子。因此使用 router.isReady 方法
router.isReady().then(() => {
    app.mount('#app')
})
```

服务端、客户端入口文件完成后，需要对路由（router）和 store 进行共享。<br />新建router.js

```javascript
import { createRouter } from 'vue-router'

const routes = [
    { path: '/', component: () => import('../src/views/index.vue') },
    { path: '/about', component: () => import('../src/views/about.vue') },
]

// 导出路由仓库
export default function (history) {
    // 工厂
    return createRouter({
        history,
        routes
    })
}
```

还需要两个vue组件，分别是index.vue、about.vue

```vue
<script>
    // 声明额外的选项
    export default {
        // 对外暴露方法,执行store
        asyncData: ({ store, route }) => {
            // 触发 action 后，会返回 Promise
            return store.dispatch('asyncSetData', route.query?.id || route.params?.id);
        },
    }
</script>
<script setup>
    import { computed } from 'vue';
    import { useStore } from 'vuex';
    import {generateRandomInteger} from '../utils'

    const clickMe = () => {
        store.dispatch('asyncSetData', generateRandomInteger(1,4));
    }

    const store = useStore();
    // 得到后赋值
    const storeData = computed(() => store.state.data);

</script>
<template>
    <div>
        <div>index</div>
        <button @click="clickMe">点击</button>
        <div style="margin-top:20px">store
            <div>id: {{ storeData?.id }}</div>
            <div>title: {{ storeData?.title }}</div>
        </div>
    </div>
</template>

<style lang='scss' scoped>
</style>
```

```vue
<script setup>
    import { ref } from 'vue'
</script>
<template>
    <div>about</div>
</template>

<style lang='scss' scoped>
</style>
```

为了方便测试，我们将App.vue修改为router-view

```vue
<template>
    <div id="nav">
        <router-link to="/?id=1">Home</router-link> |
        <router-link to="/about">About</router-link>
    </div>
    <router-view />
</template>

<style lang="scss">
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
}

#nav {
    padding: 30px;

    a {
        font-weight: bold;
        color: #2c3e50;

        &.router-link-exact-active {
            color: #42b983;
        }
    }
}
</style>
```

新建store.js

```javascript
// 实例化store
import { createStore as _createStore } from 'vuex';
// 引入数据
import { getData } from './data';

// 对外导出一个仓库
export default function createStore() {
    return _createStore({
        state: {
            // 状态数据
            data: {}
        },
        mutations: {
            // 同步数据
            SET_DATA(state, item) {
                state.data = item;
            }
        },
        actions: {
            // 异步数据
            asyncSetData({ commit }, id) {
                getData(id).then(item => {
                    commit('SET_DATA', item);
                })
            },
        },
        modules: {}
    });
}
```

为了方便测试，我们新建一个data.js文件

```javascript
export function getData(id) {
    const bookList = [
        { id: 1, title: '西游记' },
        { id: 2, title: '红楼梦' },
        { id: 3, title: '水浒传' },
        { id: 4, title: '三国演义' },
    ];

    const item = bookList.find(i => i.id == id);
    return Promise.resolve(item);
}
```

到这里我们就可以构建客户端、服务端了，命令为 **npm run build**，这里的构建会自动构建两次，分别是 build:client、build:server，当终端出现 successfully 时则表示构建成功。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/26115280/1662546544977-5f01ec4c-d993-41c1-a7f4-8cdf4dabb1df.png#clientId=u137e68a6-e804-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=514&id=u4614ed22&margin=%5Bobject%20Object%5D&name=image.png&originHeight=514&originWidth=387&originalType=binary&ratio=1&rotation=0&showTitle=true&size=36054&status=done&style=none&taskId=uff520007-d94a-4ece-907f-c222581fa83&title=%E6%9C%80%E7%BB%88%E7%BB%93%E6%9E%9C&width=387#crop=0&crop=0&crop=1&crop=1&id=VPsHO&originHeight=514&originWidth=387&originalType=binary&ratio=1&rotation=0&showTitle=true&status=done&style=none&title=%E6%9C%80%E7%BB%88%E7%BB%93%E6%9E%9C "最终结果")

<a name="49347c34"></a>
### 2.3、server.js

最后就是修改server/index.js文件，为了区别在此新建index2.js

```javascript
const express = require('express')
const { renderToString } = require('@vue/server-renderer')
const app = express();
const path = require('path');
// 构建结果清单
const manifest = require('./../dist/server/ssr-manifest.json')
const appPath = path.join(__dirname, './../dist/server', manifest['server.js'])
const createApp = require(appPath).default;
const fs = require('fs');

// 搭建静态资源目录
// 这里index必须为false，有兴趣的话可以试试前后会有什么区别
app.use('/', express.static(path.join(__dirname, '../dist/client'), { index: false }));

// 获取模板
const indexTemplate = fs.readFileSync(path.join(__dirname, './../dist/client/index.html'), 'utf-8');

// 匹配所有的路径，搭建服务
app.get('*', async (req, res) => {
    try {
        const appContent = await createApp(req);

        const html = indexTemplate
            .toString()
            .replace('<div id="app">', `<div id="app">${appContent}`)

        res.setHeader('Content-Type', 'text/html');
        res.send(html);
    } catch (error) {
        console.log('error', error)
        res.status(500).send('服务器错误');
    }

})

app.listen(9002, () => {
    console.log('success 9002')
});
```

新建模板文件：index.html

```html
<!DOCTYPE html>
<html lang="en">

    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>vue ssr</title>
    </head>

    <body>
        <div id="app"></div>
    </body>

</html>
```

然后执行 nodemon .\server\index2.js，访问[http://localhost:9002/?id=1](http://localhost:9002/?id=1)<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/26115280/1662545609152-7db2464e-fd43-48a7-9641-b70265f51016.png#clientId=u137e68a6-e804-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=403&id=uc3e85b00&margin=%5Bobject%20Object%5D&name=image.png&originHeight=403&originWidth=1040&originalType=binary&ratio=1&rotation=0&showTitle=false&size=17555&status=done&style=none&taskId=ue771b715-d6ea-425d-8e04-993b667728b&title=&width=1040#crop=0&crop=0&crop=1&crop=1&id=MAvdR&originHeight=403&originWidth=1040&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=shadow&title=)

查看网页源代码<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/26115280/1662545645861-5360bafc-4a23-4827-8375-c5ef3613915b.png#clientId=u137e68a6-e804-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=285&id=u509d59d3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=285&originWidth=1041&originalType=binary&ratio=1&rotation=0&showTitle=false&size=26241&status=done&style=none&taskId=uc13e917b-5017-435d-8b93-5592597a882&title=&width=1041#crop=0&crop=0&crop=1&crop=1&id=GybKe&originHeight=285&originWidth=1041&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)

<a name="25f9c7fa"></a>
# 总结

当浏览器访问服务端渲染项目时，服务端将 URL 传给到预选构建好的 VUE 应用渲染器，渲染器匹配到对应的路由的组件之后，执行我们预先在组件内定义的 asyncData 方法获取数据，并将获取完的数据传递给渲染器的上下文，利用 template 组装成HTML，并将 HTML 和状态 state 一并 send 给浏览器，浏览器加载了构建好的 Vue 应用后，将 state 数据同步到前端的 store 中，并根据数据激活后端返回的被浏览器解析为 DOM 元素的HTML文本，完成了数据状态、路由、组件的同步，使得白屏时间更短，有了更好的加载体验，同时更有利于 SEO 。

<a name="fb507f2c"></a>
# 参考资料

- [Server-Side Rendering (SSR)](https://vuejs.org/guide/scaling-up/ssr.html)
- [从头开始，彻底理解服务端渲染原理(8千字汇总长文)](https://juejin.cn/post/6844903881390964744)
- [彻底理解服务端渲染 - SSR原理](https://github.com/yacan8/blog/issues/30)
