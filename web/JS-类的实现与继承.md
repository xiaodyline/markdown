# JS 中类的两种实现方式，以及继承

JS 中实现“类”主要有两种方式：

1. **ES5 构造函数 + 原型**
2. **ES6 class 语法**

本质上，ES6 的 `class` 只是语法糖，底层仍然基于 **原型和原型链**。

------

# 1. ES5：构造函数 + 原型

## 1.1 定义类

```js
function Person(name, age) {
  this.name = name
  this.age = age
}

Person.prototype.sayHi = function () {
  console.log(`我叫${this.name}，今年${this.age}岁`)
}

const p1 = new Person('张三', 18)

console.log(p1.name) // 张三
p1.sayHi() // 我叫张三，今年18岁
```

这里：

```js
function Person(name, age) {
  this.name = name
  this.age = age
}
```

相当于定义了一个“类”。

```js
Person.prototype.sayHi = function () {}
```

相当于给这个类添加公共方法。

------

## 1.2 为什么方法要写在 prototype 上？

不推荐这样写：

```js
function Person(name, age) {
  this.name = name
  this.age = age

  this.sayHi = function () {
    console.log(this.name)
  }
}
```

因为每创建一个对象，都会重新创建一份 `sayHi` 方法。

```js
const p1 = new Person('张三', 18)
const p2 = new Person('李四', 20)

console.log(p1.sayHi === p2.sayHi) // false
```

更推荐写到原型上：

```js
function Person(name, age) {
  this.name = name
  this.age = age
}

Person.prototype.sayHi = function () {
  console.log(this.name)
}

const p1 = new Person('张三', 18)
const p2 = new Person('李四', 20)

console.log(p1.sayHi === p2.sayHi) // true
```

这样所有实例共用同一个方法，节省内存。

------

# 2. ES5 中的继承

假设有一个父类 `Person`：

```js
function Person(name, age) {
  this.name = name
  this.age = age
}

Person.prototype.sayHi = function () {
  console.log(`我叫${this.name}`)
}
```

现在让 `Student` 继承 `Person`。

------

## 2.1 组合继承写法

```js
function Student(name, age, score) {
  Person.call(this, name, age)

  this.score = score
}

Student.prototype = Object.create(Person.prototype)

Student.prototype.constructor = Student

Student.prototype.study = function () {
  console.log(`${this.name}正在学习，分数是${this.score}`)
}

const s1 = new Student('小明', 18, 100)

console.log(s1.name) // 小明
console.log(s1.age) // 18
console.log(s1.score) // 100

s1.sayHi() // 我叫小明
s1.study() // 小明正在学习，分数是100
```

------

## 2.2 关键点解释

### 1. 继承父类属性

```js
Person.call(this, name, age)
```

作用是借用父类构造函数，把父类中的属性添加到当前子类实例上。

相当于：

```js
this.name = name
this.age = age
```

------

### 2. 继承父类方法

```js
Student.prototype = Object.create(Person.prototype)
```

作用是让：

```js
Student.prototype
```

的原型指向：

```js
Person.prototype
```

这样 `Student` 的实例就可以访问 `Person.prototype` 上的方法。

------

### 3. 修正 constructor

```js
Student.prototype.constructor = Student
```

因为前面重写了 `Student.prototype`，所以需要把 `constructor` 指回 `Student`。

------

# 3. ES6：class 语法

## 3.1 定义类

```js
class Person {
  constructor(name, age) {
    this.name = name
    this.age = age
  }

  sayHi() {
    console.log(`我叫${this.name}，今年${this.age}岁`)
  }
}

const p1 = new Person('张三', 18)

console.log(p1.name) // 张三
p1.sayHi() // 我叫张三，今年18岁
```

------

## 3.2 class 中的 constructor

```js
constructor(name, age) {
  this.name = name
  this.age = age
}
```

`constructor` 是构造方法。

当执行：

```js
new Person('张三', 18)
```

时，会自动调用 `constructor`。

------

## 3.3 class 中的方法

```js
sayHi() {
  console.log(this.name)
}
```

这些方法本质上是挂在原型上的。

可以验证：

```js
class Person {
  sayHi() {
    console.log('hello')
  }
}

const p1 = new Person()
const p2 = new Person()

console.log(p1.sayHi === p2.sayHi) // true
```

说明 `sayHi` 是所有实例共享的。

------

# 4. ES6 中的继承

## 4.1 extends 继承

```js
class Person {
  constructor(name, age) {
    this.name = name
    this.age = age
  }

  sayHi() {
    console.log(`我叫${this.name}`)
  }
}

class Student extends Person {
  constructor(name, age, score) {
    super(name, age)

    this.score = score
  }

  study() {
    console.log(`${this.name}正在学习，分数是${this.score}`)
  }
}

const s1 = new Student('小明', 18, 100)

console.log(s1.name) // 小明
console.log(s1.age) // 18
console.log(s1.score) // 100

s1.sayHi() // 我叫小明
s1.study() // 小明正在学习，分数是100
```

------

## 4.2 super 的作用

```js
super(name, age)
```

相当于调用父类的构造函数：

```js
Person.call(this, name, age)
```

也就是说，它负责初始化父类中的属性：

```js
this.name = name
this.age = age
```

------

## 4.3 子类 constructor 中必须先调用 super

下面是错误写法：

```js
class Student extends Person {
  constructor(name, age, score) {
    this.score = score
    super(name, age)
  }
}
```

会报错。

正确写法：

```js
class Student extends Person {
  constructor(name, age, score) {
    super(name, age)
    this.score = score
  }
}
```

原因是：子类自己的 `this` 必须先通过父类构造函数创建出来。

------

# 5. ES5 和 ES6 对比

| 对比点   | ES5 构造函数                      | ES6 class         |
| -------- | --------------------------------- | ----------------- |
| 定义类   | `function Person() {}`            | `class Person {}` |
| 构造方法 | 普通函数体内部                    | `constructor()`   |
| 实例属性 | `this.xxx = xxx`                  | `this.xxx = xxx`  |
| 原型方法 | `Person.prototype.xxx`            | 直接写在 class 内 |
| 继承属性 | `Parent.call(this)`               | `super()`         |
| 继承方法 | `Object.create(Parent.prototype)` | `extends`         |
| 本质     | 原型机制                          | 原型机制语法糖    |

------

# 6. 面试回答版本

JS 中实现类主要有两种方式。

第一种是 **ES5 的构造函数加原型**。构造函数中定义实例属性，公共方法挂载到 `prototype` 上，这样多个实例可以共享方法，避免每次创建对象时都重新创建函数。

第二种是 **ES6 的 class 语法**。它通过 `class`、`constructor`、方法定义等形式让类的写法更接近传统面向对象语言，但本质上仍然是基于原型和原型链实现的。

继承方面，ES5 常用组合继承：在子类构造函数中通过 `Parent.call(this)` 继承父类属性，再通过 `Object.create(Parent.prototype)` 继承父类原型方法，并修正 `constructor` 指向。

ES6 中继承更简单，使用 `extends` 继承父类，通过 `super()` 调用父类构造函数。子类如果写了 `constructor`，必须先调用 `super()`，才能使用 `this`。