## 定义

> new 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例。 ——（来自于MDN）

举个栗子

``` javascript
function Car(color) {
    this.color = color;
}
Car.prototype.start = function() {
    console.log(this.color + " car start");
}

var car = new Car("black");
car.color; // 访问构造函数里的属性
// black

car.start(); // 访问原型里的方法
// black car start
```

可以看出 new 创建的实例有以下 2 个特性

* 1、访问到构造函数里的属性
* 2、访问到原型里的属性

## 注意点

ES6新增 symbol 类型，不可以使用 new Symbol()，因为 symbol 是基本数据类型，每个从Symbol()返回的 symbol 值都是唯一的。

``` javascript
Number("123"); // 123
String(123); // "123"
Boolean(123); // true
Symbol(123); // Symbol(123)

new Number("123"); // Number {123}
new String(123); // String {"123"}
new Boolean(true); // Boolean {true}
new Symbol(123); // Symbol is not a constructor
```

## 模拟实现

* 一个继承自 Foo.prototype 的新对象被创建。
* 使用指定的参数调用构造函数 Foo ，并将 this 绑定到新创建的对象。new Foo 等同于 new Foo()，也就是没有指定参数列表，Foo 不带任何参数调用的情况。
* 由构造函数返回的对象就是 new 表达式的结果。如果构造函数没有显式返回一个对象，则使用步骤1创建的对象

## 模拟实现第一步

new 是关键词，不可以直接覆盖。这里使用 create 来模拟实现 new 的效果。

new 返回一个新对象，通过 obj.__proto__ = Con.prototype 继承构造函数的原型，同时通过 Con.apply(obj, arguments)调用父构造函数实现继承，获取构造函数上的属性。

实现代码如下

``` javascript
// 第一版
function create() {
    // 创建一个空的对象
    var obj = new Object(),
        // 获得构造函数，arguments中去除第一个参数
        Con = [].shift.call(arguments);
    // 链接到原型，obj 可以访问到构造函数原型中的属性
    obj.__proto__ = Con.prototype;
    // 绑定 this 实现继承，obj 可以访问到构造函数中的属性
    Con.apply(obj, arguments);
    // 返回对象
    return obj;
};
```

测试一下

``` javascript
// 测试用例
function Car(color) {
    this.color = color;
}
Car.prototype.start = function() {
    console.log(this.color + " car start");
}

var car = create(Car, "black");
car.color;
// black

car.start();
// black car start
```

完美！

## 模拟实现第二步

上面的代码已经实现了 80%，现在继续优化。

构造函数返回值有如下三种情况

* 1、返回一个对象
* 2、没有 return，即返回 undefined
* 3、返回undefined 以外的基本类型

情况1：返回一个对象

``` javascript
function Car(color, name) {
    this.color = color;
    return {
        name: name
    }
}

var car = new Car("black", "BMW");
car.color;
// undefined

car.name;
// "BMW"
```

实例 car 中只能访问到返回对象中的属性。

情况2：没有 return，即返回 undefined

``` javascript
function Car(color, name) {
    this.color = color;
}

var car = new Car("black", "BMW");
car.color;
// black

car.name;
// undefined
```

实例 car 中只能访问到构造函数中的属性，和情况1完全相反。

情况3：返回undefined 以外的基本类型

``` javascript
function Car(color, name) {
    this.color = color;
    return "new car";
}

var car = new Car("black", "BMW");
car.color;
// black

car.name;
// undefined
```

实例 car 中只能访问到构造函数中的属性，和情况1完全相反，结果相当于没有返回值。

所以需要判断下返回的值是不是一个对象，如果是对象则返回这个对象，不然返回新创建的 obj对象。

所以实现代码如下：

``` javascript
// 第二版
function create() {
    // 创建一个空的对象
    var obj = new Object(),
        // 获得构造函数，arguments中去除第一个参数
        Con = [].shift.call(arguments);
    // 链接到原型，obj 可以访问到构造函数原型中的属性
    obj.__proto__ = Con.prototype;
    // 绑定 this 实现继承，obj 可以访问到构造函数中的属性
    var ret = Con.apply(obj, arguments);
    // 优先返回构造函数返回的对象
    return ret instanceof Object ? ret : obj;
};
```

