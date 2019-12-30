## 在JavaScript中借用方法

在JavaScript中，有时候需要在一个不同的对象上重用一个函数，而不是在定义它的对象或者原型中。通过使用call()，applay()和bind()，我们可以很方便地从不同的对象借用方法，而不需要继承它们 – 这是一个在专业JavaScript开发者的工具箱中很有用的工具。

这篇文章假设你已经充分了解了call()，apply() 和 bind() 以及它们的不同点。

在JavaScript中，你接触的几乎所有东西都是一个对象，除了string，number 和 boolean这样不可变的原始值。一个数组是一种对象类型，适合用于有序数据列表的遍历和修改，在它的原型中有很多有用的方法，例如slice，join，push 和 pop。

我们看到对象最常见的使用情况就是从一个数组中借用方法，因为它们都是列表类型的数据结构。最常被借用的方法是 Array.prototype.slice。

``` javascript
function myFunc() {
    // 错误, arguments是一个类数组对象, 不是一个真实的数组
    arguments.sort();
    // 借用 Array 原型中的方法 slice, 它接受一个类数组对象 (key:value)
    // 并返回一个真实的数组
    var args = Array.prototype.slice.call(arguments);
    // args 现在是一个真正的数组, 所以可以使用Array的sort()方法
    args.sort();
}
myFunc('bananas', 'cherries', 'apples');
```

借用方法是可行的，因为call和apply允许我们在一个不同的上下文中调用方法，是一个很好的方式来重用已经存在的函数，而无需让一个对象扩展自另一个对象。数组实际上在它的原型中定义了很多方法，而且一般是可重用的，下面再举两个例子是join和filter：

> // 接收一个字符串 "abc" 并输出 "a|b|c

``` javascript
Array.prototype.join.call('abc', '|');
```

> // 接收一个字符串并移除所有的非元音字母

``` javascript
Array.prototype.filter.call('abcdefghijk', function(val) {
    return ['a', 'e', 'i', 'o', 'u'].indexOf(val) !== -1;
}).join('');
```

正如你所看到的，不仅仅对象可以得益于从数组中借用方法，字符串也可以。然而，因为方法一般被定义在了原型上，所以每次我们借用方法都要写String.prototype 或者 Array.prototype，这是很多余并且很烦人的事。一种有效的方法的是使用Literals（字面量）。

Literal是一种语法语言结构， 遵循 JavaScript 的规则，MDN 上这样解释:

> 你可以使用literals来表示JavaScript中的值。这些都是固定的值，不是变量，你可以再脚本中按照字面写出来。

Literals允许我们以缩略的形式访问原型方法：

``` javascript
[].slice.call(arguments);
[].join.call('abc', '|');
''.toUpperCase.call(['lowercase', 'words', 'in', 'a', 'sentence']).split(',');
```

这样变得简单些了，但看起来还是有点丑，还是要使用 [] 和 ""来借用它们的方法。我们可以更进一步缩短，通过存储一个引用，把字面量和它的方法作为一个变量：

``` javascript
var slice = [].slice;
slice.call(arguments);

var join = [].join;
join.call('abc', '|');

var toUpperCase = ''.toUpperCase;
toUpperCase.call(['lowercase', 'words', 'in', 'a', 'sentence']).split(',');
```

通过引用被借用的方法，我们可以很方便地使用call()调用它，享受所有可重用性的好处。为了继续减少冗长，我们来看一下是否可以在每次调用的时候，不写call() 或者 apply() 就能借用一个方法：

``` javascript
var slice = Function.prototype.call.bind(Array.prototype.slice);
slice(arguments);

var join = Function.prototype.call.bind(Array.prototype.join);
join('abc', '|');

var toUpperCase = Function.prototype.call.bind(String.prototype.toUpperCase);
toUpperCase(['lowercase', 'words', 'in', 'a', 'sentence']).split(',');
```

如你所见，通过使用Function.prototype.call.bind，我们现在可以静态绑定“被借用”的来自不同本地原型的方法，但是var slice = Function.prototype.call.bind(Array.prototype.slice)到底是如何工作的呢？

`Function.prototype.call.bind` 乍一眼看起来有些复杂，但是理解它是如何工作的非常有用。

* Function.prototype.call是一个引用，用来调用一个函数并且把它的“this”值设置为使用内部提到的方法。

可能你还不明白call做了什么事情，我们模拟一个

``` javascript
    // 方便理解自己实现一个mycall
    Function.prototype.myCall = function() {
        // 处理参数
        var args = [].slice.call(arguments)
        // this 这里指的是bind传入的参数 join这个方法
        return this.call(...args)
    }

    var join = Function.prototype.myCall.bind(Array.prototype.join);

    console.log(join('abc', '|')); // a|b|c
    console.log(Array.prototype.join.call('abc', '|')) // a|b|c
```

* 记住“bind”返回一个新的函数，这个函数总是会牢记它的“this”值。因此，.bind(Array.prototype.slice)会返回一个新的函数，它的“this”被永久地设置成了 Array.prototype.slice 函数。

通过结合以上两个，我们现在有了一个新的函数，它将会调用“call”函数并且“this”限定为了“slice”函数。简单地调用slice（）便可以引用之前限定的方法。

继承很棒，但是当程序员想要重用一些对象或者模块中的常见功能时会经常求助于它。如果你正在单独使用继承来重用代码，你可能会做错事，在很多情况下，简单的借用一个方法会变得非常麻烦。

到目前为止，我们仅仅讨论了借用本地方法，但其实可以借用任何方法！使用下面的代码来计算一个运动员所得的比赛分数：

``` javascript
var scoreCalculator = {
    getSum: function(results) {
        var score = 0;
        for (var i = 0, len = results.length; i < len; i++) {
            score = score + results[i];
        }
        return score;
    },
    getScore: function() {
        return scoreCalculator.getSum(this.results) / this.handicap;
    }
};

var player1 = {
    results: [69, 50, 76],
    handicap: 8
};

var player2 = {
    results: [23, 4, 58],
    handicap: 5
};

var score = Function.prototype.call.bind(scoreCalculator.getScore);

// Score: 24.375
console.log('Score: ' + score(player1));

// Score: 17
console.log('Score: ' + score(player2));
```

尽管上面的例子是人为的，但是很容易看到用户定义的方法也能像本地方法那样被方便地借用。

Call, bind 和 apply 允许我们改变函数被调用的方式，在借用一个函数时经常被使用。大多数开发者都很熟悉借用本地方法，但是很少知道用户定义的方法也可以。

在过去的几年里，JavaScript中的函数式编程逐渐增多，我希望简短的使用Function.prototype.call.bind来借用方法将变得越来越普遍。

