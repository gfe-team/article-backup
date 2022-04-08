![扒官方文档学Ts类型编程.png](https://cdn.nlark.com/yuque/0/2022/png/2373519/1649347804151-e995a5c1-bb24-4f4f-ba3a-a5b288da507d.png#clientId=ube208e44-0dd5-4&crop=0&crop=0&crop=1&crop=1&from=drop&id=N37Zl&margin=%5Bobject%20Object%5D&name=%E6%89%92%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3%E5%AD%A6Ts%E7%B1%BB%E5%9E%8B%E7%BC%96%E7%A8%8B.png&originHeight=734&originWidth=1303&originalType=binary&ratio=1&rotation=0&showTitle=false&size=87927&status=done&style=none&taskId=u76cf1de3-a7e9-4883-8b46-98234c8e1c1&title=)
## 写作背景：󠀰

&ensp;&ensp;&ensp;&ensp; TypeScript作为JavaScript的一个超集带来了非常强大的类型系统，但作为天天泡在业务开发中的我们来说没感觉比其它面向对象的Java，C#等语言高级了多少，最近发现吵吵着类型体操的人比较多，决定翻看了一下TypeScript文档来搞搞清楚这个类型有什么高级之处，接下来就详细上手学习一下TypeScript类型编程的强大之处吧。

**重要的事情提前说：**

1. 你申明的是类型而非变量，你看到的true、false大多数均是类型而非Boolean类型的值。🕊️

2. TypeScript类型编程建议点击对应链接进[Playground](https://www.typescriptlang.org/zh/play)边看文章边调试代码学习。

## TypeScript类型操作：

&ensp;&ensp;&ensp;&ensp; TypeScript类型系统的强大之处主要体现在它允许我们通过类型来表达类型，也就是说我们可以通过现有的类型经过一系列的操作得到另一个类型（从类型创建类型），我们将通过下面表格所列举的顺序来讲解如何表达一个新的类型：
| 序号| Types | 类型 | 描述 |
| --- | --- | --- | --- |
| 1 | Generics-Types | 泛型类型 | 带参数的类型 |
| 2 | Keyof Type Operator | Keyof 类型运算符 | 使用keyof运算符创建新类型 |
| 3 | Typeof Type Operator | Typeof 类型运算符 | 使用typeof运算符创建新类型 |
| 4 | Indexed Access Types | 索引访问类型 | 使用Type['a']语法访问类型的子集 |
| 5 | Conditional Types | 条件类型 | 行为类似于类型系统中的 if 语句的类型 |
| 6 | Mapped Types | 映射类型 | 通过映射现有类型中的每个属性来创建类型 |
| 7 | Template Literal Types | 模板字符串类型 | 通过模板字符串更改属性的映射类型 |

## Generic Types：

&ensp;&ensp;&ensp;&ensp; 泛型在高级编程语言Java、C#中的应用是很广泛的，泛型的引用使得我们将类型指定的声明周期延迟到实例化时进行，使得我们的程序设计达到更高的复用程度，变得更加灵活。

### 泛型引入：

&ensp;&ensp;&ensp;&ensp; 在TypeScript开发过程中我们可以显示的来标记传入参数和返回数据的类型，当需要支持传入和返回数据类型的限制相对宽泛我们可以使用any来表示，但这样也就丢失了TypeScript的强大之处（静态类型推断）。

#### 定义固定类型的函数：
```typescript
function identity(arg: number): number {
  return arg;
}
```

#### 定义任意类型的函数：
```typescript
function identity(arg: any): any {
  return arg;
}
```

#### 使用泛型定义通用类型的函数：

1. 泛型的特点就是通用；

2. 泛型的语法：< T >，其中T是通配符，常见的通配符还有K，U等，下面代码中的Type也是通配符；
  
3. 在下面执行**identity**时通过泛型约束了传入类型为string，那么按函数功能返回的类型也将是string，可以点击进入[演练场](https://www.typescriptlang.org/zh/play?#code/GYVwdgxgLglg9mABDAJgUzLKBPAPAFWwAc0A+ACgEMAnAcwC5FCSBKR5tRAbwChFFqaKCGpIatANw8Avjx4AbIYjggoRVYgC8ydJhg5cAZyjUYYWhQBEAW2wBlE2dqWWUgPRv+APQD8QA)验证答案；

```typescript
function identity<Type>(arg: Type): Type {
  return arg;
}

let output = identity<string>("myString");
```

### 使用泛型类型变量：

1. 当我们在**identity**函数中直接读取arg变量的length属性时，编译器将会给我们抛出错误，提示Type并不存在一个名为length的属性。这是应为我们使用泛型定义的函数的重要特点就是通用，arg在实际传入的时候就可以试任意类型，就会出现传入的变量的类型不一定存在length属性。

2. 我们知道数组是肯定存在length属性的，下面的例子演示了约束类型为number但传入参数为数组但数组元素的类型为number，可以点击进入[演练场](https://www.typescriptlang.org/zh/play?ssl=5&ssc=50&pln=1&pc=1#code/GYVwdgxgLglg9mABAGzgczTMaCSATAUzFigE8AeAFVIAcCA+ACgEMAnNALkWroG0BdAJRcwIALYAjAq0QBvAFCJEEBAGc4yAgDpUaFux1E0UABaCA3IsSsCUEKyRs0h7KcsBfeZqiI4IKDT+iAC8KOiY2PhEJBSiktJMvACMiAA0iABM6QDMQpYA9PlKAHoA-EA)验证答案；

```typescript
function loggingIdentity<Type>(arg: Type[]): number {
  return arg.length;
}
let output = loggingIdentity<number>([1 , 2, 3]);
```

### 泛型类型：

&ensp;&ensp;&ensp;&ensp; 在前面我们看到的都是最长将的泛型的使用，这里开始我们就要学习泛型类型了，请仔细看代码，“：”左边的是变量的申明，“：”右边是变量应的类型，一定要记住。

#### 定义泛型函数<类型>：

1. 泛型函数和非泛型函数一样，都是先将类型参数列出，泛型类型参数同样可以使用不同的通配符来表示，但类型变量的数量和使用方式要保持一致。
2. 当然泛型类型定义还可以按对象字面量类型的方式编写，可以点击进入[演练场](https://www.typescriptlang.org/zh/play?#code/GYVwdgxgLglg9mABDAJgUzLKBPAPAFWwAc0A+ACgEMAnAcwC5FCSBKR5tRAbwChFFqaKCGpIatANw8Avjx4AbIYgC22AJLpMMHAEZGBYmSp12hlogC8pJocvJNWbFID0z-vwB6AfgVLVGjEcAJn01MCIQKApxRjCIqHMrRDjIu1RA7SceV3dEb18oFXUHTIBmRi5EAxJokxtWUxJEaTSSnBc3d3yeIA)验证答案；

```typescript
function identity<Type>(arg: Type): Type {
  return arg;
}

let myIdentity1: <Type>(arg: Type) => Type = identity;
//    ^?
let myIdentity2: <Input>(arg: Input) => Input = identity;
//    ^?
let myIdentity3: { <Type>(arg: Type): Type } = identity;
//    ^?
```

#### 定义泛型接口<类型>：

可以点击进入[演练场](https://www.typescriptlang.org/zh/play?#code/JYOwLgpgTgZghgYwgAgOIRNYCCSATDMYMATwDEQAeAFRIAcIA+ZAbwChlkAKOKAcwBcyWgwCUQkRADcbAL4c2MAK4gERAPYhkwAuGIka9Jj34Sj44UdYdkUCGCVQtvPjPnI2AG3vIAtiXxCfSF0TChsQL1SCkoQJV8AI2hmAF5tXSJSGQB6bM5OAD0AfjYgA)验证答案，定义泛型类和泛型接口类似，就不过多展开了。

```typescript
interface GenericIdentityFn<Type> {
  (arg: Type): Type;
}
 
function identity<Type>(arg: Type): Type {
  return arg;
}
 
let myIdentity: GenericIdentityFn<number> = identity;
//    ^?
```

### 泛型通用约束：

&ensp;&ensp;&ensp;&ensp; 我们在最开始有提到在从arg参数获取length时报错length在arg中不存在的提示，我们当时为了可以正常读取length就另创建了一个函数并约定形参为数组并数组元素类型为泛型的Type。那我们在不改变形参的情况下约束我们传入的Type一定包含一个length属性呢？这就体现出了通用约束的重要作用，在实际开发中也最为常见。

1. 我们定义了一个接口，并给定一个length属性；
  
2. 在尖括号中我们使用**extends**关键字来约束未来传入的Type一定是实现过Lengthwise接口的，这样我们就必定可以读取到length属性了，可以点击进入[演练场](https://www.typescriptlang.org/zh/play?strict=false#code/JYOwLgpgTgZghgYwgAgDIRAczACwO7ADOKA3gFDLIA2G2OAXMiAK4C2ARtANxkC+FZGMxAIwwAPYhq4zJlCYAkgBMMYsAE8APABV1ABxQQAHpBBLCaWrgLEAfAAo4UTI10GAlIxYdoycpSgIMGYoKSdMADoaLFweXiA)验证答案。

```typescript
interface Lengthwise {
  length: number;
}
 
function loggingIdentity<Type extends Lengthwise>(arg: Type): number {
  return arg.length;
}
```

### 泛型约束时使用类型参数：

1. 在下面的示例中，我们发现在尖括号中使用到了逗号；
  
2. 逗号前面：依旧是我们一直使用的Type；
  
3. 逗号右边：先给出答案，keyof Type 得到的将是Type属性key的集合，Key将是这个集合中的其中一个，可以点击进入[演练场](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBAKuEGcpQLxQNYRAewGZQG8BDALigEYAaKAIzICZqBjMgZmoBMyAWKAXwDcAKAD0I5MgB6AfiFDcAVwB2TYAEtsSqAHMIwAAoAnbJEOgAPHEjUA0ligQAHsAhKOSTDnxWIAPgAU2DQAVmQ+1J5kdiAAlIRCyIZ6CoZaQcEA2p4AusJ8CUIANnpQjqiEUKQU1HRQjFAsUOxQXFC8gnK6BsamoP6O1ABERIMxwl1GJhBmIP1DALajAkA)验证答案。

```typescript
type Types  = keyof {a: 1, b: 2, c: 3, d: 4 };
//    ^?

function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}
 
let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a");
getProperty(x, "m"); // Argument of type '"m"' is not assignable to parameter of type ...
```

## Keyof Type Operator：

&ensp;&ensp;&ensp;&ensp; 这里我们正式学习Keyof类型运算符，它的主要作用在上面的例子中也有提到，那么keyof的主要作用就是获取对象类型中属性名(键，key)的字符串或数字组成的联合（union）类型。当你想得到一个对象的key（的字符串）组成的联合类型时就用keyof。

### Keyof 类型运算符：

&ensp;&ensp;&ensp;&ensp; 我们不在展示讲述，因为它的作用足够的简单，你可以点击进去[演练场](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBAKuEGcpQLxQNYRAewGZQG8BDALigEYAaKAIzICZqBjMgZmoBMyAWKAXwDcAKAD0I5FAB6AfiFDQkKAAVsASwB2wVISgAPMuoCuAWxoQATgKggDJs+f7CF0Jdsw58KjcGFiJMuWcoAEFzcyIQVQQAC20CKABtdVtTCwBdMkN1dHVsAHd1R3l4ELcsPBCwiKjo33FkAOLFAFkiMBq4xPQyBGBzDQBzDNpsbAAbCCJCwSboZrKPKFb2mLr-WSA)验证答案。这里你可以考虑一下，对象的key都可以是什么类型呢？什么类型的值可以充当对象的属性名称呢？

```typescript
type Types  = keyof {a: 1, b: 2, c: 3, d: 4 };
//   ^?

type Point = { x: number; y: number };
type P = keyof Point;
//   ^?

type Arrayish = { [n: number]: unknown };
type A = keyof Arrayish;
//   ^?

type Mapish = { [k: string]: boolean };
type M = keyof Mapish;
//   ^?
```

## Typeof Type Operator：

&ensp;&ensp;&ensp;&ensp; 刚学完keyof操作符，这里就学习一个typeof操作符，typeof在JavaScript中就有，我们在查看变量类型的时候就经常使用，那么在TypeScript里面typeof的作用是什么呢？当我们在申明一个类型的时候，我们可以使用typeof来将声明的变量、属性转为其类型。什么意思呢？使用Typeof来引用我们JavaScript世界的内容转为到类型世界的内容。

### Typeof 类型运算符：

&ensp;&ensp;&ensp;&ensp; 可以使用typeof在类型上下文中使用它来引用变量或者属性的类型，可以点击进[演练场](https://www.typescriptlang.org/zh/play?#code/DYUwLgBAzmBOEF4ICIAWJjAPYQO5dmABNkBuAKHNEjAE8AHEALgjsawDNo5SIB6PhEDJ8YC-FQN4+gaPVAXHIxYASwB2Ac3ICIEAHoB+ckA)验证答案。

```typescript
let str = "hello world";

let type: typeof str;
//  ^?
```
### 案例分析-【结合ReturnType<T>】：

&ensp;&ensp;&ensp;&ensp; 在上面的入门示例中看到typeof似乎发挥的作用并不大，所以我们在了解作用和语法后结合其它类型运算符就可以表达更多的类型。

1. 在下面的示例中我们需要通过ReturnType来得到f函数的返回类型，这里可以看到我们需要将JavaScript世界的f转为类型世界，所以需要使用typeof来引用f；
2. 我们得到的P类型将是{ x: number;  y: number; }，具体请点击进[演练场](https://www.typescriptlang.org/zh/play?#code/GYVwdgxgLglg9mABMAFASkQbwFCMQJwFMoR8lNEAPALkQEYAGAGkQE9aBmRAXwG5tu2bFFYAHQogAKiALyIASsVJgAKmMIAeEeLjBkAPn4B6I3gB6AfiA)验证答案。

```typescript
function f() {
  return { x: 10, y: 3 };
}

type P = ReturnType<typeof f>;
//  ^?
```

## Indexed Access Types：

&ensp;&ensp;&ensp;&ensp; 索引访问类型和我们在编写JavaScript代码时的体验一样，也是使用中括号来传入索引值来获取内容，只不过在JavaScript中获取的内容是值，在TypeScript类型编程中获取的是类型。

### 索引访问类型：

&ensp;&ensp;&ensp;&ensp; 我们可以使用索引访问类型来查找另一种类型的特定属性，可以点击进[演练场](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBAChBOBnA9gOygXigbygQwHMIAuKVAVwFsAjBAbjL0pKkWHgEtUCG8AbDgDcW1ZMj4Q86AL50AUKEhQAgkUywEKVAG0ARIQi6AuvID0pqFAB6AfiA)验证答案。

```typescript
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"]; // 输出类型 number
//  ^?
```
### 案例分析-【本身就是类型】：

&ensp;&ensp;&ensp;&ensp; 因为索引访问类型本身就是类型，所以支持使用联合、keyof，或其他类型，可以点击进[演练场](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBAChBOBnA9gOygXigbyqghgLYQBcUiw8AlqgOYA0U+tpeAroQEYJQC+A3AChBoSFACSARkywEKVAG0ARMwhKoAHyhKCxJQF1+UYyagB6M1EDJ8YC-FQN4+gaPVylGrU3suCQReMA9APwi4NDiAEwycEhoCgDWECDIAGayUaiGphk+to7O1HTuqBzc8O6cyMgANhD4qN6WUAFBYgCCLADy8AByRNBYKizqWjo9SkYmTSEAzBFy0a0QHd3E6RmrWfZOFHluWoWe8HV+-kA)验证答案。

```typescript
type Person = { name: string, age: number };

type I1 = Person["age" | "name"];       // 输出类型 string | number
//  ^?
type I2 = Person[keyof Person];         // 输出类型 string | number | boolean
//  ^?
type AgeOrName = "age" | "name";    
type I3 = Person[AgeOrName];            // 输出类型 string | number
//  ^?
```
### 案例分析-【数组元素的类型获取】：

&ensp;&ensp;&ensp;&ensp; 这里我们需要通过number关键字来配合将数组展平，以便捕获数组字面量的元素类型，可以进[演练场](https://www.typescriptlang.org/zh/play?#code/MYewdgzgLgBAsgTwIICcUEMEwLwwNoBQMMA3jGOgLYCmAXDAERIA2AlsNQwDQzoDmdGAEYArDAC+XIqXJVBDAEIgARt14D6AJgDMEqcTIUa9BgFEAbpx79B2gBx6CAXQDcRAlAQAHajAAK1CgQ4Dgwnj4gAGbwyGiYeGAArpTKga7EGcQA9FkwgMnxgF+KgN4+gNHqMkaC0CisYHwu6oJJKYH14gQ5xAB6APwe3r5IAqHh1FExqBgICcmpKE54DDYM6dm5haXkM4HtuTA9fT4wg9SaoQFB4AtLK5m3dzAd62VNsztdvSMwANbUWLiLAgYbk+x10uHOwTAeB+CBu93hq3yxWeWxQbz23SAA)验证答案。

```typescript
const MyArray = [
  { name: "Alice", age: 15 },
  { name: "Bob", age: 23 },
  { name: "Eve", age: 38 },
];
 
type Person = typeof MyArray[number];       // 输出类型 { name: string; age: number; }
//  ^?
type Age = typeof MyArray[number]["age"];   // 输出类型 number
//  ^?
type Age2 = Person["age"];                  // 输出类型 number
//  ^?
type key = "age";
type Age3 = Person[key];                    // 输出类型 number
//  ^?
```

## Conditional Types：

&ensp;&ensp;&ensp;&ensp; 在我们学习编程的最开始阶段，当我们学习完如何输出HelloWorld，定义变量、函数后，基本就到了逻辑部分，那么上来的第一个将是IF比较逻辑符，也是每个编程语言都必不可少的。那么在TypeScript类型编程中也需要进行判断，但不是使用IF，而是使用类三元表达式，语法形式：SomeType extends OtherType ? TrueType : FalseType;

### 条件类型：

当extends左侧的SomeType可以分配给右侧OtherType时返回TrueType反之返回FalseType。

1. 在下面的例子中我们定义一个Person类和继承自Person类的Student类，我们知道Student属于Person，但不是每一个Person都属于Student，因为他/她长大了😅，可以点击进[演练场](https://www.typescriptlang.org/zh/play?#code/MYGwhgzhAEAKCmAnCB7AdtA3gKGtNYAtvAFzQQAuiAlmgOYDcu0Ydp+AroQEZJN7B0lRB2AUUiABQFiZYbToAaFmzJouvRAEoszPBQAW1CADoZ8aAF58RePzzRDxk6wvXX9gL7Zv2UJBgAZQoOABN4NApoeAAPCgjQmARkdF0BcCh4CDkqBWgAH04ePmZBNGFRcSlzHJp6ZVc1DSRlf0zs8lz6AqLNHRwHcg4AByRpWwa2LXt9I1M2iCyraAWsrx9sbAoAT1HoAFEYomGQeABGZeCwiKjY+LREuCRUDAB+RxELMgAzMBBFpgAekBeAAeq8gA)验证答案。
  
2. 注意，这里在提醒一下，下面代码中的true、false直接代表类型而非Boolean类型的值。
  
```typescript
class Person {
  name: string;
  age: number;
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
}

class Student extends Person {
  classes: string | number;
  constructor(name: string, age: number, classes: string | number) {
    super(name, age);
    this.classes = classes;
  }
}

type Example1 = Student extends Person ? true : false;
//  ^?
type Example2 = Person extends Student ? true : false;
//  ^?
```

### 案例分析-【条件类型+泛型】：

&ensp;&ensp;&ensp;&ensp; 通过上面学习的示例同样看起来相当的鸡肋，我们还是需要结合其他的类型运算符来配合使用。
  
下面的代码演示了我们函数重载的常用做法，我们需要实现的功能是当我们传入参数是number类型时返回IdLabel类型，当传入参数是string类型时返回NameLabel类型，显然在函数重载时没办法做进一步的限制，并且写起来也是相当的繁琐。
  
```typescript
interface IdLabel { id: number }
interface NameLabel { name: string }

function createLable(id: number): IdLabel;
function createLable(name: string): NameLabel;
function createLable(idOrName: number | string): IdLabel | NameLabel {
    throw "unrealized"
}
```

下面的代码是我们编写的通用类型工具，来满足重载函数的缺陷：

```typescript
type IdOrName<T extends number | string> = T extends number ? IdLabel : NameLabel;
```

下面的代码使我们使用类型工具简化后的结果：

1. 通过泛型约束形参类型：`<Type extends number | string>`；
  
2. 通过运行上面编写的**条件类型工具**得到合适的返回类型，可点击进[演练场](https://www.typescriptlang.org/zh/play?#code/JYOwLgpgTgZghgYwgAgJIBMAycBGEA2yA3ssOgFzIgCuAtnlMgL4BQoksiKAcnLRNjyESIPhEoBnMFFABzZixZgAngAcUGAPJRe-ADwAVZBAAekEOglU6DZAB9kUmSFkA+ZAF5kR0+cvX6aGQAfjQsXAJkSl0BCPwAbkUYahAEMGAAexBkBCgIOEhBfAhDNRRfCAsrGkDGByc5VwAKMm0YygMygEpKLR0xUvV3IhZkMeQwAAsoDIB3ZAAiFLy4fGAALwh0BZZWFmKwZAA3AEZPHJXC3GKmhZV1CVzgVTAFrvjxsYB6L+QYwQILB+YwAesF9hBDkcAEznXL5K44G4AZgAdCcACwnACs70++O+vwwAPwQN+yDBLCAA)验证答案；

```typescript
interface IdLabel { id: number }
interface NameLabel { name: string }

type IdOrName<T extends number | string> = T extends number ? IdLabel : NameLabel;

function createLable<Type extends number | string>(idOrName: Type): IdOrName<Type> {
    throw "unrealized"
}

let v1 = createLable("typescript");     // NameLabel
//  ^?
let v2 = createLable(3.1415);           // IdLabel
//  ^?
```

### 案例分析-【条件类型约束】：

我们通过一个案例来演示条件类型约束，我们需要设计一个通用的类型工具，当传入的泛型T中包含一个message属性，我们就返回这个属性的类型，如果不包含则返回never；

下面是我们编写的通用类型工具：

1. 使用条件类型来判断当T中存在一个message可以分配给右侧则返回通过索引访问类型取出（T["message"]）的类型；

2. 当T不存在可以分配给右侧一个message时返回never；
  
```typescript
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;
```
下面是验证的完整示例，可以进[演练场](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBAshDO8CGBzCB5AZgHgCoD4oBeKXKCAD2AgDsATeKAbygFsFk0AuKAVxoDWNAPYB3GlAC+UAPykA2gCJ2iVBEUBdKDxoQAbhABOAbgBQUUwEsa1Q5iQBjaAFFWSSwBtm5thzU94YENrFDNJcysbI3snKAARYRRvKCgAIyRDAQAKAEoePWFLOjCI0EgoV3cPOFU0AGFhKJtGEhrODBxKz3wzAHpelIA9GVLwaASUNrUGpuAW2D80LGwJntN+oZkgA)验证答案。
  
```typescript
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;
 
interface Email {
  message: string;
}
 
interface Dog {
  bark(): void;
}
 
type EmailMessageContents = MessageOf<Email>;
//  ^?
 
type DogMessageContents = MessageOf<Dog>;
//  ^?
```

### 案例分析-【展平数组得到元素类型】：

&ensp;&ensp;&ensp;&ensp; 在这个案例中我们希望传入的T是一个任意类型的数组，输出的是这个数组元素的类型，这里会用到一个特殊的索引访问“T[number]”，可以进[演练场](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBAYgNgQ2MCA7APAFQHxQLxSZQQAeKqAJgM5QKogDaAulAPyEOoCuAtgEYQATiwBchANwAoSaEhQAysEH5YiZGnRUlAS1QBzZtnFQTAelNQtg3XsnmTAPVYzw0AHK8V8JOXTd+QkYmwfb+AoJ2FlBOQA)验证答案。

```typescript
type Flatten<T> = T extends any[] ? T[number] : T;

type Str = Flatten<string[]>;   // string
//  ^?
type Num = Flatten<number>;     // number
//  ^?
```

### 案例分析-【在条件类型中如何推断】：

&ensp;&ensp;&ensp;&ensp; 在上一个案例中我们使用T[number]来得到数组元素的类型，这里我们将介绍一个新的关键字**infer**，它可以方便我们在条件类型中推断出元素的类型，可以进[演练场](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBAYgNgQ2MCA7APAFXBAfFAXim0iggA8VUATAZygEEAnJhEdAS1QDMImoAkigC2+APyCRUAFzEcAbgBQi0KQDKwfkXhIq6Wpq4BzANoBdXPKjWA9DagGmxxXesA9MSpxQAcgFdhQlhEZDR0VACAIz5LazjXCOFophd7KA8gA)验证答案。

```typescript
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;

type Str = Flatten<string[]>;   // string
//  ^?
type Num = Flatten<number>;     // number
//  ^?
```
  
下面这个案例是推断提取返回值的类型，可以进[演练场](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBA4hwCV4FcBOA7AKuCAebkAfFALxQHQQAewE6AJgM5QAUAlKcQJboBmEqKEmBp0UAPxCUGKAC4o6CADcBAbgBQoSFAByyALalY8YaIq52nBQYBGAwqqhPnzgPSvr+u6nXunAPXFNHCgAZWBBMjhEaSwcCw4SYkYIngBzBxcsvxTUdN8PKEDg7QAhAHtygBtmKJNY80skqBtKqogAQ3QAbQBdTLcPVurOnt6CgPEgA)验证答案。
  
```typescript
type GetReturnType<Type> = Type extends () => infer Return ? Return : never;
type Num = GetReturnType<() => number>;         // number
//  ^?
type Str = GetReturnType<() => string>;         // string
//  ^?
type Bools = GetReturnType<() => boolean[]>;    // boolean[]
//  ^?
```

### 案例分析-【分布式条件类型】：

&ensp;&ensp;&ensp;&ensp; 分布式条件类型（Distributive Conditional Types）来自软件翻译结果，这里我更愿意理解为“分别”，因为分布式常在服务端出现，如分布式部署，简单的理解就是将同一个分别部署到不同的机器上。那么 分布式条件类型指的是传入的类型为联合类型时，则条件类型会分别应用于该联合的每个成员。

例如下面这个是我们的类型工具，当我们传入的Type是一个联合类型时，我们将得到一个数组联合类型，且每个数组的类型分别对应Type联合类型的每一个类型。
  
```typescript
type ToArray<Type> = Type extends any ? Type[] : never;
```
  
下面是验证代码，可以进[演练场](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBAKg9gQQE5IIYgDw3BAfFAXlhyggA9gIA7AEwGcpUqQoB+YyAbQF0oAuKFQgA3CEgDcAKCiTQkKAGVgSZEgDySAHIBXALarCsRCnQY6ygJZUA5lAA+gvQCMxuKQHp3UKAD1WkoA)验证答案：
  
```typescript
type ToArray<Type> = Type extends any ? Type[] : never;
 
type StrArrOrNumArr = ToArray<string | number>;
//  ^?
```
  
当你运行上面的代码后得到的结果正如分布式条件类型的定义那样，传入的是string和number的联合类型，那么返回的将是string[]和number[]的联合类型。这是分布式条件类型的默认行为，但我们如何表示返回的结果需要是string或number类型的一个数组呢？这时候就需要使用中括号将extends两侧的类型进行包裹来避免默认的行为，可以进[演练场](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBAKg9gQQE5IIYgDw3BAfFAXigG1tIBdKCAD2AgDsATAZxNXpEoH5YdjKAXFHoQAbhCQBuAFChIUAMrAkyJAHkkAOQCuAW1WFYiFOgzNlAS3oBzKAB9hegEYTcMgPTuoUAHpdpQA)验证答案。
  
```typescript
type ToArray<Type> = [Type] extends [any] ? Type[] : never;
type StrArrOrNumArr = ToArray<string | number>;
//  ^?
```

## 写在最后：

&ensp;&ensp;&ensp;&ensp; 在这一篇中我们通过20份代码片段学习了TypeScript类型编程的前5大关键内容，在这里还是建议各位伙伴可以点击对应链接进入在线IED以边看代码边看文章学习。TypeScript类型编程的学习就和我们初学任何一种编程语言一样，将基础的语法灵活学习后才能在实战中运用自如。这里推荐一个在Github上的开源项目[type-challenges](https://github.com/type-challenges/type-challenges)，点赞高达15k之多，由Vue3的其中一位贡献者创建的学习和查阅TypeScript类型编程的项目感兴趣的伙伴也可以Fork一份自己做做看。剩下的Mapped Types和Template Literal Types在类型编程中也是很重要的两块内容，我们将单独再写一篇来详细讲解，各位尽情期待吧~

说明：
1. 文中的大量案例沿用了官方文档的示例，同样可以参考官方文档，不足之处还请指正；
2. 由于平台对外链限制，建议访问原文获得更好的阅读体验。

## 团队介绍
&ensp;&ensp;&ensp;&ensp; 高灯科技交易合规前端团队（GFE）, 隶属于高灯科技(北京)交易合规业务事业线研发部，是一个富有激情、充满创造力、坚持技术驱动全面成长的团队, 团队平均年龄27岁，有在各自领域深耕多年的大牛, 也有刚刚毕业的小牛, 我们在工程化、编码质量、性能监控、微服务、交互体验等方向积极进行探索, 追求技术驱动产品落地的宗旨，打造完善的前端技术体系。

- 愿景: 成为最值得信任、最有影响力的前端团队
- 使命: 坚持客户体验第一, 为业务创造更多可能性
- 文化: 勇于承担、深入业务、群策群力、简单开放

**Github：**[github.com/gfe-team](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fgfe-team)

**团队邮箱：**[gfe@goldentec.com](https://link.juejin.cn?target=mailto%3Agfe%40goldentec.com)

作者：GFE-小鑫同学

著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。