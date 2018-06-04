# 说说JavaScript的隐式转换
## 题目1
```javascript
//  题目1  
2 == [[2]] // true
```
很简单的看出来两边都调用toString方法，都得到“2”，所以输出true

## 题目2
```javascript
//  题目2  
parseInt(false, 16) // 250
```
乍一看，是用16进制处理false输出十进制整数，由于false并不是需要的数据类型，先通过toString转换成"false"，那么在16进制里面f和a是合法的，实际执行的是parseInt("fa", 16)，结果为 15*16+10*1 = 250

## 题目3
```javascript
//  题目3  
[1, 2, 3] > [1, 2, 2] // true
['a', 'b', 'c'] > ['a', 'b', 'b'] // true
[2, 'a'] > [1, 'b'] // true
['.', 4, 'a'] > [',', 3, 'b'] // true
```
题目3是是比较两个数组，通常大家会下面这么想

    第一个例子，数字3比2大
    第二个例子，字母里面c比b大
    第三个例子，2比1大，但是a比b小，于是觉得是第一位的权重比较大
    第四个例子，估计有人想到，难道是比的数组的第一位ASCII码

笔者自己认为，对比两个数组，实际上调用了两个数组的toString方法，那么例子可以换成下面的写法  
```javascript
//  题目3的实际比较
"1,2,3" > "1,2,2" // true
"a,b,c" > "a,b,b" // true
"2,a" > "1,b" // true
".,4,a" > ",,3,b" // true
```
这么一看实际上比较的就是两个字符串了，这样比较就很好理解了

## 题目4
题目4是实现add任意参数连加的链式调用。
```javascript
/*
* 实现add任意参数连加的链式调用
* 期望
* add() // 0
* add(1) // 1
* add(1, 2, 3) // 6
* add(1)(2)(3) // 6
* add(1, 2)(3, 4) // 10
*/
```
思路：  

    1. 任意参数很好解决，es6里面rest参数就可以了  
    2. 链式调用，那么需要返回的肯定是一个函数了，但是函数通常印象中是需要调用函数才能触发，否则就是返回函数本身
    3. 那么只要解决这个返回值就好处理了。通常输出一个函数，调用的是这个函数的valueOf或者toString方法，所以我们把toString绑定结果，输出函数本身的时候也就和调用函数的值是一致的
    

代码  
```javascript
//  题目4

// 第一种实现
const add1 = (...rest) => {
    let num = rest.reduce((a, b) => a+b, 0)
    const _add = (...args) => add1(args.reduce((a, b) => a + b, num))
    _add.toString = _add.valueOf = () => num
    return _add
}
add1.toString = () =>  "function add1() { [native code] }"

// 第二种实现
const add2 = (...rest) => {
    const _add = (...args) => add2(...rest, ...args)
    _add.toString = _add.valueOf = () => rest.reduce((a, b) => a + b, 0)
    return _add
}
add2.toString = () =>  "function add2() { [native code] }"
```

## END
隐式调用确实能写出很多酷炫的代码，搞清楚里面的逻辑能加强自己的基础，以上


作者：微博 [@itagn][1] - Github [@itagn][2]

[1]: https://weibo.com/p/1005053782707172
[2]: https://github.com/itagn






