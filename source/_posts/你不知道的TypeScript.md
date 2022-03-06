---
title: 你不知道的TypeScript
date: 2022-03-06 15:14:33
tags: typescript 基础知识
description: 总结ts的类型层级、枚举、断言等相关知识，更加深入的了解typescript
---

# 基础概括

## 1.1 枚举

### 1.1.1 背景介绍

维护别人的代码尤其是一些质量较差的代码时，我们常常会碰到以下这样的代码，这样的状态位在之后的开发中很容易混乱。

```typescript
const handle = (status: number) => {
    if (status === 2) {
        // do something
    }
}

json ={
 'name':'zhangsan'
 'email':'1234567@qq.com'
}
```

所以说这个时候就需要有一个对象可以来将这些状态为做一个集中管理了。

### 1.1.2 基础用法

ts中的枚举其他的高级语言的枚举不同的是，ts中的枚举成员的值可以是字符串/数字。

```typescript
enum NoYes {
    no = 'no',
    yes = 'yes',
}
enum NoYes {
    no = 0,
    yes = 1,
}
```

此外，在ts的枚举中也可以使用数字的递增来定义枚举成员。

```typescript
enum NoYes {
    no, // 0
    yes, // 1
}
```

但是，在定义数字型枚举作为类型时，便会出现一个松散型的类型检查，它会直接将这个枚举类型当作一个number类型。

```typescript
enum NoYes {
    no,
    yes,
}
const foo = (sig: NoYes) => {
    // do something...
}
foo(11) // ok! 这个时候是不会报错的
```

因为这个特性，很容易的引起别人对公共模块乱传参对情况，比如说例子中我们无法对函数foo的入参做一个严格的检查。
但是当枚举值是字符串时就不再有这个问题了。

```typescript
enum NoYes {
    no = 'no',
    yes = 'yes',
}
const foo = (sig: NoYes) => {
    // do something。。。
}
foo('no') // error
foo(NoYes.no) // ok!

```

所以说，在很多的情况下都是建议采用字符串作为枚举值的，可以起到一个更好的约束作用。
但是在涉及到与后端的交互的时候，也是没办法的，是数字就还是得用数字，如果说擅自将数字改成字符串的还是会引起一些不必要的麻烦的。遗憾的是，之后的ts版本，将不会再去更新会影响代码运行的内容，所以说对于数字枚举松散型的问题，在后期也是不会再去解决了。
于是，我们也可以采用这种方法：

```typescript
enum NoYes {
    no, // 0
    yes, // 1
}
const foo = (sig: keyof typeof NoYes) => {
    // do something。。。
}
foo(11) // error
foo('no') // ok!
```

相比于枚举类型的数字，字符串还是比较好理解的，同时对入参的格式限制也还是有比较准确的定义，可以很好的检查出存不存在乱传参的行为。

1.1.3 运行时的枚举
上文说到枚举是少数的会参与代码运行的ts的内容之一，因此枚举是会被编译成js代码的。
以简单的NoYes枚举为例：

```typescript
enum NoYes {
  No,
  Yes,
}
```

ts将这个枚举编译为：

```typescript
var NoYes;
(function (NoYes) {
  NoYes[NoYes["No"] = 0] = "No";
  NoYes[NoYes["Yes"] = 1] = "Yes";
})(NoYes || (NoYes = {}));
```

通过编译后的代码我们可以看出，枚举具有反向映射的功能，可以通过值反向获取到枚举成员。

```typescript
enum NoYes {
  No,
  Yes,
}
NoYes.yes // 1
NoYes[NoYes.yes] // yes
```

为了减小代码运行时的负担，也有了一个常量枚举，让枚举只是参与开发过程，而不在参与js的运行过程。

```typescript
const enum NoYes {
  No,
  Yes,
}
```

以上的NoYes枚举在编译之后是会与类型一样被删除的，而在使用也会有区别。const枚举会失去反向映射的功能

```typescript
const enum NoYes {
  No,
  Yes,
}
const val1 = NoYes.yes // ok! val1 = 1
const val2 = NoYes[NoYes.yes] // error!  val2 = undefined
```

编译完之后的代码也会去掉enum的定义，引用的也直接给一个定值

```typescript
const val1 = 1;
const val2 = undefined;
```

这样对于缩小打包后的代码体积起到了一定的作用，在开发的过程中可以根据实际情况去使用const枚举，同时这样对打包后的代码也可以起到一个更好的加密效果。

### 1.1.4 对象枚举

在没有ts的时候写js代码，我们对枚举的定义都会使用一个对象写出一个枚举

```typescript
// ./enmus.js
export const StatusEnum = {
    off: 0,
    on: 1
}

import { StatusEnum } from './enums'

// ...
const handle = (status) => {
    if (status === StatusEnum.off){
        // do something....
    }
}
// ...
```

在ts项目中，这样的方式去定义枚举也不是不可以的，但是这时候我们可以去设置一个as const只读属性。

```typescript
// ./enmus.js
export const StatusEnum = {
    off: 0,
    on: 1
} as const
```

对象枚举的优点：

```markdown
将旧的js代码改造成ts更加的方便
可以对键值加计算逻辑
枚举值可以是Symbol类型

对象枚举的缺点
无法反向映射
对于枚举值的类型描述困难，透传的时候不好去描述类型（可以自己写一个ValueOf工具类型，但是有理解成本，如下代码）
```

```typescript
export const NoYes = {
    yes: 1,
    no: 0
} as const
```

```typescript
// 获取值
declare type ValueOf<T> = T extends {[K in keyof T]: infer V } ? V : never;
const handle1 = (k: keyof typeof NoYes) => { // 通过keyof typeof约束类型，但是不好去透传
    // do something...
}
const handle2 = （v: ValueOf<typeof NoYes>) => { // 可以透传枚举值，但是ValueOf有理解成本
    // do something...
}
```

## 1.2 类型守卫

### 1.2.1 类型的层级

在前面提到了顶级类型的概念，在ts中，类型是有自己的层级的，当一个类型可以被一个类型约束时，那么这个这两个类型便构成了上下级的关系（这节我们就排除any，它是一个特殊的例子）。类型也只能在同级和下级到上级传递，他们也构成了一个单向的传递关系。这也就是任何类型都可以给unknown，never可以给任何类型的原因。

```typescript
const foo = (arg: T1 | T2 | undefined) => {
    // do something;
}
const obj1: T1 = {
//...
};
const obj2: T2 = {
//..
}; 
let obj3: T3
foo(undefined) // ok!!
foo(obj1); // ok!!
foo(obj2); // ok!!
foo(obj3); // ok!!
```

不过值得注意的是，在非基础类型内部，上下级的关系就是不一样的了。

```typescript
type T1 = {
  a: string,
  b: number
  c: boolean
}
type T2 = {
  a: string,
  b: number
}

const foo1 = (arg: T2) => {
    // do something
}
const obj1: T1 = {
  a: 'aa',
  b: 1,
  c: true
}
foo1(obj1)； // ok！！
```

但是，在开发过程中难免会遇到类型不小心被放大的情况，然后被ts提示有不严谨的地方。(这种时候可能又会有人骂骂咧咧的说ts不好了)
在这个是时候其实需要对类型做一个守卫，从而再将类型进行收窄。

### 1.2.2 类型收窄

说起类型收窄，我们首先可以想到的在js中所拥有的

```markdown
类型判断：typeof；
实例判断：instanceof；
属性判断：in；
字面量相等判断：==，===，!=，!==；
```

它们在代码书写的时候都将通过if else和switch起到一定的类型守卫的作用，ts也可以对所参与的变量起到一个类型收窄的作用。

```typescript
type T1 = {
  a: string;
  b: number;
  c: boolean;
  d: 'type1';
}
type T2 = {
  a: number;
  b: number;
  d: 'type2';
}
const foo1 = (arg?: T1 | T2 | string) => {
  if (!arg) return;
  // arg is T1 | T2 | string
  if (typeof arg === 'string') {
      // arg is string
  } else {
      // arg is T1 | T2
      if ('c' in arg) {
        // arg is T1
      }
      if (arg.d === 'type1') {
        // arg is T1
      }
      if (arg.d === 'type2') {
          // arg is T2
      }
  }
}
```

在后面的章节中会讲到，never是所有类型的子类型，所以说never是所有类型的下级类型。
所以说我们可以这么理解

```typescript
type T1 = {
    a: string | never;
    b: number | never;
    c: boolean | never;
} | never
type T2 = {
    a: string | never;
    b: number | never;
    d: string | never;
} | never
```

每一个已知类型都是会被ts联合一个never的子类型的，当然这个联合不是我们自己写上去的。
所以说我们在使用if，switch做类型收窄的时候就会发现，当我们吧所有的已知类型都考虑完之后，编辑器就会出现一个never未知类型

```typescript
type T1 = {
    a: 'a' | 'b' | 'c';
    // any other keys;
}
const foo = (arg: T1) => {
    switch(arg.a) {
        case 'a': 
            // do something...
            break;
        case 'b':
            // do something...
            break;
        case 'c':
            // do something...
            break;
        default 
            // arg.a is never
    }
}
```

通过我们上面对never以及上下子类型的理解，知道了T1类型是会被ts给完整的写成

```typescript
type T1 = {
    a: 'a' | 'b' | 'c' ｜ never;
    // any other keys;
} | never;
```

在开发中我们可以用if else和switch将类型收窄，也可能会想到通过类型断言将类型收窄。

```typescript
type T1 = {
  a: string;
  b: number;
  c: boolean;
  d: 'type1';
}
type T2 = {
  a: number;
  b: number;
  d: 'type2';
}
const  foo1 = ( arg: T1 | T2 ) => {
    // do something....
}

let t1: T1 | T2 | string | undefined
// do something...
foo1(t1) // error!！ T1 | T2 | string | undefined类型不能给T1 | T2
foo1(t1 as T1) // ok
foo1(t1 as T2) // ok
foo1(t1 as T1 | T2) // ok
```

这种情况下很大的可能是一些个人原因没定义好类型，在更多的情况下我们遇到的是一个可能为空的情况，于是我们也可以使用非空断言

```typescript
type T1 = {
  a: string;
  b: number;
  c: boolean;
  d: 'type1';
}

let t: T1 // t is T1 | undefined
// do something...
const foo = (arg: T1) => {
    // do something...
}
foo(t) // error! t的值很有可能为undefined。
foo(t!) // ok! 通过非空断言（!）断言t不是undefined和null
```

不管怎样，断言收窄还是不建议乱去使用的，更加推荐的是使用is去做一个类型收窄

```typescript
const projectType = <T>(
    data: unknown,
    cb: (arg: unknown) => boolean
): data is T => {
  return cb(data)
}

type T1 = {
    a: string;
    b: number;
}
type T2 = {
    c: boolean;
    d: number;
}
const foo = (arg?: T1 | T2) => {
    if (!arg) return;
    // arg is T1 | T2
    if (projectType<T1>(arg, (a) => !!(a as T1)?.a)) {
        // arg is T1
    } else {
        // arg is T2
    }
}
```

使用is去做类型守卫，看似麻烦了很多。但是从逻辑层面去对数据进行了一个类型收窄，相比于断言，这样可以很大的降低运行时的风险。（当然那个判定逻辑不能随便去写）

### 小结

```markdown
类型守卫的核心是将类型收窄，可以将类型收窄成它的子类型
可以灵活的使用js中的相关内容对一个类型做相应的收窄
断言只能在上下级类型断言！不可以在没有上下级关系的类型之间断言，相关的区别会在本文中的断言中做详细介绍。
断言属于欺骗编译器的行为，并不会在运行过程中起作用，在开发中不可以盲目断言！更推荐于使用if else加is去做一个逻辑层的类型守卫。
```

## 1.3 any、unknown、never

### 1.3.1 any、unknown、never对比

ts作为一个静态语言，与强类型还是有所区别的。强类型是将代码编译成另一种语言的代码的，但是静态语言还是逃脱不了是一个弱类型的本质，有很多时候还是可以逃避的。所以说很多时候我们会看到的是能有any，unknown，never这样的隐式类型。
它们的区别如下：
any属于顶级、底级类型，所有的类型都可以给any，any类型可以给其他任何类型；
unknown属于顶级类型，所有的类型都可以给unknown，但是在unknown调用方法时必须要对unknown做存在的判断；
never属于底级类型，它可以给所有已知类型，但是已知类型不可以给never类型；

### 1.3.2 any是top type和bottom type

写过ts代码的人，对any可能是非常熟悉的一个东西了，毕竟any可以解决很多的问题。
在ts的提案中，any属于顶级类型，任何类型都可以赋给它。

```typescript
const handle (item: any) => {
    // do something...
}

const str: string = "hello world!"
const num: number = 100
const isYes = true
handle(str) // ok!!
handle(num) // ok!!
handle(isYes) // ok!!
```

同时any也是底级类型，它可以赋给任何类型

```typescript
const a1: any
const str: string = a1; // ok!
const num: number = a1; // ok!
```

所以说，any的优点还是挺明显的，但是any肯定不是可以让你类型自由的工具。对于代码中还是要尽量的去写已知类型。

### 1.3.3 unknown是顶级的类型

在其他的很多文章中都说unknown和any很像或者类似，但是这个说法也不是那么准确。
unknown在ts的提案中定义为了一个顶级的类型。任何类型都可以赋给unknown，在使用unknown的时候需要将类型收窄。

```typescript
const foo = (arg: unknown) => {
    arg.push() // error!!
    if (typeof arg === 'array') {
        arg.push() // ok
    }
}
```

try catch 语句中的catch后面的error参数会是unknown，然后有的时候我们会碰到比较难处理的情况，我们这个时候也可以使用断言进行类型收窄。

```typescript
try {
  // do something...
} catch (err) {
  // err is unknown
  if((err as Error )?.message === '...') {
    const error = err as Error
    // error is Error
  }
}
```

上面的收窄方式可能还是太机械性了，重复的代码写的太多，我们也可以使用is关键字对unknown进行类型收窄

```typescript
/**
 * 对数据进行类型守卫的函数
 * @param data 守卫的数据
 * @param cb 判断守卫的函数，把能够确定的逻辑写进来，返回true就是确定这个类型
 * @returns 第二个参数返回true为这个类型，否则不是
 */
const projectType = <T>(
    data: unknown,
    cb: (arg: unknown) => boolean
): data is T => {
  return cb(data)
}

try {
  // do something...
} catch (err) {
  // err is unknown
  if(projectType<Error>(err, (e) => !!(e as Error)?.message)) {
    // err is Error
  }
}
```

### 1.3.4 never是所有类型的子类型

由上面的对比可知，never可以是所有类型的子类型，在有已知类型的时候，never就会直接合并入已知类型中，不再有never类型。

```typescript
const T1: number | never // number
const T2: unknown | never // unknown
const T3: string | number | never // string | number
const T4: never //never
```

所以说never类型可以表示为一个无法推断出来的类型，这个在很多的工具类型特别是使用infer的类型中可以看到。表示的都是无法推断出想要得到的类型。

```typescript
// 是否为空类型
type NonNullable<T> = T extends null | undefined ? never : T;
// 获取函数参数类型的元数组
type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;

// 取U中的非T部分
type Exclude<T, U> = T extends U ? never : T;
// 取U中的T部分
type Extract<T, U> = T extends U ? T : never;
```

此外，never还可以表示的是未知类型，意思为它不属于任何一种类型，这种情况下在switch和if else语句中尤为明显，对这个的处理的也能体现出，逻辑代码是否能够考虑极端情况，对代码的稳定性的提高显得尤为重要。

```typescript
enum NoYes { no, yes }
const handleErr = (e: Error) => {
    // do something;
}
const foo = (type: NoYes) => {
    switch(type) {
        case NoYes.yes: 
            // do something....
        break;
        case NoYes.no:
            // do something...
        break;
        default: 
        /* 这种case的时候就是一个never类型，
         * 这种never类型可以给其他的任何类型
        */
            handleErr(type); // ok!
    }
}
```

### 小结

```markdown
介绍到这里，或许大家可以对any，unknown与never有一个了解，在使用的时候也有相应的几点建议：
减少未知类型的出现，少写甚至不写any；
对类型的声明要做到准确，避免隐式类型；
对never类型能够做合理拦截；
对于unknown类型能够做出合理的类型收窄；
```

## 1.4 联合undefined 与可选在实际使用过程中的区别

### 1.4.1 对象中键值设置为undefined和没有它的区别

由于js的灵活性，也是能够允许对象中的键值为null和undefined。但是设置为undefined的时候最重要的一点是可以被遍历到。

```typescript
const obj = {a: 1, b: 2};
obj.b = undefined;
for (const k in obj) {
    console.log(k);
}
// a, b

const obj = {a: 1, b: 2};
// @ts-ignore
delete obj.b
for (const k in obj) {
    console.log(k);
}
// a
```

为了表示出对象中的缺别，我们就有了键值联合undefined和可选的区别了

### 1.4.2 ts类型的undefined联合和可选

许多人在写ts的时候很早的时候就会注意到那个可选值，然后有的时候也可会观察到有一些的组件库的类型是{k: string | undefined}类型。甚至还会疑惑为什么不是写 ?: (毕竟 ?: 字符少，写起来方便^_^！！)。
这个地方就牵扯到ts类型的一个小细节了。

```typescript
type T1 = {
    a: number;
    b: string;
    c: boolean ｜ undefined;
    d?: number;
}
const obj1: T1 = { a: 1, b: '1', c: true } // ok
const obj2: T1 = { a: 1, b: '1', c: true, d: 2 } // ok
const obj3: T1 = { a: 1, b: '1', d: 2 } // error，缺少键值c
const obj4: T1 = { a: 1, b: '1', c: undefined, d: 2 } // ok
const obj5: T1 = { a: 1, b: '1', c: true, d: undefined } // ok
const obj6: T1 = { a: 1, b: '1', c: true, d: '2' } // error, d的类型不对
```

当一个类型的键使用可选声明时，这个键代表的就是可有可无了（有肯定是约定好的类型），但是设置为undefined联合时，这个键就必须得有了。
而且在我们将键设置为可选时，我们可以对该键进行delete，依然以上面的obj对象为例

```typescript
delete obj3.d // ok！！
delete obj5.c // error!!
```

### 小结

```markdown
开发过程中对非基础类型中的undefined属性定义要合理
为了减少可选类型对维护的时候带来的误解，可以使用| undefined来代替可选
```

## 1.5 object、Object与{}

### 1.5.1 基本类型

在ts中，基本类型包括string、number、boolean、symbol、[]、enum、undefined、null、void、unknown、never、any等。详细可见：<https://juejin.cn/post/7006304933813157919>
它们构成了ts的基本类型，由ts内部自己定义。

### 1.5.2 非基本类型

除了上述的基本类型外，其他的类型都为非基本类型。ts2.7版本中被提出object类型，表示ts中的非基本类型。在此之前，lib.d.ts中收录了Object类型来表示非基本类型。
所以说，类似于以下类型，都称为非基本类型

```typescript
interface T1 {
  a: string;
  b: number;
  c: boolean;
  // ...
}
interface T2 {
  a: number;
  b: number;
  d: 'type2';
  // ...
}
```

同时在ts的lib.d.ts中，我们也是可以看到有Object的声明的

```typescript
/**
 * Provides functionality common to all JavaScript objects.
 */
declare var Object: ObjectConstructor;

interface ObjectConstructor {
    new(value?: any): Object;
    (): any;
    (value: any): any;

    /** A reference to the prototype for a class of objects. */
    readonly prototype: Object;

    // ...
}
```

object相当于就是将Object定义成一个基本类型。
object与Object的各有以下特点：

```markdown
object是ts内部定义的基本类型，表示的是非基本类型，不可被重写；
Object类型为lib.d.ts中声明的非基本类型，可以被改写
```

### 1.5.3 Object是所有的非基本类型的父类型

所有的非基本类型中，都会继承Object类型。这个也是ts自身赋予的，不需要我们去写。我们在使用一个非基本类型的时候，是可以使用到hasOwnProperty、valueOf、length等原型属性的。

```typescript
interface T1 {
    a: string;
    b: number;
}

const p: T1 = {
    a: 'a',
    b: 1
}
p.hasOwnProperty('a'); //ok! Object上有这个方法
```

### 小结

```markdown
object是基本类型，它表示非基本类型Object
不建议将变量类型直接声明为object或Object，除非真的只用object的原型属性
ObjectConstructor有一些方法类型定义不够准确，可以尝试去改写它
```

## 1.6 interface 和 type  自定义类型

大家使用 typescript 总会使用到 interface 和 type,但是很少能够真正区分它俩，接下来介绍下他们之间的区别
1、相同点

- 都可以描述一个对象或者函数

```typescript
// interface定义对象
interface User {
  name: string
  age: number
}

// interface定义函数
interface SetUser {
  (name: string, age: number): void;
}

// type定义对象
type User = {
  name: string
  age: number
};

// type定义函数
type SetUser = (name: string, age: number)=> void;
```

- 都允许拓展（extends）
interface 和 type 都可以拓展，并且两者并不是相互独立的，也就是说 interface 可以 extends type, type 也可以 extends interface 。 虽然效果差不多，但是两者语法不同。
interface使用extends、implements组合类型 , type则可以通过&, |符号组合、合并类型。
1 interface extends interface

```typescript
interface Name { 
  name: string; 
}

interface User extends Name { 
  age: number; 
}
```

2. type extends type

```typescript
type Name = { 
  name: string; 
}

type User = Name & { age: number  };

```

3. interface extends type

```typescript
type Name = { 
  name: string; 
}
interface User extends Name { 
  age: number; 
}
```

4. type extends interface

```typescript
interface Name { 
  name: string; 
}
type User = Name & {
  age: number;
}
```

2、不同点

- type 可以声明基本类型别名，联合类型，元组等类型

```typescript
// 基本类型别名
type Name = string

// 联合类型
interface Dog {
    wong();
}
interface Cat {
    miao();
}

type Pet = Dog | Cat

// 具体定义数组每个位置的类型
type PetList = [Dog, Pet]

- type 语句中还可以使用 typeof 获取实例的 类型进行赋值
// 当你想获取一个变量的类型时，使用 typeof
const foo = {
    a: 'a',
    b: 2,
    c: true
}
type Foo = typeof foo //{ a: string, b: 2, c: boolean }
```

- interface 能够声明合并，也是interface 一个比较强大的地方，可以重复定义添加属性，type不行

```typescript
interface User { 
    name: string 
    age: number 
} 
interface User { sex: string } 
/* User 接口为 { name: string age: number sex: string } */
```

## 1.7  & 交叉类型

在 TypeScript 中交叉类型是将多个类型合并为⼀个类型。通过 & 运算符可以将现有的多种类型叠加到 ⼀起成为⼀种类型，它包含了所需的所有类型的特性。

```typescript
type X = { 
   x: number; 
}; 
type Point = X & { y: number; }; // 定义point的类型：  合并X和y的类型

let point: Point = { 
    x: 1, 
    y: 1 
}
```

- 同名基础类型属性的合并
那么现在问题来了，假设在合并多个类型的过程中，刚好出现某些类型存在相同的成员，但对应的类型 ⼜不⼀致，⽐如：

```typescript
interface X { 
    c: string; 
    d: string; 
}
interface Y { 
    c: number; 
    e: string 
} 
type XY = X & Y; 
type YX = Y & X; 
let p: XY; 
let q: YX;
```

在上⾯的代码中，接⼝ X 和接⼝ Y 都含有⼀个相同的成员 c，但它们的类型不⼀致。对于这种情况，此 时 XY 类型或 YX 类型中成员 c 的类型是不是可以是 string 或 number 类型呢？⽐如下⾯的例⼦：

```typescript
p = { c: 6, d: "d", e: "e" }; // 接上面的示例
q = { c: "c", d: "d", e: "e" };
```

为什么接⼝ X 和接⼝ Y 混⼊后，成员 c 的类型会变成 never 呢？这是因为混⼊后成员 c 的类型为
string & number ，即成员 c 的类型既可以是 string 类型⼜可以是 number 类型。很明显这种类型
是不存在的，所以混⼊后成员 c 的类型为 never，因避免出现类似情况

- 同名⾮基础类型属性的合并

```typescript
interface D { d: boolean; }
interface E { e: string; }
interface F { f: number; }
interface A { x: D; }
interface B { x: E; }
interface C { x: F; }
type ABC = A & B & C;
let abc: ABC = {
    x: {
      d: true, e: 'semlinker', f: 666
    }
};
console.log('abc:', abc);
```

由上图可知，在混⼊多个类型时，若存在相同的成员，且成员类型为⾮基本数据类型，那么是可以成功合并。

## 1.8 Tuple 类型

我们知道数组中元素的数据类型一般都是相同的（any[] 类型的数组可以不同），如果存储的元素数据类型不同，则需要使用元组。元组中允许存储不同类型的元素，元组可以作为参数传递给函数。

- 声明一个元组mytuple，并初始化：

```typescript
let mytuple: [number, string]
var mytuple = [10,"Runoob"];
```

- 访问元组

```typescript
console.log(mytuple[0]) // 10
console.log(mytuple[1]) // Runoob
```

- 可选元组
元组类型允许在元素类型后缀一个 ? 来说明元素是可选的：

```typescript
let mytuple: [number, string?，boolean?]
let mytuple = [10,"Runoob",ture];
let mytuple1 = [10,"Runoob"];
let mytuple2 = [10,];
```

- 元组越界
可以越界添加元素（不建议），但不可越界访问，有可选元素更不建议使用元组越界，因为可选元素一般都在最后

```typescript
let mytuple: [number, string] = [10,"Runoob"];
mytuple.push('hello world')

console.log(mytuple) // [10, 'Runoob', 'hello world' ] 
console.log(list[2]) // Tuple type '[string, number]' of length '2' has no element at index '2'
```

- 命名元组类型
命名元组类型适需要 TypeScript 4.0及以上版本才能使用，它极大的改善了我们的开发体验及效率，先来看一个例子:

```typescript
type Address = [string, number]
function setAddress(...args: Address) {
  console.log(args)
}
```

当我们这样定义函数入参后，在使用函数时，编辑器的智能提示只会提示我们参数类型，丢失了对参数含义的描述。

为了改善这一点，我们可以通过命名元组类型，我们可以这样定义参数：

```typescript
type Address = [streetName: string, streetNumber: number]

function setAddress(...args: Address) {
  console.log(args)
}
```

这样，在调用函数时，我们的参数就获得了相应的语义，这使得代码更加容易维护。
这两种⽅式看起来没有多⼤的区别，但对于第⼀种⽅式，我们没法设置第⼀个参数和第⼆个参数的名称。虽然这样对类型检查没有影响，但在元组位置上缺少标签，会使得它们难于使⽤。为了提⾼开发者使⽤元组的体验，TypeScript 4.0 ⽀持为元组类型设置标签

- 典型应用 useState

```typescript
import { useState } from 'react';
const [loading, setLoading] = useState<boolean>(false);
```

## 1.9 字符串模板类型

### 1.9.1  基础语法

它的语法和 es 里的字符串模板很相似，所以上手成本也很低，先看几个🌰：

```typescript
type EventName<T extends string> = `${T}Changed`;
type T0 = EventName<'foo'>;  // 'fooChanged'
type T1 = EventName<'foo' | 'bar' | 'baz'>;  // 'fooChanged' | 'barChanged' | 'bazChanged'


type Concat<S1 extends string, S2 extends string> = `${S1}${S2}`;
type T2 = Concat<'Hello', 'World'>;  // 'HelloWorld'

字符串模板中的联合类型会被展开后排列组合：
type T3 = `${'top' | 'bottom'}-${'left' | 'right'}`;  
// 'top-left' | 'top-right' | 'bottom-left' | 'bottom-right'
```

### 1.9.2  新增关键字

为了字符串模板类型这个功能， TS 中新增了四个关键字，用于对模板字符串变量进行处理

```markdown
-  uppercase — 大写字母
- lowercase — 小写字母
- capitalize — 首字母大写 
- uncapitalize — 首字母小写 
type Cases<T extends string> = `${uppercase T} ${lowercase T} ${capitalize T} ${uncapitalize T}`;
type T11 = Cases<'bar'>;  // 'BAR bar Bar bar'
```

### 1.9.3  实现类似于正则匹配提取的功能

配合infer

```typescript
type MatchPair<S extends string> = S extends `[${infer A},${infer B}]` ? [A, B] : unknown;
type T20 = MatchPair<'[1,2]'>;  // ['1', '2']
type T21 = MatchPair<'[foo,bar]'>;  // ['foo', 'bar']
```

通过 , 分割左右两边，再在左右两边分别用一个 infer 泛型接受推断值 [${infer A},${infer B}]，就可以轻松的重新组合 , 两边的字符串。

### 1.9.4 实现 Join 方法

... 拓展运算符和 infer

```typescript
type Join<T extends (string | number | boolean | bigint)[], D extends string> =
    T extends [] ? '' :
    T extends [unknown] ? `${T[0]}` :
    T extends [unknown, ...infer U] ? `${T[0]}${D}${Join<U, D>}` :
    string;
type T30 = Join<[1, 2, 3, 4], '.'>;  // '1.2.3.4'
type T31 = Join<['foo', 'bar', 'baz'], '-'>;  // 'foo-bar-baz'
```

### 1.9.5 实战运用

- 实现 lodash get 函数

```typescript
type PropType<T, Path extends string> = string extends Path ? unknown :
    Path extends keyof T ? T[Path] :
    Path extends `${infer K}.${infer R}` ? K extends keyof T ? PropType<T[K], R> : unknown :
    unknown;
declare function get<T, P extends string>(obj: T, path: P): PropType<T, P>;
```

```typescript
const obj = { a: { b: {c: 42, d: 'hello' }}};

const value = get(obj, "a.b.c")
```

# 2、TypeScript 4.1 带来的这个新功能让 TS 支持更多字符串相关的拼接场景，其实是特别实用的，希望大家能够有所收获~

## 2.1 断言

### 2.1.1  非空断言

- 忽略 undefined 和 null 类型
问题引入：如何在类型定义时忽略 undefined 和 null 类型？

```typescript
function myFunc(maybeString: string | undefined | null) {
  const onlyString: string = maybeString;   // Error
}
```

答：使用非空断言解决：

```typescript
function myFunc(maybeString: string | undefined | null) {
  const onlyString: string = maybeString!; // true
}
```

从以上示例可以看出，非空断言是⼀个后缀表达式操作符 ! 可以⽤于断⾔操作对象是⾮ null 和⾮ undefined 类型。具体⽽⾔，x! 将从 x 值域中排除 null 和 undefined 。
具体示例如下：

```typescript
function myFunc(maybeString: string | undefined | null) { 
const onlyString: string = maybeString; // Error 
const ignoreUndefinedAndNull: string = maybeString!; // Ok 
}
```

- 确定赋值断⾔
问题引入：如何解决下面这个问题？ 与非空断言的区别
代码遮住

答：使用确定赋值断⾔解决：

```typescript
let x！: number; 
initialize(); 

console.log(2 * x); // true
function initialize() { 
    x = 10; 
}
```

通过 let x!: number; 确定赋值断⾔，TypeScript 编译器就会知道该属性会被明确地赋值。

### 2.1.2  类型断言

类型断言就是告诉ts我知道这个变量的类型是什么，它没有运行时的影响，只是在编译阶段起作用

- 类型断言有两种形式。 其一是“尖括号”<>语法：

```typescript
let someValue: any = "this is a string";
let strLength: number = (<string>someValue).length;
```

- 另一个为as语法：

```typescript
let someValue: any = "this is a string";
let strLength: number = (someValue as string).length;
```

两种形式是等价的。 至于使用哪个大多数情况下是凭个人喜好；然而，当你在TypeScript里使用JSX时，只有 as语法断言是被允许的。

## 2.2   ?.   可选链运算符

可选链的核⼼ ?. 运算符,遇到 null 或 undefined 就可以⽴即停⽌某些表达式的运⾏。
🌰  可选的属性访问的例⼦：

```typescript
const val = a?.b;
```

🌰  可选函数调用的例子：

```typescript
let result = obj.customMethod?.();
```

## 2.3  ??  空值合并运算符

问题引入：对于非空判断是否有更优的写法，如：

```typescript
 let num;
 let num1 = 1;
 result = ( num !== null && num !== undefined ) ? num : num1 ;
 ```

答：使用空值合并运算符 ??  

```typescript
 let num;
 let num1 = 1;
 result = num ?? num1 ; // result === 1
 ```

通过以上案例，空值合并运算符就是当左侧操作数为 null 或 undefined 时，其返回右侧的操作数，否则返回左侧的操作数。

```typescript
const foo = null ?? 'default string'; 
console.log(foo); // 输出："default string" 
const baz = 0 ?? 42; 
console.log(baz); // 输出：0
```

      与逻辑或 || 运算符不同，逻辑或会在左操作数为 falsy 值时返回右侧操作数。也就是说，如果你使⽤ 
|| 来为某些变量设置默认的值时，你可能会遇到意料之外的⾏为。⽐如为 falsy 值（''、NaN 或 0）时。

- 与可选链操作符 ?. 搭配使用

```typescript
interface Customer { 
name: string; 
city?: string; 
} 
let customer: Customer = { 
name: "Semlinker" 
}; 
let customerCity = customer?.city ?? "Unknown city"; 
console.log(customerCity); // 输出：Unknown city
```

- 不能与 && 或 || 操作符共⽤

```typescript
null || undefined ?? "foo"; // raises a SyntaxError 
true && undefined ?? "foo"; // raises a SyntaxError
```

 但当使⽤括号来显式表明优先级时是可⾏的，⽐如：
 (null || undefined ) ?? "foo"; // 返回 "foo"

## 2.4   ?:  可选属性

```typescript
interface Person { 
   name: string; 
   age?: number; 
} 

let lolo: Person = { 
   name: "lolo" 
}
```

注意：只读参数放第一位，必选参数第二位，可选参数次之，不确定参数放最后。

## 2.5  _  数字分隔符

⼀个数字字⾯量，你现在可以通过把⼀个下划线作为它们之间的分隔符来分组数字，分隔符不会改变数值字⾯量的值，但逻辑分组使⼈们更容易⼀眼就能读懂数字

```typescript
const inhabitantsOfMunich = 1_464_301; 
const distanceEarthSunInKm = 149_600_000; 
const fileSystemPermission = 0b111_111_000; 
const bytes = 0b1111_10101011_11110000_00001101;
```
