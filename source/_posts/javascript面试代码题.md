---
title: javascript面试代码题
toc: true
comments: true
date: 2020-10-09 23:28:52
tags: 
- 面试
- 代码
category:
- 前端
- javascript
---

# javascript语言相关的
## 防抖
设置一个超时时间后再执行function，通过局部的定时器和闭包实现
```javascript
function debounce(function, ms = 1000) {
    let timer;
    return function(...args) {
        if (timer) {
            clearTimeout(timer);
        }
        timer = setTimeout(()=>{
            func.apply(this, args)
        }, ms)
    };
}

```

## 节流
限流，通过局部变量和闭包进行限流，返回的函数可以在ms内只被调用一次

```javascript
function throttle(func, ms = 1000) {
  let canRun = true
  return function (...args) {
    if (!canRun) return
    canRun = false
    setTimeout(() => {
      func.apply(this, args)
      canRun = true
    }, ms)
  }
}
```
##  new
```javascript

function myNew(Func, ...args) {
  const instance = {};
  if (Func.prototype) {
    Object.setPrototypeOf(instance, Func.prototype)
  }
  const res = Func.apply(instance, args)
  if (typeof res === "function" || (typeof res === "object" && res !== null)) {
    return res
  }
  return instance
}

// 测试
function Person(name) {
  this.name = name
}
Person.prototype.sayName = function() {
  console.log(`My name is ${this.name}`)
}
const me = myNew(Person, 'Jack')
me.sayName()
console.log(me)
```

<!-- more -->

## call和apply
### call
1. call改变函数this指向
2. 调用函数

```javascript
Function.prototype.myCall = function(context) {
  context = context || window;
  //将函数挂载到对象的fn属性上
  var orgfn = context.fn;
  context.fn = this;
  //处理传入的参数
  const args = [...arguments].slice(1);
  //通过对象的属性调用该方法
  const result = context.fn(...args);
  //删除该属性
  delete context.fn;
  if(orgfn){
    context.fn = orgfn;
  }
  return result
};
```

1. 首先我们对参数context做了兼容处理，不传值，context默认值为window；
1. 然后我们将函数挂载到context上面,context.fn = this；
2. 处理参数，将传入myCall的参数截取，去除第一位，然后转为数组；
3. 调用context.fn，此时fn的this指向context；
1. 删除对象上的属性 delete context.fn；
3. 将结果返回。

### apply
和call不同的是参数的处理
```javascript
Function.prototype.myApply = function(context) {
  context = context || window
  var orgfn = context.fn;
  context.fn = this
  let result
  // myApply的参数形式为(obj,[arg1,arg2,arg3]);
  // 所以myApply的第二个参数为[arg1,arg2,arg3]
  // 这里我们用扩展运算符来处理一下参数的传入方式
  if (arguments[1]) {
    result = context.fn(…arguments[1])
  } else {
    result = context.fn()
  }
  delete context.fn;
  if(orgfn){
    context.fn = orgfn;
  }
  return result
};
```

## bind
用bind改变了this指向的函数
但是如果用new操作符来调用，bind将会失效
```javascript
// bind实现
Function.prototype.mybind = function(){
  // 1、保存函数
  let _this = this;
  // 2、保存目标对象
  let context = arguments[0]||window;
  // 3、保存目标对象之外的参数,将其转化为数组;
  let rest = Array.prototype.slice.call(arguments,1);
  // 4、返回一个待执行的函数
  return function F(){
    // 5、将二次传递的参数转化为数组;
    let rest2 = Array.prototype.slice.call(arguments)
    if(this instanceof F){
      // 6、若是用new操作符调用,则直接用new 调用原函数,并用扩展运算符传递参数
      return new _this(...rest2)
    }else{
      //7、用apply调用第一步保存的函数，并绑定this，传递合并的参数数组，即context._this(rest.concat(rest2))
      _this.apply(context,rest.concat(rest2));
    }
  }
};
```

## 函数柯里化

### 将一个函数柯里化
```javascript
function curry(func) {
  return function curried(...args) {
    // 关键知识点：function.length 用来获取函数的形参个数
    // 补充：arguments.length 获取的是实参个数
    if (args.length >= func.length) {
      return func.apply(this, args)
    }
    return function (...args2) {
      return curried.apply(this, args.concat(args2))
    }
  }
}

// 测试
function sum (a, b, c) {
  return a + b + c
}
const curriedSum = curry(sum)
console.log(curriedSum(1, 2, 3))
console.log(curriedSum(1)(2,3))
console.log(curriedSum(1)(2)(3))
```

### 带终止函数的柯里化
注意需要为返回的函数添加一个终止的方法

```javascript
sum(1)(2)(3).valueOf() // 6
sum(1, 2)(3)(4).valueOf() //10
sum(1, 2, 3)(4, 5)(6, 7).valueOf() // 23
sum(1)(2, 3, 4)(5).valueOf() // 15

function add(...args){
	return args.reduce((a, b) => a + b)
}

function curry(fn){
	// 收集传参
	var args = args || []
	// 定义_fn函数并返回，保证柯里化后返回的是一个可调用的函数
	var _fn = function (){
		// 将最新传入的参数和之前收集的参数集合在一起
		var _args = args.concat([].slice.call(arguments))
		// 与柯里化的不同之处在于无法使用收集参数的个数与执行函数的参数个数来确定柯里化的终止条件
		// 为了保证柯里化处理的函数可以多次调用，需不停的递归
		return curry(fn, _args)
	}
	// 如果不停的递归，没有终止条件的话，会没有执行结果输出
	// 从上面的分析中可知，当调用vauleOf方法时返回执行结果，因此可以为_fn设置一个valueOf方法，调用时执行函数
	_fn.valueOf = function () {
		return fn.apply(this, args)
	}
	return _fn
}

var sum = curry(add)
sum(1)(2)(3).valueOf() // 6
sum(4, 5, 5)(1, 3, 2)(6, 6).valueOf() // 32

```

### 不定参数的柯里化

``` javascript
// 实现一个add方法，使计算结果能够满足如下预期：
add(1)(2)(3) = 6;
add(1, 2, 3)(4) = 10;
add(1)(2)(3)(4)(5) = 15;

function add() {
    // 第一次执行时，定义一个数组专门用来存储所有的参数
    var _args = Array.prototype.slice.call(arguments);

    // 在内部声明一个函数，利用闭包的特性保存_args并收集所有的参数值
    var _adder = function() {
        _args.push(...arguments);
        return _adder;
    };

    // 利用toString隐式转换的特性，当最后执行时隐式转换，并计算最终的值返回
    _adder.toString = function () {
        return _args.reduce(function (a, b) {
            return a + b;
        });
    }
    return _adder;
}

add(1)(2)(3)                // 6
add(1, 2, 3)(4)             // 10
add(1)(2)(3)(4)(5)          // 15
add(2, 6)(1)                // 9

```

## ES5实现继承
```javascript
function create(proto) {
  function F() {}
  F.prototype = proto;
  return new F();
}

// Parent
function Parent(name) {
  this.name = name
}

Parent.prototype.sayName = function () {
  console.log(this.name)
};

// Child
function Child(age, name) {
  Parent.call(this, name)
  this.age = age
}
Child.prototype = create(Parent.prototype)
Child.prototype.constructor = Child

Child.prototype.sayAge = function () {
  console.log(this.age)
}

// 测试
const child = new Child(18, 'Jack')
child.sayName()
child.sayAge()
```

## 实现一个Array.isArray
思路很简单，就是利用Object.prototype.toString
```javascript
Array.myIsArray = function(o) { 
  return Object.prototype.toString.call(Object(o)) === '[object Array]'; 
};
```

## instance of
```javascript
// L 表示左表达式，R 表示右表达式
function instance_of(L, R) {
    var O = R.prototype;
  L = L.__proto__;
  while (true) {
        if (L === null){
            return false;
        }
        // 这里重点：当 O 严格等于 L 时，返回 true
        if (O === L) {
            return true;
        }
        L = L.__proto__;
  }
}
```

## promise封装ajax
```javascript
function ajax(method, url, body, headers) {
    const promise = new Promise(function (resolve, reject) {
        const xmlhttp = new XMLHttpRequest();
        xmlhttp.open(method, url);
        for (const key in headers) {//遍历header,设置响应头
            let value = headers[key];
            xmlhttp.setRequestHeader(key,value);
        }
        xmlhttp.onreadystatechange = function () {
            if (this.readyState != 4) {
                return;
            }
            if (this.status == 200) {
                resolve(this.response);
            } else {
                reject(new Error("数据请求失败"))
            }
        }
        xmlhttp.send(body);

    })
    return promise;
}

```

# 基础算法
## 排序算法
[参考链接](https://segmentfault.com/a/1190000021638663?utm_source=wechat_session&utm_medium=social&utm_oi=580004672081891328)

```javascript
let bubbleSort = arr => {
  for (let i = 0; i < arr.length; i++) {
    for (let j = 0; j < arr.length - i - 1; j++) {
      if (arr[j] > arr[j + 1]) {
        let temp = arr[j];
        arr[j] = arr[j + 1];
        arr[j + 1] = temp;
      }
    }
  }
  return arr;
};

let selectSort = arr => {
  for (let i = 0; i < arr.length; i++) {
    Let min = arr[i]; // 初始化最小值为第一个元素
    let index = i; // 最小值下标
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[j] < min) { // 发现更小的数则交换位置
        min = arr[j];
        index = j;
      }
    }
    // 将当前趟最小值移动至其最终位置
    let temp = arr[i];
    arr[i] = min;
    arr[index] = temp;
  }
};

let insertSort = arr => {
  for (let i = 1; i < arr.length; i++) {
    let key = arr[i];
    let j = i - 1;
    while (j >= 0 && arr[j] > key) {
      arr[j + 1] = arr[j];
      j--;
    }
    arr[j + 1] = key;
  }
  return arr;
};

let shellSort = function (arr) {
  let len = arr.length, temp, gap = 1;
  // 动态定义间隔序列
  while (gap < len / 5) {
    gap = gap * 5 + 1;
  }
  for (gap; gap > 0; gap = Math.floor(gap / 5)) {
    for (let i = gap; i < len; i++) {
      temp = arr[i];
      for (let j = i - gap; j >= 0 && arr[j] > temp; j -= gap) {
        arr[j + gap] = arr[j];
      }
      arr[j + gap] = temp;
    }
  }
  return arr;
}

let merge = (left, right) => {
  let result = [];
  while (left.length && right.length) {
    if (left[0] <= right[0]) {
      result.push(left.shift());
    } else {
      result.push(right.shift());
    }
  }
  while (left.length) {
    result.push(left.shift());
  }
  while (right.length) {
    result.push(right.shift());
  }
  return result;
}

//采用自上而下的递归方法
let mergeSort = arr => {
  let len = arr.length;
  if (len < 2) {
    return arr;
  }
  let middle = Math.floor(len / 2),
    left = arr.slice(0, middle),
    right = arr.slice(middle);
  return merge(mergeSort(left), mergeSort(right));
}


var devide = function (array, start, end) {
  if(start >= end) return array;
  var baseIndex = Math.floor((start + end) / 2),
          i = start,
          j = end;

  while (i <= j) {
      while (array[i] < array[baseIndex]) {
          i++;
      }
      while (array[j] > array[baseIndex])  {
          j--;
      }

      if(i <= j) {
          var temp = array[i];
          array[i] = array[j];
          array[j] = temp;
          i++;
          j--;
      }
  }
  return i;
}

var quickSort = function (array, start, end) {
    if(array.length < 1) {
        return array;
    }
    var index = devide(array, start, end);
    if(start < index -1) {
        quickSort(array, start, index - 1);
    }
    if(end > index) {
        quickSort(array, index, end);
    }

    return array;
}

```

## 树的相关操作
### 前中后序遍历
### 层次遍历

## 链表
### 链表反转
### 链表找中点
### 链表判断是否有环



