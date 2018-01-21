```javascript
/*
* 作者：蔡东-uestc-2017届-cs
* 转载请注明出处并保留原文链接
*/
```
## 在项目中通过解构赋值优化代码

    笔者发现自己做项目写代码的时候总是想着先完成任务，然后再优化代码
    可是项目做完了总是忘记优化代码，代码习惯还是不好
    虽然在写ES6，解构赋值一直没使用，项目中很多时候可以用解构赋值优化
    今天就来谈谈解构赋值怎么优化代码
    
在笔者做koa2项目时，数据库用的是mongoDB，于是总会有一些很繁琐的代码，比如下面这些，从数据库取值然后赋值变量，最后返回给前端
```javascript
//  假设一个接口需要从mongoDB查询一个人的信息返回给前端
this.getUserInfo = async (ctx, next) => {
    const params = ctx.request.body;
    let condition = { name: params.name };
    let result = await this.DBModule.User.getUserInfo(condition);
    if(result && result.status === 'success'){
        //  处理数据，把代码单独提出来做性能优化
        let user = this.resUserInfo(result);
        ctx.body = {
            code: 200,
            msg: '查询数据成功'，
            data: user
        }
    } else {
        ctx.body = {
            code: 500,
            msg: '服务器错误'
        }
    }
}
```
上面的代码是一个通过用户名查询用户信息的接口。接口的功能是，如果成功查询结果有数据，那么返回这个人的部分信息，否则返回'-'，首先按照业务逻辑写代码。下面首先根据这个逻辑来编写代码。
```javascript
this.resUserInfo = (result) => {
    let email, age, sex, birthday, phone;
    //  判断返回的数组是否是空的
    if(result && result.length>0 ){
        email = result[0].email;
        age = result[0].age;
        sex = result[0].sex;
        birthday = result[0].birthday;
        phone = result[0].phone;
    } else {
        email = '-';
        age = '-';
        sex = '-';
        birthday = '-';
        phone = '-';
    }
    return [ email, age, sex, birthday, phone ];
}
```
逻辑是这么写，不过感觉这样的代码质量不好，扩展不方便，我们看到有很多重复性的代码了，看起来首先if else好像可以优化，通过三元式或者 || 就能达到，进行第一步代码优化
```javascript
this.resUserInfo = (result) => {
    let email, age, sex, birthdat, phone; 
    //  用三元式缩短代码，不过看起来还是需要输入重复的属性
    email = result && result.length>0 ? result[0].email : '-';
    age = result && result.length>0 ? result[0].age : '-';
    sex = result && result.length>0 ? result[0].sex : '-';
    birthday = result && result.length>0 ? result[0].birthday : '-';
    phone = result && result.length>0 ? result[0].phone : '-';
    //  假设返回的是数组，之后好进行对比
    return [ email, age, sex, birthday, phone ];
}
```
代码虽然缩短了，但是代码质量仍然不高，看起来属性也重复写了，我们不希望属性也同时出现两次，我们再进一步优化代码
```javascript
this.resUserInfo = (result) => {
//  通过对象解构赋值优化代码
    let {
        email = "-",
        age = "-",
        sex = "-",
        birthday = "-",
        phone = "-"
    } = result && result.length>0 ? result[0] : {};
    return [ email, age, sex, birthday, phone ];
}
```
突然发现如果把user写成一行，整个函数就3行代码，而且扩展和维护也很方便，这段代码意思呢是首先设置user在解构赋值的默认值为'-'，然后通过三元式条件赛选，如果通过条件，则执行解构赋值的模式匹配

    之前在函数里面用到了对象的解构赋值，既然是函数，那么可以用函数的解构赋值来代替对象的解构赋值吧，下面看看优化的代码
    
```javascript
this.resUserInfo = (result) => {
//  通过函数解构赋值优化代码
    let obj  = result && result.length>0 ? result[0] : {};
    return this.getVal(obj)
}
this.getVal = ({email="-",age="-",sex="-",birthday="-",phone="-"} = {}) => {
    return [ email, age, sex, birthday, phone ];
}
```
函数的解构赋值，设置参数对象的属性默认值为'-'，如果传入的参数为空对象，那么匹配到undefined，根据解构赋值的规则，那么变量获取默认值'-'，否则获取参数。函数的解构赋值呢还有另外一种方法。
```javascript
this.resUserInfo = (result) => {
//  通过函数解构赋值优化代码
    let obj  = result && result.length>0 ? result[0] : undefined;
    return this.getVal(obj)
}
this.getVal = ({email,age,sex,birthday,phone} = {email:"-",age:"-",sex:"-",birthday:"-",phone:"-"}) => {
    return [ email, age, sex, birthday, phone ];
}
```
这个和上面的函数解构赋值代码不一样了，之前是对要参数对象的属性设置默认值，这个是对整个参数对象设置默认值，只有参数为undefined时候才可以触发默认值。
## 总结

    解构赋值很好的解决变量赋值的问题，并且能够设置默认值，这样的功能在代码中很常见。
    利用好解构赋值可以提升代码质量，减少重复性代码。
    还有数组的解构赋值，在给变量赋值的时候也是能优化很多代码。

作者：微博 [@itagn][1] - Github [@itagn][2]

[1]: https://weibo.com/p/1005053782707172
[2]: https://github.com/itagn