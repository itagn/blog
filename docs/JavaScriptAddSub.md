# JavaScript中任意两个数加减的解决方案

## 写在前面的话
本文是从初步解决到最终解决的思路，文章篇幅较长    
虽然是一篇从0开始的文章，中间的思维跳跃可能比较大    
代码的解析都在文章的思路分析和注释里，全文会帮助理解的几个关键词

> * Number.MAX_SAFE_INTEGER 和 Number.MIN_SAFE_INTEGER
> * 15长度的字符串
> * padStart 和 padEnd
> * 
    
## 分析填坑思路
相信很多前端都知道这段神奇的代码吧
```javascript
console.log(0.1 + 0.2 === 0.3)  // false
console.log(0.3 - 0.2 === 0.1)  // false
```
网络上有很多文章解释，这里就不剖析了。    
至少我们可以知道，小数加减是存在问题的！    
那怎么解决小数的加减呢？有一个思路：

    既然小数加减存在问题，那么避开这个问题。
    直接把小数转换成整数后加减计算，这总可以吧。

小数的坑现在转到了整数，再看看整数加减的坑...    
```javascript
const max = Number.MAX_SAFE_INTEGER
console.log(max)  // 9007199254740991
console.log(max + 2)  // 9007199254740994

const min = Number.MIN_SAFE_INTEGER
console.log(min)  // -9007199254740991
console.log(min - 2)  // -9007199254740994
```
`Number.MAX_SAFE_INTEGER` 是何物？    
根据 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number/MAX_SAFE_INTEGER) 里面的定义

    常量表示在 JavaScript 中最大的安全整数

同理可知，`Number.MIN_SAFE_INTEGER` 也就是最小的安全整数    
整数的加减在最大安全整数和最小安全整数以内的计算才是稳稳的    
计算结果安全了么？emmm好像还有一个问题...    
```javascript
console.log(10 ** 21)  // 1e+21
console.log(999999999999999999999)  // 1e+21
```
从上面的结果可以看到，不可能忍受的是

    1.最后的输出结果显示的是科学计数法
    2.科学计数法表示的数并不能准确知道真实的数是多少

既然数字的显示存在这样的问题，把输入结果和输出结果都用字符串表示    
```javascript
console.log(`${10 ** 21}`)  // '1e+21'
console.log('' + 10 ** 21)  // '1e+21'
console.log((10 ** 21).toString())  // '1e+21'
```
我们发现即使直接就转换成字符串仍然会显示为科学计数法，那么可以直接输入字符串了，跳过转成字符串的过程
## 解决整数加法的坑
在这里先试着解决整数加法的问题      
这里有几个可能性

    1.输入的数字都在安全整数以内相加之后，且计算的结果也在安全整数之内，则直接输出结果
    2.如果不满足上面条件的...（等下再说）

```javascript
const MAX = Number.MAX_SAFE_INTEGER
const MIN = Number.MIN_SAFE_INTEGER
/**
* @param { number } num 需要检查的整数
* @return { boolean } 返回数字是否为安全的整数
*/
function isSafeNumber(num) {
    // 即使 num 成了科学计数法也能正确的和 MAX, MIN 比较大小
    return MIN <= num && num <= MAX
}
/**
* @param { string } a 相加的第一个整数字符串
* @param { string } b 相加的第二个整数字符串
* @return { string } 返回相加的结果
*/
function IntAdd(a = '', b = '') {
    let resulte = '0'
    const intA = Number(a), intB = Number(b)
    if (intA === 0) return b
    if (intB === 0) return a
    if (isSafeNumber(intA) && isSafeNumber(intB) && isSafeNumber(intA + intB)) {
        resulte = intA + intB
    } else {
        resulte = IntCalc(a, b)
    }
    return resulte
}
function IntCalc(a, b) {
    // TODO
}
```
如果不满足上面条件的呢？    
笔者的思路是

    获取数字转成字符串拆分成多个部分（数组），每一个部分的长度为 Number.MAX_SAFE_INTEGER 转成字符串后的长度减一（15），长度不足15的用字符‘0’填充首部，再计算每个部分的结果后拼接在一起
    同时考虑到正负号的问题，拆分后的计算需要带上符号

长度减一的原因是接下来每部分的所有计算都是安全的，不需要在考虑是数字计算结果为安全的整数    
同时每部分计算后的结果存在问题以及笔者的解决方案

    注意：下面会使用15这个数字，15上面说过了，是Number.MAX_SAFE_INTEGER的长度减一
    1.计算结果为0
        那么这个部分赋值15个字符‘0’组成的字符串，即‘000000000000000’
    2.计算结果为负数
        那么向上一级数组借10的15次方，同时高位（下一级数组）减一，低位用10的15次方再加上这个负数，做为这个部分的结果
    3.计算结果为正数，判断长度：
        如果长度超过15，那么去掉结果的第一位字符（因为进位，第一个字符一定是‘1’），同时高位（下一级数组）加一
        如果长度没有超过15，向首部补充0直到长度足够15
        如果长度等于15，直接添加到结果中

直接上代码吧，里面会有详细的注释
```javascript
const MAX = Number.MAX_SAFE_INTEGER
const MIN = Number.MIN_SAFE_INTEGER
const intLen = `${MAX}`.length - 1  // 下面会频繁用到的长度 15

function isSafeNumber(num) {
    // 即使 num 成了科学计数法也能正确的和 MAX, MIN 比较大小
    return MIN <= num && num <= MAX
}

// 整数加法函数入口
function intAdd(a = '0', b = '0') {
    const statusObj = checkNumber(a, b)
    if (!statusObj.status) {
        return statusObj.data
    } else {
        const tagA = Number(a) < 0,  tagB = Number(b) < 0
        const strA = `${a}`, strB = `${b}`
        const lenA = tagA ? strA.length - 1 : strA.length
        const lenB = tagB ? strB.length - 1 : strB.length
        const maxLen = Math.max(lenA, lenB)
        const padLen = Math.ceil(maxLen / intLen) * intLen  // 即为会用到的整个数组长度
        const newA = tagA ? `-${strA.slice(1).padStart(padLen, '0')}` : strA.padStart(padLen, '0')
        const newB = tagB ? `-${strB.slice(1).padStart(padLen, '0')}` : strB.padStart(padLen, '0')
        let result = intCalc(newA, newB)
        // 去掉正负数前面无意义的字符 ‘0’
        const numberResult = Number(result)
        if (numberResult > 0) {
            while (result[0] === '0') {
                result = result.slice(1)
            }
        } else if (numberResult < 0) {
            while (result[1] === '0') {
                result = '-' + result.slice(2)
            }
        } else {
            result = '0'
        }
        console.log(result)
        return result
    }
}

/**
* @param { string } a 相加的第一个整数字符串
* @param { string } b 相加的第二个整数字符串
* @return { string } 返回相加的结果
*/
function intCalc(a, b) {
    let result = '0'
    const intA = Number(a), intB = Number(b)
    // 判断是否为安全数，不为安全数的操作进入复杂计算模式
    if (isSafeNumber(intA) && isSafeNumber(intB) && isSafeNumber(intA + intB)) {
        result = `${intA + intB}`
    } else {
        const sliceA = a.slice(1), sliceB = b.slice(1)
        if(a[0] === '-' && b[0] === '-') {
            // 两个数都为负数，取反后计算，结果再取反
            result = '-' + calc(sliceA, sliceB, true)
        } else if (a[0] === '-') {
            // 第一个数为负数，第二个数为正数的情况
            const newV = compareNumber(sliceA, b)
            if (newV === 1) {
                // 由于 a 的绝对值比 b 大，为了确保返回结果为正数，a的绝对值作为第一个参数
                result = '-' + calc(sliceA, b, false)
            } else if (newV === -1) {
                // 道理同上
                result = calc(b, sliceA, false)
            }
        } else if (b[0] === '-') {
            // 第一个数为正数，第二个数为负数的情况
            const newV = compareNumber(sliceB, a)
            if (newV === 1) {
                // 由于 b 的绝对值比 a 大，为了确保返回结果为正数，b的绝对值作为第一个参数
                result = '-' + calc(sliceB, a, false)
            } else if (newV === -1) {
                // 道理同上
                result = calc(a, sliceB, false)
            }
        } else {
            // 两个数都为正数，直接计算
            result = calc(a, b, true)
        }
    }
    return result
}

/**
* @param { string } a 比较的第一个整数字符串
* @param { string } b 比较的第二个整数字符串
* @return { object } 返回是否要退出函数的状态和退出函数返回的数据
*/
function checkNumber(a, b) {
    const obj = {
        status: true,
        data: null
    }
    const typeA = typeof(a), typeB = typeof(b)
    const allowTypes = ['number', 'string']
    if (!allowTypes.includes(typeA) || !allowTypes.includes(typeB)) {
        console.error('参数中存在非法的数据，数据类型只支持 number 和 string')
        obj.status = false
        obj.data = false
    }
    if (Number.isNaN(a) || Number.isNaN(b)) {
        console.error('参数中不应该存在 NaN')
        obj.status = false
        obj.data = false
    }
    const intA = Number(a), intB = Number(b)
    if (intA === 0) {
        obj.status = false
        obj.data = b
    }
    if (intB === 0) {
        obj.status = false
        obj.data = a
    }
    const inf = [Infinity, -Infinity]
    if (inf.includes(intA) || inf.includes(intB)) {
        console.error('参数中存在Infinity或-Infinity')
        obj.status = false
        obj.data = false
    }
    return obj
}

/**
* @param { string } a 比较的第一个整数字符串
* @param { string } b 比较的第二个整数字符串
* @return { boolean } 返回第一个参数与第二个参数的比较
*/
function compareNumber(a, b) {
    if (a === b) return 0
    if (a.length > b.length) {
        return 1
    } else if (a.length < b.length) {
        return -1
    } else {
        for (let i=0; i<a.length; i++) {
            if (a[i] > b[i]) {
                return 1
            } else if (a[i] < b[i]) {
                return -1
            }
        }
    }
}

/**
* @param { string } a 相加的第一个整数字符串
* @param { string } b 相加的第二个整数字符串
* @param { string } type 两个参数是 相加（true） 还是相减（false）
* @return { string } 返回相加的结果
*/
function calc(a, b, type = true) {
    const arr = []  // 保存每个部分计算结果的数组
    for (let i=0; i<a.length; i+=intLen) {
        // 每部分长度 15 的裁取字符串
        const strA = a.slice(i, i + intLen)
        const strB = b.slice(i, i + intLen)
        const newV = Number(strA) + Number(strB) * (type ? 1 : -1)  // 每部分的计算结果，暂时不处理
        arr.push(`${newV}`)
    }
    let num = ''  // 连接每个部分的字符串
    for (let i=arr.length-1; i>=0; i--) {
        if (arr[i] > 0) {
            // 每部分结果大于 0 的处理方案
            const str = `${arr[i]}`
            if (str.length < intLen) {
                // 长度不足 15 的首部补充字符‘0’
                num = str.padStart(intLen, '0') + num
            } else if (str.length > intLen) {
                // 长度超过 15 的扔掉第一位，下一部分进位加一
                num = str.slice(1) + num
                if (i >= 1 && str[0] !== '0') arr[i-1]++
                else num = '1' + num
            } else {
                // 长度等于 15 的直接计算
                num = str + num
            }
        } else if(arr[i] < 0) {
            // 每部分结果小于 0 的处理方案，借位 10的15次方计算，结果恒为正数，首部填充字符‘0’到15位
            const newV =  `${10 ** intLen + Number(arr[i])}`
            num = newV.padStart(intLen, '0') + num
            if (i >= 1) arr[i-1]--
        } else {
            // 每部分结果等于 0 的处理方案，连续15个字符‘0’
            num = '0'.padStart(intLen, '0') + num
        }
    }
    return num
}
```
测试结果    
这一部分的代码请看 [这里](https://github.com/itagn/math/blob/master/demo/intAdd_v1.js)  
```javascript
console.log(MAX)  // 9007199254740991
intAdd(`${MAX}`, '2')  // '9007199254740993'
intAdd(`${MAX}`, '10000000000000000')  // '19007199254740991'
// 下面测试10的二十一次方的数据 1000000000000000000000
intAdd(`${MAX}`, '1000000000000000000000')  // '1000009007199254740991'
intAdd(`${MAX}`, `-${10 ** 16}`)  // '-992800745259009'
// 仍然存在一个问题，就是不要使用计算中的字符串，如下
intAdd(MAX, `${10 ** 21}`)  // '10.0000000071992548e+21'
intAdd(MAX, `-${10 ** 21}`)  // '0'
```
当然考虑到由于一般计算不会使用大数，书写字符串相加确实感觉怪怪的，可以在函数内加入判断，是科学计数法的提示并转换为10进制数，进行代码改进
```javascript
// 整数加法函数入口
function intAdd(a = '0', b = '0') {
    const statusObj = checkNumber(a, b)
    if (!statusObj.status) {
        return statusObj.data
    } else {
        let newA, newB, maxLen
        const tagA = Number(a) < 0,  tagB = Number(b) < 0
        const strA = `${a}`, strB = `${b}`
        const reg = /^\-?(\d+)(\.\d+)?e\+(\d+)$/
        if(reg.test(a) || reg.test(b)) {
            console.warn('由于存在科学计数法，计算结果不一定准确，请转化成字符串后计算')
            a = strA.replace(reg, function(...rest){
                const str = rest[2] ? rest[1] + rest[2].slice(1) : rest[1]
                return str.padEnd(Number(rest[3]) + 1, '0')
            })
            b = strB.replace(reg, function(...rest){
                const str = rest[2] ? rest[1] + rest[2].slice(1) : rest[1]
                return str.padEnd(Number(rest[3]) + 1, '0')
            })
            maxLen = Math.max(a.length, b.length)
        } else {
            const lenA = tagA ? strA.length - 1 : strA.length
            const lenB = tagB ? strB.length - 1 : strB.length
            maxLen = Math.max(lenA, lenB)
        }
        const padLen = Math.ceil(maxLen / intLen) * intLen  // 即为会用到的整个数组长度
        newA = tagA ? `-${strA.slice(1).padStart(padLen, '0')}` : strA.padStart(padLen, '0')
        newB = tagB ? `-${strB.slice(1).padStart(padLen, '0')}` : strB.padStart(padLen, '0')
        let result = intCalc(newA, newB)
        // 去掉正负数前面无意义的字符 ‘0’
        const numberResult = Number(result)
        if (numberResult > 0) {
            while (result[0] === '0') {
                result = result.slice(1)
            }
        } else if (numberResult < 0) {
            while (result[1] === '0') {
                result = '-' + result.slice(2)
            }
        } else {
            result = '0'
        }
        console.log(result)
        return result
    }
}
```
继续测试代码    
这一部分的代码请看 [这里](https://github.com/itagn/math/blob/master/demo/intAdd_v2.js)  
```javascript
// 警告：由于存在科学计数法，计算结果不一定准确，请转化成字符串后计算
intAdd(MAX, 10 ** 21)  // '1000009007199254740991'
// 警告：由于存在科学计数法，计算结果不一定准确，请转化成字符串后计算
intAdd(MAX, 10 ** 21 + 2)  // '1000009007199254740991'

intAdd(MAX, NaN) // 报错：参数中不应该存在 NaN
intAdd(MAX, {}) // 报错：参数中存在非法的数据，数据类型只支持 number 和 string

// 大数计算
intAdd('9037499254750994', '-9007299251310995')  // '30200003439999'
intAdd('8107499231750996', '-9007299254310995')  // '-899800022559999'
intAdd('-9907492547350994', '9007399254750995')  // '-900093292599999'
intAdd('9997492547350994', '9997399254750995')  // '19994891802101989'
intAdd('-9997492547350994', '-9997399254750995')  // '-19994891802101989'
intAdd('-4707494254750996000004254750996', '9707494254750996007299232150995')  // '5000000000000000007294977399999'
intAdd('-4707494254750996900004254750996', '9707494254750996007299232150995')  // '4999999999999999107294977399999'
```
## 解决整数减法的坑
加法和减法同理，只需要把第二个参数取反后利用加法运算就可以了，由于之前已经提取了模板，可以直接定义减法函数
```javascript
// 整数减法函数入口
function intSub(a = '0', b = '0') {
    const newA = `${a}`
    const newB = Number(b) > 0 ? `-${b}`: `${b}`.slice(1)
    const statusObj = checkNumber(newA, newB)
    if (!statusObj.status) {
        return statusObj.data
    } else {
        const result = IntAdd(newA, newB)
        return result
    }
}
```
测试结果
```javascript
IntSub('9037499254750994', '-9007299251310995')  // '18044798506061989'
IntSub('8107499231750996', '-9007299254310995')  // '17114798486061991'
IntSub('-9907492547350994', '9007399254750995')  // '-18914891802101989'
IntSub('9997492547350994', '9997399254750995')  // '93292599999'
IntSub('-4707494254750996000004254750996', '9707494254750996007299232150995')  // '-14414988509501992007303486901991'
IntSub('-4707494254750996900004254750996', '9707494254750996007299232150995')  // '-14414988509501992907303486901991'
```
## 解决小数加法的坑
JavaScript中小数加减的坑是由于浮点精度的计算问题，网上能查到很多相关的文章，但是笔者不打算从浮点计算入手。    
既然之前已经解决了整数加减的问题，同样可以利用整数的加减原理来实现小数的计算。    
    
    整数加法代码中经常出现 `padStart` 这个向前补齐的函数，因为在整数前加字符‘0’的对本身没有影响。
    小数也有这个原理，往尾部补‘0’同样对小数没有影响，然后再补齐后的数通过整数加减来计算。

基于整数加法的思想实现
```javascript
// 小数加法函数入口
function floatAdd(a = '0', b = '0') {
    const statusObj = checkNumber(a, b)
    if (!statusObj.status) {
        return statusObj.data
    } else {
        const strA = `${a}`.split('.'), strB = `${b}`.split('.')
        let newA = strA[1], newB = strB[1]
        const maxLen = Math.max(newA.length, newB.length)
        const floatLen = Math.ceil(maxLen / intLen) * intLen
        newA = newA.padEnd(floatLen, '0')
        newB = newB.padEnd(floatLen, '0')
        newA = strA[0][0] === '-' ? `-${newA}` : newA
        newB = strB[0][0] === '-' ? `-${newB}` : newB
        let result = intCalc(newA, newB)
        let tag = true, numResult = Number(result)
        // 去掉正负数后面无意义的字符 ‘0’
        if (numResult !== 0) {
            if (numResult < 0) {
                result = result.slice(1)
                tag = false
            }
            result = result.length === floatLen ? `0.${result}` : `1.${result.slice(1)}`
            result = tag ? result : `-${result}`
            let index = result.length - 1
            while (result[index] === '0') {
                result = result.slice(0, -1)
                index--
            }
        } else {
            result = '0'
        }
        console.log(result)
        return result
    }
}
```
测试结果    
这一部分的代码请看 [这里](https://github.com/itagn/math/blob/master/demo/floatAdd.js)  
```javascript
floatAdd('0.9037499254750994', '-0.9007299251310995')  // '0.0030200003439999'
floatAdd('0.8107499231750996', '-0.9007299254310995')  // '-0.0899800022559999'
floatAdd('-0.9907492547350994', '0.9007399254750995')  // '-0.0900093292599999'
floatAdd('0.9997492547350994', '0.9997399254750995')  // '1.9994891802101989'
floatAdd('-0.9997492547350994', '-0.9997399254750995')  // '-1.9994891802101989'
floatAdd('-0.4707494254750996000004254750996', '0.9707494254750996007299232150995')  // '0.5000000000000000007294977399999'
floatAdd('-0.4707494254750996900004254750996', '0.9707494254750996007299232150995')  // '0.4999999999999999107294977399999'
```
## 解决小数减法的坑
与整数减法的原理相同，可以直接定义减法函数
```javascript
// 小数减法函数入口
function floatSub(a = '0', b = '0') {
    const newA = `${a}`
    const newB = Number(b) > 0 ? `-${b}`: `${b.slice(1)}`
    const statusObj = checkNumber(newA, newB)
    if (!statusObj.status) {
        return statusObj.data
    } else {
        const result = floatAdd(newA, newB)
        return result
    }
}
```
测试结果    
以上部分的代码请看 [这里](https://github.com/itagn/math/blob/master/demo/numCalc.js)  
```javascript
floatSub('0.9037499254750994', '-0.9007299251310995')  // '1.8044798506061989'
floatSub('0.8107499231750996', '-0.9007299254310995')  // '1.7114798486061991'
floatSub('-0.9907492547350994', '0.9007399254750995')  // '-1.8914891802101989'
floatSub('0.9997492547350994', '0.9997399254750995')  // '0.0000093292599999'
floatSub('-0.9997492547350994', '-0.9997399254750995')  // '-0.0000093292599999'
floatSub('-0.4707494254750996000004254750996', '0.9707494254750996007299232150995')  // '-1.4414988509501992007303486901991'
floatSub('-0.4707494254750996900004254750996', '0.9707494254750996007299232150995')  // '-1.4414988509501992907303486901991'
```
## 解决整数加小数的通用问题
由于在实际中遇到的数字很多情况是整数加小数的，下面开始分析    

    这里的解决思路仍然是往前补0和往后补0
    把整数和小数都补充完整后，合在一起进行整数相加
    最后根据之前保存的整数的长度，插入小数点
    剩下的就是把无意义的0排除掉，输出结果

这里在遇到一方没有小数的时候
```javascript
// 任意数加法函数入口
function allAdd(a = '0', b = '0') {
    const statusObj = checkNumber(a, b)
    if (!statusObj.status) {
        return statusObj.data
    } else {
        const strA = `${a}`.split('.'), strB = `${b}`.split('.')
        let intAs = strA[0], floatA = strA.length === 1 ? '0' : strA[1]
        let intBs = strB[0], floatB = strB.length === 1 ? '0' : strB[1]
        const tagA = intAs > 0, tagB = intBs > 0
        const maxIntLen = Math.max(intAs.length, intBs.length)
        const arrIntLen = Math.ceil(maxIntLen / intLen) * intLen
        const maxFloatLen = Math.max(floatA.length, floatB.length)
        const arrFloatLen = Math.ceil(maxFloatLen / intLen) * intLen
        intAs = tagA ? intAs.padStart(arrIntLen, '0') : intAs.slice(1).padStart(arrIntLen, '0')
        intBs = tagB ? intBs.padStart(arrIntLen, '0') : intBs.slice(1).padStart(arrIntLen, '0')
        let newA = floatA === '0' ? intAs + '0'.padEnd(arrFloatLen, '0') : intAs + floatA.padEnd(arrFloatLen, '0')
        let newB = floatB === '0' ? intBs + '0'.padEnd(arrFloatLen, '0') : intBs + floatB.padEnd(arrFloatLen, '0')
        newA = tagA ? newA : `-${newA}`
        newB = tagB ? newB : `-${newB}`
        let result = intCalc(newA, newB)
        const numResult = Number(result)
        if (result.length > arrIntLen) {
            result = result.slice(0, -arrFloatLen) + '.' + result.slice(-arrFloatLen)
        }
        // 去掉正负数前面后面无意义的字符 ‘0’
        if (numResult !== 0) {
            if (numResult > 0) {
                while (result[0] === '0') {
                    result = result.slice(1)
                }
            } else if (numResult < 0) {
                while (result[1] === '0') {
                    result = '-' + result.slice(2)
                }
                result = result.slice(1)
                tag = false
            }
            let index = result.length - 1
            while (result[index] === '0') {
                result = result.slice(0, -1)
                index--
            }
        } else {
            result = '0'
        }
        if (result[result.length - 1] === '.') {
            result = result.slice(0, -1)
        }
        if (result[0] === '.') {
            result = '0' + result
        }
        console.log(result)
        return result
    }
}

// 任意数减法函数入口
function allSub(a = '0', b = '0') {
    const newA = `${a}`
    const newB = Number(b) > 0 ? `-${b}`: `${b}`.slice(1)
    const statusObj = checkNumber(newA, newB)
    if (!statusObj.status) {
        return statusObj.data
    } else {     
        const result = allAdd(newA, newB)
        return result
    }
}
```
测试结果    
以上部分的代码请看 [这里](https://github.com/itagn/math/blob/master/demo/calc.js) 
```javascript
// 30200003439999.0030200003439999
allAdd('9037499254750994.9037499254750994', '-9007299251310995.9007299251310995')
// 5000000000000000007294977399998.9100199977440001
allAdd('9707494254750996007299232150995.8107499231750996', '-4707494254750996000004254750996.9007299254310995')
// 19994891802101990.9994891802101989
allAdd('9997492547350994.9997492547350994', '9997399254750995.9997399254750995')
// 30200003439999.0030200003439999
allSub('9037499254750994.9037499254750994', '9007299251310995.9007299251310995')
// 18044798506061990.8044798506061989
allSub('9037499254750994.9037499254750994', '-9007299251310995.9007299251310995')
// 17144998486501991.714499848650199
allSub('8107499231750996.8107499231750996', '-9037499254750994.9037499254750994')
```
## 总结
本文篇幅太长，所以代码部分没有细说（全在注释）    
主要分析了解决问题的整个思路，抓住几个重点理解

> * 1.Number.MAX_SAFE_INTEGER 和 Number.MIN_SAFE_INTEGER 之间的计算才是可信任的
> * 2.小数加减的浮点精度问题转移到整数来解决
> * 3.超大的数加减的时候，分区计算（理由是第1点）
> * 4.拆分成每部分15长度的字符串（理由是Number.MAX_SAFE_INTEGER的长度为16，无论如何加减都是满足第一点的，这样就不需要去注意加减的安全性问题了）
> * 5.科学计数法的问题，匹配是否为科学计数法的数，然后转换成十进制，同时提出警告，因为科学计数法的数存在误差，计算会存在不准确性

代码有很多地方可以优化，完成的比较潦草（轻喷）    
各位大佬有修改意见的 [欢迎提出](https://github.com/itagn/math/issues)    

**感谢观看**

作者：微博 [@itagn][1] - Github [@itagn][2]

[1]: https://weibo.com/p/1005053782707172
[2]: https://github.com/itagn
