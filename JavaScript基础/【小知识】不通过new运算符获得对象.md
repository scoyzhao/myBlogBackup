## 1.利用字面量
```javascript
let a = {},
    b = [],
    c = /abc/g,
    d = function() {},
```

<br>

## 2.利用dom api
```javascript
let e = document.createElement('p)
```

<br>

## 3.利用JavaScript内置对象的api
```javascript
let f = Object.create(null),
    g = Object.assign({}, {}),
    h = JSON.parse('{}')
```

<br>

## 4.利用装箱转化
```javascript
let i = Object(null)
```
