## Proxy
<br>

Proxy是ES6新增的一个对象，用于修改某些操作的默认行为，等同于在语言层面做出修改  
简而言之，就是对目标对象加一个“拦截层”，可以用它代理某些操作~  
<br>

```javascript
const obj = new Proxy({}, {
  get: (target, key, receiver) => {
    console.log(123)
  } 
})

obj.a = 1 // 123
```

可以看得出，通过proxy，我们代理了obj的get的行为  
<br>

### proxy的拦截操作
<br> 

**get(target, propKey, receiver)**  
**set(target, propKey, value, receiver)**  
这俩个方法的receiver可以参考Reflect.get的部分  
**has(target, propKey)**: 拦截propKey in proxy的操作，返回一布尔值  
...  
<br>

### 用法
<br>

1.vue3的双向绑定已经改为用Proxy实现  
2.编写web客户端接口：  

```javascript
const service = createService('xxx')
service.users().then(...)

// 通过proxy拦截它，就不需要为每种数据写适配代码了
const createService = (baseUrl) => {
  return new Proxy({}, {
    get: (target, propKey, receiver) => {
      return () => httpGet(baseUrl + '/' + propKey)
    }  
  })
} 
```

3.实现数据库的ORM层  
<br><br>

## Reflect
<br>

Reflect是为了操作对象而提供的新API，设计目的有：  
1.将Object上一些明显属于语言内部的方法放到Reflect上，也就是从Reflect上可以获得语言内部的方法，例如Object.defineProperty  
2.修改某些Object方法的返回，让其更合理，如Object.defineProperty在无法定义属性的时候会报错，而Reflect.defineProperty失败则会返回false  
3.让Object的操作都变成函数行为，如`name in Obj => Reflect.has(obj, name)`等  
4.Reflect的方法与Proxy的方法一一对应，在通过Proxy修改默认行为的时候，可以再Reflect上获取默认行为  
<br>

### 静态方法
<br>

**Reflect.get(target, name, receiver)**: 返回target对象的name属性。如果name属性部署了读取函数（getter），则读取函数的this绑定receiver  

```javascript
const myObject = {
  foo: 1,
  bar: 2,
  get baz() {
    return this.foo + this.bar
  }
}

const myReceiverObject = {
  foo: 4,
  bar: 4,
}

Reflect.get(myObject, 'baz', myReceiverObject) // 8
```

**Reflect.set(target, name, value, receiver)**: 设置target对象的name属性等于value。如果name属性设置了赋值函数，那么复值函数的this绑定到reveiver  

```javascript
const myObject = {
  foo: 1,
  set bar(value) {
    return this.foo = value
  }
}

const myReceiverObject = {
  foo: 4,
}

Reflect.set(myObject, 'bar', 1, myReceiverObject)

console.log(myObject)// {foo: 1}
console.log(myReceiverObject) // {foo: 1}
```

**Reflect.has(obj, name)**: 对应`name in obj`语法，返回布尔  
...  
<br><br>

## 利用Proxy、Reflect实现观察者模式
<br>

```javascript
/*
 * 函数自动观察数据对象的模式，一旦对象有变化，那么函数就执行
 * @Author: scoyzhao
 * @Date: 2020-12-23 15:37:05
 * @Last Modified by: scoyzhao
 * @Last Modified time: 2020-12-23 15:44:38
 */


const observers = new Set()
const addObserver = fn => observers.add(fn)

const observable = obj => {
  obj, {
    set: (target, key, value, receiver) => {
      observers.forEach(observer => observer())
      return Reflect.set(target, key, value, receiver)
    }
  }
}
```


