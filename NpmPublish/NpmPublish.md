作者：蔡东-uestc-2017届
转载请注明出处并保留原文链接

## 如何发布一个npm包
**注册**：
首先在 **[npm](https://www.npmjs.com/)** 社区注册你的账号

**本地**：
**linux**系统下 or **Windows**系统的git bash / cmder
```cmd
$ mkdir test && cd test
$ npm init
```
跟着对应的输入信息，最后会储存到**package.json**里面

    包名称：（可回车）
    第一个版本号：（可回车）或者自定义 x.x.x
    描述：用英文简短介绍下包
    git仓库地址：如果包的源代码放在github上面，就贴上地址
    关键词：英文写点关键词
    作者：英文网名哦
    许可证：为了无私的开源，建议MIT
    
然后确认，输入yes。不满意的话执行 npm init 再来一次
文件布局如下

    index.js
    package.json
    LICENSE
    README.md
    lib /
        core.js
    test / 
        xxx

1. index.js就是package.json里面提到的入口文件（可以重命名）
2. package.json用来配置你的包的信息
3. lib/core.js 用来存到你的核心代码（可以重命名）
4. 生成的LICENSE就是你的许可证（不知道怎么写可以借鉴别人）
5. 用来完整介绍你的包的markdown文件
6. test存放你的测试代码
```javascript
//  index.js文件里面可以输入一下部分
const core = require('./lib/core.js');
module.exports = core;
```
在core.js文件写入你的、完整可运行的、nodejs脚本（lib文件夹的文件自由扩展）
如果需要其他依赖，安装依赖 npm install --save xxx，会自动把信息储存到package.json
在本地经过测试（或者人工测试）后暂无bug，检查完成后可以提交到npm社区
```cmd
$ npm login
Username:
Password: 
Email: 
$ npm publish
```
**注意**

    如果想要修改提交的任何部分，都需要重新发布npm包
    从第二次发布开始，每次发布都需要修改package.json里面的版本信息version
    每次发布希望都是检查完毕，请确保代码的正确性。
    不要为了毫无必要的问题去发布新的版本。
    
作者：微博 [@itagn][1] - Github [@itagn][2]

[1]: https://weibo.com/p/1005053782707172
[2]: https://github.com/itagn