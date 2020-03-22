---
title: 2020年的Array能玩什么花样
date: 2020/03/22 13:01:00
updated: 2020/03/22 13:01:00
categories: 
- 前端知识
tags: 
- 数组
- 前端开发
---

嗨~我的老伙计们！

让我们来瞧瞧现在能用哪些数组方法了！还有什么能比知道这些用法更令人兴奋的呢？

这篇文章不是对Array全部用法的总结，只是给大伙瞧瞧比较好用的功能。

<!-- more -->

# 创建数组
## 构造函数
我敢打赌，没有人不知道这个用法：

```javascript
let a = new Array(10) // 创建长度为10，值均为undefined的数组
```

根据[es标准](https://tc39.es/ecma262/#sec-array-constructor-array)，Array直接当函数，用法没什么两样：

```javascript
let a = Array(10) // 直接当函数用也行
```

## 字面量
当然了，最方便的创建方法就是字面量声明了：

```javascript
let a = [] // 声明一个长度为0的数组
```

## Array.from
`Array.from()` 可以从一个类似数组，或者可迭代对象创建一个新的，浅拷贝的数组实例：

```javascript
Array.from({ length: 3 }) // [undefined x 3]
Array.from('hello') // ['h', 'e', 'l', 'l', 'o']
Array.from([1, 2, 3], x => x + 1) // [2, 3, 4]
```

## Array.of
`Array.of()` 方法创建一个具有可变数量参数的新数组实例，而不考虑参数的数量或类型。

```javascript
// 用of创建不容易出错，因为在只有一个参数时结果与多个参数时也是一致的
Array.of(3) // [3]
Array.of(1, 2, 3) // [1, 2, 3]
// 对比
Array(3) // [undefined x 3]
Array(1, 2, 3) // [1, 2, 3]
```

# 数组的方法
## 判断
### Array.isArray
`Array.isArray()` 用于确定传递的值是否是一个 `Array`。

```javascript
Array.isArray([1, 2, 3]) // true
Array.isArray({ x: 123 }) // false
```

### Array.prototype.includes
`includes()` 方法用来判断一个数组是否包含一个指定的值，如果包含则返回 true，否则返回 false。

```javascript
const arr = [1, 2, 3]
arr.includes(2) // true

// 这个方法等同于
function isInclude(arr, val) {
  return arr.indexOf(val) < 0 ? false : true
}
isInclude(arr, 2) // true
```

## 找值找索引
现在有更好用的方法 `find()`  和 `findIndex()` 来找值找索引了：

```javascript
const arr = [1, 2, 3]
arr.find(item => item > 1) // 2，只找最近的一个
arr.findIndex(item => item > 1) // 1
```

## 遍历和测试
有一大堆方法能够遍历数组，map 和 forEach 这类的就不讲了，有些方法还能测试数组：

```javascript
const arr = [1, 2, 3]
arr.every(item => item < 4) // true，每个元素都满足条件时返回true，空数组始终返回true
arr.some(item => item < 2) // true，只要有一个元素满足条件，就返回true，空数组始终返回false
arr.filter(item => item < 3) // [1, 2]，过滤满足条件的元素，组成新数组

// flat()方法深遍历数组，将遍历到的元素组成一个新数组，默认参数为1
const brr = [1, 2, [3, 4, [5, 6]]]
brr.flat() // [1, 2, 3, 4, [5, 6]]
brr.flat(2) // [1, 2, 3, 4, 5, 6]， 遍历两层
brr.flat(Infinity) // [1, 2, 3, 4, 5, 6]， 展开全部

const crr = [1, undefined, 2]
crr.flat() // [1, 2] // 空项会被移除

// flatMap()方法映射每一个元素，然后将结果压缩成一个数组
// 这儿有个拆词的例子
const drr = ['I love eat', 'cakes']
drr.flatMap(item => item.split(' ')) // ['I', 'love', 'eat', 'cakes']
```

## 取迭代器
`entries()` 和 `keys()` 方法帮助我们获取数组迭代器：

```javascript
const arr = ['a', 'b', 'c']
var iterator1 = arr.keys()
iterator1.next().value // 'a'
iterator1.next().value // 'b'
iterator2.next().value // 'c'

var iterator2 = arr.entries()
iterator2.next().value // [0, 'a']
iterator2.next().value // [1, 'b']
iterator2.next().value // [2, 'c']
```

## 累加
`reduce()` 方法对数组中的每个元素执行一个reducer函数（升序执行），将结果汇总为单个返回值，第一个参数是reducer函数，第二个参数是初始值。

```javascript
const arr = [1, 2, 3]
arr.reduce((sum, currentVal, index, arr) => sum + currentVal) // 6，相当于1+2+3
arr.reduce((sum, currentVal, index, arr) => sum + currentVal, 4) // 10，相当于1+2+3+4
```

---

好了伙计们，为了找到这些小花样我可花了不少时间，你的老板看到你在认真学习会很开心的，说不定会给你加工资呢。看在上帝的份上，支持关注一波吧！
