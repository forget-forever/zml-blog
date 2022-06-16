---
title: ts逻辑的抽象与复用
date: 2022-05-28 10:53:21
tags: [typescript, 泛型, 抽象]
---

### ts类型
写过ts的人应该都对类型有这一个比较深刻的认识
在ts中，要求的是每一个变量都必须要有类型

<!--more-->

就拿最简单的一个foo函数来说
```typescript
const foo = (arg1: string, arg2: number) => {
    // do something
}
```
在函数声明时，它的所有的参数都是需要我们去声明类型的，如果说我们没有没有指定类型，那它是会报错的。(当然你可以配置noImplicitAny来让ts的隐式any类型不报错，但是这样的话写ts的意义又在哪里呢)

同时对于函数的返回值，ts在这个时候会对返回值做了一次推导，可以自动检测出函数的返回值的类型。
```typescript
const foo1 = (arg: string | number) ) => {
    return 1;
}
```

### ts类型的逆变与协变
> Typescript的协变和逆变和C# Scala中的类似，但是Typescript的会自动算出来接口属于协变还是逆变，C# Scala中需要显示声明in out标记接口。 在typescript需要在tsconfig中使用strictFunctionTypes参数开启逆变检查，否则就是双变(协变或者逆变)。


拿以下以下类型
Dog extends Animal，我们可以记成 Dog < Animal

当我们定义一个函数，运行的时候就会有一个这样的问题
```typescript
declare let fooAnimal: (animal: Animal) => void;
declare let fooDog: (dog: Dog) => void;

fooDog = fooAnimal // 正确，因为Dog类型有Animal类型的全部属性，

fooAnimal = fooDog // 错误，Animal类型不一定含有Dog类型的属性

```
以上的例子中说明函数类型发生了协变，且协变过程是自动的，不需要做多余的操作的。

接下来我们再拿函数返回的例子
```typescript
declare let fooAnimal: () => Animal;
declare let fooDog: () => Dog;

fooDog = fooAnimal // 错误

fooAnimal = fooDog // 正确

```
上述的例子中可以看到函数的类型发生了逆变

> 协变和抗逆变的意义在于泛型类型的类型转换带来的类型安全问题。
> 协变类型的接口只能允许派生类泛型赋值给父类泛型I<Dog> -> I<Animal>
> 逆变类型的接口只能允许父类泛型赋值给派生类泛型 I<Animal> -> I<Dog>

### 逻辑的复用
通过上面对函数类型协变和逆变的学习，我们可以发现，其实一个函数的参数类型我们可以只定义所需要的几个类型
```typescript
type Teacher = {
    name: string;
    age: number;
    job: string;
}

type Student = {
    name: string;
    age: number;
    job: string;
    grades: number;
}

const showMsg = (msg: {name: string, age: number}) => {
    return `姓名: ${msg.name};\n年龄: ${msg.age}`
}
declare let student: Student 
declare let teacher: Teacher
showMsg(student) // ok!
showMsg(teacher) // ok!
```
- 对于相应的逻辑块，只需要维护好自己所需要的几个属性;
- 通过这种方式可以更好做到一个逻辑的复用，对于入参的要求也只需要给到所需要的几个属性，在调用时ts会自动的对类型做出相应的逆变&协变

### 逻辑的抽象
出自于一个逻辑复用的延伸，很多时候我们会遇到以下问题：
- 入参与出参的耦合
- 入参与入参的耦合

这个时候就需要对一块逻辑做抽象处理了
```typescript
type Teacher = {
    name: string;
    age: number;
    job: string;
}

type Student = {
    name: string;
    age: number;
    job: string;
    grades: number;
}

const showMsg = <T extends {name: string, age: number}>(msg: T， getMoreMsg: (msg: T) => string) => {

    return `姓名: ${msg.name};\n年龄: ${msg.age};\n${getMoreMsg(msg)}`
}
declare let student: Student 
declare let teacher: Teacher
showMsg(student, (msg) => `成绩: ${msg.grades}`) // ok!
showMsg(teacher, (msg) => `职务: ${msg.job}`) // ok!

```
没错，可以进一步的将所需要的属性写入泛型的继承父类中
这样函数在调用的时候便会自动去完成对应的协变
