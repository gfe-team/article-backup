
## 1. ajax现状
ajax 是大多数 web 应用程序背后的核心技术，它允许页面向 web 服务发出异步请求，因此数据可以不经过页面往返服务器无刷新显示数据
术语 Ajax 不是一种技术，相反，它指的是从客户端脚本加载服务器数据的方法
所有现代浏览器都通过 XMLHttpRequest 构造函数原生支持 XHR 对象
目前有两种Ajax API: 早期 XMLHttpRequest 和现代 Fetch，目前在web端仍然使用针对XMLHttpRequest封装的ajax库最多，比如：axios，jquery，SuperAgent，Request等 
本文基于我们在实际项目中的业务场景，针对axios的封装实现http请求，并剥离出来发布到内网NPM服务器上，目前我们涉及到的Vue3技术项目已经陆续投入使用，并且我们也在持续优化。

## 2. axios介绍
axios 是一个基于 promise 网络请求库，作用于node.js 和浏览器中。它是 isomorphic 的(即同一套代码可以运行在浏览器和node.js中)。在服务端它使用原生 node.js http 模块, 而在客户端 (浏览端) 则使用XMLHttpRequests。
特性：
· 从浏览器创建 XMLHttpRequests
· 从 node.js 创建 [http](http://nodejs.org/api/http.html) 请求
· 支持 Promise API
· 拦截请求和响应
· 转换请求和响应数据
· 取消请求
· 自动转换JSON数据
· 客户端支持防御XSRF

axios 是一个轻量的HTTP客户端，它基于 XMLHttpRequest 服务来执行 HTTP 请求，支持丰富的配置，支持 Promise，支持浏览器端和 Node.js 端。自Vue2.0起，Vue作者尤雨溪宣布取消对 vue-resource 的官方推荐，转而推荐 axios。现在 axios 已经成为大部分 Vue 开发者的首选。

## 3. 封装http
### 3-1. 抽象axios，创建Class  GAxios
#### 3-1-1.定义参数类型
针对官方给出的常用配置, 再结合我们在项目中的一些常用配置
```typescript
// http请求配置
export interface RequestOptions {
  // 请求参数拼接到url
  joinParamsToUrl?: boolean;
  // 格式化请求参数时间
  formatDate?: boolean;
  //  是否处理请求结果
  isTransformRequestResult?: boolean;
  // 是否加入url
  joinPrefix?: boolean;
  // 接口地址， 不填则使用默认apiUrl
  apiUrl?: string;
  // 错误消息提示类型
  errorMessageMode?: 'none' | 'modal';
  // 自定义处理请求结果
  customTransformResult?: Function | null;
}

// Axios初始化配置,  AxiosRequestConfig对象为Axios的请求配置
export interface CreateAxiosOptions extends AxiosRequestConfig {
  prefixUrl?: string;
  transform?: AxiosTransform;
  requestOptions?: RequestOptions;
  messageError?: Function | null;
  modalError?: Function | null;
}
```
#### 3-1-2.创建GAxios
基于以上配置信息,我们首先创建axios的类 GAxios(名字自己定义,主要用于区分Axios对象)
```typescript
export class GAxios {
  // TODO ......
}
```
GAxios对象内部应该包含我们针对Axios对象的一系列操作:示例初始化、配置项、拦截器、请求方法等等。
```typescript
export class GAxios {
  private axiosInstance: AxiosInstance;
  private options: CreateAxiosOptions;

  constructor(options: CreateAxiosOptions) {
    this.options = options;
    this.axiosInstance = axios.create(options);
  }
  
  // 创建axios实例
  private createAxios(config: CreateAxiosOptions): void {
    this.axiosInstance = axios.create(config);
  }
  
  // 拦截器配置
  private setupInterceptors(): void {
    // TODO ......
  }
  
  // 请求方法
  request<T>(config: AxiosRequestConfig, options?: RequestOptions): Promise<T> {
    // TODO ......
    
    return new Promise((resolve, reject) => {};
  }
}
```

#### 3-1-3.拦截器配置
拦截器定义四种:请求/响应拦截、请求/响应错误拦截
```typescript
// 请求之前的拦截器
requestInterceptors?: (config: AxiosRequestConfig) => AxiosRequestConfig;

// 请求之后的拦截器
responseInterceptors?: (res: AxiosResponse<any>) => AxiosResponse<any>;

// 请求之前的拦截器错误处理
requestInterceptorsCatch?: (error: Error, options?: CreateAxiosOptions) => void;

// 请求之后的拦截器错误处理
responseInterceptorsCatch?: (error: Error, options?: CreateAxiosOptions) => void;
```
#### 3-1-4.定义请求方法
```typescript
class GAxios {
    // 请求方法
    request<T>(config: AxiosRequestConfig, options?: RequestOptions): Promise<T> {
        // Merge参数opt
        
        // 自定义请求转换

        // 取消重复请求
      
        // 拦截器

        // 请求方法
        return new Promise((resolve, reject) => {
            this.axiosInstance
                .request(opt)
                .then((res) => {
                    
                    resolve(res);
                })
                .catch((e: Error) => {
                    
                    reject(e);
                });
        });
    }
}
```
### 3-2. 创建Class AxiosHttp
```typescript
// GAxios的具体实现
class AxiosHttp {
    private http: GAxios = null;
}
```
#### 3-2-1.定义默认参数
```typescript
      {
            timeout: 6 * 10 * 1000,// 1min
            // 基础接口地址
            baseURL: '',
            // 接口可能会有通用的地址部分，可以统一抽取出来
            prefixUrl: '',
            headers: { 'Content-Type': ContentTypeEnum.JSON },
            // 数据处理方式
            transform,
            // 配置项，下面的选项都可以在独立的接口请求中覆盖
            requestOptions: {
                // 默认将prefix 添加到url
                joinPrefix: true,
                // 需要对返回数据进行处理
                isTransformRequestResult: true,
                // post请求的时候添加参数到url
                joinParamsToUrl: false,
                // 格式化提交参数时间
                formatDate: true,
                // 消息提示类型
                errorMessageMode: 'none',
                // 接口地址
                apiUrl: '',
                // 自定义数据转换
                customTransformResult: data => data,
                // 自定义消息框方法
                messageBox: data => data
            }
        }
```
#### 3-2-2.创建http对象
在AxiosHttp类内部实例化GAxios对象
```typescript
this.http = new GAxios({...});
```
#### 3-2-3.实现常用请求方法
在AxiosHttp类内部提供我们常用的几种请求方法: get、post、put、delete
除此之外我们仍要提高http.request方法,以满足更多场景需要.
#### 3-2-4.设置token方法
token信息通常都是携带在请求头中, 所以我们需要提供自定义设置Heder方法
```typescript
public setToken(token: string): void {
  this.http.setHeader({
      Authorization: `Bearer ${token}`
  });
}
```
#### 3-2-5.返回内置defHttp对象
```typescript
public getInstance(): GAxios {

        return this.http;
}
```
通过获取内置Axios的对象可以实现更多不同场景需要
### 3. 在vue3环境中导出
vue3中都是通过app.use方式注册组件,这里我们实现当前http库的注册与使用.当我们调用app.use方式时,实际执行的是当前组件的install方法
#### 3-3-1.Install方法
```typescript
const uniqueKey = Symbol();
const install = (app: App, config: Partial<CreateAxiosOptions>): void => {

    app.provide(uniqueKey, new AxiosHttp(config));
}


export default {
    install
}
```

#### 3-3-2.通过provide，inject使用
```typescript
export default defineComponent({
  setup() {
      const axiosHttp = useAxiosHttp();
      // axiosHttp 即为注入的http对象
      
      // 在实际开发场景中,axiosHttp可以在项目入口处挂载到全局对象
      // 例:app.config.globalProperties.$http = axiosHttp;
  }
});
```

#### 3-3-3.发布当前package
在测试完Demo中用例后, 可以准备[发布](https://www.jianshu.com/p/4c009867060a)我们的第一版package了,这里以私有npm服务器为例.

1. 通过nrm改签当前源为内网服务器.
1. 登录账号npm login
1. 发布npm publish


## 4. 常见组件开发规范介绍
通常一套组件、library等的研发过程也是比价繁琐的,不同的团队会根据时间情况进行取舍.
即使如何简化像分支管理、命名规范、通讯规则等这些都是避免不了的.

![UI组件研发流程.png](https://cdn.nlark.com/yuque/0/2022/png/26114887/1655113759223-b9bfbe10-89ed-4eb6-bfd4-d1c77829d6fc.png#clientId=u722f626c-c962-4&crop=0&crop=0&crop=1&crop=1&from=ui&id=u9ea50331&margin=%5Bobject%20Object%5D&name=UI%E7%BB%84%E4%BB%B6%E7%A0%94%E5%8F%91%E6%B5%81%E7%A8%8B.png&originHeight=1690&originWidth=393&originalType=binary&ratio=1&rotation=0&showTitle=false&size=96652&status=done&style=none&taskId=ub10ec876-4535-4254-809c-13f558f2897&title=)

## 5. 实践
在我们的业务项目(vue3)中,安装我们已发布的包. 
[通过.npmrc文件指定@gfe开头的包源地址](https://blog.csdn.net/huzhenv5/article/details/107999532)
`npm install @gfe/request --save`
`yarn add @gfe/request`
## 6. 接入使用
```typescript
import { useLogout } from "@/hooks/useLogout";
import router from "@/router";
import { BASE_API } from '@/utils/env';
import AxiosHttp from '@gfe/request';
import { message, Modal } from 'ant-design-vue';
import { App } from 'vue';

export function setupHttp(app: App<Element>) {

    app.use(AxiosHttp, {
        baseURL: BASE_API,
        messageError: async (msg: string, status: number) => {
            if (`${status}` == `401`) {
                await useLogout();
                router.replace({ name: 'Login' });
            }
            message.error(msg);
        },
        modalError: (msg: string) => {
            Modal.error({
                title: '错误提示',
                content: msg
            })
        }
    });
}
```
## 7. 总结

1. 当前组件库文档使用vuepress展示,源码在/docs文件夹下
1. 配置仓库认证信息.npmrc，用于私有仓库读取或上传npm包，不用每次发布都执行npm login操作
```typescript
always-auth=true
_auth="用户名:密码"的base64编码
```

3. 源码地址
