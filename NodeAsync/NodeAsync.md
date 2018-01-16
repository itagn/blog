```javascript
/*
* 作者：蔡东-uestc-2017届-cs
* 转载请注明出处并保留原文链接
*/
```

## Node异步的解决方案

    在我思考node异步的解决方案，首先想到的是回调，如何解决回调地狱呢？
    ES6里面提出了一种方案，*和yield加promise。
    很有名的npm包 co，也就是通过*和yield实现的。
    ES7又提出了新的语法糖，async和await加promise。
    而我在做koa2项目的时候，都是将async和await加promise用到每一个异步中。
    
    如果再让我想想解决异步的方案，之前用node写爬虫的时候，为了实现同步（那时候还不会es6和koa2）。
    通常使用res.on('data')处理接收到数据时的操作，res.on('end')处理接收完数据时的操作。
    算起来这也是一种异步的解决方案。
    
    解决异步的是回调
    promise是异步回调的语法糖
    *和yield | async和await 是为了解决回调地狱。
    
    最近看了《深入浅出node》，我发现还忽略了koa2出现的另外一种异步解决方案。
    流程控制库，封装好了的各种中间件，中间件的写法参数中也带了next参数。
    为了获取koa2项目启动的时间，我们就是通过koa2的洋葱模型和自定义插入next()，来实现计时。
    
    通过《深入浅出node》，异步编程的解决方案可以分为三种
    1.事件发布/订阅模式
    2.Promise/Deferred模式
    3.流程控制库
    
## 事件发布/订阅模式
Node事件中具有的事件监听模式的方法，这些回调函数就是事件监听器。

- [x] on / addListener
- [x] emit
- [x] once
- [x] removeListener
- [x] removeAllListeners

```javascript
//  事件订阅
emitter.on('success', function(data){
    console.log(data);  //  hello wolrd
});
//  事件发布
emitter.emit('success', 'hello wolrd');
```
事件发布/订阅模式本身没有异步调用和同步调用的问题，emit函数多半是在异步中触发订阅函数，所以广泛应用在异步编程中。

- 如果同一个事件添加了超过10个监听器会收到警告，调用emitter.setMaxListeners(0)可以去掉限制。
- 如果没有给error添加事件监听，会作为异常抛出，外部还是没有捕获这个异常的话，会引起线程退出。如果给error事件添加了事件监听，错误会由监听器来处理。res.on('error');

## Promise/Deferred模式
Promise/Deferred模式包含两个部分，顾名思义，就是Promise和Deferred。

    Promise的三个状态，pending（进行中）、fulfilled（已成功）和rejected（已失败）
    Promise状态只能由异步操作的结果决定，其他操作无法改变状态。
    状态一旦改变，不可逆转，只能发展的是 pending -> fulfilled | rejected
    
《深入浅出node》p84通过继承node的events模块，通过事件发布/订阅模式的实现简单的Promise（我有修改）
```javascript
//  通过继承node的events模块来获取事件订阅方法，util封装了继承的方法
var Promise = function(){
    EventEmitter.call(this);
};
util.inherits(Promise, EventEmitter);
//  通过原型模式添加Promise的then方法
Promise.prototype.then = funtion (fulfilledHandler, errorHandler){
    //  判断参数是否为函数，函数才可以添加事件订阅，success、error、progress都是继承得到的事件
    if(typeof fulfilledHandler === 'function'){
        this.once('success', fulfilledHandler);  //  添加事件订阅
    }
    if(typeof errorHandler === 'function'){
        this.once('error', errorHandler);
    }
}
```
事件订阅了，那么还需要通过事件发布触发这些事件订阅，实现这个的地方就叫Deferred
```javascript
//  通过构造函数模式构造Deferred
var Deferred = function(){
    this.state = 'pending';
    this.promise = new Promise();
}
//  通过原型模式构造Deferred的方法
Deferred.prototype.resolve = function(data){
    this.state = 'fulfilled';
    this.promise.emit('success', data);  //  事件发布触发对应的事件订阅
}
Deferred.prototype.reject = function(err){
    this.state = 'failed';
    this.promise.emit('error', err);
}
//  《深入浅出node》 p87（我有修改）
Deferred.prototype.all = function(promises){
    var count = promises.length;  //  获取promises数组的个数
    var that = this;  //  保存this的副本，下面如果调用this，作用域不在Deferred了
    var queue = [];  //  保存promise队列
    promises.forEach(function(promise, index){  //  遍历
        //  添加事件订阅，第一个为fulfilledHandler，第二个为errorHandler
        promise.then(
            function (data){
                //  保存所有的promise成功的数据
                queue[index] = data;
                if(--count === 0){
                    //  当promise队列全部都添加了事件订阅，开始通过事件发布来触发
                    that.resolve(queue);
                }
            }, function(err){
                //  触发error，切换状态
                --count;
                that.reject(err);
            });
    });
    return this.promise;
}
```
这里的all()方法操作多个promise，只有所有的异步操作都成功了，这个整的异步操作才算成功，其中一个异步操作失败，这个整的异步操作就失败。
## 流程控制库

    流程控制库是非模式的，灵活性高。

**1.尾触发与Next**
```javascript
//  看看koa2中间件的代码
var app = Koa();
app.use(async(ctx, next) => {
    const start = new Date();
    await next();
    const ms = new Date() - start;
    console.log(`${ctx.method} ${ctx.url} - ${ms}ms`);
});
app.use(json());
app.use(logger());
app.use(router.routes(), router.allowedMethods());
app.listen(7777);
```
通过app.use()的方法注册中间件，可以在第5行看到，可以自定义加入next()方法，这次自定的next()是由于koa2的洋葱模型，最后可以统计到整个启动花费的时间。
```javascript
//  koa2中实现简单的中间件
async function(ctx, next){
    //  中间件
}
```
**2.async**

    最知名的流程控制模块async，流程控制是开发过程中的基本需求。

- 异步的串行执行，async.series()，顺序执行异步函数

```javascript
//  《深入浅出node》 p95
async.series([
    function(callback){
        fs.readFile('file1.txt', 'utf-8', callback);
    },
    function(callback){
        fs.readFile('file2.txt', 'utf-8', callback);
    }
], function(err, data){
    console.log(data);  //  [file1.txt, file2.txt]
});
```

- 异步的并行执行，async.parallel()，并发自行异步函数

```javascript
//  《深入浅出node》 p96
async.parallel([
    function(callback){
        fs.readFile('file1.txt', 'utf-8', callback);
    },
    function(callback){
        fs.readFile('file2.txt', 'utf-8', callback);
    }
], function(err, data){
    console.log(data);  //  [file1.txt, file2.txt]
});
```

作者：微博 [@itagn][1] - Github [@itagn][2]

[1]: https://weibo.com/p/1005053782707172
[2]: https://github.com/itagn