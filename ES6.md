# ES Module



## Es Module 认识



`Nodejs` 借鉴了 `Commonjs` 实现了模块化 ，从 `ES6` 开始， `JavaScript` 才真正意义上有自己的模块化规范。



### **Es Module 有什么优势**

- 借助`Es Module` 的静态导入导出的优势，实现了 `tree shaking`。
- `Es Module` 可通过 `import()` 懒加载方式实现代码分割。



### ES6 module 特性

**1.静态语法**

ES6 module 的引入和导出是静态的，`import` 会自动提升到代码的顶层 ，**`import` , `export` 不能放在块级作用域或条件语句中**。

**2.执行特性**

ES6 module 和 Common.js 一样，对于相同的 js 文件，会保存静态属性。

但是不同的是 ，`CommonJS` 模块同步加载并执行模块文件，ES6 模块**提前加载并执行模块文件**，ES6 模块在预处理阶段分析模块依赖，在执行阶段执行模块，两个阶段都采用深度优先遍历，执行顺序是子 -> 父。

**3.导出绑定**

**不能修改import导入的属性**，修改会产生错误。



## export 导出和 import 导入



所有通过 export 导出的属性，在 import 中可以通过结构的方式，解构出来。



### **export 正常导出，import 导入**

~~~javascript
// 导出
const name = 'ming' 
const age = '12'
export { name, author }
export const say = function (){
    console.log('hello , world')
}
// 导入
import { name , age , say } from './a.js'
~~~

- export { }， 与变量名绑定，命名导出；
- import { } from 'module'， 导入 `module` 的命名导出 ；
- 这种情况下 import { } 内部的变量名称，要与 export { } 完全匹配。



### **默认导出 export default**

~~~javascript
// 导出
const name = 'ming'
const age = '12'
const say = function (){
    console.log('hello , world')
}
export default { name, author, say } 
// 导入
import mes from './a.js'
console.log(mes) // { name: 'ming',age: '12', say:Function }
~~~

- `export default anything` 导入 module 的默认导出。`anything` 可以是函数，属性方法，或者对象。
- 对于引入默认导出的模块，`import anyName from 'module'`， anyName 可以是自定义名称。



### **混合导入｜导出**

~~~javascript
// 导入
export const name = 'ming'
export const author = 'aa'
export default  function say (){
    console.log('hello , world')
}
// 导出方式1
import theSay , { name, author as  bookAuthor } from './a.js'
console.log(
    theSay,     // ƒ say() {console.log('hello , world') }
    name,       // "ming"
    bookAuthor  // "aa"
)
// 导出方式2
import theSay, * as mes from './a'
console.log(
    theSay, // ƒ say() { console.log('hello , world') }
    mes // { name:'ming' , author: "aa" ...}
)
~~~



### **重命名导入**

~~~JavaScript
import { bookName as name, say, bookAuthor as author } from 'module'
console.log( name , bookAuthor , author )
~~~



### 重定向导出

把当前模块作为一个中转站，一方面引入 module 内的属性，然后把属性再给导出去。

~~~javascript
export * from 'module' // 第一种方式
export { name, author, ..., say } from 'module' // 第二种方式
export { bookName as name, bookAuthor as author, ..., say } from 'module' //第三种方式
~~~

- 第一种方式：重定向导出 module 中的所有导出属性， 但是**不包括 `module` 内的 `default` 属性**。
- 第二种方式：从 module 中导入 name ，author ，say 再以相同的属性名导出。
- 第三种方式：从 module 中导入 name ，对部分属性重命名后导出 。



### **无需导入模块，只运行模块**

~~~javascript
import 'module' 
~~~

执行 module 不导出值  多次调用 `module` 只运行一次



### **动态导入**

~~~javascript
const promise = import('module')
~~~

`import('module')`，动态导入返回一个 `Promise`。为了支持这种方式，需要在 webpack 中做相应的配置处理。



### import 使用场景

**1.动态加载**

首先 `import()` 动态加载一些内容，可以放在条件语句或者函数执行上下文中。

~~~javascript
if(isRequire){
    const result  = import('./b')
}
~~~

**2.懒加载**，如路由懒加载

**3.React 中的动态加载**

~~~JavaScript
const LazyComponent =  React.lazy(()=>import('./text'))
class index extends React.Component{   
    render(){
        return <React.Suspense fallback={ <div className="icon"><SyncOutlinespin/></div> } >
               <LazyComponent />
           </React.Suspense>
    }
~~~



### **tree shaking 实现**

Tree Shaking 在 Webpack 中的实现，是用来尽可能的删除没有被使用过的代码，一些被 import 了但其实没有被使用的代码。



## ES Module 总结



- ES6 Module 静态的，不能放在块级作用域内，代码发生在编译时。
- ES6 Module 的值是动态绑定的，可以通过导出方法修改，可以直接访问修改结果。
- ES6 Module 可以导出多个属性和方法，可以单个导入导出，混合导入导出。
- ES6 模块提前加载并执行模块文件，
- ES6 Module 导入模块在严格模式下。
- ES6 Module 的特性可以很容易实现 Tree Shaking 和 Code Splitting。



# class



## 基础



### 由来



生成对象实例的传统方法是通过**构造函数**，ES6 中 class 我们可以看作是构造函数的另一种写法。所以 class 就像一个语法糖，它让对象原型的写法更加清晰、更容易理解。

在类中定义方法不需要 function，直接放上去就行了

~~~js
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  toString() {
    return '(' + this.x + ', ' + this.y + ')';
  }
}
~~~



类的数据类型是函数，类本身指向构造函数，所以可以直接对 class 使用 new。

类中 prototype 对象的 constructor 属性指向类本身，这和 function 定义的构造函数的行为是一样的

~~~js
typeof Point // function
Point === Point.prototype.constructor // true
~~~



构造函数的 `prototype` 属性，在 ES6 的 class 上仍然存在。所有 class 的方法都是定义在 class 的 `prototype` 属性上。所以说，在调用类实例的方法，本质上是调用原型上的方法。

既然类的方法都是定义在 `prototype` 上，那我们便可以通过 `Object.assign` 将多个方法添加到类上面

~~~js
class Point {
  constructor() {
    // ...
  }

  toString() {
    // ...
  }

  toValue() {
    // ...
  }
}
// 等同于
Point.prototype = {
	constructor() {},
    toString() {},
    toValue() {},
}
// 在原型上添加方法
Object.assign(Point.prototype, {})
~~~



注意，**类的内部所有方法是定义在原型上，且不可枚举**，这一点和 ES5 的行为不一致。

ES5 上构造函数内部的方法也是定义在原型上，并且可枚举

~~~js
class Point {
  constructor(x, y) {
    // ...
  }

  toString() {
    // ...
  }
}

Object.getOwnPropertyNames(Point.prototype) // 定义在原型上
// ["constructor","toString"] 

Object.keys(Point.prototype) // 不可枚举
// [] 
~~~





### constructor



`constructor()` 是类的默认方法，通过 new 命令生成实例时会调用该方法。

类必须要有 `constructor()` 方法，如果没有显式定义，会默认添加空的 `constructor()`

`constructor()` 默认返回实例对象 this，但也可以指定返回其他对象

~~~js
class Foo {
  constructor() {
    return Object.create(null); // 指定其他对象
  }
}
new Foo() instanceof Foo  // false
~~~

类必须使用 `new` 来调用，否则会报错，普通函数不使用 `new` 也可以执行。



### 类的实例



和 ES5 一样，`constructor` 定义的属性属于自身，否则都定义在原型上

~~~js
class Point{
    // 显式定义
    constructor(x,y){
        this.x = x;
        this.x = y;
    }
}
point.hasOwnProperty('x') // true
~~~



和 ES5 一样，类的所有实例共享一个原型对象

这意味着，我们可以通过 `__proto__` 属性来为类添加方法，不过需谨慎使用，这样会改变类的原始定义，影响其他实例

~~~js
var p1 = new Point(2,3);
var p2 = new Point(3,2);
p1.__proto__ === p2.__proto__   //true
// 在原型对象添加方法
p1.__proto__.printName = function () { return 'Oops' };
~~~



### getter和setter



对某个属性设置寸值函数 `getter` 和取值函数 `setter`，拦截该属性的存取行为。

所设置的属性定义在类的原型上

~~~js
class MyClass {
  constructor() {}
  get prop() {
    return 'getter';
  }
  set prop(value) {
    console.log('setter: '+value);
  }
}
let inst = new MyClass();

inst.prop = 123;
// setter: 123
inst.prop
// 'getter'
Object.getOwnPropertyNames(MyClass.prototype)
// ['constructor', 'prop']
~~~



getter 和 setter 函数是设置在属性的描述对象对象上，这点和 ES5 一致

~~~js
var descriptor = Object.getOwnPropertyDescriptor(
  MyClass.prototype, "prop"
);
"get" in descriptor  // true
"set" in descriptor  // true
~~~



### 属性表达式



类的属性名称可以使用表达式

~~~js
let methodName = 'getArea';
class Square {
  constructor(length) {
    // ...
  }
  [methodName]() {
    // ...
  }
}
~~~



### class 表达式



和函数一样，类也可以使用表达式的形式定义。如果类的内部引用到类本身，需要给类命名，否则可以省略

~~~js
const MyClass = class Me {
  getClassName() {
    return Me.name;
  }
};
let inst = new MyClass();
~~~

利用 class 表达式，实现立即执行的 class

~~~js
const p1 = new class Point {}(1, 2)
~~~



### 注意点

1. 严格模式：类和模块的内部，默认是严格模式，不需要使用 `use strict` 指定运行模式
2. 类不存在提升
3. 类的 `name` 属性：本质上 ES6 的类是 ES5 的构造函数的一层封装，name 属性正是被 class 继承而来
4. Generator 方法：如果在某个方法之前添加 `*`，则表示该方法是一个 Generator 函数
5. this 的指向：类内部方法的 `this` 默认指向类的实例，如果在类外部的作用域被单独使用，`this` 会指向该方法运行时所在的环境（**因为 class 内部是严格模式，所以 this 指向 undefined**），会找不到 this 对应的方法导致报错



this 的指向问题

~~~js
class Logger {
    printName(name = "there") {
        console.log(this);
        this.print(`Hello ${name}`);
    }
    print(text) {
        console.log(text);
    }
}
const logger = new Logger();

logger.printName();
// Hello, there
const { printName } = logger;
printName();
// TypeError...
~~~

解决方法：

1. 在构造函数中绑定 this
2. 使用箭头函数，箭头函数内部的 `this` 总是指向定义时所在的对象

~~~js
 constructor() {
     this.printName = this.printName.bind(this);
 }
// 或者
printName = () => {}
~~~



### 静态方法



类中定义的所有方法都会被实例继承，如果在定义的方法前面加 `static` 关键字，则表示该方法不会被实例继承，而是通过类来直接调用

~~~js
class Foo {
    static print() {
        console.log('1')
    }
}
Foo.print(); // 1
const f1 = new Foo();
f1.print(); // TypeError
~~~



如果静态方法中包含 `this` 关键字，该 `this` 指向的是类本身

父类的方法可以被子类继承 `extends`，在子类中，父类的静态方法可以从 `super` 对象上调用



### 静态属性



静态属性是指 class 自身的属性，而不是定义在实例上（this）的属性。

定义静态属性新写法

~~~js
class Foo {
    static prop = 123;
}
Foo.prop; // 123
~~~



### 实例属性的新写法



实例属性除了定义在 `constructor` 里的 `this` 上，也可以定义在类的最顶层。好处是简洁、实例属性一眼可见

~~~js
class Foo {
    constructor(){
        this.count = 0;
    }
}
// 等同于
class Foo {
    count = 0;
}
~~~



### 私有方法和私有属性



特点：只能在类内部使用，外部不能访问

在类内部的属性或者方法前面添加 `#` 号表示类的私有属性，如果在类外部使用会报错

私有属性也可以设置 `getter` 和 `setter`

私有属性不限于 `this` 引用，只要是在类的内部，实例也可以引用私有属性

在私有属性、方法前面添加 `static` 关键字，表示这是一个静态的私有属性、方法，在外部引用仍然报错



### in 运算符



之前是通过 `try...catch` 去判断某个是否存在某个属性，可读性很差

现在可以通过 `... in ...` 来判断某个对应是否存在某个属性，存在返回 `true`，否则 `false`



### 静态块



静态属性有个问题，属性要么在类的外部进行初始化，要么在 `constructor` 里面。

ES2022 进入静态块解决这个问题

~~~js
class C {
    static x = 1;
	static y;

    static {
        try {
          const obj = getData(this.x);
          this.y = obj.y;
        }
        catch {
          this.y = ...;
        }
    }
}
~~~



### new Target



通过 `new.target` 属性，我们可以判断构造函数是否通过 `new` 命令或 `Reflect.constructor` 调用的

~~~js
function Person(name) {
  if (new.target !== undefined) {
    this.name = name;
  } else {
    throw new Error('必须使用 new 命令生成实例');
  }
}
// 另一种写法
function Person(name) {
  if (new.target === Person) {
    this.name = name;
  } else {
    throw new Error('必须使用 new 命令生成实例');
  }
}
var person = new Person('张三'); // 正确
var notAPerson = Person.call(person, '张三');  // 报错
~~~

class 中使用 `new.target`

~~~js
class Rectangle {
  constructor(length, width) {
    console.log(new.target === Rectangle);
    this.length = length;
    this.width = width;
  }
}
var obj = new Rectangle(3, 4); // 输出 true
~~~



## 继承



简介



class 通过 `extends` 关键字实现继承

子类必须在 `constructor()` 方法中调用 `super`，否则会报错，因为**子类的 `this` 必须先通过父类的构造函数完成塑造，才能得到父类同样实例的属性和方法**，然后进行其他加工，添加子类自己的实例属性和方法。如果不调用 `super` 就得不到自己的 `this` 对象

由于子类必须调用 `super`，所以新建子类实例时，父类的构造函数会先执行一次

如果子类没有显式定义 `constructor` ，该方法会默认添加，并且里面会自动调用 `super`

除了私有属性，父类的所有属性和方法，都会被子类继承，包括静态方法。

**私有属性只能在定义它的 class 里面使用**，但可以通过对私有属性加一层读写封装来读取到父类的私有属性

~~~js
class Foo {
  #p = 1;
  getP() {
    return this.#p;
  }
}

class Bar extends Foo {
  constructor() {
    super();
    console.log(this.getP()); // 1
  }
}
~~~



### Object.getPrototypeOf()

用于获取某个子类的父类

~~~js
class Point { /*...*/ }
class ColorPoint extends Point { /*...*/ }
Object.getPrototypeOf(ColorPoint) === Point
~~~



### super



1. 代表父类的构造函数，子类构造函数的执行必须先执行 super()。super() 只能用在子类的构造函数中，子类的 super() 相当于 `A.prototype.constructor.call(this)`

2. `super` 作为对象时，在普通方法中，指向父类的原型对象（可访问父类的方法...）；在静态方法中，指向父类（父类的静态方法...）
3. 在子类的普通方法中通过 `super` 调用父类方法时，方法内部 `this` 指向当前的子类实例；因此如果通过 `super` 对某个属性赋值，要注意该属性是针对子类的属性
4. 在子类的静态方法中用 `super` 调用父类的方法时，方法里的`this`指向当前的子类，而不是子类的实例
5. 使用 `super` 的时候，必须显式指定是作为函数、还是对象，否则会报错；且只能在子类构造函数中使用 super()，在其他地方使用会报错



### 类的 prototype 和 proto



子类的 `__proto__` 指向父类的 `prototype`

子类的 `prototype` 的 `__proto__`，表示方法的继承，指向父类的 `prototype`

~~~js
class A {
}

class B extends A {
}
B.__proto__ === A 
B.prototype.__proto__ === A.prototype
~~~



子类实例的 `__proto__` 的 `__proto__` 属性，指向父类实例的 `__proto__` 属性。即子类的原型的原型，是父类的原型

~~~js
var p1 = new Point(2, 3);
var p2 = new ColorPoint(2, 3, 'red');

p2.__proto__.__proto__ === p1.__proto__ // true
~~~



### 原生构造函数继承



原生构造函数有：

- Boolean()
- Number()
- String()
- Array()
- Date()
- Function()
- RegExp()
- Error()
- Object()



ES6 允许继承原生构造函数来定义子类，因为 ES6 可以通过 class 先新建父类的实例对象 this，然后用子类的构造函数修饰 this，这样父类的所有行为都能继承。这在 ES5 是不可行的

继承 `Object` 的子类，有一个**行为差异**，即如果 `Object` 方法不是通过 `new Object()` 这种形式调用，ES6 规定 `Object` 构造函数会忽略参数









