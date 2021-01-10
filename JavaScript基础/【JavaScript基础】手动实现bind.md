## bind核心作用：绑定this、初始化参数
<br>

根据MDN可以知道，bind用法为`func.bind(thisArg[, arg1[, arg2 ...]])`，从这个功能出发，实现一个简单版：  

```javascript
Function.prototype.myBind = function (context, ...args1) {
  if (typeof this !== 'function') {
    throw new Error(`${this.name} is not a function`)
  }

  let _this = this

  return function (...args2) {
    return _this.apply(context, [...args1, ...args2])
  }
}
```
<br><br>

## new操作符
<br>

上述方法，类似手动实现`apply,call`已经可以满足大部分需求了，但是在MDN中，还有一个例外的情况，那就是当使用new调用绑定函数的时候，`thisArg参数无效`  

```javascript
function animal(name) {
    this.name = name
}
let obj = {}

let cat = animal.bind(obj)
cat('lily')
console.log(obj.name)  //lily

let tom = new cat('tom')
console.log(obj.name)  //lily
console.log(tom.name)  //tom
```

如果使用上述绑定：  

```javascript
Function.prototype.myBind = function (context, ...args1) {
  if (typeof this !== 'function') {
    throw new Error(`${this.name} is not a function`)
  }

  let _this = this

  return function (...args2) {
    return _this.apply(context, [...args1, ...args2])
  }
}

function animal(name) {
    this.name = name
}
let obj = {}

let cat = animal.myBind(obj)
cat('lily')
console.log(obj.name)  //lily

let tom = new cat('tom')
console.log(obj.name)  //tom
console.log(tom.name)  //undefined
```

从上面可以看出，绑定函数作为构造函数的时候，**不能改变this的指向**，所以这个时候tom是一个空对象  
<br>

### 分类处理
<br>

```javascript
Function.prototype.bind = function (context, ...args1) {
  if (typeof this !== 'function') {
    return
  }

  let self = this
  let fBound = function(...args2) {
    let _this = this instanceof self ? this : context //检测是否使用new创建
      return self.apply(_this, [...args1, ...args2])
  }

  if (this.prototype) {
    fBound.prototype = this.prototype
  }

  return fBound
}
```
假设我们将调用bind的函数称为C，将fBound的prototype原型对象指向C的prototype原型对象（上例中就是self），这样的话如果将fBound作为构造函数（使用new操作符）实例化一个对象，那么这个对象也是C的实例，this instanceof self就会返回true。这时就将self指向新创建的对象的this上就可以达到原生bind的效果了（不再固定指定的this）。否则，才使用oThis，即绑定指定的this。  

但是这样做会有什么影响？将fBound的prototype原型对象直接指向self的prototype原型对象，那么当修改fBound的prototype对象时，self（上述C函数）的prototype对象也会被修改！！考虑到这个问题，我们需要另外一个function来帮我们做个中间人来避免这个问题，我们看看MDN是怎么实现bind的。  
<br>

### MDN的做法
<br>

```javascript
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    if (typeof this !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }

    var aArgs = Array.prototype.slice.call(arguments, 1),//这里的arguments是跟oThis一起传进来的实参
      fToBind = this,
      fNOP    = function() {},
      fBound  = function() {
        return fToBind.apply(this instanceof fNOP
          ? this
          : oThis,
          // 获取调用时(fBound)的传参.bind 返回的函数入参往往是这么传递的
          aArgs.concat(Array.prototype.slice.call(arguments)));
      };

    // 维护原型关系
    if (this.prototype) {
      // Function.prototype doesn't have a prototype property
      fNOP.prototype = this.prototype;
    }
    fBound.prototype = new fNOP();

    return fBound;
  };
}
```

发现了吗，和上面经过改造的方案相比，最主要的差异就在于它定义了一个空的function fNOP，通过fNOP来传递原型对象给fBound（通过实例化的方式）。这时，修改fBound的prototype对象，就不会影响到self的prototype对象啦～而且fNOP是空对象，所以几乎不占内存。  






