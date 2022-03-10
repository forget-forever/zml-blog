---
title: typecript，让类型约束成为一种习惯
date: 2022-01-08 18:36:31
tags: #typescript #类型编程
description: 伴随着ts4.3的发布，ts4.4也已经处于beta阶段了，ts成为了一个前端规范的趋势，随之vue、react和其他的plugin都开始了使用ts进行编写，ts已经成为了一个前端开发者所必需熟悉的一个东西。能否写出准确的ts类型也成为了一段代码的质量的评判条件之一
---
## 基础知识概述

伴随着ts4.3的发布，ts4.4也已经处于beta阶段了，ts成为了一个前端规范的趋势，随之vue、react和其他的plugin都开始了使用ts进行编写，ts已经成为了一个前端开发者所必需熟悉的一个东西。能否写出准确的ts类型也成为了一段代码的质量的评判条件之一。

 ts类型并不是只能有类型定义，然后给逻辑代码用这么简单。也并不是说代码懂得了一个any这样的全能类型，然后就到处使用any，只关心逻辑代码这块（能跑就行！！！！）。

<!--more-->

<h2>1、是开发和维护过程中的工具</h2>

对于ts类型，他也是有自己的编程逻辑的。对于它，我们可以将它理解成一个我们项目开发与维护的工具，这个取决于我们对这个工具的了解程度和利用的程度，利用的好，它是规范我们的逻辑代码的一大利器，代码中的变量便会在明确的类型指引下快速且高效的开发。

<ul><li><strong>约束某一块代码的具体功能</strong></li></ul>

比如说在一个函数中，我们因为有了一个函数的约束，我们可以很明确的知道这个段逻辑块要完成的是一个什么样的功能，不得不说的是，很多人觉得ts是个累赘，那就是很多人都是先写逻辑，后写类型的，就是一种差不多的想法，代码能跑就行，最后只能写成了anyscript。 比如说：

```typescript
const foo = (arg: A): B => {
    // do something....
}
```

我们便能知道这个函数是一个要将A类型的变量处理处理成B类型的函数，这段代码有这样的作用就通过类型就可以显而易见了，当然前提是A和B的类型是明确的，不能写一个any，Object草草了事，当然在返回类型是也要尽量的准确点，比如说，有的时候为了一个求一个逻辑代码写的顺畅，盲目的让类型迎合变量。

```typescript
const foo = (a: A): string | number | boolean => {
    // do something...
}
```

```typescript
const foo = (a: string | number | object | boolean): B => {
    // do something...
}
```

这样的做法在ts编程的时候是不可取（不建议）的，其实在纯js中也是不建议这样的，这样失去了逻辑快单一功能的原则，会让后续的对它的维护显得特别的困难，在之后的调试过程中也会显得格外的困难。

可能，我们会真的碰到这个函数逻辑就是要这样的耦合，那么我们不妨试试另一种类型声明的方式。

```typescript
interface foo {(a: A, b: string): void} // 参数a为类型A时，那么b的类型为string
interface foo {(a: B, b: number): void} // 参数a为类型B时，那么b的类型为number
interface foo {(a: C, b: number): D} // 参数a为类型C时，那么b的类型为number，且函数会有返回值D。
```

这三个都是给一个函数声明类型，然后可以很有效的做到逻辑耦合但是类型不耦合，同时也对函数的调用起到了类型校验的作用，不再是像以前写着一个联合类型，让人调用函数的时候觉得这个类型有点傻乎乎的样子，什么也不懂。

这样的方式可以在很大的程度上，让函数的调用变得轻松，不至于让写出的东西让别人无法调用或者类型靠as去断言。

```typescript
declare interface Foo {
  (a: string, b: string): void;
}// 参数a为类型string时，那么b的类型为string
declare interface Foo {(a: boolean, b: number): void} // 参数a为类型boolean时，那么b的类型为number
const foo: Foo = (a: string | boolean, b: number | string) => {
  // do something......
}

foo('1', 2); // error,a为string类型时，b参数的类型为string
foo(false, 2); // ok👌
foo(true, '2') // error,第一个参数为boolean时，第二个参数必须为数字
```

###### 注：这样声明的类型只能用interface，原因可以见下文的interface、type和class的区别

 对于函数的逻辑块的类型声明可以这样，同理，组件中我们也可以做到。 其实组件中特别是想要复用性高一点的组件，一般都不会太建议太高的耦合度了，但是，方法也是有的了。

在react中，如果是函数式组件，我们可以跟上面说的函数定义一样的去做

```typescript
interface IProps {
    a: A1;
    b: B1;
};
interface IProps {
    a: A2;
    b: B2;
}
const ComponentA: React.FC<IProps>;
```

同样，这样的耦合度对于这个组件的编写还是不太好的，但是很大程度上可以简化组件的调用，让别人可以更好的去调用它。 在class组件中，这样的做法会显得更加简便

```typescript
interface ComponentA {
    props: {....}
    func(): A
}
interface ComponentA {
    props: {....}
}
class ComponentA extends Component {
    constructor() {
        this.state = {.....}
    }
    render() {
        return (....)
    }
}
```

这样子对组件的编写过程起到的作用还是比较小的，但是对于组件的调用却有很大的意义，它可以查出组件错误的调用方式。

<ul><li><strong>约束静态数据</strong></li></ul>

作为工具，说白了就是没有它也一样，照样可以做出我想要的东西。但是ts能够一直发展过来，能够受到这么多的前端开发者的青睐自有它的原因。它确实是可以很好的去约束我们的代码，约束我们开发过程中所制造出的各种数据，这个也是取决于，自己对ts的理解程度的。 比如说我们在定义如下数据的类型时

```typescript
const schedule = {
  '00:00': 0,
  '00:30': 0,
  '01:00': 0,
   // ..... 省略，每隔30一个
  '12:00': 0,
  '12:30': 1,
  '13:00': 1,
  '13:30': 1,
  '22:30': 0,
  '23:00': 0,
  '23:30': 0,
};
```

在只知道interface，type的时候会一项一项的列出来

```typescript
type ISchedule = {
  '00:00': number,
  '00:30': number,
  '01:00': number,
   // ..... 省略，每隔30一个
};
```

然后再深入，知道了[in]，然后又会觉得，类型不过如此，会直接在写出

```typescript
type ISchedule = {
    [k in string]: number
}
```

之后，更加深入的去知道了自带的工具类型

```typescript
type ISchedule = {
    [k in string]: number
}
```

很明显，对ts的使用程度就可以体现出来了，写法的不同，ts发挥的作用也都是不同的。

 后面两种的写法很明显只是为了规避eslint的报错而写的类型的，定义太宽泛，这个时候如果是前端自己写类型定义可以使用类型的模板字符串

```typescript
type N = 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 0
type Time<T extends 0 | 1 | 2> = T extends 0 | 1 ? `${T}${N}` : `${T}${0|1|2|3}`
enum CheckEnum {}
type A = Record<`${Time<0 | 1> | Time<2>}:${'00' | '30'}`, CheckEnum>
```

模板字符串，它集成了[in]语法糖，让所有的可能类型自由组合，可以自动对所有的情况做一次遍历。 当然这样写只是为了提示一下ts的新特性中还有类型模板字符串，但是也可以看得出，ts的类型也越来越完善，可以让类型定义的越来越精确。有了这个模板字符串，很多的以前我们就写一个string的类型，我们都可以给定一个更加精确的定义，来保证我们的类型的准确性 比如说，我们给request定义url的时候我们就可以不写string，完全可以定义一个独有的IUrl类型，来规范url的编写。

```typescript
type IUrl = `/${string}`
export request = <R>(url: IUrl, options) => Promise<T>
```

这样就可以避免犯前面忘了加 / 这样的低级错误。 此外，类型模版字符串甚至可以用于校验电话号码，时间格式等字符串类型的数据，预防静态数据认为输入是校验太少而出现的错误。

<ul><li><strong>类型的断言</strong></li></ul>

其实ts作为工具，它和eslint的功能差不多，都是一个规范代码书写，可快发提效的手段。同时也会遇到一些ts无法准确做出判断的时候，毕竟ts是不参与逻辑代码的计算的，应该说，是不支持解耦之后的代码的类型运算。

```typescript
type Foo1 = {
  value: number,
  type: 'a'
}
type Foo2 = {
  value: string,
  type: 'b'
}
const foo = (arg: Foo1 | Foo2) => {
  if (arg.type === 'a'){
    // arg is Foo1
    console.log(arg)
  }
  if (arg.type === 'b') {
    // arg is F002
    console.log(arg)
  }
}
```

在这种前后耦合的情况下类型还是可以会能有自己的推导的，具体的，可以去看类型的合成与拆分，这也是在某个函数一定要耦合的时候建议的做法。

###### 好了以上的还是题外话，想提醒一下，是不是类型的联合都还搞不清楚。对于一些情况，比如说我们在一些dialog中，我们有时候会习惯用一个对象来驱动弹窗的显隐。让弹窗的显隐通过是否有数据驱动

```typescript
<Dialog visible={!!data} close={data.close}> // error, data可能为undefined
 // some thing
</Dialog>
```

这种情况下就难免会给data定义undefined的联合类型的了，到了一些方法中，这个主要是因为数据与视图解耦了，ts就会提醒你某个参数有可能为undefined，但是为undefined的时候数据都是不执行的，其实前面做一个非空检验也没什么大不了的，但是这个时候就看的出ts就比较傻了，此时我们的断言就可以用上了

```typescript
<Dialog visible={!!data} close={data!.close}> // ok, 这里可以用一个非空断言
 // some thing
</Dialog>
```

大部分的时候非空断言( ! )，我们基本上就够用了，可以解决大部分类型推导不过来的问题。

但是还有很多时候，ts的使用程度不同的人会出现不一样的情况，有的人对类型的定义严格，有的人定义的宽松，为什么会这样可以看前面的概述。但是问题还是要解决的，我们这个时候如果真的非常肯定不会出问题，那么我们不妨试试as断言。

```typescript
type Foo1 = {
  value: number ｜ string,
  type: string
}
type Foo2 = {
  value: string,
  type: 'b'
}
const foo = (arg: Foo2) => {
  // do something
}
const a: Foo1 = {
    .....
}
foo(a) // error, Foo1类型不能给Foo2类型！
foo(a as Foo2) //ok，Foo2类型只是比Foo1类型更小，此处的断言可以告诉ts，我比你更清楚这个数据
```

从这个小例子中我们可以看出，断言对很多类型逃避主义的人其实也是一个非常大的福音，类型定义的时候宽泛就好了，衔接不上的时候直接as unknown as ...、as any as ...。如果有这种的行为，我也只能说，干的漂亮！ts技术又有了一点提升，只要代码能跑，还真让别人挑不出一点ts的毛病。 话说回来，还是好好的重视类型报错、严格的定义类型吧，如果真的是有非要有耦合的情况，可以看看上面本节的第一点说的方法吧。ts要不了多少时间的，真的可以避免错误，还有减少很多找bug的时间。

<ul><li><strong>全局类型声明</strong></li></ul>

应该很多人想过一个问题，为什么我们可以不需要引入，就可以用Record、Omit、Partail这样的工具类型。还有就是，我们自己开发过程中其实也写出了很多的好用的类型，有的是可以完全脱离某个项目，直接到处都可以用的，比如说我们Omit的源码。

```typescript
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

###### 它的第二个参数受的是any约束，虽然也有它的原因，但是我们完全可以自己再写一个MyOmit

```typescript
type MyOmit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;
```

###### 这样就可以使得MyOmit更符合我们的需求

###### 下面的问题来了，如果我们只是这样的定义一下，别的地方想使用每次都得要去import，但是这个东西本来就是开发的时候稍微用一样，打包的时候都是去掉的。那么有什么办法可以别的地方直接用就好了？？

###### 其实这个也很简单，只需要在根目录下定义一个.d.ts，文件就好了

```typescript
// public.d.ts
declare type MyOmit<T, K extends keyof T> = Pick<T, Exclude<keyof T, K>>;
```

这样就好了，就可以在整个项目中使用了。

如果你以为这一点就这么完了，那你想的肯定是太简单了，另外拓展一下，其实类型也是可以改写的😂😂。

比如说，在使用Object.entries的时候，通过lib.es2017.d.ts中的源码可以看到，entries的类型定义还是有点不太好，不太适合日常的ts类型开发。

```typescript
     /**
         * Returns an array of key/values of the enumerable properties of an object
     * @param o Object that contains the properties and methods. This can be an object that you created or an existing Document Object Model (DOM) object.
     */
    entries<T>(o: { [s: string]: T } | ArrayLike<T>): [string, T][];
```

这个时候，我们就可以考虑一下改写它的类型，当然不是去修改编辑器的代码，别想太多。

```typescript
// .d.ts
declare interface ObjectConstructor {
  entries<T>(obj: T): [keyof T, T[keyof T]][];
}
```

<h2>2、是一个具有编程思想的语言</h2>

###### 上一节中我们从typescript是一个规范开发的工具入手，讲述了ts可以有的几大功能。那么这一节，就讲述一下typescript其实也是有一定的编程思想的。它也是有自己的变量声明、条件语句、循环语句、作用域的，当你习惯了这个编程思想之后，就能够更加深刻的感受到ts所带来的便利

<ul><li><strong>interface、type、class的区别</strong></li></ul>

###### 很多人对interface、type的理解仅仅只是一个声明类型方式的区别。 但是事实上，它就是类型定义的一个区别。 只不过，有以下几点的区别

###### 1、type，class定义的类型不可以重复，interface定义的类型可以重复

```typescript
type Record = {...} // error, Record已经被定义过
interface ObjectConstructor {
    ......
} // ok👌，interface 定义的类型可以重复定义，重新给Object的一些原型方法写类型
```

###### 这就是在上节中所讲的类型可以被重写的另一个知识点。 2、type声明的对象可以使用[in]，interface，class的不可以

```typescript
type Keys = "小王" | "小文"
type X = {
  [key in Keys]: string
}
const test: X = {
    '小王': '肌肉男',
    '小文': '也是肌肉男'
}

interface XX {
    [k in keys]: string // error!!!
}
```

###### 3、interface，class使用extends、implements组合类型，type则可以通过&, |符号组合、合并类型

```typescript
interface Animals1 {
    ...
}
interface Animals2 {
    ...
}
interface Cat extends Animals1, Animals2 {
    ...
}

type Dog = Animals1 & Animals2 & {
    ...
}
```

###### 4、interface定义的类型可以被改写，type、class定义的类型不可以被改写

```typescript
// a.d.ts
export interface Foo {
    aa: string;
    ....
}
// {aa: string, ...}
```

```typescript
// b.d.ts
import type { Foo } from 'a'
interface Foo {
    aa: number;
    ...
}
// {aa: number; ...}
```

###### 5、type可以使用typeof、keyof、infer去反推类型，interface、class不可以

```typescript
const foo = {
    a: 'a',
    b: 2,
    c: true
}
type Foo = typeof foo //{ a: string, b: 2, c: boolean }
```

从以上的几点区别可以看出，类型的定义上，interface和class更像是给ecmascript增加了接口的概念，让类型与代码可以耦合，真正的将js变成了强类型语言（当然总是联合类型就另说了，基本上不会在意这么多了，基本上就不会管类型声明上的区别了）。

```typescript
interface Animal1 {
    a: string;
}
interface Animal2 {
    b: number;
}
interface Animal extends Animal1, Animal2 {
    ...
} // interface 可以使用extends集成
class Animal implements Animal1, Animal2 {
    ...
} // 也可以使用class的extends和implements进行继承
```

###### 可以看出来使用的还是java里面的那一套。 之后的type的类型定义，便又是另一个编程思想了，它就是一个类型编程的思想了。它可以使用typeof，keyof，infer等各种类型推导。同时可以使用[in]等对类型各种各样的推导操作

```typescript
const defaultData = {
    name: string;
    age: number;
}
type IDefaultData = typeof defaultData // {name: string; age: number}
type Ikeys = keyof defaultData
```

###### 同时也可以开始有了通过各种工具类型（有自带的也有自己写的 ），对类型进行各种运算

```typescript
const defaultData = {
    name: string;
    age: number;
}
type IDefaultData = typeof defaultData // {name: string; age: number}
type Ikeys = keyof defaultData

type IData = Record<string, typeof defaultData>;
type IList  = Record<string, Pick<IDefaultData, 'name'> & {sex: number}>
```

```typescript
type Foo<T> = T extends { a: infer U; b: infer U } ? U : never;
type T0 = Foo<{a: string, b: string}> // string
type T1 = Foo<{a: string, b: number}> // string | number
type T6 = Foo<{a: number, c: string, b: symbol}> // symbol
```

<ul><li><strong>类型的推导</strong></li></ul>

###### 类型的推导那就是比较考验对类型够不够理解了，主要涉及到的ts知识点就为typeof、keyof、infer。 不过值得说的就是infer对于业务代码中的类型使用还是没那么多的。用的最多的还是typeof和keyof，它可以让人更快速的去得到想要的类型

```typescript
type Foo1 = {
    a: A1 //就当它是A1类型吧
    b: B1 // 就当它是B1类型吧
}

type Foo2 = Record<keyof Foo1, Foo>
```

###### 而对于infer，我们很多时候可以用它来制作工具类型，我们这个时候是可以参考Parammeters、ReturnType等工具类型

```typescript
type Parameters<T extends (...args: any) => any> = T extends (...args: infer P) => any ? P : never;

/**
 * Obtain the parameters of a constructor function type in a tuple
 */
type ConstructorParameters<T extends abstract new (...args: any) => any> = T extends abstract new (...args: infer P) => any ? P : never;

/**
 * Obtain the return type of a function type
 */
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
```

<ul><li><strong>类型的条件语句</strong></li></ul>

###### 这个又是一个关于extends的作用了，extends它不仅可以在interface上可以使用继承(或许有的人的理解仅限于此)。同时可以有一个约束的功能，其实很像继承的反推

```typescript
interface Lengthwise {
  length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
  console.log(arg.length); // ok👌， arg 受Lengthwise约束，因此lenght是有的
  return arg;
}
```

```typescript
type Foo<T extends 0 | 1 | 2> = T extends 0 | 1 ? `${T}${N}` : `${T}${0|1|2|3}`
// 也可以通过extends的约束功能给类型做一个三元表达的条件语句运算`
```

<ul><li><strong>类型的循环遍历</strong></li></ul>

既然是可以编程的，那就不可以缺少循环遍历的语句了。ts的类型也是可以通过[in]对类型进行循环遍历的。同样的，这只能在type中使用

```typescript
type Foo = {
    a: string;
    b: number;
    c: boolean;
    d: symbol;
}
type A = {
    [k in keyof Foo]: B;
}
// {a: B; b: B; c: B; d: B}

interface A {
    [k in keyof Foo]: B;
} // error,别问为什么不可以，语言就是这样
```

###### 甚至乎，我们还可以使用类型模版字符串，快速的写出更加标准的类型

```typescript
type Foo = {
    a: string;
    b: number;
    c: boolean;
    d: symbol;
}
type A = {
    [ `get${k}` in keyof Foo]: B;
}
// {geta: B; getb: B; getc: B; getd: B}

```

###### 同样，作为

<ul><li><strong>静态数据的类型</strong></li></ul>

###### 说到这个，我们就不得不说的了，很多时候，我们写一个string，number其实也算是一个类型的敷衍 比如说，我们在定义一个mode，或者status时

```typescript
type Foo  = {
    status: number;
}
type Foo = {
    mode: string;
}
```

###### 这样的类型定义，其实很明显也是在逃避的，毕竟后面的status或者mode也有可能是要用的。后面的代码很有可能就会这样

```typescript
if(status === 0) {
    // do something
}
if (mode === 'xxx') {
    // do something
}
```

###### 这样就是让后面的维护者风中凌乱了，很不利于维护，同时那个number和string也很容易写错。 其实对于这种静态的类型，建议是不应该用个number和string的。 对于前端自产自销的mode或者status，不妨试试

```typescript
if(status === 0) {
    // do something
}
if (mode === 'xxx') {
    // do something
}
```

###### 如果是对于后端返回的而且要用到的，这个时候就可以使用枚举了

```typescript
enum IStatus {
    off,
    on
} // {off: 0, on: 1}
type Foo  = {
    status: Istatus;
}// 在之后的过程中都可以使用这个IStatus枚举
```

###### 通过这个就不得不说一下了，其实enum与相似，它既可以参与逻辑，也可以参与类型。在类型中它可以表示一个基本类型。在逻辑中，它就可以充当出一个constant的作用了 对于静态的数据，可以做一点拓展知识，那就是as const

```typescript
cosnt arr = ['a', 'b', 'c'] 
type Foo = typeof arr // string[]
// 这个时候ts的反推类型就是string[]
```

```typescript
cosnt arr = ['a', 'b', 'c'] as const
type Foo = typeof arr // ['a', 'b', 'c']
// 这个时候ts的反推类型就是只读熟悉了，它可以用来弥补readonly的一些缺陷
```

<ul><li><strong>动态类型的定义</strong></li></ul>

###### 是编程，那么就得有变量的声明，和数据的自顶向下的数据流，ts中也是一样的，这个东西便是泛型了。有了它，就可以正式的将类型带入了编程的行列。 逻辑代码上泛型可以跟函数跟类进行耦合，去写出一个更加可用的模块

```typescript
const foo = <T extends {type: 'a'|'b', val: any}>(arg: T) => {
    switch (arg.type){
        case 'a': ...
        case 'b': ...
    }
    return arg.val
}
```

```typescript
class Foo<T, U> {
    a: T
    b: U
    foo: (a: T) => void
}
```

###### 类型编程上我们可以通过泛型，写出各种好用的工具类型，提升我们的开发效率 我们可以拿我们熟知的protable的类型定义说起

```typescript
export declare type ProSchema<T = Record<string, unknown>, Extra = unknown, V = ProSchemaComponentTypes, ValueType = 'text'> = {
    /** @name 确定这个列的唯一值 */
    key?: React.ReactText;
    /**
     * 支持一个数字，[a,b] 会转化为 obj.a.b
     *
     * @name 与实体映射的key
     */
    dataIndex?: keyof T;

    render?: (dom: React.ReactNode, entity: T, index: number, action: ProCoreActionType, schema: ProSchema<T, Extra> & {
        isEditable?: boolean;
        type: V;
    }) => React.ReactNode;
}
```
