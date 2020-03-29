### `==`和`===`的区别
第一种是`==`比较，它会自动转换数据类型再比较，很多时候，会得到非常诡异的结果 
第二种是`===`比较，它不会自动转换数据类型，如果数据类型不一致，返回false，如果一致，再比较 
由于JavaScript这个设计缺陷，不要使用`==`比较，始终坚持使用`===`比较
```
false == 0; // true
false === 0; // false
```

### `NaN`
`NaN`这个特殊的Number与所有其他值都不相等，包括它自己　　
```
NaN === NaN; // false
```
唯一能判断`NaN`的方法是通过`isNaN()`函数
```
isNaN(NaN); // true
```

### 浮点数的比较
```
1 / 3 === (1 - 2 / 3); // false
```
这不是JavaScript的设计缺陷。浮点数在运算过程中会产生误差，因为计算机无法精确表示无限循环小数。要比较两个浮点数是否相等，只能计算它们之差的绝对值，看是否小于某个阈值
```
Math.abs(1 / 3 - (1 - 2 / 3)) < 0.0000001; // true
```

### foreach

`iterable`内置的`forEach`方法，它接收一个函数，每次迭代就自动回调该函数。以`Array`为例：

```
var a = ['A', 'B', 'C'];
a.forEach(function (element, index, array) {
    // element: 指向当前元素的值
    // index: 指向当前索引
    // array: 指向Array对象本身
    alert(element);
});
```

`Set`与`Array`类似，但`Set`没有索引，因此回调函数的前两个参数都是元素本身：

```
var s = new Set(['A', 'B', 'C']);
s.forEach(function (element, sameElement, set) {
    alert(element);
});

```

`Map`的回调函数参数依次为`value`、`key`和`map`本身：

```
var m = new Map([[1, 'x'], [2, 'y'], [3, 'z']]);
m.forEach(function (value, key, map) {
    alert(value);
});

```

如果对某些参数不感兴趣，由于JavaScript的函数调用不要求参数必须一致，因此可以忽略它们。例如，只需要获得`Array`的`element`：

```
var a = ['A', 'B', 'C'];
a.forEach(function (element) {
    alert(element);
});
```

