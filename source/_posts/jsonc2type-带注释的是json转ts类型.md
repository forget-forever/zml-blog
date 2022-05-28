---
title: 'jsonc2type,带注释的是json转ts类型'
date: 2022-03-10 16:40:22
tags: [jsonc, ts类型, npm包]
---
## 工具介绍

在做一个脚手架工具的时候想把带注释的json数据转化成ts类型，网上找了很多的js包，都没有找到合适的包。
本来typeofJsonc挺好用的，但是它转化出来的类型是一个分散的类型
这样的jsonc数据

<!--more-->

```typescript
{
    // 属性带双引号
    "code": 0,
    // 属性带单引号
    'data': { 
        /** 
         * 多行注释 
         * 多行注释
         */
        list: [{
            name: 'hello', // 属性不带引号
            age: 18
        }, {
            name: "world",
            age: 16,
            nickName: 'lucky' // 尾部单行注释
        }]
    }
}
```

会被转成这样的typescript类型
```typescript
export interface ReqBodyOther {
	/** 属性带双引号 */
	code: number;
	data: Data;
}

/** 属性带单引号 */
export interface Data {
	/**
	 * 多行注释
	 * 多行注释
	 */
	list: List[];
}

export interface List {
	/** 属性不带引号 */
	name: string;
	age: number;
	/** 尾部单行注释 */
	nickName?: string;
}
```
- 体验地址： [https://wulunyi.github.io/typeof-sjsonc-web/build/index.html](https://wulunyi.github.io/typeof-sjsonc-web/build/index.html)

看着转化结果挺符合自己的需求的，但是有的时候我还是想要把类型合并成一个，因为我那边要自动生成类型文件，如果这样的分散会导致许多的类型名称重名，还有就是我需要去指定开始节点。
对于我提出的这些需求，我对typeofJsonc进行了一些改造，能够指定自己要开始提取类型的节点


## 使用方法

```bash
yarn add jsonc2type
npm install jsonc2type
```

```typescript
import jsonc2type from 'jsonc2type';

const jsonc2type(`{
  a: 'aaaa', // comments
  b: 1234, // comments
  c: {a: 13432, d: '13424'}
}`, { startNode: 'c', name: 'type'})
/* type Type = {
  a: number,
  d: string,
}
*/

const jsonc2type(`{
  a: 'aaaa', // comments
  b: 1234, // comments
  c: {a: 13432, d: '13424'}
}`, { startNode: 'type', name: 'type'})
/* type Type = {
*  /* comments */
*  a: string,
*  /* comments */
*  d: string,
*  c: {
*    a: number,
*    d: string,
*  }
* }
*/
```

