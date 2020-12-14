## 浅拷贝和赋值不是一回事
<br>

### 基本类型的赋值
在JavaScript中，基本类型包括`number`、`string`、`boolean`、`undefined`和`null`，他们的赋值相对来说比较简单，并且赋值以后两个变量**互不影响**

```javascript
const a = 10
const b = a
a = 20 // a:20, b:10
```
<br>

### 引用数据类型的赋值
引用数据类型，包括`Array`、`Object`，他们赋值后两个变量共享一个数据内存空间，改变一个，另一个也会变化
**浅拷贝和深拷贝**就是为了解决这个问题

<br><br>

## 浅拷贝
<br>

### 对象浅拷贝
方法：**Object.assign**
如果对象中有数据失引用类型，那么创建新对象的过程中，会将这个引用类型的地址也放到新对象中
<br>

### 数组浅拷贝
方法：**Array.prototype.concat和Array.prototype.slice**
和对象一样，如果数组中存在非简单类型，也会出现联系
<br>

### 浅拷贝小结
如果数组、对象中的都是简单类型，那么变量之间没有关系，如果有引用数据类型，那么仍然会存在联系

<br><br>

## 深拷贝
<br>

### 利用json和json字符串的转换
通过**JSON.stringify和JSON.parse**
这种方法可以解决一定的问题：
1.循环引用
2.无法拷贝特殊对象：Reg、Date、Set、Map等会丢失
3.无法拷贝函数

![](https://raw.githubusercontent.com/scoyzhao/FigureBed/master/blog/54/0.png)
<br>

### 手写遍历
需要考虑很多东西
1.属性是基本类型
2.属性是对象
3.属性是数组
4.循环引用等等

```javascript
function deepCopy(originObj, map = new WeakMap()) {
    // 判断是否为基本数据类型
    if(typeof originObj === 'object') {
        // 判断是都否为数组
        const cloneObj = Array.isArray(originObj) ? [] : {};
        // 判断是否为循环引用
        if(map.get(originObj)) {
            return map.get(originObj);
        }
        map.set(originObj, cloneObj);
        for(const prop in originObj) {
            cloneObj[prop] = deepCopy(originObj[prop], map);
        }
        return cloneObj;
    } else {
        return originObj;
    }
}
```

map存储对象是防止循环引用，如果存在，就直接赋值上去并且返回
用WeakMap的弱引用特性，可以让他不需要我们手动设置为null就可以被系统回收
比如`let obj = { name: '123' }; const target = new WeakMap(); target.set(obj, '123'); obj = null;`这段代码中，target和obj的存在就是弱引用，当下一次垃圾回收机制执行的时候，这块内存就会释放掉
<br>

### 参考lodash（v4.17）对上述实现的优化
<br>

#### 性能优化
上面用了for...in来做遍历，相比for和while，**while**效率最高。
```javascript
function forEach(array, iteratee) {
    let index = -1;
    const length = array.length;
    while (++index < length) {
        iteratee(array[index], index);
    }
    return array;
}

function clone(target, map = new WeakMap()) {
    if (typeof target === 'object') {
        const isArray = Array.isArray(target);
        let cloneTarget = isArray ? [] : {};

        if (map.get(target)) {
            return map.get(target);
        }
        map.set(target, cloneTarget);

        const keys = isArray ? undefined : Object.keys(target);
        forEach(keys || target, (value, key) => {
            if (keys) {
                key = value;
            }
            cloneTarget[key] = clone2(target[key], map);
        });

        return cloneTarget;
    } else {
        return target;
    }
}
```
通过这样的改写，性能会加强一些
<br>

#### lodash源码实现
1.简单类型就返回
2.获取类型的tag，通过toString的方法，对于null和undefined做了处理
```javascript
...
if (value == null) {
  return value === undefined? '[object Undefined]': '[object Null]'
}

return toString.call(value)
...
```
**判断一个object是不是null，如果用非严格等号为false是不对的。。。可以用他和null去非严格等号判断，若为true，那么他不是undefined就是null**

3...巴拉巴拉巴拉

以上是一些我看源码的理解，有点云里雾里，结合ConarLi的blog，综合记录如下：
<br><br>

### 其它数据类型
<br>

#### 合理判断引用类型

除了Object和Array，还需要考虑function和null
```javascript
function isObject(target) {
    const type = typeof target;
    return target !== null && (type === 'object' || type === 'function');
}
```
typeof的返回值有：
基本类型 =》 基本类型
null，obj =》 obj
func =》 func
undefined =》undefined

其中，undefined是一种数据类型，null的数据类型是Object
ps：从lodash源码看，还有更多的类型，例如Map等。。。
<br>

#### 获取数据类型
每个引用类型都有toString方法，返回 **[object type]**
<img src="https://raw.githubusercontent.com/scoyzhao/FigureBed/master/blog/54/1.jpg" width = "100%"  alt="图片名称" align=center />

```javascript
function getType(target) {
    return Object.prototype.toString.call(target);
}
```

而且通过分类，可以分为可以遍历的类型和不可以便利的类型
```javascript
const mapTag = '[object Map]';
const setTag = '[object Set]';
const arrayTag = '[object Array]';
const objectTag = '[object Object]';

const boolTag = '[object Boolean]';
const dateTag = '[object Date]';
const errorTag = '[object Error]';
const numberTag = '[object Number]';
const regexpTag = '[object RegExp]';
const stringTag = '[object String]';
const symbolTag = '[object Symbol]';
```

这里缺少的是JSON、Undefined，null, global，中间两种按普通的一起处理，可以直接复制
其他两种额，json按对象处理，global ？？？
<br>

#### 可遍历的类型
需要拿到初值，方法是通过构造函数
```javascript
function getInit(target) {
    const Ctor = target.constructor;
    return new Ctor();
}

function clone(target, map = new WeakMap()) {

    // 克隆原始类型
    if (!isObject(target)) {
        return target;
    }

    // 初始化
    const type = getType(target);
    let cloneTarget;
    if (deepTag.includes(type)) {
        cloneTarget = getInit(target, type);
    }

    // 防止循环引用
    if (map.get(target)) {
        return map.get(target);
    }
    map.set(target, cloneTarget);

    // 克隆set
    if (type === setTag) {
        target.forEach(value => {
            cloneTarget.add(clone(value,map));
        });
        return cloneTarget;
    }

    // 克隆map
    if (type === mapTag) {
        target.forEach((value, key) => {
            cloneTarget.set(key, clone(value,map));
        });
        return cloneTarget;
    }

    // 克隆对象和数组
    const keys = type === arrayTag ? undefined : Object.keys(target);
    forEach(keys || target, (value, key) => {
        if (keys) {
            key = value;
        }
        cloneTarget[key] = clone(target[key], map);
    });

    return cloneTarget;
}
```
<br>

#### 不可遍历类型
```javascript
function cloneOtherType(targe, type) {
    const Ctor = targe.constructor;
    switch (type) {
        case boolTag:
        case numberTag:
        case stringTag:
        case errorTag:
        case dateTag:
            return new Ctor(targe);
        case regexpTag:
            return cloneReg(targe);
        case symbolTag:
            return cloneSymbol(targe);
        default:
            return null;
    }
}
```

对于**Symbol**：
```javascript
function cloneSymbol(targe) {
    // valueof 返回原始值，通过Object方法实现创建对象
    return Object(Symbol.prototype.valueOf.call(targe));
}
```

对于正则：
```javascript
function cloneReg(targe) {
    const reFlags = /\w*$/;
    // .source方法获取前面的
    const result = new targe.constructor(targe.source, reFlags.exec(targe));
    // 可读可写的整形属性，指定下一次匹配的位置，需要加一哈~
    result.lastIndex = targe.lastIndex;
    return result;
}
```
<br>

#### 克隆函数
需要区分箭头函数和普通函数，箭头函数是没有prototoype的
1.箭头函数直接用evel拼接就行了
2.普通函数通过正则匹配获取参数和内容，然后用 new Function 即可

```javascript
function cloneFunction(func) {
    const bodyReg = /(?<={)(.|\n)+(?=})/m;
    const paramReg = /(?<=\().+(?=\)\s+{)/;
    const funcString = func.toString();
    if (func.prototype) {
        console.log('普通函数');
        const param = paramReg.exec(funcString);
        const body = bodyReg.exec(funcString);
        if (body) {
            console.log('匹配到函数体：', body[0]);
            if (param) {
                const paramArr = param[0].split(',');
                console.log('匹配到参数：', paramArr);
                return new Function(...paramArr, body[0]);
            } else {
                return new Function(body[0]);
            }
        } else {
            return null;
        }
    } else {
        return eval(funcString);
    }
}
```
<br>

#### 比较完善的代码
<img src="https://raw.githubusercontent.com/scoyzhao/FigureBed/master/blog/54/2.jpg" width="100%" alt="图片名称" align=center />
<br><br>


## 小结
参考大神博客，学习了lodash关于深克隆的实现，以及有一些对于自己知识体系的补充
