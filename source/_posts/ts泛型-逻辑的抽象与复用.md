---
title: ts泛型-逻辑的抽象与复用
date: 2022-05-01 10:53:21
tags: [typescript, 泛型, 抽象]
---

### ts类型
&emsp;&emsp;写过ts的人应该都对类型有这一个比较深刻的认识
&emsp;&emsp;在ts中，要求的是每一个变量都必须要有类型
就拿最简单的一个foo函数来说
```typescript
const foo = (arg1: string, arg2: number) => {
    // do something
}
```
&emsp;&emsp;在函数声明时，它的所有的参数都是需要我们去声明类型的，如果说我们没有没有指定类型，那它是会报错的。(当然你可以配置noImplicitAny来让ts的隐式any类型不报错，但是这样的话写ts的意义又在哪里呢)

&emsp;&emsp;同时对于函数的返回值，ts在这个时候会对返回值做了一次推导，可以自动检测出函数的返回值的类型。
```typescript
const foo1 = (arg: string | number) ) => {
    return 1;
}
```

### ts类型的逆变与协变
ts与其他的强类型的类型声名不同的是，ts的类型是允许协变
比如说在Java中
```java
Class Dog1 {
    public void wang() {
        System.printf.out('wang');
    }
}
Class Dog2 {
    public void wang() {
        System.printf.out('wang');
    }
}
Dog1 dog1 = new Dog1()
Dog2 dog2 = new Dog2()

```
其中的dog1和dog2实例虽然有着相同属性，但是他们源自于不同的类，所以说他们是不可以相互转化的
```java
dog1 = dog2 // error
```

但是在ts中
```typescript
type Dog1 = {
    wang: () => void
}
type Dog2 = {
    wang: () => void
}
cosnt dog1
```

##### 类型的协变
在ts中如果一个类型从

