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
    let _this = this instanceof self ? this : oThis //检测是否使用new创建
      return self.apply(_this, [...args1, ...args2])
  }

  if (this.prototype) {
    fBound.prototype = this.prototype
  }

  return fBound
}
```





