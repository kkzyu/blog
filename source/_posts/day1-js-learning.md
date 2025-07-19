---
layout: posts
title: day1-js-learning
date: 2025-07-16 17:50:58
tags: [面试准备, web前端, JS]
---
1. JS基本数据类型及它们的区别
- `string`
- `number`：整数或浮点数。64位**双精度浮点数**，包含`Infinity`,`-Infinity`,`NaN`。
- `bigInt`：表示**大于**2^53-1的**整数**，不能与number直接运算（需显式转换）。
- `boolean`：表示true或false。
- `null`：**表示“无”或“空对象指针”**，需要通过let obj = null来赋值，但是typeof null会返回object。主要用于赋值给一些可能会返回对象的变量，作为初始化。
- `undefined`：**变量已声明但未赋值**，函数无发返回值时也会默认返回undefined。
- `symbol`：表示创建后独一无二且不可变的数据类型，主要是为了解决可能出现的全局变量冲突的问题，不可通过for...in来遍历。
- `object`：引用类型。表示复杂数据结构，包括{}/[]/function/Date等。

【原始数据类型vs引用数据类型】
- **引用数据类型**：存放在堆中，引用数据类型占据空间大，且大小不固定——undefined，null，boolean，number，string，symbol，bigInt。如果存在栈中，会影响程序运行的性能。**引用数据类型在栈中存储了指针**，该指针指向堆中该实体的起始地址——对象/数组和函数。
- **原始数据类型**：存放在栈中，栈中简单数据段，占据空间小，使用频繁的数据——对象/数组和函数。

【堆vs栈】
- 在数据结构中，栈中数据的存取方式为**先进后出**。堆是一个优先队列，**按照优先级来进行排序**，优先级可以按照数据大小来规定。
- 在操作系统中，栈区内存**由编译器自动分配释放**，存放函数的参数值，局部变量的值等。其操作方式类似于数据结构中的栈。堆区内存一般**由开发者分配释放**，若开发者不释放，程序结束时可能由垃圾回收机制来回收。

2. 数据类型检测的方式有哪些？
- `typeof`：这里可以检测每一种数据类型，除了object属于引用类型，其他都属于原始数据类型。其中数组[]，对象{}，和null都会被判断为object。比如：
```javascript
console.log(typeof undefined); //undefined
```
- `instanceof`：obj instanceof 引用数据类型，会返回true或者false。注意instanceof只能正确判断引用数据类型。其内部运行机制是判断是否能在其原型链中找到该类型的原型（是否存在一个构造函数的prototype属性）。比如：
```javascript
console.log(true instanceof Boolean); //false
console.log([] instanceof Array); //true
```
- `constructor`：用于判断数据类型，以及帮助对象实例来访问它的构造函数。比如：
```javascript
console.log((function() {}).constructor === Function); //true

// 需要注意，如果创建一个对象来改变它的原型，constructor就不能用来判断数据类型了
function Fn(){};
Fn.prototype = new Array();
var f = new Fn();
console.log(f.constructor===Fn); //false
console.log(f.constructor===Array); //true
```
- `Object.prototype.toString.call()`：使用Object对象的原型方法toString来判断数据类型。比如：
```javascript
var a = Object.prototype.toString;
console.log(a.call(2));
```

3. 判断数组的方法有哪些？
- 通过Object.prototype.toString.call()：
```javascript
Object.prototype.toString.call(obj).slice(8,-1) ==='Array';
```
- 通过原型链：
```javascript
obj.__proto__===Array.prototype;
```
- 通过ES6的Array.isArray()作判断：
```javascript
Array.isArray(obj);
```
- 通过instanceof作判断：
```javascript
obj instanceof Array;
```
- 通过Array.prototype.isPrototypeOf：
```javascript
Array.prototype.isPrototypeOf(obj);
```

4. JS中的this的作用？
this是执行上下文中的一个属性，简单来说：**它指向最后一次调用这个方法的对象。**
(1) 默认模式：全局环境下的this**指向全局对象window**。
(2) 隐式调用：方法调用-->指向调用它的对象，如果一个函数作为一个对象的方法来调用的话那么this会指向这个对象：
```javascript
const person = {
    name:"Alice",
    satHi(){
        console.log(`Hi,I'm${this.name}`); //this-->person
    }
};
person.sayHi();//函数作为一个对象的方法来调用
```
(3) 隐式调用：方法被分离后-->this丢失（此时就指向window），当一个函数不是一个对象的属性时，**直接作为函数来调用时，this指向全局对象。**
```javascript
const person = {
    name:"Alice",
    satHi(){
        console.log(`Hi,I'm${this.name}`); //this-->person
    }
};
const sayHi = person.sayHi;
sayHi(); // "Hi,I'm undefined"此时方法已被分离（被借给了别人），this就指向window。
```
(4) 显式调用：call/apply-->立即调用函数并强制绑定this
```javascript
function greet() {
    console.log(`Hello, ${this.name}`);
}
const user = { name: "Bob" };
greet.call(user); //"Hello,Bob"(this-->Bob)
```
(5) 显式调用：bind-->返回一个永久绑定this的新函数，这个函数的this除了使用new时会被改变，其它情况都不会改变。
```javascript
const boundGreet = greet.bind(user);
boundGreet(); //"Hello,Bob"(this永远指向user)
```
(6) 构造函数：new绑定-->指向新创建的对象，如果一个函数用new来调用时，函数执行前会创建一个对象，此时this指向这个新创建的对象。
```javascript
function Person(name) {
    this.name = name; //this-->新对象
}
const alice = new Person("Alice");
console.log(alice.name); //"Alice"，这里相当于隐式调用中的方法调用
```
(7) 箭头函数：继承**外层作用域**的this。箭头函数不会创建自己的this，它只会在自己的作用域的上一层继承this。所以箭头函数中的this的指向在它定义时已经确定了，之后永远不会改变。
```javascript
const group = {
    title:"Our Group",
    members:["Alice","Bob"],
    showMembers(){
        this.members.forEach(member => {
            console.log(`${this.title}:${member}`);
        });//继承了group的this
    }
};
group.showMembers();
```
```javascript
var id = 'GLOBAL';
var obj = {
    id:'OBJ',
    a: function() {
        console.log(this.id);
    },
    b: () => {
        console.log(this.id);
    }
};
obj.a(); //'OBJ'
obj.b(); //'GLOBAL'
new obj.a(); //undefined，new调用时，this是新对象
new obj.b(); // Uncaught TypeError: obj.b is not a constructor，箭头函数不能作为构造函数，无法用new调用
```

5. let VS const VS var
- 块级定义域：由{}包裹，let和const具有块级作用域，而块级作用域解决了两个问题：
    - 内层变量可能覆盖外层变量
    - 用来计数的循环变量泄露为全局变量
- 变量提升：var存在变量提升，即变量可以在未声明的时候使用。
- 给全局添加属性：浏览器的全局对象是window，node的全局对象是global。var声明的变量为全局变量，并且会将该变量添加为全局对象的属性。
- 重复声明：var声明变量时，可以重复声明，并且后者会覆盖前者。
- 暂时性死区：在使用let和const命令声明变量之前，该变量都是不可用的。在未声明前使用会发生报错。
- 初始值设置：在变量声明时，var和let可以不用设置初始值。
- 指针指向：let和var创建的变量是可以更改指针指向（可以重新赋值），但const声明的变量是不允许改变指针的指向。

6. new操作符的实现原理
假设要用new Person('Alice')来创建一个实例（类似于建造一栋属于Alice的房子）
(1) 建地基：首先创建了一个新的空对象
```javascript
let newObject = {};
// 开发商买了一块空地{}，准备在这盖房子
```
(2) 设计图纸：设置原型，将对象的原型设置为函数的prototype
```javascript
newObject.__proto__ = Person.prototype;
// 继承构造函数的原型，类似开发商拿到建筑师（Person）的设计图纸（prototype）在newObject上建造房子，这样实例就可以访问Person.prototype的方法
```
(3) 建造房屋：让函数的this指向这个对象，执行构造函数的代码
```javascript
let result = Person.apply(newObject,arguments);
// 执行构造函数，构造函数内的this就是新对象newObject。类似施工队（构造函数Person）拿着Alice的要求（arguments），在空地（newObject）上盖房子。
```
(4) 验收房屋：判断函数的返回值类型，如果是值类型，返回创建的对象。如果是引用类型，就返回这个引用类型的对象。
```javascript
function NormalBakery() {
  this.owner = "小明";
  // 没有 return 或者 return 5; (非对象)
}
let bakery1 = new NormalBakery();
console.log(bakery1); // 输出: { owner: '小明' } (返回了新创建的对象)


function SpecialBakery() {
  this.owner = "小红";
  let superBakery = { name: "超级烘焙中心", location: "市中心" };
  return superBakery; // 显式返回了一个对象
}
let bakery2 = new SpecialBakery();
console.log(bakery2); // 输出: { name: '超级烘焙中心', location: '市中心' }
// new 创建的那个 { owner: '小红' } 对象被丢弃了。
```
new操作符的实现原理可以总结为以下代码——
```javascript
function objectFactory(Constructor, ...args) {
    // 创建地基obj，并把constructor的原型指向obj，这样obj就可以使用到constructor的方法（获得设计图纸）
    const obj = Object.create(Constructor.prototype);
    // 根据设计图纸来建造房屋，把毛胚房obj根据constructor方法建造起的精装房屋命名为result，具体建造工具名为args
    const result = Constructor.apply(obj,args);

    // 判断result的类型是否是一个对象，如果是的话则返回精装房result，如果不是则返回毛胚房obj
    return result instanceof Object ? result : obj;
}

function Person(name){
    this.name = name;
}
// 指定constructor为Person，具体的建造工具是Alice
const Alice = objectFactory(Person, 'Alice');
console.log(Alice.name); //Alice
```

7. 数组有哪些原生方法？
- 数组和字符串的转换方法：toString()/toLocalString()/join()。
```javascript
let fruits=["apple","banana","mango"];
let date = [new Date()];

console.log("toString():",fruits.toString());
// toString():apple,banana,mango

console.log("toLocalString():",data.toLocalString('zh-CN'));
// toLocalString():2025/7/19下午4:00:00

console.log("join('-'):",fruit.join('-'));
// join('-'):apple-banana-mango,join方法可以指定转换为字符串时的分隔符
```
- 数组尾部操作的方法：pop()/push()
```javascript
let fruits=["apple","banana","mango"];

fruit.pop();
console.log("执行pop()后：",fruit);
// 执行pop()后：['apple','banana']

fruit.push("orange");
console.log("执行push()后：",fruit);
// 执行push()后：['apple','banana','orange']0
```