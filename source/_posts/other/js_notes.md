---
title: Javascript学习笔记
date: 2017-07-21 16:36:48
tags:
- javascript
- node.js
categories:
- javascript
---

JavaScript不区分整数和浮点数，统一用Number表示

JavaScript 大小写敏感

用  // 和 /* */ 做注释

&& 与运算 ||或运算

==比较，会自动转换类型 ， === 比较不会转换类型，不同类型false

NaN 不与其他数值相等，判断非数值用isNaN()

js 用null 表空值

js 列表[] 可以由多种不同类型的数值

pop and push 对列表的尾部进行操作

unshift给列表头添加数据，shift = pop 头部版

splice 对列表各种操作

```javascript
var arr = ['Microsoft', 'Apple', 'Yahoo', 'AOL', 'Excite', 'Oracle'];
// 从索引2开始删除3个元素,然后再添加两个元素:
arr.splice(2, 3, 'Google', 'Facebook'); // 返回删除的元素 ['Yahoo', 'AOL', 'Excite']
arr; // ['Microsoft', 'Apple', 'Google', 'Facebook', 'Oracle']
// 只删除,不添加:
arr.splice(2, 2); // ['Google', 'Facebook']
arr; // ['Microsoft', 'Apple', 'Oracle']
// 只添加,不删除:
arr.splice(2, 0, 'Google', 'Facebook'); // 返回[],因为没有删除任何元素
arr; // ['Microsoft', 'Apple', 'Google', 'Facebook', 'Oracle']
```

新浏览器才能用map和set

将所有数值绑定到一个全局变量中，这样避免命名空间重用

```javascript
// 唯一的全局变量MYAPP:
var MYAPP = {};

// 其他变量:
MYAPP.name = 'myapp';
MYAPP.version = 1.0;

// 其他函数:
MYAPP.foo = function () {
    return 'foo';
};
```

for (xxx of xxx) { } 遍历用



构造原型

```javascript
//原型函数
function Student(std){
  this.name = std.name || "xiaoming";
  this.grade = std.grade || 1;
}
//定义公用函数
Student.prototype.hello(){
  alert("Hello, "+this.name+"!")
}

//定义构造函数
function createStudent(std){
  return new Student(std || {})
}
```

有class但是很多浏览器不支持



要加载一个新页面，可以调用`location.assign()`。如果要重新加载当前页面，调用`location.reload()`方法非常方便。