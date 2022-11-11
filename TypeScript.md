## TS类型



### 基础类型

常用：boolean number string array enum any void

少用：tuple null undefined never



### 对象类型

interface 和 type 区别：type 更加强大，interface 可以进行声明合并，type 不行。



### 数组类型

常见写法：

~~~typescript
interface Item {
  id: number;
  name: string;
}
const arr1 = Item[] = [{ id: 1, name: '俊劫'}]
const arr2: Array<Item> = [{ id: 1, name: '俊劫'}]
~~~



### 元组 tuple

与数组类似，类型注解不太一样

~~~typescript
// 数组 某个位置的值可以是注解中的任何一个
const LOL: (string | number)[] = ["zed", 25, "darts"];

// 元祖 每一项数据类型必须一致
const LOL: [string, string, number] = ["zed", "darts", 25];
~~~



### 联合| 交叉&

联合类型：某个变量可能是多个 `interface` 中的一个，用 `|` 分割

交叉类型：由多个类型组成，用 `&` 连接



### 枚举 enum

提高代码可维护性，统一维护某些枚举值。提高可读性，一看就知道我要找的是什么值。



### 泛型 T (type)

泛型：泛指的类型，不确定的类型。可以理解为一个占位符（T 只是使用习惯，用任何占位符都可以）

~~~typescript
// T 自定义名称
function myFun<T>(params: T[]) {
  return params;
}
myFun<string> (["123", "456"]);

// 定义多个泛型
function join<T, P>(first: T, second: P) {
  return `${first}${second}`;
}
join <number, string> (1, "2");
~~~



### 断言

手动指定一个值的类型。`value as type` 或 `<type>value`。

注意：在 `tsx` 语法中必须使用前者语法。



### in 

类似数组和字符串中的 `includes` 方法



### void 和 never

返回值类型，也是基础类型。

~~~typescript
// 没有返回值 void
function f(): void {
    console.log(123)
}
// 执行不完
function errorf(): never {
    throw new Error();
  	console.log("Hello World");
}
~~~



### 类型检测



#### 1.typeof

可用来获取一个变量或对象的类型

~~~typescript
interface A {
  name: string;
}

const a: A = { name: "ming" };
type aa = typeof a; // type aa = A
~~~



#### 2.instanceof

~~~typescript
class NumberObj {
  	count: number;
}
if (first instanceof NumberObj && second instanceof NumberObj) {
    ...
}
~~~



#### 3.keyof

类似`Object.keys()`，只不过 `keyof` 取 interface 的键。

~~~typescript
interface Point {
    x: number;
    y: number;
}

type keys = keyof Point; // type key = x | y; 
~~~



## Utility Types



可以理解为基于 ts 封装的工具类型。



### Partial

`Partial<T>`：将 T 中所有属性转换为可选属性，返回的类型可以是 T 的任何子集

~~~typescript
nterface UserModel {
  name: string;
  age?: number;
}

type JUserModel = Partial<UserModel>
// 等同于
type JUserModel = {
    name?: string | undefined;
    age?: number | undefined;
}
~~~



### Required

`Require<T>`：通过将 T 的所有属性设置为必选属性来构造一个新的类型。

~~~typescript
type JUserModel2 = Required<UserModel>
// =
type JUserModel2 = {
    name: string;
    age: number;
}
~~~



### Readonly

`Readonly<T>`：将 T 中所有类型设置为只读

~~~typescript
type JUserModel3 = Readonly<UserModel>

// =
type JUserModel3 = {
    readonly name: string;
    readonly age?: number | undefined;
}
~~~



### Record

`Record<K, T>`：构造一个类型，该类型具有一组属性为K，每个属性类型为T。用于将一个类型的属性映射为另一个类型。

~~~typescript
type TodoProperty = 'title' | 'desc';
type Todo = Record<TodoProperty, string>;
// = 
type Todo = {
    title: string;
    desc: string;
}
~~~



### Pick

`Pick<T, K>`：在一个声明好的对象中，挑选一部分出来组成一个新的声明对象。

~~~typescript
interface Todo {
    name: string;
    sex: boolean;
    desc: string;
}
type SubTodo = Pick<Todo, 'name' | 'age'>
// =
type TodoBase = {
    name: string;
    sex: boolean;
}
~~~



### Omit

`Omit<T, K>`：从 T 中去除 K 的其他属性，与 Pick 相对



### Exclude

`Exclude<T, U>`：从 T 中排除可分配给 U 的属性，剩余属性组成新的类型

~~~typescript
type T0 = Exclude<'a' | 'b' | 'c', 'a'>
// =
type T0 = 'b' | 'c'
~~~



### Extract

`Extract<T, U>`：从 T 中抽出分配给 U 的属性构成新的类型，与 Exclude 相反

~~~typescript
type T0 = Extract<'a' | 'b' | 'c', 'a'>; 
// = 
type T0 = 'a'
~~~



### NonNullable

`NonNullable<T>`：去除 T 中的 null 和 undefined 类型



### Parameters

`Parameters<T>`：返回函数的参数类型所组成的元组类型

~~~typescript
function foo(x: number): Array<number> {
  return [x];
}
type P = Parameters<typeof foo>; // -> [number]
~~~



### ReturnType

`ReturnType<T>`：function T 返回的类型

~~~typescript
type T0 = ReturnType<() => string>;  // string
type T1 = ReturnType<(s: string) => void>;  // void
~~~



### InstanceType

`InstanceType<T>`：返回构造函数类型 T 的实例类型

~~~typescript
class C {
  x = 0;
  y = 0;
}
type T0 = InstanceType<typeof C>;  // C
~~~

