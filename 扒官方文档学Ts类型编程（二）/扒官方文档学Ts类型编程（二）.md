![](https://cdn.nlark.com/yuque/0/2022/png/26114887/1650446511486-af897d6b-b683-4a76-92ad-17470aa50233.png)

## 写作背景：󠀰

&ensp;&ensp;&ensp;&ensp; 承接上一篇[扒官方文档学Ts类型编程](https://juejin.cn/post/7084128507109834782#comment)来继续扒完类型编程的后两个章节Mapped Types和Template Literal Types，同样准备了演练场的代码可以同步调试观察输出的类型来学习。

## TypeScript类型操作：

&ensp;&ensp;&ensp;&ensp; TypeScript类型系统的强大之处主要体现在它允许我们通过类型来表达类型，也就是说我们可以通过现有的类型经过一系列的操作得到另一个类型（从类型创建类型），我们将通过下面表格所列举的顺序来讲解如何表达一个新的类型：

| 序号 | Types | 类型 | 描述 |
| --- | --- | --- | --- |
| 1 | Generics-Types | 泛型类型 | 带参数的类型 |
| 2 | Keyof Type Operator | Keyof 类型运算符 | 使用keyof运算符创建新类型 |
| 3 | Typeof Type Operator | Typeof 类型运算符 | 使用typeof运算符创建新类型 |
| 4 | Indexed Access Types | 索引访问类型 | 使用Type['a']语法访问类型的子集 |
| 5 | Conditional Types | 条件类型 | 行为类似于类型系统中的 if 语句的类型 |
| 6 | Mapped Types | 映射类型 | 通过映射现有类型中的每个属性来创建类型 |
| 7 | Template Literal Types | 模板字符串类型 | 通过模板字符串更改属性的映射类型 |

## Mapped Types：

&ensp;&ensp;&ensp;&ensp; 通过使用映射类型可以再不重新定义的前提下创建另一种新的类型，映射类型也是一种通用类型，接下来我们通过一系列的示例来感受一下映射类型的使用。

**正式开始前需要明确以下4点：**

1. 使用映射类型的最基础是通过索引类型访问来实现的；
1. 使用映射类型前应该有一个类型；
1. 使用keyof关键字来得到输入类型中key组成的联合类型；
1. 使用in关键字可以遍历联合类型。

### 入门案例：HelloWorld

了解一个最简单的映射类型工具的使用。

#### 下面这块代码是我们待输入的类型：

```typescript
type FeatureFlags = {
  darkMode: () => void;
  newUserProfile: () => void;
};
```

#### 类型工具说明：

&ensp;&ensp;&ensp;&ensp; 借助下面的通用映射类型工具，将输入类型Type的key来作为新对象类型的key，value的类型统一为boolean类型进行约束。

```typescript
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};
```

#### 我们使用一下这个类型工具：

```typescript
type FeatureOptions = OptionsFlags<FeatureFlags>;
//  ^?
```

[去演练场验证结果](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBAYhCGwCuAnCMA28DmBnKAvFAN4BQUUAJvCgNYCyA9pRAFxQAUAlIQHxQA3RgEtKAbnJQAdhADuAVVwQUABRSMAZsIxtOPAvyGiJAXwmlQkKAHkwwYYym5MOXAB4AKuAj8iZCgDaaoyQKKBQwlJQtBAgmlBekAC67ABGjIw68FKm5pbQcIioELb2jvhEpQ5OLnhuhchotbi8EgD0bRQAegD8pEA)
### 案例-在映射类型中使用修饰符：

&ensp;&ensp;&ensp;&ensp; 在映射类型中同样支持在JavaScript中对对象的修饰属性，readonly，属性可选。在映射类型中还可以对已经存在的这些修饰符进行删除。

#### 剔除输入类型中的readonly修饰

##### 下面这块代码是我们待输入的类型：

```typescript
type LockedAccount = {
  readonly id: string;
  readonly name: string;
};
```

##### 类型工具说明：

&ensp;&ensp;&ensp;&ensp; 借助下面的通用的映射类型工具，将输入类型中的readonly删除掉，得到一个所有属性均非只读的对象类型；

```typescript
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};
```

##### 我们使用一下这个类型工具：

```typescript
type UnlockedAccount = CreateMutable<LockedAccount>;
//  ^?
```

[去演练场验证答案](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBAMg9gYwNYQCYEEELgVwHbBQC8UA3gFBRQBOEAhqnHgDYhQCWqAXFAM7DV2eAOYBuSjXqMWbPHQC2EHv0EjxAX3HlQkKAGFadYBACyOYHQBGzCAB4AKuAgA+YmQkBaQ9NZQA2gAK1HCQ1KAceFAoIHAAZlCOkAC6PIkQgcGhoEkaWjrQAKosiCgYWLgEbgb0xmYW1nbwyGiY2PjAzuIA9F1UAHoA-ORAA)

#### 剔除输入类型中的可选修饰：

##### 下面这块代码是我们待输入的类型

```typescript
type MaybeUser = {
  id: string;
  name?: string;
  age?: number;
};
```

##### 类型工具说明：

&ensp;&ensp;&ensp;&ensp; 借助下面的通用的映射类型工具，将输入类型中的可选修饰符删除掉，得到一个所有属性均必填的对象类型；

```typescript
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
};
```

##### 我们使用一下这个类型工具：

```typescript
type User = Concrete<MaybeUser>;
//  ^?
```

[去演练场验证答案](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBAsghiARhAqgZwgJygXigbwCgooBLAEwC4o1hNSA7AcwG5ioG4BbCAfmtr1mbEnCZ9qDAK5dkmNgF82hUJCgBhAPYMAxpgjAIAHgAq4CAD5cBdgG0ACpk2RMoMgygBrCCE0AzKDNIAF0AWn5A8wcnF1BgxWVVaHQsay1dfUMjeCRUDEwLNgB6IpIAPV5CIA)

### 案例-搭配模板字符类型使用

&ensp;&ensp;&ensp;&ensp; 模板字符类型可以方便我们链接/扩展为更多的字符串类型，使用类同我们在JavaScript中模板字符串的使用，但模板类型作用在类型位置。
模块字符类型我们会在下面单独讲，这个案例使用到的**Capitalize**会将传入的**Property**转为首字母大写。

#### 下面这块代码是我们待输入的类型

```typescript
interface Person {
    name: string;
    age: number;
    location: string;
}
```

#### 类型工具说明：

&ensp;&ensp;&ensp;&ensp; 借助下面的通用的映射类型工具，我们可以为输入的类型Type增加对应的已get为前缀的函数。我们通常在定义完对象属性后会增加对应属性获取的函数而不是直接对外暴露这个属性。

```typescript
type Getters<Type> = {
    [Property in keyof Type as `get${Capitalize<string & Property>}`]: () => Type[Property]
};
```

#### 我们使用一下这个类型工具：

```typescript
type LazyPerson = Getters<Person>;
//  ^?
```

[去演练场验证答案](https://www.typescriptlang.org/zh/play?#code/JYOwLgpgTgZghgYwgAgArQM4HsTIN4BQyxyIcAthAFzIZhSgDmA3ESXI9aQK7kBG0ViWQAbLAjhhgOGnQYgWBAL4ECYAJ4AHFAHEIYSFAwAeACpaIAPmQBefG2IBtVFCzaoG5KGQBrCOqwYZHNtZDgMZAADTjAAEjwAYThNYDA4EWAALwhjOSZkADI0V3cNSyVIgF0aAAoASltrEIhnEugNSuVWNQtkABk4TPV0IxxbZD0DTGMR7BBLVgB6ReIAPQB+AiA)

### 案例-映射类型+内置的类型工具

使用内置的类型工具Exclude来配合映射类型剔除掉输入类型的指定属性后创建一个新的类型。

#### 下面这块代码是我们待输入的类型

```typescript
interface Circle {
    kind: "circle";
    radius: number;
}
```

#### 类型工具说明：

&ensp;&ensp;&ensp;&ensp; 借助下面的通用的映射类型工具，在使用索引类型方式key时使用as关键字来对Property进行进一步的处理，使用Exclude剔除掉所包含的kind，从而得到一个新的类型。

```typescript
type RemoveKindField<Type> = {
    [Property in keyof Type as Exclude<Property, "kind">]: Type[Property]
};
```

#### 我们使用一下这个类型工具：

```typescript
type KindlessCircle = RemoveKindField<Circle>;
//  ^?
```

[去演练场验证答案](https://www.typescriptlang.org/zh/play?#code/JYOwLgpgTgZghgYwgAgMLCggNig3gKGSOQGtQATALmQCIENsIaBuQ4qOc4AVwGdqQ3ALYAjaKwC++fGACeABxQAlCEID2ANwgBpCgDFgELOQA8AFQUQAfMgC8yAsWQBtAApQ1iqHOShSEWTUYZAtFZDheZABRAA9sbnIIE3dPaDkAGloyEHIaKwBdalCINw8vOXz8CVYZS2RdHJxeXnRMHDtkFXUtBvIDI1NWxitWAHpRogA9AH58IA)

### 案例-映射联合类型

#### 下面这块代码是我们待输入的类型

```typescript
type SquareEvent = { kind: "square", x: number, y: number };
type CircleEvent = { kind: "circle", radius: number };
```

#### 类型工具说明：

&ensp;&ensp;&ensp;&ensp; 借助下面的通用的映射类型工具，在输入类型做了约束，必须要包含king且类型为string的属性，在遍历Events的时候使用as取E中名为kind的value作为新类型的key。

```typescript
type EventConfig<Events extends { kind: string }> = {
    [E in Events as E["kind"]]: (event: E) => void;
}
```

#### 我们使用一下这个类型工具：

```typescript
type Config = EventConfig<SquareEvent | CircleEvent>
//  ^?
```

[去演练场验证答案](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBAygjgVwIYCcIFEBuEB2woC8UA3lANYCWOAJgFxQBEAzoqhAwDRQAe9OCAWwBGEFFxB9BIlFAC+AbgBQoSFADCFFAGMANhmx5CJclTqMtm3ey4ok1CgiaThouUuXhoWXMDUB7HAAzCgBzAB5vPCYoCG5gXGpo0koaeiZgFCoQuQA+I2JFKCKoAG10KCooSOBopGj0EoYU6gYAXVb6AAoIA2B6dABKQjzMPwpqJVlFD1V-INCjarng8PhkNGqoAB91Sz1qnMUAeiOigD0AfkUgA)

## Template Literal Types：

&ensp;&ensp;&ensp;&ensp; 模板字符类型的语法同JavaScript中模板字符串，但使用的位置不同，模板字符类型应用在类型位置。通过使用模板类型来扩展/链接内容创建新的字符类型。

### 入门案例：HelloWorld

&ensp;&ensp;&ensp;&ensp; 下面使我们的入门案例，通过模板插值将类型**World**插入到了类型**Greeting**中，最终创建的类型为“hello world”，注意这里面的“hello world”是类型而不是值。

```typescript
type World = "world";
 
type Greeting = `hello ${World}`;
//  ^?
```

[去演练场验证答案](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBA6g9gJwDYBMoF4oCIDujVYDcAUFMaJFAOIIQTACWAdgOYZQAGAFhEknFAAkAb3jIUAXw4kA9DKhQAegH5iQA)

### 案例-插值位置使用联合类型：

&ensp;&ensp;&ensp;&ensp; 下面的案例可以得到一个结果，当插值位置是用联合类型是，结果将是由每个联合成员便是的每个可能的字符串组成的集合。

```typescript
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";
 
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
//  ^?
```

[去演练场验证答案](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBAogtgQwJYBsAyB7AxglECSAIgM5QC8UARAO4QpYZwQD6EiqlUAPlW8iswAWEBABMkAOwDmlANwAoUJCgAxDBmAQATphx4ipCpQBm6zVubAkwPJx4mz25sQgTRGY8bnyoi8NABBFHRsXAIScigAAwASAG94fl0wg25VRx1Q-RIAX2YkUSiFAHpiqCgAPQB+eSA)

### 案例-多个插值位置使用联合类型：

&ensp;&ensp;&ensp;&ensp; 下面的案例可以得到一个结果，当每个插值位置均使用联合类型时，结果的数量将是每个联合元素个数相乘的积。

```typescript
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";

type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`;
type Lang = "en" | "ja" | "pt";
 
type LocaleMessageIDs = `${Lang}_${AllLocaleIDs}`;
//  ^?
```

[去演练场验证答案](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBAogtgQwJYBsAyB7AxglECSAIgM5QC8UARAO4QpYZwQD6EiqlUAPlW8iswAWEBABMkAOwDmlANwAoUJCgAxDBmAQATphx4ipCpQBm6zVubAkwPJx4mz25sQgTRGY8bnzF4aAEEUdGxcAhJyKAADABIAb3h+XVCDblVHHRD9EgBfZiRRSIUlaDQEaQjKVzsqACsEasowYG8oX2UkvABZCGJiBCkwwyi40ulcuMDgvUHswvkAenmoKAA9AH55IA)

### 案例-在类型中使用字符串联合

案例介绍：

1. 我们通常会在做一系列操作的函数前面定义一个明显的前缀；
2. 在这个案例中实现监听对象属性的变化；
3. 需要有一个**on**函数来接收，监听事件的名称我们做特定的约束格式为：key+Changed；callback是一个没有返回值的函数。

```typescript
type PropEventSource<Type> = {
    on(eventName: `${string & keyof Type}Changed`, callback: (newValue: any) => void): void;
};

/// Create a "watched object" with an 'on' method
/// so that you can watch for changes to properties.
declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;
```
```typescript
const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26
});
 
person.on("firstNameChanged", () => {});

 // Prevent easy human error (using the key instead of the event name)
person.on("firstName", () => {});

// It's typo-resistant
person.on("frstNameChanged", () => {});
```
[去演练场验证答案](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBACgTgezAUQG4QHbAMoIK5wDGEAPACrgQB8UAvFAN4BQUrUCGAFBOlgHIBDALYQAXFAAGAEgYBnYHACWGAOZQAZFADWEEAgBmUCpAC+AYQAWA1RAAmEgDRRCAgDauARgMJbxnDBAA7gBqbnhiUNYgAJR0NKgIirbR4glJANxMJplMAPT5UGZwEALA0AJQAESBpYQWduweAFYQhMCVUIGKwBaRGFAA5BwDUCI9CLZ5BbIIUD2lUHp4ztadtb36CHDOVjayc7NgiJBwwIoQsgB0TLatrgLFUPp4GG2KHKMCOgDq63YA8s1WsByJQqJwEM1xMYICkjJQNLBjmhMDh8ERSDCqDlCBx5FATjN+vQhF8IL9gHUAUC2pxmKx9Io4PJBCJxJVsAJEsyIJUHCwoPcWcIIpUAEocax8gUCFQRABMADYstFMlAmISOJcOJxKozmcBWRBLNY5bY+VBOLFaDQGCZVUwWAV4DxUVASrIQFALHhSf0IHBENtOHhZMo1D1oDovcp5CVbOxDJH3bxgFAMCLohqA0TtVw9UzhSILVa4ox7TkCgBJYADfagMAIAC0xTD8mswGzzK1Or1BqNJps5qcpZt5dVQA)

### 案例-模板字符类型推导

这个案例是上一个案例的升级版本，增加了对返回类型的推导，使得这个类型工具将更加的完善。

```typescript
type PropEventSource<Type> = {
  on<Key extends string & keyof Type>
    (eventName: `${Key}Changed`, callback: (newValue: Type[Key]) => void): void;
};

declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;
```
```typescript
const person = makeWatchedObject({
  firstName: "Saoirse",
  lastName: "Ronan",
  age: 26
});

person.on("firstNameChanged", newName => {
  console.log(`new name is ${newName.toUpperCase()}`);
});

person.on("ageChanged", newAge => {
  if (newAge < 0) {
    console.warn("warning! negative age");
  }
})
```

[去演练场验证答案](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBACgTgezAUQG4QHbAMoIK5wDGEAPACrgQB8UAvFAN4BQUUCGJA0hCFBAB7BMAEwDOUUcDgBLDAHMoAMigBrHggBmUCpCotWUABQR0WAHIBDALYQAXFAAGAEgbcQAXwDCACwvyIwg4ANFCEFgA24QBGFoQq9oYYEADuAGoReHbalADabgC6AJR0NKgI0sKF9mUVANxM7vVMwhCE4RZw0Bp4GITA0uxQVhZqAOoWwITeAQDyUQBWrcDklFSGCAv2OhBV2ZBKsIgopjj4RKTbVE2E7JJQkHCig-TDYxNTswtLhsysGtKPYCWGz2ABE2As5UeEFBQX07UkwKyoIASuw-LD9BY5FkAEwANgahSaDyeGAAdOxDKD-oCkT4-DjhLCoElkkiSox9DcME9whByeEEHJDA42azrNBpOIXGykeTgAgAKpgB6eCyiCCGQruBzEokkiCPdiUjDU7EQBn+ZkhNkAQRxnN+UGkWkSKQd0BIUAADMVnaweXyBckOmbQaG4BhZHIAISsiByCbSdBQC2g-WsdxEoA)

### 整理内置的模板字符操作类型：

#### 将每个字符均转为大写：

```typescript
type Greeting = "Hello, world"
type ShoutyGreeting = Uppercase<Greeting>
//  ^?

type ASCIICacheKey<Str extends string> = `ID-${Uppercase<Str>}`
type MainID = ASCIICacheKey<"my_app">
//  ^?
```

[去演练场验证答案](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBA4gThCwCWA7A5lAvFARACQgBsiB7AGigHdS4iATXAKFEigGUALUgV1HkQoM2KAFUwkOAGMAhgGcIAHgFI06AHxMA9FqhQAegH4mLcNACC7AMIBJG1ZlTOEANIQQi9sDhQIAD2AIVHo5KDlvNXURAAMbABEAWgASAG9xSVkFT291AF9o0zYAWRk0eJFLW3tHZzcPXABbEAB9GQlcTR09IyA)

#### 将每个字符均转为小写：

```typescript
type Greeting = "Hello, world"
type QuietGreeting = Lowercase<Greeting>
//  ^?

type ASCIICacheKey<Str extends string> = `id-${Lowercase<Str>}`
type MainID = ASCIICacheKey<"MY_APP">
//  ^?
```

[去演练场验证答案](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBA4gThCwCWA7A5lAvFARACQgBsiB7AGigHdS4iATXAKFEigEUBXZJeRFDNigAZUlQhwAxgEMAzhAA8fJGnQA+JgHpNUKAD0A-ExbhoAQQDKAYQCSNq9MkALCAGkIIBReBwoEAB7AEKj0slCyPqpqQgAGyPQAtAAkAN6i4lJyit5wagC+MSZsALLSaDYAIkKWtvaOLu6euMUAmgD6ZgAKnbga2rqGQA)

#### 将第一个字符转为大写

```typescript
type LowercaseGreeting = "hello, world";
type Greeting = Capitalize<LowercaseGreeting>;
//  ^?
```

[去演练场验证答案](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBAMg9gdwgJwMYEMDOEDiyITACWAdgOZQC8UARABYQA2jcANFAnMowCY0DcAKFCQoeAsXJUoAYXRgiwdIyIAvCAB54SNFlz5CpMgD4hAejNQoAPQD8QA)

#### 将第一个字符转为小写：

```typescript
type UppercaseGreeting = "HELLO WORLD";
type UncomfortableGreeting = Uncapitalize<UppercaseGreeting>;
//  ^?
```

[去演练场验证答案](https://www.typescriptlang.org/zh/play?#code/C4TwDgpgBAqmkCcDGBDAzhA4giFgEsA7AcygF4oAiACQFEAZegeSgHUmAlegEUoG4AUKEixCSAPYBbAGbiEwFACMANlhx4ipCjDEow+BcvwAvCAB44iVBmy4CJAHyCA9M6hQAegH4gA)

## 写到最后：

&ensp;&ensp;&ensp;&ensp; 至此TypeScript类型编程的7大块内容就已经过了一遍了，模板字符类型的案例还需要多熟悉熟悉。在官网还有一些提供的内容类型工具可以直接供我们在实际开发中使用，这里给出[Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)的地址方便大家查询。类型编程和我们以往的编程一样，同样在乎基础的学习和大量的练习。上次推荐的开源类型挑战项目[type-challenges](https://github.com/type-challenges/type-challenges)你有练习打卡吗？

由于平台对外链限制，建议访问原文获得更好的阅读体验，文中的大量案例沿用了官方文档的示例，同样可以参考官方文档，不足之处还请指正。


## 团队介绍

&ensp;&ensp;&ensp;&ensp; 高灯科技交易合规前端团队（GFE）, 隶属于高灯科技(北京)交易合规业务事业线研发部，是一个富有激情、充满创造力、坚持技术驱动全面成长的团队, 团队平均年龄27岁，有在各自领域深耕多年的大牛, 也有刚刚毕业的小牛, 我们在工程化、编码质量、性能监控、微服务、交互体验等方向积极进行探索, 追求技术驱动产品落地的宗旨，打造完善的前端技术体系。

- 愿景: 成为最值得信任、最有影响力的前端团队
- 使命: 坚持客户体验第一, 为业务创造更多可能性
- 文化: 勇于承担、深入业务、群策群力、简单开放


**Github：**[github.com/gfe-team](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fgfe-team)

**团队邮箱：**[gfe@goldentec.com](https://link.juejin.cn?target=mailto%3Agfe%40goldentec.com)

作者：GFE-小鑫同学
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

