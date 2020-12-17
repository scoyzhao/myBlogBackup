## Constructor 构造函数
<br>

**构造函数**在ES6以前就是函数，只是首字母大写：  

```javascript
function Person() {}
```

<br><br>

## prototype 原型
<br>

**原型**指的是一个属性，也就是实例``继承``的那个属性  
在原型上定义的属性，通过``继承``，让实例获得了这个属性，这个过程是在**new**操作符中实现的  

**原型与构造函数的关系**，构造函数中又一个``prototype``的属性，通过它，构造函数可以访问到原型  

<img src="https://raw.githubusercontent.com/scoyzhao/FigureBed/master/blog/58/0.png"  alt="图片名称" align=center /><br>

<img src="https://raw.githubusercontent.com/scoyzhao/FigureBed/master/blog/58/1.png" width = "100%"  alt="图片名称" align=center /><br>
<br><br>

## instance 实例
<br>

有了构造函数，就可以通过new操作服创建实例，通过**instanceof**操作符，可以检测它原型链是否包含后面  
另外，在原型上加的属性，实例是可以访问到的  

<img src="https://raw.githubusercontent.com/scoyzhao/FigureBed/master/blog/58/2.png" width = "100%"  alt="图片名称" align=center /><br>

```javascript
function Person() {}
Person.prototype.type = { name: 'person' }
const p = new Person()
p instanceof Person // true
p instanceof Object // true
p.type // { name: 'person' }
```
<br><br>

## \_\_proto\_\_ 隐式继承
``__proto__``是实现原型链的关键，通过这个属性，可以访问到原型  

<img src="https://raw.githubusercontent.com/scoyzhao/FigureBed/master/blog/58/3.png" width = "100%"  alt="图片名称" align=center /><br>

<br>

### 隐式原型与显示原型
<br>

#### 作用
显示原型（prototype）：用来实现基于原型的继承与属性的共享  
隐式原型（\_\_proto\_\_）：构成原型链，实现基于原型的继承  
<br>

#### \_\_proto\_\_的指向
指向构造他的函数的显示原型，这里当然不止实例有，对象都有  

``Object.create``方法创建的对象，会把参数的第一个参数作为新建对象的原型，通过Object.create(null)创建的对象是‘干净的’，它的原型直接是null，所以他不会包含toString等Object原型上的方法  
<br>

#### *一些原型的例子

```javascript
// 1.
function Bar(){}
//这时我们想让Foo继承Bar
Foo.prototype = new Bar()
Foo.prototype.__proto__ === Bar.prototype //true

// 2.我们不想让Foo继承谁，但是我们要自己重新定义Foo.prototype
Foo.prototype = {
  a:10,
  b:-10
}
//这种方式就是用了对象字面量的方式来创建一个对象，根据前文所述
Foo.prototype.__proto__ === Object.prototype

// 3.既然是构造函数那么它就是Function（）的实例，因此也就指向Function.prototype
Object.__proto__ === Function.prototype

// 4.
Function instanceof Object // true
Object instanceof Function // true
Function instanceof Function //true
Object instanceof Object // true
Number instanceof Number //false
```
<br><br>

## constructor 构造函数
<br>

构造函数可以通过prototype属性访问原型，原型也可以通过构造函数属性访问构造函数：  

```javascript
function Person() {}
Person.prototype.constructor === Person // true
```

<img src="https://raw.githubusercontent.com/scoyzhao/FigureBed/master/blog/58/4.png" width = "100%"  alt="图片名称" align=center />

这里的constructor是原型的一个属性，和上面的构造函数不是一个东西  
<br><br>

## 原型链
<br>

当访问p的一个属性的时候，会先在它本身的属性上，如果没有找到，就会顺着它的\_\_proto\_\_找到构造他的构造函数到原型上找，如此往复，直到Object -> null为止，搜索不到那么这个属性就是不存在的  

<img src="https://raw.githubusercontent.com/scoyzhao/FigureBed/master/blog/58/5.png" width = "100%"  alt="图片名称" align=center /><br>

<img src="https://raw.githubusercontent.com/scoyzhao/FigureBed/master/blog/58/6.png" width = "100%"  alt="图片名称" align=center /><br>
