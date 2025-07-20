---
layout: posts
title: day1-js-learning
date: 2025-07-16 17:50:58
tags: [面试准备, web前端, JS]

---

### 1. JS基本数据类型及它们的区别

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

### 2. 数据类型检测的方式有哪些？

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

### 3. 判断数组的方法有哪些？

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

### 4. JS中的this的作用？

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

### 5. let VS const VS var

- 块级定义域：由{}包裹，let和const具有块级作用域，而块级作用域解决了两个问题：
  - 内层变量可能覆盖外层变量
  - 用来计数的循环变量泄露为全局变量
- 变量提升：var存在变量提升，即变量可以在未声明的时候使用。
- 给全局添加属性：浏览器的全局对象是window，node的全局对象是global。var声明的变量为全局变量，并且会将该变量添加为全局对象的属性。
- 重复声明：var声明变量时，可以重复声明，并且后者会覆盖前者。
- 暂时性死区：在使用let和const命令声明变量之前，该变量都是不可用的。在未声明前使用会发生报错。
- 初始值设置：在变量声明时，var和let可以不用设置初始值。
- 指针指向：let和var创建的变量是可以更改指针指向（可以重新赋值），但const声明的变量是不允许改变指针的指向。

### 6. new操作符的实现原理

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

### 7. 数组有哪些原生方法？

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

- 数组的截取方法：slice()

```javascript
let arr = [1,2,3,4,5];
let subArr = arr.slice(1,3);
console.log(subArr);//[2,3],从1开始到3（不包含）
console.log(arr);//[1,2,3,4,5],不改变原数组
```

- 数组插入的方法：splice()

```javascript
let arr = [1,2,3,4];
// 删除从索引1开始的2个元素
arr.splice(1,2);
console.log(arr);// [1,4]

// 插入元素，从索引1开始，删除0个元素，插入5和6
arr.splice(1,0,5,6);
console.log(arr);//[1,5,6,4]
```

- 数组首部操作的方法：shift()/unshift()

```javascript
let arr = [1,2,3];
let removed = arr.shift();
// shift()删除数组的第一个元素，并返回被删除的元素
console.log(removed);// 1
// shift()会影响原数组
console.log(arr);// [2,3]

let newLength = arr.unshift(0);
// unshift()在数组开头添加一个或多个元素，并返回新的数组长度
console.log(newLength);// 3
// unshift()会影响原数组
console.log(arr);// [0,2,3]
```

- 数组重排序的方法：reverse()/sort()

```javascript
let arr = [1,3,2];
arr.reverse();
//将数组中的元素顺序反转，会改变原数组
console.log(arr);// [2,3,1]

arr.sort((a,b) => a-b);// 按升序排列
console.log(arr);// [1,2,3]
arr.sort((a,b) => b-a);// 按降序排列
console.log(arr);// [3,2,1]
```

- 数组链接的方法：concat()

```javascript
let arr1 = [1, 2];
let arr2 = [3, 4];
let newArr = arr1.concat(arr2);
console.log(newArr); // 输出 [1, 2, 3, 4]
console.log(arr1); // 输出 [1, 2]（原数组未改变）
```

- 数组的索引方法：indexOf()/lastindexOf()

```javascript
// 返回指定元素在数组中第一次出现的索引，如果没有找到则返回 -1。
let arr = [1, 2, 3, 2];
let index = arr.indexOf(2);
console.log(index); // 输出 1

// 返回指定元素在数组中最后一次出现的索引，如果没有找到则返回 -1。
let index = arr.lastIndexOf(2);
console.log(index); // 输出 3
```

- 数组的迭代方法：every()/some()/filter()/map()/forEach()

```javascript
// every()检查数组中所有元素是否都满足条件。如果全部满足，则返回true；否则返回false
let arr = [2,4,7];
let result = arr.every(item => item % 2 ===0);
console.log(result);// false

// some()检查数组中是否有至少一个元素满足条件。如果有满足的，则返回true；否则返回false
let result = arr.some(item => ite, % 2 === 0);
console.log(result);// true

// filter()返回一个新数组，包含所有满足条件的元素
let result = arr.filter(item => item % 2 ===0);
console.log(result); //[2,4]

//map()对数组中的每个元素进行操作，返回一个新的数组
let result = arr.map(item => item % 2);
console.log(result);// [0,0,1]

//forEach()对数组中的每个元素执行一次提供的函数，没有返回值
 arr.forEach(item => console.log(item));
// 1
// 2
// 3

// forEach()会针对每个元素执行提供的函数，对数据的操作会改变原数组，该方法没有返回值；
// map()不会改变原数组的值，返回一个新数组，新数组中的值为原数组调用函数处理之后的值。
```

- 数组的归并方法：reduce()/reduceRight()

```javascript
// reduce()从左到右遍历数组，将数组元素累积为一个值
let arr = [1,2,3,4];
let sum = arr.reduce((arr,curr)=>arr+curr,0);
// 第一个arr原始数组，第二个arr累积器，curr当前元素，0arr的初始值
console.log(sum); // 10

//reduceRight()从右到左遍历数组，将数组累积为一个值
let sum = arr.reduce((arr,curr)=>arr+curr,0);
console.log(sum); // 10
```

### 8. for in VS for of

for in循环主要是为了遍历对象而生（遍历的是对象的整个原型链），不适用于遍历数组。获取的是对象的键名label。

for of循环可以用来遍历数组，类数组对象，字符串/set/map以及generator对象。获取的是对象的键值value。



### 9. 对原型/原型链的理解

在JS中是使用构造函数来新建一个对象的，每一个构造函数的内部都有一个prototype属性，它的属性值是一个对象——这个对象包含了可以由该构造函数的所有实例共享的属性和方法。当使用构造函数新建一个对象后，在这个对象的内部将包含一个指针，该指针指向构造函数prototype属性对应的值。

当访问一个对象的属性时，如果这个对象内部不存在这个属性，那么它就会去它的原型对象里找这个属性，这个原型对象又会有自己的原型，于是就这样一直找下去，这就是原型链的概念。

JS对象是通过引用来传递的，创建的每个新对象实体中并没有一份属于自己的原型副本。当修改原型时，与之相关的对象也会继承这一改变。

> - 当你看到一个对象的时候，你可以通过prototype去向上追寻它的原型。比如let obj = new Person()，那么`obj.__proto__`可以通过原型链来追寻到obj的原型，即Person的prototype的属性。
> - 因为Person就是一个构造函数，而这个构造函数里面有一个对象叫prototype，实例的`__proto__`指向了它，它包含了这个构造函数所有实例共享的所有属性和对象，也即Person所包含的属性和对象，obj也能够拥有。比如说Person里面有一个name属性，但obj没有的话，那么如果调用obj.name，就会根据obj的原型链找到Person，来访问Person的name属性。
> - 注意实例对象没有.prototype属性，只有构造函数才有。所以不存在obj.prototype，应该用obj.constructor.prototype。与此同时，每个实例对象都有一个`.__proto__`属性，指向构造函数的.prototype。

```javascript
function Person(){}
Person.prototype.name='Alice';

let obj = new Person();
console.log(obj.name); // Alice
```



### 10. 原型修改/重写/指向

- **修改原型：**目的是为了在原有的原型对象上添加或修改属性或方法，而不是替换整个原型对象。不影响已有实例，所有通过new创建的实例都能立即访问到新增的方法。修改原型不会影响构造函数的constructor属性（默认为object）。
- **重构原型：**指完全替换构造函数的整个原型对象，通常是通过一个新对象字面量或新的对象来赋值给prototype，这样更容易做模块化设计以及一次性定义多个方法。**必须手动设置 `constructor`** ，否则构造函数指向会变成 `Object`。

```javascript
function Person(name){
	this.name = name
}
// 修改原型，为Person的原型添加了一个名为getName的方法（多了一个属性），该方法是一个空函数。此时，Person.prototype仍然是一个对象，继承自Object.prototype，并未发生仍和改变。
Person.prototype.getName = function(){}

var obj = new Person('Alice')
console.log(obj.__proto__ === Person.prototype) // true，二者都等于Object
console.log(obj.__proto__ === obj.constructor.prototype) //true,因为obj的构造函数就是Person

// 重写原型，重写了Person.prototype，将其指向一个新的对象，这个对象有一个getName方法，它是一个空函数。此时所有通过new Person()创建的对象实例的__proto__都指向这个新的原型对象。
Person.prototype = {
	getName:function(){} //getName是Person.prototype上的一个属性（而不是函数），指向了一个函数
}

var obj = new Person('Alice')

console.log(obj.__proto__ === Person.prototype) // true
console.log(obj.__proto__ === obj.constructor.prototype) //false，这里obj的构造函数不是Person了。
//通过new Person()创建的对象obj的__proto__指向的是Person.prototype，而Person.prototype是一个对象字面量，它的constructor属性默认指向Object，所以obj.constructor===Object。

//如果希望obj.constructor === Person，应该
Person.prototype={
	constructor:Person, //手动设置构造函数
	getName:funvtion(){}
};

obj.__proto__ //Person.prototype
Person.prototype.__proto__ //Object.prototype
obj.__proto__.__proto__ //Object.prototype
Object.prototype.__proto__;// null
```

由于Object是构造函数，原型链的终点就是Object，也就是`Object.prototype.__Proto__`，而`Object.prototype.__Proto__===null`，所以原型链的终点是null。



### 11. 对闭包的理解

**【闭包是指有权访问另一个函数作用域中变量的函数】**，创建闭包的最常见方式就是在一个函数内创建另一个函数，创建的函数可以访问到当前函数的局部变量。比如fun1(){fun2(){}}中，fun2就是闭包。

用途1： 使我们在函数外部能够访问到函数内部的变量。通过使用闭包，可以通过在外部调用闭包函数，从而在外部访问到函数内部的变量，可以用这种方法来创建私有变量。

> 可以理解为闭包创建了一个“特权”通道，允许外部通过这个通道来间接操作函数内部的变量，而不能直接访问，从而实现了数据的封装和保护。

用途2：延长变量的生命周期，使已经运行结束的函数上下文中的变量对象继续留在内存中，因为闭包函数保留了整个变量对象的引用，所以这个变量对象不会被回收。

> 正常情况下，一个函数执行完毕后，它的局部变量就会被垃圾回收机制销毁。但如果闭包仍然需要这些变量，它们就会被“挽留”在内存中。

```javascript
//用途1
function sayName(name){
    let Myname = name;
    return {
        sayMyname: function(Myname){
            console.log(`My name is ${Myname}.`)
        },
        SetName: function(){
            return Myname;
        }
    }
}

const aliceName = sayName('Alice');
console.log(aliceName.SetName()); // 'Alice'

//用途2
const JohnName = sayName.SetName('John') //sayName.SetName函数此时已经执行完毕，按照逻辑JognName变量应该被销毁了
JohnName(); // My name is John.
```



### 12. 全局作用域VS函数作用域

全局作用域中的变量可以在整个程序中被访问。函数内部可以访问全局变量，但如果在函数内部声明了同名变量，则优先使用局部变量。如果在函数中没有重新声明变量，而是直接修改全局变量的值，那么全局变量的值也会改变，否则不改变。var声明的全局变量会成为window对象的属性，而let和const声明的变量虽然也是全局变量，但不会绑定到window上。

**【作用域链】**：本质是一个指向变量对象的指针列表。在当前作用域中查找所需变量，如果找不到就沿着作用域链一次向上级作用域查找，直到访问到window对象。













