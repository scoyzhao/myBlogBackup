## 引用类型
除了Object，还有一些引用类型变量，并不是Object构造的，但是他们原型链的终点都是Object，这些类型都是引用类型：  

```
Array
Date
RegExp
Function
```

<br>

### 包装类型
JavaScript还包括几个**包装类型**：  

```
Boolean
Number
String
```

<img src="https://raw.githubusercontent.com/scoyzhao/FigureBed/master/blog/56/0.png" width = "100%"  alt="图片名称" align=center />

引用类型和包装类型的主要区别就在于**对象的生存期**  
使用new操作服创建的引用类型实力，在执行流离开当前作用域之前，都会保留  
而基本类型只存在于一行代码执行的瞬间，完成以后就被销毁了，这就意味着，我们不能给在运行时的基本类型添加属性和方法  

```javascript
var name = 'ConardLi'
name.color = 'red';
console.log(name.color); // undefined
```

在上述代码中，其实执行了一个拆箱和装箱的过程  
<br>

### 装箱和拆箱
既然原始类型不能扩展属性和方法，那么是如何调用原始类型的方法呢？  

```javascript
const name = '123'
const name1 = name.substring(2)
```

在上述代码中，执行了以下几个操作：  
1.调用`new String(123)`创建包装类型实例  
2.在实例上调用substring方法  
3.销毁实例  

拆箱的原则，遵循**toPrimitive原则**，一般会调用**toString、valueOf**方法，也可以重写toPrimitive方法  

<img src="https://raw.githubusercontent.com/scoyzhao/FigureBed/master/blog/56/1.png" width = "100%"  alt="图片名称" align=center />

1.引用类型转换为**String**的先调用**toString**然后调用**valueOf**  
2.引用类型转换为**Number**的先调用**valueOf**然后调用**toString**  

如果某个过程已经转为基本类型就停了，如果都没有就抛出错误，**TypeError**  

<img src="https://raw.githubusercontent.com/scoyzhao/FigureBed/master/blog/56/2.png" width = "100%"/>

思考了一下，如果是true的装箱类型，true - 1 返回的是 0 ，这是因为 **类型转换。。。。**  
<br><br>

## 类型转换
<br>

### 隐式类型转换
<img src="https://raw.githubusercontent.com/scoyzhao/FigureBed/master/blog/56/3.png" width = "100%"/>
符合这个规则，值得注意的是undefine和null  

从以上表来看，会被当作**if**中false的有：**null， undefined， NaN, '', 0, false**  
<br>

### ==
1.NaN  
NaN和任何比较，都返回**false**  

2.Boolean  
Boolean和任何值比较，都会先把自己转为**Number**  

```javascript
null == false
undefined == false
```

以上两个都是false，因为false会先转为0，然后比较就错了。。  

3.String和Number  
String先转为Number  

4.null和undefined  
null和undefined结果是true，和其余的比较都是false  

5.引用类型  
引用类型和原始类型比，引用类型先拆箱  

```javascript
[] == ![]
[null] == false
[undefined] == false
```

1.true，先右边，false，然后false转为0  
2/3.根据**toPrimitive规则**，数组元素为这俩，那么当作空字符串处理  
<br>

### 如何让 a == 1 && a == 2 && a == 3
```javascript
const a = {
   value:[3,2,1],
   valueOf: function () {return this.value.pop(); },
}
```
<br><br>

## 类型判断
如何判断类型，可以参考[【JavaScript基础】赋值、浅拷贝和深拷贝](/detail/54/)  
<br>

### typeof
用途：判断**基本类型和函数**
```javascript
typeof 'ConardLi'  // string
typeof 123  // number
typeof true  // boolean
typeof Symbol()  // symbol
typeof undefined  // undefined
type function() {} // function
```

而且他对于null的判断，也会判断为Object  
<br>

### instanceof
判断类型具体是什么类型的对象，跟原型链有关  
<br>

### toString
未被overwrite的返回**\[object xxx\]**，其中，xxx为具体类型  
<br>
