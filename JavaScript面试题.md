# JavaScript 面试题

## 基础语法

### 1. 数据类型
**问题**: JavaScript 有哪些数据类型？如何判断变量的数据类型？

**答案**:
- 基本数据类型：`undefined`, `null`, `boolean`, `number`, `string`, `symbol`, `bigint`
- 引用数据类型：`object`（包括数组、函数、日期等）

判断方法：
```javascript
// typeof 操作符（注意 null 的特殊情况）
typeof 123         // "number"
typeof "hello"     // "string"
typeof null        // "object" (历史遗留问题)

// Object.prototype.toString.call()
Object.prototype.toString.call([])     // "[object Array]"
Object.prototype.toString.call({})     // "[object Object]"
Object.prototype.toString.call(null)   // "[object Null]"

// instanceof 操作符
[] instanceof Array    // true
```

### 2. 变量提升
**问题**: 什么是变量提升？var、let、const 的区别？

**答案**:
```javascript
// var 变量提升
console.log(a); // undefined (不会报错)
var a = 1;

// let/const 暂时性死区
console.log(b); // ReferenceError
let b = 2;

// 区别：
// var: 函数作用域，可重复声明，有变量提升
// let: 块级作用域，不可重复声明，有暂时性死区
// const: 块级作用域，不可重复声明，有暂时性死区，声明时必须初始化
```

### 3. 作用域和作用域链
**问题**: 解释 JavaScript 的作用域和作用域链

**答案**:
```javascript
var a = 1;
function outer() {
  var b = 2;
  function inner() {
    var c = 3;
    console.log(a, b, c); // 1 2 3
  }
  inner();
}
outer();

// 作用域链：inner -> outer -> global
// 变量查找顺序：当前作用域 -> 上级作用域 -> ... -> 全局作用域
```

## 高级概念

### 4. 闭包
**问题**: 什么是闭包？闭包的应用场景？

**答案**:
```javascript
// 闭包定义：函数和其词法环境的组合
function createCounter() {
  let count = 0;
  return function() {
    return ++count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2

// 应用场景：
// 1. 数据私有化
// 2. 模块化
// 3. 回调函数
// 4. 函数柯里化
```

### 5. 原型和原型链
**问题**: 解释 JavaScript 的原型和原型链机制

**答案**:
```javascript
// 构造函数
function Person(name) {
  this.name = name;
}

// 原型方法
Person.prototype.sayHello = function() {
  console.log(`Hello, I'm ${this.name}`);
};

const person = new Person('Alice');

// 原型链
person.__proto__ === Person.prototype        // true
Person.prototype.__proto__ === Object.prototype  // true
Object.prototype.__proto__ === null          // true

// 属性查找顺序：实例 -> 构造函数原型 -> Object.prototype -> null
```

### 6. this 指向
**问题**: JavaScript 中 this 的指向规则

**答案**:
```javascript
// 1. 默认绑定（非严格模式指向 window，严格模式为 undefined）
function fn() {
  console.log(this);
}

// 2. 隐式绑定（调用对象）
const obj = {
  name: 'obj',
  fn() {
    console.log(this.name); // 'obj'
  }
};

// 3. 显式绑定（call、apply、bind）
fn.call(obj);
fn.apply(obj);
const boundFn = fn.bind(obj);

// 4. new 绑定
function Constructor() {
  this.name = 'instance';
}
const instance = new Constructor();

// 5. 箭头函数（继承外层作用域的 this）
const arrow = () => {
  console.log(this); // 继承定义时的 this
};
```

## 异步编程

### 7. 事件循环
**问题**: 解释 JavaScript 的事件循环机制

**答案**:
```javascript
// 执行顺序：同步代码 -> 微任务 -> 宏任务
console.log('1');

setTimeout(() => console.log('2'), 0);  // 宏任务

Promise.resolve().then(() => console.log('3'));  // 微任务

console.log('4');

// 输出：1 4 3 2

// 宏任务：setTimeout, setInterval, I/O, UI 渲染
// 微任务：Promise.then, queueMicrotask, MutationObserver
```

### 8. Promise
**问题**: Promise 的基本用法和实现原理

**答案**:
```javascript
// 基本用法
const promise = new Promise((resolve, reject) => {
  // 异步操作
  if (/* 操作成功 */) {
    resolve(value);
  } else {
    reject(error);
  }
});

promise
  .then(value => {
    // 处理成功
  })
  .catch(error => {
    // 处理错误
  })
  .finally(() => {
    // 无论成功失败都执行
  });

// Promise.all - 所有成功才成功
Promise.all([promise1, promise2]).then(results => {
  // 所有 promise 都成功
});

// Promise.race - 最先完成的结果
Promise.race([promise1, promise2]).then(result => {
  // 最先完成的 promise 结果
});
```

### 9. async/await
**问题**: async/await 的使用和错误处理

**答案**:
```javascript
// async 函数返回 Promise
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Error:', error);
    throw error;
  }
}

// 并发执行
async function fetchMultiple() {
  const [data1, data2] = await Promise.all([
    fetchData1(),
    fetchData2()
  ]);
  return { data1, data2 };
}
```

## 高阶函数

### 10. 防抖和节流
**问题**: 实现防抖和节流函数

**答案**:
```javascript
// 防抖：延迟执行，重复触发会重新计时
function debounce(func, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      func.apply(this, args);
    }, delay);
  };
}

// 节流：固定时间间隔执行
function throttle(func, delay) {
  let lastExecTime = 0;
  return function(...args) {
    const currentTime = Date.now();
    if (currentTime - lastExecTime >= delay) {
      func.apply(this, args);
      lastExecTime = currentTime;
    }
  };
}

// 使用示例
const debouncedSearch = debounce(search, 300);
const throttledScroll = throttle(handleScroll, 100);
```

### 11. 深拷贝
**问题**: 实现深拷贝函数

**答案**:
```javascript
function deepClone(obj, map = new WeakMap()) {
  // 处理 null 和基本类型
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }
  
  // 处理循环引用
  if (map.has(obj)) {
    return map.get(obj);
  }
  
  // 处理日期
  if (obj instanceof Date) {
    return new Date(obj);
  }
  
  // 处理正则
  if (obj instanceof RegExp) {
    return new RegExp(obj);
  }
  
  // 处理数组和对象
  const cloned = Array.isArray(obj) ? [] : {};
  map.set(obj, cloned);
  
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      cloned[key] = deepClone(obj[key], map);
    }
  }
  
  return cloned;
}
```

## ES6+ 新特性

### 12. 解构赋值
**问题**: ES6 解构赋值的使用场景

**答案**:
```javascript
// 数组解构
const [a, b, ...rest] = [1, 2, 3, 4, 5];
console.log(a, b, rest); // 1 2 [3, 4, 5]

// 对象解构
const { name, age, ...others } = { name: 'Alice', age: 25, city: 'NYC' };

// 函数参数解构
function greet({ name, age = 18 }) {
  console.log(`Hello, ${name}, you are ${age} years old`);
}

// 嵌套解构
const { user: { profile: { email } } } = response;
```

### 13. 模板字符串
**问题**: 模板字符串的高级用法

**答案**:
```javascript
// 基本用法
const name = 'World';
const greeting = `Hello, ${name}!`;

// 多行字符串
const html = `
  <div>
    <h1>${title}</h1>
    <p>${content}</p>
  </div>
`;

// 标签模板
function highlight(strings, ...values) {
  return strings.reduce((result, string, i) => {
    const value = values[i] ? `<mark>${values[i]}</mark>` : '';
    return result + string + value;
  }, '');
}

const highlighted = highlight`Hello, ${name}!`;
```

### 14. 箭头函数
**问题**: 箭头函数与普通函数的区别

**答案**:
```javascript
// 区别：
// 1. this 绑定不同
const obj = {
  name: 'obj',
  arrow: () => {
    console.log(this.name); // undefined（继承外层 this）
  },
  normal: function() {
    console.log(this.name); // 'obj'
  }
};

// 2. 没有 arguments 对象
const arrow = (...args) => {
  console.log(args); // 使用剩余参数
};

// 3. 不能作为构造函数
const Arrow = () => {};
// new Arrow(); // TypeError

// 4. 没有原型
console.log(Arrow.prototype); // undefined
```

### 15. Set 和 Map
**问题**: Set 和 Map 的使用场景和方法

**答案**:
```javascript
// Set - 唯一值集合
const set = new Set([1, 2, 2, 3]);
console.log(set); // Set {1, 2, 3}

set.add(4);
set.delete(1);
console.log(set.has(2)); // true
console.log(set.size); // 3

// 数组去重
const unique = [...new Set(array)];

// Map - 键值对集合
const map = new Map();
map.set('key1', 'value1');
map.set(obj, 'object key');

console.log(map.get('key1')); // 'value1'
console.log(map.has('key1')); // true
map.delete('key1');
```

## 性能优化

### 16. 内存管理
**问题**: JavaScript 的内存管理和垃圾回收

**答案**:
```javascript
// 内存泄漏常见场景：

// 1. 全局变量
window.myData = largeData; // 手动清除：delete window.myData

// 2. 闭包引用
function createHandler() {
  const largeData = new Array(1000000);
  return function(event) {
    // 使用 largeData
  };
}

// 3. 事件监听器未移除
element.addEventListener('click', handler);
// 记得移除：element.removeEventListener('click', handler);

// 4. 定时器未清除
const timer = setInterval(callback, 1000);
// 记得清除：clearInterval(timer);
```