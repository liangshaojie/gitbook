我们知道this绑定规则一共有5种情况：

- 1、默认绑定（严格/非严格模式）
- 2、隐式绑定
- 3、显式绑定
- 4、new绑定
- 5、箭头函数绑定

其实大部分情况下可以用一句话来概括，`this总是指向调用该函数的对象`。

但是对于箭头函数并不是这样，是根据`外层（函数或者全局）作用域（词法作用域）`来决定`this`。

对于箭头函数的`this`总结如下：

- 箭头函数不绑定this，箭头函数中的this相当于普通变量。
- 箭头函数的this寻值行为与普通变量相同，在作用域中逐级寻找。
- 箭头函数的this无法通过bind，call，apply来直接修改（可以间接修改）。
- 改变作用域中this的指向可以改变箭头函数的this。

eg. function closure(){()=>{//code }}，在此例中，我们通过改变封包环境closure.bind(another)()，来改变箭头函数this的指向。

## 题目1


```javascript
/**
 * 非严格模式
 */

var name = 'window'

var person1 = {
  name: 'person1',
  show1: function () {
    console.log(this.name)
  },
  show2: () => console.log(this.name),
  show3: function () {
    return function () {
      console.log(this.name)
    }
  },
  show4: function () {
    return () => console.log(this.name)
  }
}
var person2 = { name: 'person2' }

person1.show1()
person1.show1.call(person2)

person1.show2()
person1.show2.call(person2)

person1.show3()()
person1.show3().call(person2)
person1.show3.call(person2)()

person1.show4()()
person1.show4().call(person2)
person1.show4.call(person2)()
```

正确答案如下：


```javascript
person1.show1()                 // person1，隐式绑定，this指向调用者 person1 
person1.show1.call(person2)     // person2，显式绑定，this指向 person2

person1.show2()                 // window，箭头函数绑定，this指向外层作用域，即全局作用域
person1.show2.call(person2)     // window，箭头函数绑定，this指向外层作用域，即全局作用域

person1.show3()()               // window，默认绑定，这是一个高阶函数，调用者是window 类似于`var func = person1.show3()` 执行`func()`
person1.show3().call(person2)   // person2，显式绑定，this指向 person2
person1.show3.call(person2)()   // window，默认绑定，调用者是window

person1.show4()()               // person1,箭头函数绑定，this指向外层作用域，即person1函数作用域
person1.show4().call(person2)   // person1,箭头函数的this无法通过bind，call，apply来直接修改
person1.show4.call(person2)()   // person2,间接修改(改变作用域中this的指向可以改变箭头函数的this)
```

最后一个`person1.show4.call(person2)()`有点复杂，我们来一层一层的剥开。

- 1、首先是`var func1 = person1.show4.call(person2)`，这是显式绑定，调用者是`person2`，`show4`函数指向的是`person2`。
- 2、然后是`func1()`，箭头函数绑定，`this`指向外层作用域，即`person2`函数作用域

首先要说明的是，箭头函数绑定中，this指向外层作用域，并不一定是第一层，也不一定是第二层。

因为没有自身的this，所以只能根据作用域链往上层查找，直到找到一个绑定了this的函数作用域，并指向调用该普通函数的对象。

## 题目2

这次通过构造函数来创建一个对象，并执行相同的4个`show`方法。


```javascript
var name = 'window'

function Person (name) {
  this.name = name;
  this.show1 = function () {
    console.log(this.name)
  }
  this.show2 = () => console.log(this.name)
  this.show3 = function () {
    return function () {
      console.log(this.name)
    }
  }
  this.show4 = function () {
    return () => console.log(this.name)
  }
}

var personA = new Person('personA')
var personB = new Person('personB')

personA.show1()
personA.show1.call(personB)

personA.show2()
personA.show2.call(personB)

personA.show3()()
personA.show3().call(personB)
personA.show3.call(personB)()

personA.show4()()
personA.show4().call(personB)
personA.show4.call(personB)()
```
正确答案如下：


```javascript
personA.show1()                     // personA，隐式绑定，调用者是 personA
personA.show1.call(personB)         // personB，显式绑定，调用者是 personB

personA.show2()                     // personA，首先personA是new绑定，产生了新的构造函数作用域，
				                    // 然后是箭头函数绑定，this指向外层作用域，即personA函数作用域
personA.show2.call(personB)         // personA，同上(箭头函数的this无法通过bind，call，apply来直接修改)

personA.show3()()                   // window，默认绑定，调用者是window
personA.show3().call(personB)       // personB，显式绑定，调用者是personB
personA.show3.call(personB)()       // window，默认绑定，调用者是window

personA.show4()()                   // personA，箭头函数绑定，this指向外层作用域，即personA函数作用域
personA.show4().call(personB)       // personA，箭头函数绑定，call并没有改变外层作用域，
							        // this指向外层作用域，即personA函数作用域
personA.show4.call(personB)()       // personB，解析同题目1，最后是箭头函数绑定，
							        // this指向外层作用域，即改变后的person2函数作用域
```

题目一和题目二的区别在于题目二使用了new操作符。

使用 new 操作符调用构造函数(构建了一个函数作用域，箭头函数上层就是这个作用域，而不是第一题的全局作用域)，实际上会经历一下4个步骤：

- 创建一个新对象；
- 将构造函数的作用域赋给新对象（因此this就指向了这个新对象）；
- 执行构造函数中的代码（为这个新对象添加属性）；
- 返回新对象。