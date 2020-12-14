## JavaScript数据类型
<br>

### 原始类型
7种（Null，Undefined，Boolean，Number，String， Symbol【es10新增了BigInt】）  
<br>

### 对象类型
Object，Array、Function等都属于对象类型  
<br>

### 区分原因
原始类型为不可变得，在内存里是按栈存储的：  
<img src="https://raw.githubusercontent.com/scoyzhao/FigureBed/master/blog/55/0.png" width = "100%"  alt="图片名称" align=center />

对象类型的值实际是存储在堆内存里的，在栈内存里只是存了一个内存地址：  
<img src="https://raw.githubusercontent.com/scoyzhao/FigureBed/master/blog/55/1.png" width = "100%"  alt="图片名称" align=center />
<br>

### 复制
对于原始数据，直接复制就行  
对象类型，他们的复制其实就氛围**浅拷贝和浅拷贝**，这部分可以参考之前的博客  

<br>

### 比较
对于原始类型，比较就只比较他们的值  
而对象类型，比较其实是他们的内存地址  
<br>

### 值传递和引用传递
有这么一种**错觉**，那就是在函数中使用对象作为参数的时候，传递的是**引用**，或者理解成指针  
其实，在JS中呢，所有函数的参数都是值传递  
对于对象类型来说，他们的‘值’，其实就是他们的内存地址  
<br>

## Null和Undefined
他们都代表**无/空**，可以这样区分：  
<br>

### null
表示赋值过的对象，值为null，为空  

所以对象的某个值为null是正常的，null转数字为0  
<br>

### undefined
表示缺少值，表示这里应该有一个值，但是还没有定义  
如果对象的某个值为undefined，这是不对的，应该把这个属性删掉，他转化为数字为NaN  

现有的编程，一般有一些编程规范建议**使用 void 0 代替undefined**，这是为什么呢？  
Undefined类型表示未定义，他的类型只有一个值，就是undefined，任何变量在赋值前都可以用**全局**undefined来表达这个值，或者void运算来把任意一个表达式编程undefined值  
<br>

### 二者区别

但是，由于JavaScript中undefined是一个变量，而不是一个**关键词**，特别是在自定义作用域中，还可以给这个变量赋值，为了避免无意识的篡改，所以可以用void 0来获取undefined的值  

Null类型也表示空，但是null是JavaScript中的一个关键字，所以任何位置都是可以使用他的  
<br><br>

## Number
1.会出现的问题就是精度的丢失。。。  
2.JavaScript中有+—0的区分，出发的时候需要留意  
<br>

### 精度丢失
计算机中数据都是按二进制存储的，会在计算的时候用二进制计算，计算完之后转换为十进制，而小数的二进制都是无限循环的  

那么应该如何精确计算呢？ `Math.abs(0.1 + 0.2 - 0.3) <= Number.EPSILON` , 也就是值应该小于最小精度  
<br><br>

## String
<br>

### 最大长度
String表示文本数据，最大长度是**2^53 - 1**，这个长度，并不是理解的字符数  

String的含义并非是“字符串”，而是字符串的**UTF16编码**，字符串的操作，实际上都是针对UTF16编码的。所以字符串的最大长度，受字符串的编码长度影响。  

JavaScript字符串把每个UTF16单元当作一个自负来处理，所以处理非BMP（基本字符区域，某个范围的码点）应当注意  
<br>


### 不可变性
字符串是没法改变的，一旦构造出来，无法用任何方式改变字符串的内容  
<br><br>

## Symbol
每个从Symbol()返回的值都是唯一的  
<br>

### 独一无二
它是原始类型，所以不是通过new操作符号创建的，而是通过 `const s = Symbol()`  
这样，可以作为对象的key的就除了字符串，还有Symbol类型了，当他作为对象属性的时候，**不可以用点操作符**  
在创建的时候，可以选用一个**字符串**用于描述，如果参数为对象的时候，调用它的toString方法  

同一个字符创建的Symbol也是不同的，如果想要使用**同一个Symbol**，那么可以用**Symbol.for**方法，它会用描述参数在全局注册，如果相同的，就使用同一个，而Symbol()不会在全局注册  
通过**Symbol.keyFor()**可以查看登记的key  
<br>

### 不可枚举
Symbol作为对象属性的时候，for...in, Object.getOwnPropertyNames, Object.keys等都无法获取到  
可以调用**Object.getOwnPropertySymbols()**来获取，返回数组  
<br>

### 应用
1.不污染属性  
2.作为私有属性  
使用他作为对象的属性，就可以做到非私有的内部方法的效果  
3.防止xss攻击  
react渲染组件，会校验某个Symbol，如果是通过**JSON**注入代码，而JSON代码无法添加Symbol，就会在渲染的时候被忽略  
<br>

### 内置Symbol
ES6内置里12种，其中：  
1.Symbol.iterator: 指向该对象默认的遍历器方法，也就是有这个属性的对象可以用**for...of**  
2.Symbol.toStringTag: 在对象上调用Object.prototype.toString，如果他存在，就会返回[Object xxx]，这个xxx就是他决定的  
3.Symbol.toPrimitive: 指的是该对象转为原始类型的时候的值  
