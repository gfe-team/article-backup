今天带来一篇不一样的组件库开发文章，强烈推荐先收藏后阅读。
当你看了很多的从0到1开发的组件库的文章后会发现跟你主流使用的组件库总是天差地别，但又不是不能用。这样的组件库其实就严重缺少了工程化的支撑，缺少了全局的考虑。如果你已经看过了那些文章，那么你将通过这次介绍的6件事让你的组件库提升到更高的高度~
## 1. 留一个好的印象给Contributor
### 1.1 背景描述
在 Workspace 模式的项目中各个子包都会有一定的相互依赖存在，当你未能构建某个特定子包时就会造成另外一个子包无法启动，为了避免这样的问题出现，也为了给Contributor留下一个好的印象，我决定在你拿到项目安装依赖之后就帮你把该做的事情都做好，达到开箱即可体验的目的。
### 1.2 回归案例
在这次的案例中按 Workspace 模式开发的组件库项目包含了有ui、docs和example三个子包，其中docs和example都依赖ui子包构建后的产物，那么我需要做的就是在你安装项目依赖后自动实现ui包构建。
### 1.3 技术调研
在查看[NPM文档](https://docs.npmjs.com/cli/v7/using-npm/scripts#npm-install)后得知在执行依赖安装前后其实都会触发特定的钩子，我将利用这一特性，在触发到`postinstall`钩子后自动执行ui包构建。
### 1.4 实现过程
在 Workspace 下的 package 下添加构建ui包的脚本；

- 通过 `--filter` 来限制脚本执行的子包集；
```json
{
    "scripts": {
        "postinstall": "pnpm -r --filter=@gfe/ui run build"
    }
}
```
![auto-post-install.gif](https://cdn.nlark.com/yuque/0/2022/gif/2373519/1667989730290-70740a1b-0f1e-441f-a68a-67d46b936f85.gif#averageHue=%231e1e1e&clientId=u455f52d9-8b64-4&crop=0&crop=0&crop=1&crop=1&from=drop&id=u3396a9bc&margin=%5Bobject%20Object%5D&name=auto-post-install.gif&originHeight=486&originWidth=1350&originalType=binary&ratio=1&rotation=0&showTitle=false&size=91213&status=done&style=none&taskId=ucc8c80da-5375-41e6-aefb-ef587739492&title=)
## 2. Build Tools API 更适合
### 2.1 背景描述
在一些常规的项目中通常只需要应用到`webpack`或者`vite`的配置文件就可以让项目正常的运行及构建，当你的项目变得复杂的时候就或不停的往配置文件中增加更多的`loader`或`plugins`来充实构建工具的功能，在组件库开发的时候如果你仅使用配置文件来实现的话这一切将变得很复杂，所以就需要应用到构建工具提供的API来在构建脚本的函数中动态调用，实现更加灵活的执行。
### 2.2 回归案例
在这次的案例中将利用Vite构建工具来实现每个组件的打包，在后期使用组件时既可以引入全量的组件包又可以选择性的使用某一个指定的组件包。在组件包中我们还将利用脚本来充实组件包的内容达到更好的使用体验，这也是纯配置文件不那么容易搞定的事情。
### 2.3 实现过程
#### 2.3.1 组件全量编译
全量编译不需要配置过多的信息，因为它会按照你在vite配置执行编译，只需要调用vite提供的`build`函数就可以了~
```typescript
import { build } from "vite";

const buildAll = async () => {
    // 全量打包
    await build();
}

buildAll();
```
#### 2.3.2 组件分包编译
分包编译就需要通过遍历组件目录得到符合组件包特征的组件列表，在遍历组件列表的时候实时配置组件编译选项再执行`build`函数。
```typescript
// 按组件分别打包
const srcDir = path.resolve(__dirname, "../src/");

// 提取包含index.ts入口的组件目录
const componentsDir = fs.readdirSync(srcDir).filter(filename => {
    const componentDir = path.resolve(srcDir, filename);
    const isDir = fs.lstatSync(componentDir).isDirectory();
    return isDir && fs.readdirSync(componentDir).includes("index.ts");
})

// 遍历需要打包的组件分别打包
for (let name of componentsDir) {
    const outDir = path.resolve(output, name)

    const customConfig = {
        lib: {
            entry: path.resolve(srcDir, name),
            name,
            fileName: "index",
            formats: ["esm", "umd"]
        },
        outDir,
    }

    await build({ build: customConfig, } as InlineConfig);
};
```
#### 2.3.3 调整package信息
在全量构建完成和每个分包构建完成都应该给它们维护其专属的package信息，在使用组件时将很有用。
```typescript
// 输出package信息函数
function outputPkgFile(filepath, pkg) {
    fs.outputFile(path.resolve(filepath, `package.json`), JSON.stringify(pkg, null, 2), `utf-8`);
}
```
全量构建的package信息不需要全部重写，可以导入ui包下的package信息，在此基础上进行修改，补充main、module、types信息，分别指向umd输出产物路径、esm输出产物路径、.d.ts输出产物路径（在组件库的使用体验很重要一节会详细说明）；
```typescript
outputPkgFile(output, {
    ...require("../package.json"),
    main: `gfe-ui.umd.js`,
    module: `gfe-ui.esm.js`,
    types: `gfe-ui.d.ts`,
});
```
每个分包构建后的package信息除了上面的三个属性外name属性也需要调整成每个组件自己的名称，因为它属于这个组件；
```typescript
outputPkgFile(outDir, {
    name: `@gfe-ui/${name.toLocaleLowerCase()}`,
    main: `index.umd.js`,
    module: `index.esm.js`,
    types: `../types/${name}/index.d.ts`
});
```
#### 2.3.4 输出自述文档
自述文档在没有项目的根目录下都会有一份，在ui包的根目录下的自述文档描述了组件库的安装、导入及使用的方式，在输出的产物中也需要包含这个文件；
```typescript
fs.copyFileSync(path.resolve("./README.md"), path.resolve(output, `README.md`));
```
输出结构：
```typescript
dist                                   
├─ Button                     
│  ├─ styles                  
│  │  ├─ index.css            
│  │  └─ style.css            
│  ├─ index.esm.js            
│  ├─ index.esm.js.map        
│  ├─ index.iife.js           
│  ├─ index.iife.js.map       
│  ├─ index.umd.js            
│  ├─ index.umd.js.map        
│  └─ package.json                                                                  
├─ gfe-ui.d.ts                
├─ gfe-ui.esm.js              
├─ gfe-ui.esm.js.map          
├─ gfe-ui.iife.js             
├─ gfe-ui.iife.js.map         
├─ gfe-ui.umd.js              
├─ gfe-ui.umd.js.map          
├─ package.json               
└─ README.md                  
```
## 3. 顺手的工具远胜与一切
### 3.1 背景描述
前端项目从不需要构建到使用`webpack`、`vite`构建，还有最近新出现的`Turbopack`，无论选择哪种构建工具都会遇到某一些文件是没办法直接处理的，那么首先就会去寻找对应的插件来看是否可能满足要求，我想说的是其实没有那么必要拘泥于使用一种构建工具，有更顺手的构建工具配合将是一种不错的选择。
### 3.2 回归案例
在这次的案例中构建组件的主要是基于`vite`来做的，但是在less模块的构建中我选择了相对熟悉的`gulp`来编写构建脚本，通过遍历组件文件夹来提取到所有的less模块文件，在分别注册`gulp`任务，最后交由`gulp`来统一执行，定制化程度高，我想不到什么样的`vite`插件可以这么灵活的实现；
### 3.3 实现过程

1. 配置`gulp`构建`less`模块的脚本，使用到了`gulp-less`模块支持，如果需要对less构建完的结果做进一步处理，比如要压缩，就可以继续使用`pipe`处理，因为`gulp`基于 node 强大的流(stream)能力：
```typescript
import gulp from "gulp";
import less from "gulp-less";

// gulp core code
const register = (name, src, dist) => {
  gulp.task(name, function () {
    return gulp.src(src)
      .pipe(less())
      .pipe(gulp.dest(dist));
  });
}

// TODO register task

// 导出所有 gulp task
export default gulp.series(...tasks);
```

2. 确定组件文件的和输出文件的路径，如果在`vite.config.ts`中指定了`outDir`选项将会被优先使用：
```typescript
const srcDir = path.resolve(__dirname, "../src/");
const output = path.resolve(require("../vite.config.ts").build?.outDir || "../dist");
```

3. 通过组件包特征（包含入口`index.ts`文件）来确定所有组件的目录：
```typescript
const componentsDir = fs.readdirSync(srcDir).filter(filename => {
  const componentDir = path.resolve(srcDir, filename);
  const isDir = fs.lstatSync(componentDir).isDirectory();
  return isDir && fs.readdirSync(componentDir).includes("index.ts");
})
```

4. 最后遍历组件注册任务：
```typescript
register(`${name}Task`, 
    path.resolve(entryDir, 'style.less'), 
    path.resolve(outDir, 'styles')
);
```
![build-less.gif](https://cdn.nlark.com/yuque/0/2022/gif/2373519/1668060539821-d1fce004-bb28-4ef2-a78c-96881409eac3.gif#averageHue=%231e1e1e&clientId=ua4bd44d8-ef9d-4&crop=0&crop=0&crop=1&crop=1&from=drop&id=uf474c0a1&margin=%5Bobject%20Object%5D&name=build-less.gif&originHeight=502&originWidth=1263&originalType=binary&ratio=1&rotation=0&showTitle=false&size=196534&status=done&style=stroke&taskId=u9aa0b787-5a1d-439a-a4ac-4cd51fea271&title=)
## 4. 组件库的使用体验很重要
### 4.1 背景描述
在编码中使用组件库的时候你是因为什么原因去不停的翻找组件文档的？是因为组件名不会写？还是因为组件属性忘记了？还是因为其他？解决这些不大不小的问题也是提升组件库使用体验很关键一点，那么你有没有考虑找一些VSCode插件实时提示组件的一些信息呢？或者使用快捷键来生成代码片段？
### 4.2 回归案例
在组件库开发初期就决定使用Ts作为组件库开发的基本语言，良好的利用其类型系统的优势来达到后期使用组件时的便利性，那么就需要为组件生成它所对应的类型文件并且正确的进行配置，在SFC组件中使用时还需要配置`Volar`插件使用。
### 4.3 实现过程
#### 4.3.1 生成dts文件：
利用`vite-plugin-dts`插件来实现dts文件的生成，插件的配置不建议配置到`vite.config.ts`中，在分包构建的时候发现依然会触发一次该插件的执行，快速的组件编译将导致内存被快速消耗殆尽，建议在全局构建时调用`build`函数时动态传入。
```typescript
import dts from "vite-plugin-dts";

// 全量打包
await build({
    // 支持生成.d.ts类型文件
    // 修改为仅全量打包阶段生成dts
    plugins: [
        dts({
            outputDir: "./dist/types",
            insertTypesEntry: false, // 插入TS 入口
            copyDtsFiles: true, // 是否将源码里的 .d.ts 文件复制到 outputDir
        }) as unknown as PluginOption
    ]
});
```
#### 4.3.2 入口文件改造
上面的配置我们禁用的入口文件的插入，因为插件生成的入口不太符合我们的要求，我们需要进一步的利用脚本改造，使得组件的类型可以在使用是得到识别；

1. 确认全量包的package信息，让types属性与类型入口文件对应；
2. 确认每个分包的package信息，让types属性与每个组件的类型入口文件对应；
3. 定义类型入口文件模板，这里应用了 Handlebars 模板引擎：
```typescript
export * from './types/index'
import GFEUI from './types/index'
export default GFEUI

declare module 'vue' {
    export interface GlobalComponents {
        {{#each components}}
        {{name}}: typeof import("./types/index").{{component}},
        {{/each}}
    }
}
```

4. 获取组件信息列表生成模板所需的元数据，这里使用了import动态导入组件入口文件进行分析获取：
```typescript
async function getComponents(input) {
    const entry = await import(`file://${input}`);
    return Object.keys(entry)
        .filter(k => k !== 'default')
        .map(k => ({
            name: entry[k].name,
            component: k,
        }))
}
```

5. 利用模板引擎替换生成入口文件并输出到指定位置：
```typescript
function generateCode(meta, filePath: string, templatePath: string) {
    if (fs.existsSync(templatePath)) {
        const content = fs.readFileSync(templatePath).toString();
        const result = handlebars.compile(content)(meta);
        fs.writeFileSync(filePath, result);
    }
    console.log(`🚀 ${filePath} 创建成功`)
}
```

6. 脚本编写完成后可以将脚本的执行放置到全量构建之后，因为这个时候既生成的各个dts文件，有利用脚本生成了类型入口文件，在SFC组件中使用指定了类型入口的组件库时将获得类型的提示及约束的效果。

![auto-create-dts-entry.gif](https://cdn.nlark.com/yuque/0/2022/gif/2373519/1668041897665-13070746-edc0-4e0e-a194-8ed1075915e3.gif#averageHue=%23201f1f&clientId=u36a04b9c-949b-4&crop=0&crop=0&crop=1&crop=1&from=drop&id=u5df634c2&margin=%5Bobject%20Object%5D&name=auto-create-dts-entry.gif&originHeight=486&originWidth=1350&originalType=binary&ratio=1&rotation=0&showTitle=false&size=280026&status=done&style=stroke&taskId=ua0dd090c-ffc5-4ee1-a86f-20b8937b85f&title=)
![types-prompt.gif](https://cdn.nlark.com/yuque/0/2022/gif/2373519/1668060443670-cae2a5a8-89b2-477f-97bf-34a8e22b57f7.gif#averageHue=%2320201f&clientId=ua4bd44d8-ef9d-4&crop=0&crop=0&crop=1&crop=1&from=drop&id=ub2c1dc64&margin=%5Bobject%20Object%5D&name=types-prompt.gif&originHeight=342&originWidth=664&originalType=binary&ratio=1&rotation=0&showTitle=false&size=86516&status=done&style=stroke&taskId=u9002f2da-8262-4cb2-9626-7e26f7eb117&title=)
## 5. 高效维护属性列表文档
### 5.1 背景描述
在使用开源社区的组件库的时候主要关注的就是怎么安装？什么效果？有哪些属性？每个组件的属性列表将决定了你是否会使用这个组件库，因为属性列表决定了功能是否可以实现（轻松），而效果则是次要的。那你有没有想过一个组件库那么多的组件，每个组件又有那么多的属性你会怎么样来维护呢？
### 5.2 回归案例
在这次组件库开发时我选择了使用`Babel`来对每个组件中定义的属性列表文件进行解析，得到属性列表文件中定义的属性和属性上附带的注释，将解析到的数据组合整理成组件库文档中属性列表的语法格式，并输出覆盖旧的属性列表，那么将这段脚本添加到组件库编译后的流程中就实现了每次构建完组件库后对应文档的属性列表也就是最新的。
### 5.3 实现过程
#### 5.3.1 AST 结构分析：
下面两张图是通过 [AST Explorer](https://astexplorer.net/) 工具对组件源码片段的分析；

1. 在第一张图中可以看到`type`为`Identifier`的对象中`name`属性存放了组件选项的名称；
2. 在第二张图中可以看到`type`为`CommentBlock`的对象中`value`属性存放了选项上配置的所有注释数据；
3. 这两块内容及其它一些数据共同组成了`type`为`ExportNamedDeclaration`表达式；

![image.png](https://cdn.nlark.com/yuque/0/2022/png/2373519/1666709065852-29757c05-8f30-4182-9a7d-744cff59e17e.png#averageHue=%23f5f5f4&clientId=u623d1471-1055-4&crop=0&crop=0&crop=1&crop=1&errorMessage=unknown%20error&from=paste&height=929&id=u28030b0a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=929&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=41061&status=error&style=stroke&taskId=u05e5f3b2-5c15-406d-9067-91934e4108b&title=&width=1920)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/2373519/1666709153583-3691ccde-76b1-4e7e-b27b-0497cb9fc2ae.png#averageHue=%23f5f5f4&clientId=u623d1471-1055-4&crop=0&crop=0&crop=1&crop=1&errorMessage=unknown%20error&from=paste&height=929&id=uc0d446ff&margin=%5Bobject%20Object%5D&name=image.png&originHeight=929&originWidth=1920&originalType=binary&ratio=1&rotation=0&showTitle=false&size=41417&status=error&style=stroke&taskId=u2c13da67-fbeb-4351-a623-badc3b5ee09&title=&width=1920)
#### 5.3.2 插件开始前初始化容器：
在`pre()`函数中可以使用`this.set(key, value)`方式来存储`attributeList`数据，在这个函数中可以将MD表格的`header`和`split line`部分先存储起来；
```typescript
// 插件执行前初始化存储容器
pre(this: PluginPass, file: BabelFile) {
  console.info(
    `\u001b[33m将要生成的组件属性列表将合并至对应组件库文档.\u001b[39m\n`
  )
  this.set('attributeList', [
    ['属性名', '说明', '类型', '可选值', '默认值'],
    ['------', '----', '----', '-----', '-----']
  ]);
}
```
#### 5.3.3 存储每一次解析数据：
因为通过AST工具分析可以看到我们的每块属性都属于一个`ExportNamedDeclaration`，那么在 visitor 中就配置`ExportNamedDeclaration(path, state)`函数来解析，在每次进入到`ExportNamedDeclaration(path, state)`函数时需要通过`state`获取到已经存储的属性列表数据，并在每次操作结束后将新解析到的数据再存储到`state`中；
```typescript
visitor: {
  ExportNamedDeclaration(
    path: NodePath<t.ExportNamedDeclaration>,
    state: PluginPass
  ) {
    const attributeList = state.get('attributeList');

  	// TODO
    
    state.set('attributeList', attributeList);
  }
}
```
#### 5.3.4 解析注释数据：
解析注释需要使用到`doctrine`模块，在Babel将组件源码解析为对应的AST结构后，注释信息将存储在对应的`leadingComments`属性中，通过`doctrine`模块提供的`parse`函数将解析注释信息为方便操作的对象模式；
接着需要定义一个`Comment`类型结构来存储每一个选项的注释数据；
```typescript
import doctrine from "doctrine";

/**
 * 定义注释所对应的对象类型
 */
type Comment = {
  describe: string
  type: any
  options?: any
  default?: any
} | undefined

/**
 * 使用doctrine模块解析在AST结构leadingComments中存在的每一个元素
 * @param comment 
 * @returns 
 *
const parseComment = (comment) => {
  if (!comment) {
    return;
  }
  return doctrine.parse(comment, {
    unwrap: true,
  });
};
```
#### 5.3.5 完成注释数据解析&属性列表数组组合：
通过Debug发现每次进入`ExportNamedDeclaration(path, state)`函数后通过`path.node.leadingComments`取出的数据将增加当前解析到选项的注释数据，为避免重复处理可以通过一个`skip`标识来跳过已经处理过的数据；

将解析到的注释数据和`declaration`中取到的选项名称组合到数组中并存储到`state`，完成一次解析~
```typescript
visitor: {
  ExportNamedDeclaration(
    path: NodePath<t.ExportNamedDeclaration>,
    state: PluginPass
  ) {
    const attributeList = state.get('attributeList');
    let _comment: Comment = undefined;
    path.node.leadingComments?.forEach(comment => {
      if (!Reflect.has(comment, "skip")) {
        // 解析注释数据
        const tags = parseComment(comment.value)?.tags;
        _comment = {
          describe: tags?.find(v => v.title === "gDescribe")?.description || '--',
          type: tags?.find(v => v.title === "gType")?.description || '--',
          options: tags?.find(v => v.title === "gOptions")?.description || '--',
          default: tags?.find(v => v.title === "gDefault")?.description || '--',
        };
        Reflect.set(comment, "skip", true);
      }
    });
    attributeList.push([
      (path.node.declaration as t.TypeAlias).id.name.substr(1).toLocaleLowerCase(),
      _comment!.describe,
      _comment!.type,
      _comment!.options,
      _comment!.default,
    ])
    state.set('attributeList', attributeList);
  }
}
```
#### 5.3.6 拼装属性列表文档&合并到组件文档
下面通过“|”分割的数据组成的格式即为MD文档表格的风格；
```markdown
属性名 | 说明 | 类型 | 可选值 | 默认值
------ | ---- | ---- | ----- | -----
size | 尺寸 | string | "large"<br> "default"<br> "small" | --
```
在 **Babel **解析期间将组件属性和属性上的注释组合成一个属性列表的二维数组，通过`transformMarkdown`函数将这个二维数组通过`join(' | ')`函数组合成目标风格；
```
/**
 * 整合属性列表表格
 * @param attributeList 
 * @returns 
 */
const transformMarkdown = (table: Array<Array<String>>) => table.map(v => v.join(' | ')).join('\n');
```
#### 5.3.7 输出Markdown表格：
在`post()`函数中取出最终得到的属性列表数据，通过提供的`transformMarkdown()`函数将属性列表二维数组转换成MD表格风格的文本；
```typescript
// 将所有的命名导出表达式解析完成后，将容器中存储的数据转换为MD文件并输出
post(this: PluginPass, file: BabelFile) {
  const attributeList = this.get('attributeList');
  const output = transformMarkdown(attributeList);
  const root = path.parse(file.opts.filename).dir;
  fs.writeFileSync(path.join(root, 'api-docs.md'), output);
}
```
#### 5.3.8 合并表格至原组件文档：
默认属性列表为文档最后一部分，通过分割文本取出无属性列表的第一部分文档拼接新的属性列表后重写组件文档，将`rewriteCompDocs`函数重新加到`post()`函数的末尾将实现合并功能~
```typescript
const rewriteCompDocs = (root: string, output: string) => {
  const compName = path.parse(root).name;
  const compPath = path.resolve(__dirname, `../../docs/components/${compName}`);
  if (fs.existsSync(compPath)) {
    const compDocs = path.resolve(compPath, 'index.md');
    const raw = fs.readFileSync(compDocs, { encoding: 'utf-8' });
    const noAttrPart = raw.split(/^##[\s][\S]{1,}[\s]属性列表$/gm)[0];
    const content = `${noAttrPart.trimEnd()}\n\n## ${compName} 属性列表\n${output}`;
    fs.writeFileSync(compDocs, content, { encoding: 'utf-8' });
  }
}
```
![auto-create-api.gif](https://cdn.nlark.com/yuque/0/2022/gif/2373519/1668060316807-bd445390-1a70-4e47-8018-389c538afde7.gif#averageHue=%231e1e1e&clientId=ua4bd44d8-ef9d-4&crop=0&crop=0&crop=1&crop=1&from=drop&id=u0c75b061&margin=%5Bobject%20Object%5D&name=auto-create-api.gif&originHeight=512&originWidth=1375&originalType=binary&ratio=1&rotation=0&showTitle=false&size=99410&status=done&style=stroke&taskId=u61a584b5-994a-4fa8-9546-39a260184c2&title=)
## 6. 保证组件包一致性的关键
### 6.1 背景描述
当你参与到一个项目的开发过程中后你在做一块新的功能的时候总是会找一下以前的代码里有没有这样的踪影，有一部分原因是为了不重复编写代码，有一部分原因是想照搬照抄，但是还有一部分原因是想与现有的代码保持一致，就比如说组件的命名方式，目录的命名方式等。在你看到的所有糟糕的代码都可能是各写各的这种风格导致的。
### 6.2 回归案例
在这次组件库开发案例中我在准备好第一个组件的结构和风格后就着手准备组件模板和命令生成组件脚本编写，通过在终端交互输入组件的名称就可以生成规范的组件包，包括了组件包的内容、命名的风格等；

- 组件包目录示例：
```latex
Button                
├─ __test__           
│  └─ Button.spec.ts  
├─ api-docs.md        
├─ Button.tsx         
├─ index.ts           
├─ interface.ts       
├─ style.less         
└─ types.ts
```
### 6.3 技术调研
Plop.js 是一个微型生成器框架，其内置了Handlebars引擎，通过简单的编写得到获取终端交互的能力，通过编写Handlebars模板来固定组件包各个文件的风格，很适合这样的应用场景；
### 6.4 实现过程
通过组件包中其中一个模板的编写来体验 Plop.js 的使用；

1. 定义`plopfile.js`文件，使用统一入口注册`Generator`：
```typescript
const componentGenerator = require('./plop-templates/component/prompt')

module.exports = function(plop) {
  plop.setGenerator('component', componentGenerator)
}
```

2. 定义`component`的`Generator`，完成收集组件名称和组件文件输出的目的：
```javascript
"use strict";

module.exports = {
    description: "generate a component",
    prompts: [
        {
            type: "input",
            name: "name",
            message: "Tips：Component name should be UpperCamelCase，\nPlease enter a component name：",
            validate: (v) => {
                return !v || v.trim() === "" ? `${name} is required` : true;
            },
        },
    ],
    actions: (data) => {
        const name = "{{ titleCase name }}";
        const actions = [
            {
                type: "add",
                path: `packages/ui/src/${name}/${name}.tsx`,
                templateFile: "plop-templates/component/component.hbs",
                data: { name },
            },
        ];
        return actions;
    },
};
```

3. 编写`component.hbs`，替换了组件和样式class的命名，且命名方式使用`titleCase`模式：
```html
import { defineComponent } from "vue";
// import {  } from "./interface";

export default defineComponent({
    name: '{{ titleCase name }}',
    setup(props, { slots }) {
        return () => (<div class={'g-{{ kebabCase name }}'}>
            <h1>{{ titleCase name }}</h1>
        </div>)
    }
})
```

4. 当这一切都搞定后就可以运行`plop`来启动脚本了，最好还是将脚本执行配置到package中使用：
```json
{
    "scripts": {
        "new": "plop"
    }
}
```
![quickly-create-components.gif](https://cdn.nlark.com/yuque/0/2022/gif/2373519/1668060143349-098ed8f6-0191-4582-9292-b1a003997610.gif#averageHue=%231e1e1e&clientId=ua4bd44d8-ef9d-4&crop=0&crop=0&crop=1&crop=1&from=drop&id=u14bb5647&margin=%5Bobject%20Object%5D&name=quickly-create-components.gif&originHeight=512&originWidth=1375&originalType=binary&ratio=1&rotation=0&showTitle=false&size=92813&status=done&style=stroke&taskId=u7de81662-cd5b-45f5-bb6a-9aa713a3d26&title=)
## 总结：
无论是刚参与到组件库开发、正处在组件库开发期间还是在使用组件库过程中都有考虑，倾力打造一款优秀体验的组件库，在组件库的工程化方面和组件开发当中还有哪些可以提升体验的地方欢迎一起讨论~
