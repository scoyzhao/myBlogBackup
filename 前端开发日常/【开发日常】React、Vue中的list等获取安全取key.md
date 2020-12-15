## 背景
在React、Vue中渲染诸如**List**等的时候，需要用到key，如果是后端返回的，可以用**id**，它可能存在一定的安全问题，但是也没啥毛病  
另外，如果是自己操作，比如我最近遇到的，要操作一个列表，点击新增，就需要自定义一个key  
<br><br>

## 两个不太好的解决办法
我用了时间戳测试，会出现奇怪的bug（同时渲染两个？？？）  
<br>

### 使用index作为key
简单来说，操作增删可能会出现渲染错乱的问题  
<br>

### 使用Math.random()
每次key都是随机的，性能不好，而且key每次都变，影响交互，还可能会出现奇奇怪怪的bug  
<br><br>

## 一种还算可以的方法
这种也是我之前用的方法，全局生命一个表示key的数据，然后每次添加都取改变这个数据  
而且这个对于存储毫无疑义，对后端来说是一个冗余数据  
<br><br>

## 比较安全的解决方法

<img src="https://raw.githubusercontent.com/scoyzhao/FigureBed/master/blog/57/0.png" width = "100%"  alt="图片名称" align=center />

