# 1.项目中安装echarts #

```shell
cnpm install echarts --save
```
但是ionic项目依赖于angular和typeScript，所以再安装ts支持包

```shell
cnpm install @types/echarts -–save
```
官网给出的一段建议：
在 3.1.1 版本之前 ECharts 在 npm 上的 package 是非官方维护的，从 3.1.1 开始由官方 EFE 维护 npm 上 ECharts 和 zrender 的 package。
所以还得安装zrender

```shell
cnpm install zrender
```
# 2.引入使用 #


 - 在所需页面的name.ts引入如下
```typescript
import echarts from 'echarts';
```



 - 使用如下

```typescript
initCharts(){
  echarts.init(this.myCharts.nativeElement).setOption({
    xAxis: {
      type: 'category',
      data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
    },
    yAxis: {
      type: 'value'
    },
    series: [{
      data: [820, 932, 901, 934, 1290, 1330, 1320],
      type: 'line'
    }]

  },true);
}

```

 - 调用
```typescript
constructor(public navCtrl: NavController, public navParams: NavParams) {
 console.log(echarts);
  this.initCharts();
}

```

 - 页面显示
  找到name.html定义一个呈现图表的div如下
```html
<ion-header>

  <ion-navbar>
    <ion-title>charts</ion-title>
  </ion-navbar>

</ion-header>
<ion-content padding>
// #charts通过ViewChild获取dom节点
  <div class="charts" #charts>

  </div>
</ion-content>

```

 - 在name.ts文件一如dom节点

```typescript
 @ViewChild('charts') myCharts:ElementRef
```

 - 运行ionic serve报错如下图

![clipboard.png](E:\git\liukai90.github.io\cordova+ionic\img\error.png)
大概意思是说获取不到dom节点，也就是我们展示地图的那个div，打印了之后发现是null或者undefine。后面就打印了一下ionic页面的生命周期，测试了一下ionic页面的生命周期函数。

生命周期函数	触发时刻	顺序
constructor()	创建页面时触发	1
ionViewDidLoad()	当页面加载时触发	2
ionViewWillEnter()	当将要进入页面时触发	3
ionViewDidEnter()	进入页面时触发	4

在constructor() 和 ionViewDidLoad()initCharts()函数中是获取不到dom节点的，因为页面还没有加载完成，在ionViewWillEnter()和ionViewDidEnter()是能获取到dom节点的，顾名思义只有当页面加载完成时才能获取到dom节点，所以再在页面加载完成后的生命周期函数里可以获取到改为


```
ionViewWillEnter(){
 
}

```
或

```
ionViewDidEnter(){
  this.initCharts();
}

```
运行ionic serve 完美解决

![clipboard.png](E:\git\liukai90.github.io\cordova+ionic\img\result.png)

## 3.完整代码示例 ##
Charts.ts如下

```ts
import {Component, ElementRef, ViewChild} from '@angular/core';
import { IonicPage, NavController, NavParams } from 'ionic-angular';
import echarts from 'echarts';

/**
 * Generated class for the ChartsPage page.
 *
 * See https://ionicframework.com/docs/components/#navigation for more info on
 * Ionic pages and navigation.
 */

@IonicPage()
@Component({
  selector: 'page-charts',
  templateUrl: 'charts.html',
})
export class ChartsPage {
  @ViewChild('charts') myCharts:ElementRef

  constructor(public navCtrl: NavController, public navParams: NavParams) {
   console.log(echarts);

  }

  ionViewDidLoad() {
    // this.initCharts();
    console.log('ionViewDidLoad ChartsPage');
  }
  // ionViewDidEnter(){
  //   this.initCharts();
  // }
  ionViewWillEnter(){
    this.initCharts();
  }
  initCharts(){
    console.log(document.getElementById('charts'));
    echarts.init(this.myCharts.nativeElement).setOption({
      xAxis: {
        type: 'category',
        data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
      },
      yAxis: {
        type: 'value'
      },
      series: [{
        data: [820, 932, 901, 934, 1290, 1330, 1320],
        type: 'line'
      }]

    },true);
  }

}

```
charts.html

```html
<!--
  Generated template for the ChartsPage page.

  See http://ionicframework.com/docs/components/#navigation for more info on
  Ionic pages and navigation.
-->
<ion-header>

  <ion-navbar>
    <ion-title>charts</ion-title>
  </ion-navbar>

</ion-header>



<ion-content padding>
  <div class="charts"  #charts>

  </div>
</ion-content>

```
charts.scss

```
page-charts {
  .charts{
    height: 500px;

  }

}

```