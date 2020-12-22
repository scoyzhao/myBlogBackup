## “类”
<br>

JavaScript是少有的不通过**类**，直接创建对象的语言  
在JavaScript中，根本就不存在类，所有的一切都是对象  
<br><br>

## “构造函数”
<br>

```javascript
function Foo() {}
var a = new Foo()
```

在Javascript中，可以通过**new**操作符和函数子啊一起来创建“类”，在上一节可以知道，其实本质是链接了原型链，而不是和面向类的语言一样，**复制类**，在JavaScript中并没有这样的机制，实际上，这个操作只是让这两个对象相互关联  
**new Foo()**并没有直接创建关联，这个关联只是一个意外的副作用，这样一个对象就可以通过**委托**访问另一个对象的属性和函数了，似乎“实现了”类的创建  
更直接的方法是通过**Object.create(..)**  

<br>

Foo.prototype默认有一个共有且不可枚举的属性**constructor**，指向它关联函数Foo，此外，a也有constructor属性，也指向Foo，它是通过**原型链**，使用了Foo.prototype的属性。。。。。。  
也就是说，它是可以被修改的，所以用它来表示是否“继承”某个“类”，是很不靠谱的。。。。。。  
<br>

### new 的作用
<br>

```javascript
var o1 = new Object()
// 一般用__proto__代替[[Prototype]]
o1.[[Prototype]] = Base.prototype
Base.call(o1)
```

上述代码很容易让人认为Foo是一个构造函数，其实他和普通函数并没有区别，大小写也是为了区分普通函数  
它和普通函数的区别在于new，new操作符会**劫持**所有的普通函数，**构造**一个对象并且赋值给a，这是new的一个副作用（无论如何都会构造一个对象），而劫持的函数本身和普通函数并没有什么区别  
<br><br>

## 原型继承
<br>

```javascript
function Foo(name) {
  this.name = name
}

Foo.prototype.myName = function () {
  return this.name
}

function Bar(name, age) {
  // 这里模拟super
  Foo.call(this, name)
  this.age = age
}

// 这样创建的关联，没有constructor属性，需要的话自行添加
Bar.prototype = Object.create(Foo.prototype)
```

以上是一个原型继承的demo，关键点在于创建关联，通过以下方法也可以，但是都存在一定的问题：  

```javascript
Bar.prototype = Foo.prototype
Bar.prototype = new Foo()
```

第一个只是让Bar.prototype直接引用后者  
第二个可以创建对象的关联，但是如果Foo有一些副作用（例如注册到其他对象，给this添加属性等），就会影响到Bar“构造”的实例  
其实也可以通过Bar.prototype.\_\_proto\_\_来修改，但是在不兼容的浏览器里不起作用  
而使用Object.create也有一个缺点是创建联系的对象被删掉了，无法进行修改  
<br>

ES6提供了函数**Object.setPrototypeOf(...)**方法，可以用标准且可靠的方法来修改关联`Object.setPrototypeOf(Bar.prototype, Foo.prototype)`  
<br><br>

## 对象关联
<br>

**Object.create(null)**会创建一个拥有空链接的对象，这个对象无法进行委托。由于这个对象没有原型链，instanceof操作符没法进行判断，因此总是返回false，所以可以作为“字典”，因为他不会收到原型链的干扰，所以很适合存储诗句  
它的原理可以通过它的polyfill代码看出来：  

```javascript
Object.create = function(o) {
  function F() {}
  F.prototype = o
  return new F()
}
```

如果传入null，那么就没有原型链了，因为直接指向最末尾了  
<br><br>

## 面向委托的设计
<br>

用面向委托的方法重写上面的代码：  

```javascript
Foo = {
    init(name) {
        this.name = name
    },
    sayName() {
        console.log('name: ' + this.name);
    }
}

Bar = Object.create(Foo);

Bar.inits = function(name, age) {
    Foo.init.call(this, name);
    this.age = age;
}

Bar.sayAge = function() {
    console.log('age: ' + this.age);
}

var bar = Object.create(Bar);

bar.inits('bar', 14);

bar.sayName();
bar.sayAge();
```

可以看到，全部代码没有prototype，没有丑陋的的模拟类的super的语法，看起来还是很清爽的  
在ES6来到以后，有了**Class**相关的语法，有了super关键字，一切看起来都很好，但是它是语法糖，并不是真正实现了类，对于开发来说是更友好了，但是对于理解JavaScript的原型来说，又多了很多的陷阱  
<br>

### ES6 Class
<br>

```javascript
class Foo {
    constructor(name) {
        this.name = name;
    }
    sayName() {
        console.log('name: ' + this.name);
    }
}

class Bar extends Foo {
    constructor(name, age) {
        super(name);
        this.age = age;
    }
    sayAge() {
        console.log('age: ' + this.age);
    }
}

var bar3 = new Bar('bar3', 15);

bar3.sayName();
bar3.sayAge();
```

首先它在语法层面上实现了“类”，他的语法解决了典型的原型风格的，语法简洁，便于开发  
但是  
它隐藏了JavaScript对象中最重要的机制：**委托**，实际上让出现问题的时候更加难以定位，对于新手来说更难理解JavaScript的设计  


