![WEB项目部署发版-通知用户在线更新方案](https://cdn.nlark.com/yuque/0/2022/png/1671893/1650004015475-cfd440c6-85d2-443c-9ca2-644e3de8efff.png)
# 技术背景

项目使用的框架为 vue2，构建工具为 webpack

# 问题

项目迭代频繁，每次上线后，需要通知销售人员（用户）手动刷新，才可使用系统，不然会出现一直使用旧版本以及会出现一些不可预知的错误。

# 原因

vue 框架开发是一种单页 Web 应用（**SPA**）

### 通俗解释

&ensp;&ensp;&ensp;&ensp; 尽在 Web 页面初始化时加载相应的 HTML，JavaScript 和 CSS。一旦页面加载完成，SPA 不会因为用户的操作而进行页面的重新加载或者跳转；取而代之的时利用路由机制实现 HTML 内容的变化，UI 与用户的交互，避免页面的重新加载。

#### 优点

&ensp;&ensp;&ensp;&ensp; 用户体验好、快、内容的改变不需要重新加载整个页面，避免了不必要的跳转和重复渲染；甚至于上面的一点，SPA 相对对服务器压力小；前后端职责分离，架构清晰，前端进行交互逻辑，后端负责数据处理

#### 缺点

&ensp;&ensp;&ensp;&ensp; 初次加载耗时多；为实现单页 Web 应用功能及显示效果，需要在加载页面的时候将 JavaScript、CSS 统一加载，部分页面按需加载，前进后退路由管理。

#### 总结

&ensp;&ensp;&ensp;&ensp; **由于是单页面应用，用户一进入网站，就会把 HTML 与相对应的资源利用浏览器的缓存机制，停留在本地，这样的好处，就是再次进入网站，可以利用缓存机制，可快速进入网站。这也就造成了我们项目版本迭代后，用户因缓存机制，会继续使用旧迭代版本，需要用户手动刷新去服务器请求获取新的 HTML，来使用新迭代版本。**

&ensp;&ensp;&ensp;&ensp; **用户手动刷新获取新版本 html（需要在 nginx 配置，在 nginx.conf 文件做设置，让 index.html 不缓存）**

# 需求

项目迭代后，用户使用会有版本更新的提示或者自动刷新，实现版本升级用户有感知

# 方案

## 原理图


![微信截图_20220415210725.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b84abb019d324855a8c8a0ed16db11a7~tplv-k3u1fbpfcp-watermark.image?)

```html
版本号为打包是自动创建的当前时间戳
接口获取的版本号：是webpack在构建的时候自动创建的版本号，生成 一个版本接口文件
静态页版本号：webpack构建过程中自动创建的版本号，自动写入index.html页面meta信息中
```

## 代码

### nginx

配置-在 nginx.conf 文件做设置，让 index.html 不缓存

```nginx
location = /index.html {
  root	 /usr/share/nginx/html;
  add_header Cache-Control "no-cache, no-store";
}
```

### version.js

创建版本文件

```javascript
const fs = require('fs')  // 引入文件模块
const Timestamp = new Date().getTime()
fs.writeFile('public/version.json', '{"version":' + Timestamp + '}\n', function (err) {
  if (err) {
    return console.log(err)
  }
  })
```

### package.json

vue 构建命令：

```json
"build:prodb": "node version.js && vue-cli-service build --mode prodb"
```

version.js 为版本号自动生成的文件，为请求接口所用。

### vue.config.js

vue2-配置文件-vue.config.js

```javascript
// 获取版本号数据
const bpmVersion = process.env.NODE_ENV === 'production'
|| process.env.NODE_ENV === 'prodb' ? require('./public/version.json') : {
  version: 'dev'
}
module.exports = {
  pages: {
    index: {
      // page 的入口
      entry: 'src/main.js',
      // 模板来源
      template: 'public/index.html',
      // 在 dist/index.html 的输出
      filename: 'index.html',
      version: bpmVersion.version, // 版本号
      // 在这个页面中包含的块，默认情况下会包含
      // 提取出来的通用 chunk 和 vendor chunk。
      chunks: ['chunk-vendors', 'chunk-common', 'index']
    }
  }
```

### index.html

index.html-页面配置版本号字段存储

```html
<meta name="version" content="<%= htmlWebpackPlugin.options.version%>" />
```

### generate-asset-webpack-plugin

也可以通过`generate-asset-webpack-plugin`插件创建

```tsx
const GeneraterAssetPlugin = require('generate-asset-webpack-plugin');
const {
    name
} = require('./package.json');
const timestamp = new Date().getTime();
const version = `v${timestamp}`;

const createJson = function () {
    const data = {
        app: {
            name,
            version
        }
    };

    return JSON.stringify(data);
};
plugins: [
 // 生成版本标识
  new GeneraterAssetPlugin({
      filename: 'app.json',
      fn: (compilation, cb) => {

          cb(null, createJson());
      }
  })
],

chainWebpack: (config) => {
    config
        .plugin('html')
        .tap(args => {
            args[0].name = name;
            args[0].version = timestamp;
            return args
        });
}
```

生成 app.json 文件内容如下:

```json
{"app":{"name":"app-name","version":"v1649818515910"}}
```

**目前是页面的版本号与接口的版本号，都已经准备就绪。剩下来就是两个版本比对。**
关于比对，为了避免不必要的接口比对，特意将比对时机定格在进入路由-加载页面完成之后，这样用户体验效果会好些（纯粹个人体会）。

### router.js

**比对代码**

router.js

```javascript
router.afterEach(async (to, form) => {
  // 生产环境提示升级
  if (process.env.NODE_ENV === 'production' || process.env.NODE_ENV === 'prodb') {
    const checkVersion = await store.dispatch('checkVersion')
    if (!checkVersion) { // 获取的版本号不等时
      Message.warning('正在自动升级新版本...', 2, () => {
        window.location.reload() // 版本不同 刷新 获取最新版本
      })
    }
  }
})
```

### store.js

store.js

```javascript
actions: {
  checkVersion ({ commit, state }) {
    return new Promise(resolve => {
      Axios.get('/version.json?v=' + new Date().getTime(),
                { headers: { 'Cache-Control': 'no-cache' },
                 baseURL: window.location.origin })
      //反正就是要请求到json文件的内容, 并且禁止缓存
        .then(res => {
        const version = res.version
        const clientVersion = Number(document.querySelector('#BPMVersion').content
                                     || '')
        resolve(version === clientVersion)
      })
    })
  }
}
```

## 优化

功能目前是可以达到需求要求，但我们还可以加入版本号的查看，方便知道目前使用的是哪个版本 以及什么时候上线的

### store.js

```javascript
checkVersion ({ commit, state }) {
  return new Promise(resolve => {
    Axios.get('/version.json?v=' + new Date().getTime(),
              { headers: { 'Cache-Control': 'no-cache' },
               baseURL: window.location.origin })
    // 反正就是要请求到json文件的内容, 并且禁止缓存
      .then(res => {
      const version = res.version
      const clientVersion = Number(document.querySelector('#BPMVersion')
                                   .content || '')
      // 以下是查看版本号上线时间 - start
      console.info('%c Environment ' + '%c ' + process.env.NODE_ENV + ' ',
                   'padding: 1px; border-radius: 3px 0 0 3px;
                   color: #fff; background:#606060',
                   'padding: 1px; border-radius: 0 3px 3px 0;
                   color: #fff; background:#42c02e')
                   console.info('%c Build Date ' + '%c ' +
                   dateFtt('yyyy-MM-dd hh:mm:ss',
                   new Date(clientVersion)) + ' ',
        'padding: 1px; border-radius: 3px 0 0 3px;
      color: #fff; background:#606060',
      'padding: 1px; border-radius: 0 3px 3px 0;
      color: #fff; background:#1475b2')
      console.info('%c Last Build Date ' + '%c ' +
                   dateFtt('yyyy-MM-dd hh:mm:ss',
                           new Date(version)) + ' ',
                   'padding: 1px; border-radius: 3px 0 0 3px;
                   color: #fff; background:#606060',
                   'padding: 1px; border-radius: 0 3px 3px 0;
                   color: #fff; background:#1475b2')
                   // -end
                   resolve(version === clientVersion)
    })
  })
    }
```

### **版本号查看效果**

页面提示

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc48c9149e7b418eb6d2aed0220287be~tplv-k3u1fbpfcp-zoom-1.image)

### 控制台输出

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/265ca467c8ad480c8c36e6ce99e66dfd~tplv-k3u1fbpfcp-zoom-1.image)

# 下期预告

&ensp;&ensp;&ensp;&ensp; 针对目前项目中使用 webpack 打包速度越来越慢，希望自己在 vite 工具做多一些尝试，等踩完坑之后。4 月底会出 vite 与 webpack 的对比文章及实例，以及踩到的所有坑分享给大家

# 写到最后

&ensp;&ensp;&ensp;&ensp; 目前代码只适用于 vue2 版本，其里面原理可以借鉴到其他框架。因方案全程只是前端自己参与，所以在每次切换路由的时候，会造成需要请求一次版本号接口的比对，会造成接口的些许浪费吧，目前暂时没有想到更好的办法来解决这个问题，期待后续会有优化这方面的方案吧。

## 团队介绍
高灯科技交易合规前端团队（GFE）, 隶属于高灯科技(北京)交易合规业务事业线研发部，是一个富有激情、充满创造力、坚持技术驱动全面成长的团队, 团队平均年龄27岁，有在各自领域深耕多年的大牛, 也有刚刚毕业的小牛, 我们在工程化、编码质量、性能监控、微服务、交互体验等方向积极进行探索, 追求技术驱动产品落地的宗旨，打造完善的前端技术体系。

- 愿景: 成为最值得信任、最有影响力的前端团队
- 使命: 坚持客户体验第一, 为业务创造更多可能性
- 文化: 勇于承担、深入业务、群策群力、简单开放



**Github：**[github.com/gfe-team](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fgfe-team)

**团队邮箱：**[gfe@goldentec.com](https://link.juejin.cn?target=mailto%3Agfe%40goldentec.com)

作者：GFE团队

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
