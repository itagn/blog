# Vue+TS+Echarts制作可视化组件
写在前面的话  
为什么用vue+typescript呢？  
vue是项目成员们平时的技术栈，typescript是因为项目TL推荐typescript。  
那么就去好好看看vue + typescript怎么玩了。  
一开始笔者是拒绝的，typescript配合vue一开始遇到很多坑，不过还是继续下去了

## vue配置typescript
vue推出了一个使用typescript的js库 [vue-class-component](https://github.com/vuejs/vue-class-component)  
笔者又发现一个基于这个仓库加强版的js库 [vue-property-decorator](https://github.com/kaorun343/vue-property-decorator)  
所以先安装这两个npm包吧
```javascript
$ npm install vue-class-component
$ npm install vue-property-decorator
```
typescript是不能直接在浏览器运行的，还要安装ts-loader和配置webpack的解析模块  
```javascript
$ npm install ts-loader
```
另外还需要配置tsconfig.json，里面包含了ts的解析的详细信息  
最后在src目录下创建一个vue-global.d.ts文件，让vue认识ts文件  
```javascript
declare module "*.vue" {
    import Vue from "vue";
    export default Vue;
}
```
继续修改src目录下的main.js重命名为main.ts，然后去webpack修改入口文件main.js改为main.ts  
现在配置工作告一段落，开始使用typescript吧
## vue中使用typescript
根据[ vue-class-component ](https://github.com/vuejs/vue-class-component)的使用规则来使用typescript  
创建一个新的文件 test.vue
```javascript
<template>
    <div> {{ msg }} </div>
</template>
<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
@Component
export default class test extends Vue {
    msg: string = '';
    mounted(){
        this.msg = 'hello world!';
    }
}
</script>
```
去添加一个test的路由，访问localhost:8080/test之后可以看到hello world!
## Typescript的坑
根据之前的经验，发现vue+typescript好像可以没遇到过啥坑，那么开始配合echarts吧  
笔者喜欢在main.ts里面添加全局函数，所以先引入echarts全局函数  
```javascript
import Vue from 'vue';
import echarts from 'echarts';
Vue.prototype.$echarts = echarts;
```
然后创建一个图表组件饼图pie.vue，由于配置项官网在echarts有很多，这里就不描述了  
```javascript
<template>
    <div class="pie">
        <div class="chart"></div>
    </div>
</template>
<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
@Component
export default class pie extends Vue {
    mounted(){
        let dom = document.querySelector('.pie .chart');
        let chart = this.$echarts.init(dom);
        let options = {
            //  echarts官网有官网实例
        }
        chart.setOption(options);
    }
}
</script>
```
这样一看写法没啥问题啊，但是报错了~！  
```javascript
Failed to compile.
TS2339: Property '$echarts' does not exist on type 'pie'.
```

难道说没有this里面没有$echarts的方法，在main.ts里面注册的全局函数不存在吗  
马上输出this看看结果
```javascript
mounted(){
    console.log(this);
}
```
然后在浏览器输出了一个vue对象 VueComponent  
最终在VueComponent.__proto__.__proto__ 里面找到了$echarts方法  
既然this里面可以通过原型找到echarts的方法，为啥不能用呢？  
这个问题笔者也不知道，但是笔者用了另外的办法解决了这个问题
```javascript
mounted(){
        let dom = document.querySelector('.pie .chart');
        let that = Object.assign(this);
        let chart = that.$echarts.init(dom);
        let options = {
            //  echarts官网有官网实例
        }
        chart.setOption(options);
    }
```
既然可以通过原型找到，但是this却不能直接使用  
那么笔者把this的引用通过浅复制复制给that，那么that就可以获取到eahrts的方法  

这个问题可能在哪里遇到呢  
1. 在main.ts里面注册全局函数 
1. 父组件通过prop传值，在子组件获取到的prop值也会存在这个问题

## Echarts的坑
除了typescript的坑，echarts也到坑了  
由于制作的是图表组件，这是一个子组件，父组件可能会有很多个图表组件  
echarts不是自适应的，所以需要我们手动配置自适应。  
那么我们先配置一下自适应吧，在每个子组件内部解决自适应。
```javascript
//  图表子组件
mounted(){
    window.onresize = function(){
        chart.resize();
    }
}
```
父组件调用了多个图表子组件后，刷新，拖动浏览器窗口，发现了问题  
只有一个echarts满足了自适应，其他的没有变化  
想一想，应该是最后加载的window.onresize覆盖了前面的所有的自适应  
解决办法呢，看来子组件不能解决自适应的问题，需要在父组件中解决自适应了  
```javascript
//  父组件
mounted(){
    let charts = [chart1, chart2, chart3, chart4, chart5];
    window.onresize = function(){
        charts.forEach(val => val.resize() );
    }
}
```
现在思路整理清楚了，那么我们需要解决的问题就是，把在子组件注册的echarts对象传给父组件  
父组件给子组件通信是通过prop传递值，子组件给父组件通信就需要通过事件触发了  

**子组件** 部分，子组件初始化完echarts对象后通过事件触发与父组件通信
```javascript
//  子组件
mounted(){
    let dom = document.querySelector('.pie .chart');
    let that = Object.assign(this);
    let chart = that.$echarts.init(dom);
    uploadChart(chart);
    let options = {
        //  echarts官网有官网实例
    }
    chart.setOption(options);
}
uploadChart(chart){
    emit('upChart', chart);
}
```
**父组件** 部分，父组件在子组件绑定触发的事件和接收通信的数据
```javascript
//  子组件
<template>
    <div>
        <pie @upChart="getChart" />
    </div>
</template>
<script lang="ts">
import { Vue, Component } from 'vue-property-decorator';
import pie from './pie.vue';
@Component({
    components: {
        pie
    }
})
export default class pie extends Vue {
    charts = [];
    mounted(){
        window.onresize = function (){
            this.charts.forEach(val => val.resize() );
        }
    }
    getChart(data){
        //  这里的data就是子组件传递的值
        this.charts.push(data);
    }
}
</script>
```
通过子组件向父组件通信解决了关于echarts组件自适应的问题  
结合父组件通过prop给子组件传递数据，那么子组件的兄弟组件之间也可以传递数据了

## 总结
数据的可视化需求越来越多，也会跟着大数据一起火，以后会成为前端工程师必备的技能。  
除了echarts可视化库，还有[【d3js】](https://github.com/d3/d3)、阿里的[【antv】](https://antv.alipay.com/zh-cn/index.html)、3d的js库[【threejs】](https://github.com/mrdoob/three.js)

作者：微博 [@itagn][1] - Github [@itagn][2]

[1]: https://weibo.com/p/1005053782707172
[2]: https://github.com/itagn




